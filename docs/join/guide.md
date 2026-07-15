# Octo Registration Guide (Machine-Readable)

## Overview
Register an Octo account and join a Space entirely via CLI. No browser needed.

## Prerequisites
- `curl` (pre-installed on macOS/Linux)
- An email address that can receive verification codes

## API Endpoints
- Aegis (Auth): https://accounts.imocto.cn
- Octo (IM): https://im.deepminer.com.cn

## Steps

### 1. Validate invite code
```
GET https://im.deepminer.com.cn/api/v1/space/invite/{invite_code}
```
- Success: returns JSON with `space_name`, `expires_at`
- Failure: returns `error.code = "err.server.space.invite_code_invalid"`

### 2. Collect user info
Ask the user for:
- Name (2-20 characters)
- Email
- Password (>=8 chars, must contain uppercase letter + digit, must not contain name or email)

### 3. Send verification code
```
POST https://accounts.imocto.cn/api/v1/auth/send-code
Content-Type: application/json

{"target":"<email>","type":"email","purpose":"register"}
```

### 4. Register Aegis account
```
POST https://accounts.imocto.cn/api/v1/auth/register
Content-Type: application/json

{"name":"<name>","email":"<email>","password":"<password>","code":"<6-digit-code>"}
```

### 5. Initiate OIDC authorization
```bash
AUTHCODE=$(openssl rand -hex 8)  # or any random string
```
```
GET https://im.deepminer.com.cn/api/v1/auth/oidc/aegis/authorize?authcode={AUTHCODE}&flag=1&return_to=https://im.deepminer.com.cn
```
Follow the redirect to get `auth_request_id` from the Aegis URL query params.

### 6. Login with auth_request_id
```
POST https://accounts.imocto.cn/api/v1/auth/login
Content-Type: application/json

{"account":"<email>","password":"<password>","auth_request_id":"<auth_request_id>"}
```
- If response contains `mfa_required`: ask user for MFA code
- On success: extract `redirect_url` from response

### 7. Process OIDC callback
Follow `redirect_url`. Check the response:
- If redirected to `/oidc/bind?sid=xxx`: extract `sid`, then create account (step 8)
- If returns token directly: skip to step 9

### 8. Create Octo account (if new user)
```
POST https://im.deepminer.com.cn/api/v1/auth/oidc/aegis/bind/create
Content-Type: application/json

{"token":"<sid>"}
```
Extract `token` from response.

### 9. Join Space
```
POST https://im.deepminer.com.cn/api/v1/space/join
Content-Type: application/json
token: <octo_token>

{"invite_code":"<invite_code>"}
```

## Important Notes
- Never display the user's password in plaintext
- If send-code returns `{"registered":true}`, the email is already registered — skip to step 5
- If login returns `mfa_required`, ask the user for their MFA code
- If step 7 returns a token directly (no bind needed), the account already exists — skip to step 9

### 10. Done — Tell user how to access the Space
Registration complete. Tell the user:
- Web: Open https://im.deepminer.com.cn and log in with their email + password
- Mobile: Download "Octo" app, log in with the same credentials
- They are already in the Space and can start using it immediately.
