---
task: "Add user authentication endpoint with JWT"
sources:
  - type: issue
    ref: "AUTH-42"
  - type: codebase
    paths: ["src/routes/", "src/middleware/", "src/models/user.ts"]
  - type: docs
    paths: ["CLAUDE.md", "docs/api-conventions.md"]
created: 2026-03-15
status: verified
---

# Verification Contract: Add user authentication endpoint with JWT

## Context
The application needs a login endpoint that authenticates users with email/password and returns a JWT token. The codebase uses Express with TypeScript, follows repository pattern for data access, and has an existing middleware chain for request validation. Rate limiting middleware exists at `src/middleware/rateLimit.ts` and should be reused. The team convention (per CLAUDE.md) is to keep route handlers thin — business logic belongs in service layer.

## Functional Verification
- [ ] `[auto]` POST /auth/login returns 200 and a JWT token with valid credentials
  <!-- verify: TOKEN=$(curl -s -X POST localhost:3000/auth/login -H 'Content-Type: application/json' -d '{"email":"test@example.com","password":"testpass123"}' | jq -r '.token') && [ "$TOKEN" != "null" ] && echo "PASS" || echo "FAIL" -->
- [ ] `[auto]` POST /auth/login returns 401 with invalid credentials
  <!-- verify: STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST localhost:3000/auth/login -H 'Content-Type: application/json' -d '{"email":"test@example.com","password":"wrong"}') && [ "$STATUS" = "401" ] && echo "PASS" || echo "FAIL" -->
- [ ] `[auto]` POST /auth/login returns 400 when email or password is missing
  <!-- verify: STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST localhost:3000/auth/login -H 'Content-Type: application/json' -d '{"email":"test@example.com"}') && [ "$STATUS" = "400" ] && echo "PASS" || echo "FAIL" -->
- [ ] `[auto]` JWT token contains user ID and email in payload
  <!-- verify: TOKEN=$(curl -s -X POST localhost:3000/auth/login -H 'Content-Type: application/json' -d '{"email":"test@example.com","password":"testpass123"}' | jq -r '.token') && echo $TOKEN | cut -d. -f2 | base64 -d 2>/dev/null | jq '.userId, .email' -->
- [ ] `[auto]` Rate limiting activates after 5 failed login attempts
  <!-- verify: for i in $(seq 1 6); do STATUS=$(curl -s -o /dev/null -w "%{http_code}" -X POST localhost:3000/auth/login -H 'Content-Type: application/json' -d '{"email":"test@example.com","password":"wrong"}'); echo "Attempt $i: $STATUS"; done -->

## Craft Verification
- [ ] `[auto]` No secrets or passwords hardcoded in source files
  <!-- verify: grep -rn "password\s*[:=]\s*['\"][^'\"]*['\"]" src/ --include="*.ts" | grep -v test | grep -v '.env' | grep -v 'schema\|type\|interface\|validation' -->
- [ ] `[auto]` Auth logic lives in service layer, not in route handler
  <!-- verify: grep -c "bcrypt\|jwt\|compare\|sign\|verify" src/routes/auth.ts || echo "0" -->
- [ ] `[auto]` Route handler is under 20 lines
  <!-- verify: awk '/login/,/^}/' src/routes/auth.ts | wc -l -->
- [ ] `[manual]` Naming follows existing codebase conventions (camelCase, descriptive)

## Contextual Verification
- [ ] `[auto]` All existing tests still pass
  <!-- verify: npm test -->
- [ ] `[auto]` New endpoint has test coverage
  <!-- verify: npm test -- --testPathPattern="auth" --verbose 2>&1 | grep -E "PASS|FAIL|Tests:" -->
- [ ] `[manual]` Error responses don't leak internal details (stack traces, DB errors)
- [ ] `[manual]` JWT secret is loaded from environment variable, not config file
