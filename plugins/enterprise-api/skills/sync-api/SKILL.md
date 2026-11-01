---
name: sync-api
description: Build against the Sync Enterprise API. Use when getting a DPoP-bound access token, signing the client assertion or DPoP proofs, calling Sync API endpoints, fixing their errors, or verifying Sync webhooks.
---

# Sync Enterprise API

## Before you start

Get these from the developer:

- The client ID and base URL. Both are in the token program on the client's page in [API keys](https://developers.sync.vc/keys).
- The path to the client's private key file.

Keep the private key and webhook signing secret out of code, commits, logs and chat. Load them at runtime from a file or secret store.

## Get a token

In TypeScript, use the program on the client's page. Otherwise, send:

```bash
curl $BASE_URL/v1/oauth/token \
  -H "DPoP: $PROOF" \
  -d grant_type=client_credentials \
  -d client_id=$CLIENT_ID \
  -d client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer \
  -d client_assertion=$ASSERTION
```

```json
{ "access_token": "sync_at_test_…", "token_type": "DPoP", "expires_in": 300 }
```

Tokens last 5 minutes, so reuse them.

## Call the API

Send a new proof with every request. Make `GET /v1/whoami` the first call.

```bash
curl $BASE_URL/v1/whoami \
  -H "Authorization: DPoP $ACCESS_TOKEN" \
  -H "DPoP: $PROOF"
```

It returns the tenant, the client, the mode (`test` or `live`) and when the token expires. The endpoint list is in the [API reference](https://developers.sync.vc/docs/reference). It's signed-in only, so ask the developer.

## Sign the assertion and proof

Sign both with the private key: `ES256` for P-256, `EdDSA` or `Ed25519` for Ed25519, `PS256` for RSA.

| Assertion claim | Value |
|---|---|
| `iss`, `sub` | Client ID |
| `aud` | Base URL |
| `exp` | At most 5 minutes away |
| `jti` | Unique, single use |

The proof's header has `typ: dpop+jwt` and the public key as `jwk`.

| Proof claim | Value |
|---|---|
| `htm` | The request method |
| `htu` | The request URL, without the query |
| `iat` | Now, within 60 seconds |
| `jti` | Unique, single use |
| `ath` | Base64url SHA-256 of the access token. API calls only. |

## Errors

The token endpoint returns `{ "error": "<code>" }`. The API returns `401` with a problem body whose `code` names the error. A `401` means the request didn't run, so retrying is safe.

| Code | Fix |
|---|---|
| `invalid_client` | Check the assertion, the client's status and its IP allowlist. |
| `invalid_token` | Get a new token from an allowed IP. |
| `invalid_dpop_proof` | Sign a new proof for this request with the key. |
| `wrong_token_type` | Send `Authorization: DPoP <token>` from this environment. |
| `tenant_inactive` | Email help@sync.vc. |
| `rate_limited` | Wait `Retry-After` seconds. |

## Webhooks

An admin adds an endpoint in [Webhooks](https://developers.sync.vc/webhooks) and copies its signing secret with **Show**. We POST each event:

```json
{"id":"0f8c2a51-…","type":"deal.created","timestamp":"2026-11-01T14:02:11.000Z","data":{"object":"deal","id":"…"}}
```

Headers are `webhook-id` (the event ID), `webhook-timestamp` (send time) and `webhook-signature`. The only event is `deal.created`.

1. Verify the raw body before you parse it. We sign per [Standard Webhooks](https://www.standardwebhooks.com), so use its library, like `svix` in TypeScript:

   ```ts
   import { Webhook } from "svix";

   const body = await request.text();
   const event = new Webhook(secret).verify(body, Object.fromEntries(request.headers));
   ```

2. Return `400` if verification throws.
3. Skip any `webhook-id` you've already processed. Events can arrive more than once and out of order.
4. Return a `2xx` within 10 seconds. Anything else, including a redirect, fails. We try each event up to 12 times over about 34 hours, then stop.
