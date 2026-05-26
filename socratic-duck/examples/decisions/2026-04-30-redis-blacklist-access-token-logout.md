# Redis blacklist for immediate access token revocation on logout

**Date:** 2026-04-30

---

## Context

The `POST /auth/logout` endpoint invalidated the refresh token in the database, but the JWT access token remained valid until its natural expiration (15 minutes). In contexts where the app is used from shared devices, logout must take effect immediately. A stolen or leaked access token could be used during that window with no way to revoke it.

The project already had Redis (`RedisModule`, `ioredis` client) in use by another module, so no new infrastructure was added.

## Decision

On logout, if the frontend includes the current access token in the `Authorization` header, the `jti` is extracted and stored in Redis with a TTL equal to the token's remaining lifetime. On every authenticated request, `JwtStrategy.validate()` checks that the `jti` is not in the blacklist before authorizing access.

## Alternatives rejected

- **Reduce the access token TTL to 5 minutes:** shrinks the risk window without extra infrastructure, but does not eliminate it. Logout would remain "eventual", not immediate. Rejected because it does not solve the problem on shared devices.
- **Store access tokens in the database (stateful sessions):** allows full revocation but eliminates the benefits of stateless JWTs and adds a database query to every authenticated request. Rejected due to scalability impact and as a design regression.

## Trade-offs accepted

- Every authenticated request performs an `EXISTS` on Redis. Although it is O(1) and Redis was already in the stack, it adds a round-trip to every authenticated request.
- If Redis goes down, a decision is needed between failing all authenticated requests or bypassing the blacklist (security degradation). The failure-mode decision is deferred.
- The frontend now has a new responsibility: send the access token on logout. If it does not, immediate revocation does not apply for that client.

## Assumptions

- **The frontend will send the access token in the `Authorization` header on logout.** Revisit if reports appear of logout not revoking, or if logs show logouts without the header.
- **The `jti` is always present in issued access tokens.** Revisit if the JWT library is changed or the payload is modified.
- **The token's remaining TTL is reliably computable from the `exp` claim.** Revisit if clock skew is ever introduced between nodes.
- **Logout volume will not saturate Redis with blacklisted `jti` entries.** Revisit when daily logouts exceed 100k or Redis memory usage exceeds 70%.

## Sub-decisions

### Redis key format

- **Decision:** `blacklist:jti:<jti>`.
- **Why:** namespaced prefix to differentiate from other Redis uses and to ease inspection.
- **Trade-off:** slightly longer key, negligible memory cost.

### Behavior when the `Authorization` header is missing on logout

- **Decision:** the refresh token is invalidated anyway, the access token expires by natural TTL. A warning is logged.
- **Why:** safe degradation without breaking the endpoint for older clients.
- **Trade-off:** a 15-minute window where the access token remains valid in that specific case.

### Cleanup of blacklist keys

- **Decision:** rely on native Redis TTL. No cleanup job is implemented.
- **Why:** Redis expires keys automatically. Any additional job would be redundant.
- **Trade-off:** none relevant at current volume.

## Weak points

- The failure mode when Redis is down was not decided. The session identified it but deferred it, and the current code throws an exception if Redis does not respond. That means a Redis outage translates into 100% 401 errors on authenticated endpoints. This is a fragile point that should be resolved before a real incident, not after.
- The assumption that the frontend sends the access token on logout was not verified by inspecting the frontend code during the session. If the frontend has more than one logout path (for example, automatic expiration vs user click), not all of them may be sending the token.
- The latency impact of the additional `EXISTS` was not measured. The claim "O(1), minimal impact" is theoretically correct but not backed by a benchmark in this project.

## Deferred

- **Behavior when Redis is down (fail-closed vs fail-open):** its own decision. Revisit before the next security release. Candidate for a separate session.
- **Mass revocation per user (logout from all devices):** out of scope for this decision. Requires indexing `jti` by `user_id` or changing the model. Candidate for a separate session when requested by product.
- **Memory-usage monitoring specifically for the blacklist in Redis:** revisit when the infrastructure dashboard is added.

## References

- `src/auth/auth.service.ts` (`logout()` method that writes to Redis).
- `src/auth/strategies/jwt.strategy.ts` (`validate()` method that checks the blacklist).
- `src/auth/auth.controller.ts` (logout endpoint accepting the `Authorization` header).
- `src/redis/redis.module.ts` (global Redis client, `REDIS_CLIENT`).
- `docs/api/auth.md` (endpoint and token model documentation).
