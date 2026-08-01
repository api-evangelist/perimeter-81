---
name: Define network objects and update a Harmony SASE firewall policy
description: Create reusable address and service objects, then apply them by updating a network's firewall policy, using the Harmony SASE (Perimeter 81) Public API.
api: openapi/perimeter-81-openapi-original.json
operations:
- postObjectsAddresses
- getObjectsAddresses
- postObjectsServices
- getFirewallPolicy
- updateFirewallPolicy
---

# Define objects and update a firewall policy

Use this skill to build reusable network objects and enforce them in a network's firewall policy.

## Authentication (required first)
Obtain a bearer token via `POST /v1/auth/authorize` (`{"grantType":"api_key","apiKey":"<API_KEY>"}`) and send `Authorization: Bearer <access_token>`. See `authentication/perimeter-81-authentication.yml`.

## Steps
1. **Create an address object** — `postObjectsAddresses` (`POST /v2.3/objects/addresses`). List/verify with `getObjectsAddresses` (`GET /v2.3/objects/addresses`) and capture the `objectId`.
2. **Create a service object** — `postObjectsServices` (`POST /v2.3/objects/services`) for the ports/protocols the rule targets.
3. **Read the current policy** — `getFirewallPolicy` (`GET /v2.3/networks/{networkId}/policy`) so you edit against the current rule set.
4. **Apply the rule** — `updateFirewallPolicy` (`PUT /v2.3/networks/{networkId}/policy`) referencing the address/service objects created above.

## Rules
- Objects are reusable across policies; prefer referencing an `objectId` over inlining addresses.
- `PUT /policy` replaces policy state — read-modify-write from `getFirewallPolicy` to avoid dropping existing rules.
- Rate limit: 500 requests / 5 minutes / IP. Handle 403 (permission) and 404 (network/object not found) per `errors/perimeter-81-problem-types.yml`.
