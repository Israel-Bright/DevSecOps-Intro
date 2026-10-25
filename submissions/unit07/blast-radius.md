# Unit 07 — Blast radius and Task 4

Student: Israel Imhomoh

## Predictions compared with measurements

| Experiment | Plan prediction | Measurement | Finding / gap |
| --- | --- | --- | --- |
| E1: stop juice-a | Surviving replica handles traffic; possible latency increase | All 596 requests returned 200; maximum API latency 2.128s | Replica redundancy worked for this failure, with a measurable latency cost. |
| E2: stop idp | APIs fail closed; homepage and edge health remain 200 | Products 70/149 successes; configuration 71/149; 60 API 500s each; homepage 38/149 responses were 502; health 149/149 successful | Shared identity defeated replica redundancy, but the predicted homepage isolation did not hold. Additional pool failures widened the measured radius. |
| E3: pause idp | API errors after roughly 10s; homepage/health available | Products 55/149 successes; configuration 54/149; API worst times about 10.006s; homepage 41/149 responses were 502; health 149/149 successful | Hung identity produced the predicted delay. Homepage failures again exceeded the identity-only prediction. |

The extra 502s are not all attributable to identity. `results/error.log` shows browser `/socket.io/` timeouts, temporary upstream disabling and `no live upstreams`, including on `/`. This is evidence of interference through a common upstream pool. A controlled browser-free comparison would be needed to isolate its contribution; no such experiment is claimed here.

## Radius: role, location and time

**Role:** The observed visitor could load the page shell but could not see products during E2/E3. Both probed API routes were affected regardless of which application replica would serve them. Other users or administrative workflows relying on `/api/` or `/rest/` would share the configured identity gate, but no authenticated account, checkout or admin workflow was directly tested.

**Location:** The injected faults were confined to one container in the local `u07` project. E1 affected one replica; E2/E3 affected a dependency shared by both. The observed radius reached API-backed content and, intermittently, homepage requests through the edge's shared upstream pool. No deletion, corruption, confidentiality loss or persistent data loss was demonstrated. No non-lab system was intentionally targeted.

**Time:** E1's stopped interval was 60 seconds (14:48:54–14:49:54 UTC). E2's idp outage lasted 60 seconds (14:57:58–14:58:58), but errors remained at the final CSV launch, 58 seconds after restoration, and in the immediate follow-up probe through 15:00:00. A later probe at 15:04:24–15:04:26 was successful; the precise recovery time is unknown. E3's pause lasted 61 seconds (15:08:13–15:09:14). Its last failing API launch was 15:10:05, 51 seconds after restoration; all sampled launches from 15:10:06 through the end succeeded. These spans include intermittent failures and are not continuous-downtime estimates.

## Larger-system implication

In a hypothetical deployment with three zones and 40 services, placing replicas in different zones would not remove dependence on one shared identity provider. Every service synchronously requiring that provider could lose protected operations together; independently served public content might survive. For illustration only, 1,000 identity-dependent requests per second over a 60-second total identity outage exposes up to 60,000 arriving requests to failure or delay. This is a stated scaling assumption, not traffic measured by this lab or a claim of lost transactions. Actual loss depends on retries, queues, caches, route policy and recovery tails.

## Task 4: fail faster while remaining closed

Changed only `proxy_read_timeout` inside `location = /_auth` from 10s to 2s (edge configuration line 34). The two-second connection timeout and all route policies stayed the same. `nginx -t` passed and the edge was reloaded. The diff is `results/task4.diff`.

E3 was the worse identity experiment by API delay and whole-run API success count, so its pause/unpause fault was repeated as **E2b**, following the lab's requested filename. E2b is not a second stopped-idp run. It retained the 150-second probe with roughly 30 seconds before injection, 60 seconds paused and 60 seconds recovery; browser refresh traffic was included again.

The decision preserves fail-closed behaviour for `/api/` and `/rest/`: a failed identity subrequest does not bypass the gate. `/` remains an unguarded route by design, not a new fail-open fallback. The benefit is shorter waiting on a hung provider. The trade-off is rejection of legitimate requests when an otherwise functioning provider takes more than two seconds to respond; clients may also retry sooner. This lab's idp is an always-approve stand-in, not a real token verifier.

| Run | Endpoint | 200 / total | Success | 500 | 502 | Worst seconds |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| E3 | `/` | 108/149 | 72.48% | 0 | 41 | 0.041737 |
| E3 | `/api/Products` | 55/149 | 36.91% | 50 | 44 | 10.005826 |
| E3 | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.007075 |
| E3 | `/rest/admin/application-configuration` | 54/149 | 36.24% | 50 | 45 | 10.006264 |
| E2b | `/` | 112/149 | 75.17% | 0 | 37 | 0.043747 |
| E2b | `/api/Products` | 69/149 | 46.31% | 58 | 22 | 2.010765 |
| E2b | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.010973 |
| E2b | `/rest/admin/application-configuration` | 69/149 | 46.31% | 58 | 22 | 2.013497 |

Maximum API 500 response time fell from 10.006264s to 2.013497s, a reduction of about 79.88%. This measures faster failure, not restored availability. The Products/configuration API 500 count increased from 50 each to 58 each: shorter waiting gives fewer requests a chance to survive pending until unpause. Overall API successes increased to 69 each, but the separate 502 counts also changed; one mixed-traffic run does not establish that the timeout edit caused every success-rate difference.

E2b's pause lasted 60 seconds, 15:19:36–15:20:36 UTC. Application errors persisted intermittently through launches at 15:21:09, 33 seconds after restoration. Remaining launches from 15:21:10 through 15:21:34 succeeded, corroborated by twelve 200s in the final terminal probe. The recovery-tail difference is observed, but is not isolated proof of an effect from the timeout change.

## Residual risk

The identity provider, edge and host remain common dependencies. APIs still fail while identity is unavailable. Health stayed 200 in all 149 E2b samples. The repeat also retained 37 homepage 502s and 22 502s on each API endpoint, so the application-pool problem remains. The E2/E3 browser screenshots show empty product lists; the repeat's CSV/logs establish faster responses, not a fully working browser or checkout. Truthful dependency health, polling-aware upstream configuration and independent failure domains are possible future work, not changes made in this one-change experiment.
