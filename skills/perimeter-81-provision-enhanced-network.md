---
name: Provision a Harmony SASE enhanced network with a region and IPSec tunnel
description: Create an enhanced (software-defined) network, add a regional point-of-presence, then connect a site over a dynamic IPSec tunnel, using the Harmony SASE (Perimeter 81) Public API.
api: openapi/perimeter-81-openapi-original.json
operations:
- createEnhancedNetwork
- listEnhancedRegions
- createEnhancedRegion
- createDynamicTunnel
- getEnhancedNetworkHealth
---

# Provision an enhanced network with a region and tunnel

Use this skill to stand up a new Harmony SASE enhanced network and connect a site.

## Authentication (required first)
1. Generate an API key in the Web Management Console (Settings -> API Support). API access requires a Premium Plus or Enterprise plan.
2. Exchange it for a 60-minute access token: `POST /v1/auth/authorize` with body `{"grantType": "api_key", "apiKey": "<API_KEY>"}`.
3. Send `Authorization: Bearer <access_token>` on every call. Base URL (US): `https://api.perimeter81.com/api/rest` (regional EU/AU/IN gateways also available).

## Steps
1. **Create the network** — `createEnhancedNetwork` (`POST /v2.3/networks/enhanced`). Capture the returned `networkId`. Provisioning may be asynchronous (HTTP 202); poll the async-operation status endpoint until complete.
2. **Add a region** — `createEnhancedRegion` (`POST /v2.3/networks/enhanced/{networkId}/regions`) to place a point-of-presence. Use `listEnhancedRegions` (`GET /v2.3/networks/enhanced/{networkId}/regions`) to confirm and get the `regionId`.
3. **Connect a site** — `createDynamicTunnel` (`POST /v2.3/networks/enhanced/{networkId}/tunnels/ipsec/dynamic`) to establish a dynamic IPSec tunnel from the customer edge.
4. **Verify** — `getEnhancedNetworkHealth` (`GET /v2.3/networks/enhanced/{networkId}/health`).

## Rules
- Rate limit: 500 requests per 5 minutes per IP — batch and back off on 429/limit.
- Treat 202 responses as async: poll for completion before dependent steps.
- Errors are plain JSON (`components.schemas.Error`, not RFC 9457). 401 = re-authorize; 402 = plan/license insufficient; 403 = key lacks the required permission; 404 = network/region not found. See `errors/perimeter-81-problem-types.yml`.
