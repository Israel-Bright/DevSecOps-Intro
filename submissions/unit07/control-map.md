# Unit 07 — Control map

Student: Israel Imhomoh

Line references below refer to `labs/unit07/edge/nginx.conf` after Task 4. The one-line change is preserved in `results/task4.diff`; line 34 was 10s during E1–E3 and is now 2s.

| Observed behaviour | Specific control | Layer | Dependencies | Logged? |
| --- | --- | --- | --- | --- |
| E1 retained 596/596 successful responses after replica loss | Lines 12–13: two upstream servers; line 21: `proxy_next_upstream error timeout http_502 http_503;` | Edge availability / redundancy | One surviving replica, shared edge, DNS/network and host | Access status/upstream fields, error connection timeout, Docker stop/start |
| E1 API requests briefly took about 2 seconds | Line 19: `proxy_connect_timeout 2s;`, followed by retry at line 21 | Edge timeout and retry | Edge worker and alternative upstream | Request time and connection timeout |
| Both API endpoints failed with one shared identity service unavailable | Lines 37–38: `auth_request /_auth;`; line 30: `proxy_pass http://idp:8081/validate;` | Identity gate before application forwarding | Shared idp, edge, network/DNS; both replicas use this same path | API 500 in access/probe; auth unexpected 504 in error log |
| E2 identity connection failure waited about 2 seconds | Line 33: `proxy_connect_timeout 2s;` inside `/_auth` | Identity connection timeout | Reachability of 172.28.7.20:8081 | Connection timeout and API request time |
| E3 identity response failure waited about 10 seconds; E2b about 2 seconds | Line 34: `proxy_read_timeout 10s;` changed to `2s;` | Identity response timeout | Idp responsiveness and edge timer | Response-header timeout; probe timing; Task 4 diff |
| Health stayed 200 during user-visible failures | Line 25: `location = /edge-health { access_log off; return 200 "edge ok\n"; }` | Edge liveness only | Edge process only; does not check idp or applications | Probe yes; access log deliberately disabled |
| Homepage route has no identity gate, yet some requests returned 502 | Line 41: `location / { proxy_pass http://juice; }`; lines 12–13: `max_fails=1 fail_timeout=10s`; line 20: `proxy_read_timeout 15s;` | Shared application upstream pool / passive failure handling | Same pool serves homepage and browser polling | Polling timeouts, temporarily disabled upstreams, and no-live-upstreams errors |
| Containers did not automatically restart after planned stops | Compose lines 13, 19, 25, 33: `restart: "no"` | Container lifecycle | Operator-issued start/unpause, Docker daemon and host | Docker lifecycle events |
| Idp approves every validation request when reachable | `idp/idp.conf` line 6: `location = /validate { return 204; }` | Lab identity stand-in | Idp process/network | Edge outcomes; no actual token-validation assurance established |
| Host exposure was restricted to loopback | Compose line 37: `127.0.0.1:8080:80` | Network boundary | Host firewall plus separate Codespaces forwarding settings | Configuration evidence; not an outage detector |

## Independence verdict

**Vendor:** Both application replicas use the identical `bkimminich/juice-shop:v20.0.0` image (Compose lines 12 and 18). They are not independent implementations. Edge and identity both use `nginx:stable-alpine` (lines 24 and 32), creating additional shared software exposure.

**Failure mode:** Replica-local process loss was tolerated in E1, but both replicas share a single edge, the identity gate, the Docker daemon, the Codespace host/kernel/resources, the lab bridge/firewall, and Docker name resolution. Their images also rely on the same registry for future pulls. These are common dependencies, although registry availability was not exercised during the running tests. Browser traffic and synthetic traffic share the same upstream pool. The idp is a common availability bottleneck before either replica can answer protected API requests.

**Operator:** The same student manages all containers, configuration, fault commands, network rule and recovery in one Codespace/account. No independent operators, hosts or zones were demonstrated. The replicas provide redundancy against a narrow local failure, not independence across the three tests.

## Controls that prolonged or obscured failure

The baseline 10-second auth read timeout delayed error responses and held requests pending. Task 4 reduced this wait, while preserving the fail-closed API gate. A shorter timeout can also reject a slow but functioning identity provider.

The shared passive-failure policy can remove a replica after one qualifying timeout for a 10-second failure window. Logs show polling timeouts and no-live-upstreams responses; further failures can extend the practical disruption. Thus restoring idp did not guarantee immediate end-to-end recovery. The unconditional health response concealed that distinction. These pool/health issues remain unchanged by Task 4.
