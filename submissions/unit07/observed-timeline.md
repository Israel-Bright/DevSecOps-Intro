# Unit 07 — Observed timeline

Student: Israel Imhomoh  
Environment: GitHub Codespaces; Docker Compose project `u07`; branch `feature/unit07`.

## Evidence and measurement method

All times below are UTC on 2026-10-24. Each experiment contains 596 requests (149 per endpoint). The baseline contains 120 requests. Results below are whole-run observations, including steady state, injection, and recovery; they are not fault-window-only success rates. Sources: `results/probe-baseline.csv`, `probe-E1.csv`, `probe-E2.csv`, `probe-E3.csv`, `probe-E2b.csv`, `docker-events.csv`, `access.log`, and `error.log`.

The probe's `ts` is the launch timestamp, at one-second precision, not response completion time. Concurrent requests finish out of order. Approximate error visibility is launch time plus `seconds`; Nginx log timestamps corroborate completion. A first/last bad timestamp spans intermittent errors and is not proof of uninterrupted downtime. Docker's event file includes multiline `exec_create`/`exec_start` descriptions; only anchored timestamp/action/container lifecycle records are used for injection times.

The injection plan was committed and pushed before E1, as verified in the terminal: `e21152773f46829b4ff3754ad8cd498026847f6c`, 14:46:21Z. The archive contains the plan, but not Git history. The original predictions are preserved.

## Timestamped events

| UTC | Event and observed result |
| --- | --- |
| 14:42:58–14:43:27 | Baseline: all four endpoints returned 200. |
| 14:48:23 | E1 probe began. |
| 14:48:53–14:48:54 | `juice-a` kill, stop, and die events. |
| 14:48:56 | Nginx logged connection timeouts to 172.28.7.11; retries preserved successful responses. |
| 14:49:54 | `juice-a` start event: 60 seconds after stop. |
| 14:50:52 | Last E1 launch; all 596 requests succeeded. |
| 14:57:27 | E2 probe began. |
| 14:57:58 | `idp` kill/stop/die events. |
| 14:57:59 | First failing API launches; responses completed around 14:58:01 after the 2-second connection timeout. |
| 14:58:33 | First recorded homepage 502 launch. |
| 14:58:58 | `idp` started after 60 seconds. |
| 14:59:56 | E2 recording ended with application 502s still present; recovery was not established within this recording. |
| 14:59:58–15:00:00 | Follow-up terminal probe still showed application 502s and health 200. |
| 15:04:24–15:04:26 | Later terminal probe returned twelve 200s. Exact E2 recovery time is not established by the saved CSV. |
| 15:06:11 | Diagnostic exec event; terminal checks reported both replicas reachable directly. |
| 15:07:43 | E3 probe began. |
| 15:08:13 | `idp` paused; first failing API launches. |
| 15:08:23 | Auth timeout errors logged, about 10 seconds after pause. |
| 15:09:06 | First E3 homepage 502 launch. |
| 15:09:14 | `idp` unpaused after 61 seconds. |
| 15:10:05 | Last failing API launches; last homepage failure was 15:10:04. |
| 15:10:06–15:10:12 | Remaining sampled launches returned 200 for all endpoints. Recovery persisted in the 15:10:14–15:10:16 terminal check. |
| 15:15:02 | Configuration validation/reload after an unsuccessful text replacement; timeout had not changed. |
| 15:17:54 | Corrected 10s-to-2s edit validated and reload issued; diff and subsequent successful probes verified in terminal. |
| 15:19:05 | E2b probe began (repeat of the E3 pause fault with the new timeout). |
| 15:19:36 | `idp` paused; first failing API launches. |
| 15:19:38 | Auth timeout errors logged, about 2 seconds after pause. |
| 15:20:36 | `idp` unpaused after 60 seconds. |
| 15:21:09 | Last failing application launches. |
| 15:21:10–15:21:34 | Remaining sampled launches returned 200; terminal check at 15:21:36–15:21:38 also returned twelve 200s. |

## Per-endpoint measurements

| Run | Endpoint | 200 / total | Success | 500 | 502 | Worst seconds |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| baseline | `/` | 30/30 | 100.00% | 0 | 0 | 0.041553 |
| baseline | `/api/Products` | 30/30 | 100.00% | 0 | 0 | 0.105242 |
| baseline | `/edge-health` | 30/30 | 100.00% | 0 | 0 | 0.007346 |
| baseline | `/rest/admin/application-configuration` | 30/30 | 100.00% | 0 | 0 | 0.026444 |
| E1 | `/` | 149/149 | 100.00% | 0 | 0 | 0.050706 |
| E1 | `/api/Products` | 149/149 | 100.00% | 0 | 0 | 2.127584 |
| E1 | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.010772 |
| E1 | `/rest/admin/application-configuration` | 149/149 | 100.00% | 0 | 0 | 2.015182 |
| E2 | `/` | 111/149 | 74.50% | 0 | 38 | 0.051441 |
| E2 | `/api/Products` | 70/149 | 46.98% | 60 | 19 | 2.010067 |
| E2 | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.011849 |
| E2 | `/rest/admin/application-configuration` | 71/149 | 47.65% | 60 | 18 | 2.011927 |
| E3 | `/` | 108/149 | 72.48% | 0 | 41 | 0.041737 |
| E3 | `/api/Products` | 55/149 | 36.91% | 50 | 44 | 10.005826 |
| E3 | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.007075 |
| E3 | `/rest/admin/application-configuration` | 54/149 | 36.24% | 50 | 45 | 10.006264 |
| E2b | `/` | 112/149 | 75.17% | 0 | 37 | 0.043747 |
| E2b | `/api/Products` | 69/149 | 46.31% | 58 | 22 | 2.010765 |
| E2b | `/edge-health` | 149/149 | 100.00% | 0 | 0 | 0.010973 |
| E2b | `/rest/admin/application-configuration` | 69/149 | 46.31% | 58 | 22 | 2.013497 |

## Detection and browser observations

No automated alert delivery was configured; “detection” here means observable evidence. Docker events recorded each injected stop/pause at the times above. E1 produced no HTTP error-status alert: connection-timeout logs and API latency (up to 2.128s) exposed the lost replica. E2's first API errors were visible approximately 3 seconds after stop; E3's approximately 10 seconds after pause; E2b's approximately 2 seconds after pause. The 1-second launch resolution limits precision.

`/edge-health` returned 200 for every sample in all runs. An HTTP-status-only monitor polling it would never have detected these application outages. It also has `access_log off`, so the probe is the evidence for its successful responses. Docker “Up” alone did not prove application recovery.

During E2 and E3, the browser rendered the Juice Shop header and “All Products” page but displayed no products (“0 of 0”). See `results/E2-browser.png` and `results/E3-browser.png`. The screenshot is evidence of unavailable displayed content, not proof of lost database records. The browser uses additional routes, including `/socket.io/`, beyond the four synthetic endpoints.

## Log evidence

Representative complete lines from `results/error.log`:

```text
2026/10/24 14:48:56 [error] 29#29: *413 upstream timed out (110: Operation timed out) while connecting to upstream, client: 172.28.7.1, server: , request: "GET /rest/admin/application-configuration HTTP/1.1", upstream: "http://172.28.7.11:3000/rest/admin/application-configuration", host: "127.0.0.1:8080"
```

```text
2026/10/24 14:58:01 [error] 29#29: *1440 upstream timed out (110: Operation timed out) while connecting to upstream, client: 172.28.7.1, server: , request: "GET /rest/admin/application-configuration HTTP/1.1", subrequest: "/_auth", upstream: "http://172.28.7.20:8081/validate", host: "127.0.0.1:8080"
```

```text
2026/10/24 15:08:23 [error] 29#29: *2591 upstream timed out (110: Operation timed out) while reading response header from upstream, client: 172.28.7.1, server: , request: "GET /api/Products HTTP/1.1", subrequest: "/_auth", upstream: "http://172.28.7.20:8081/validate", host: "127.0.0.1:8080"
```

```text
2026/10/24 15:19:38 [error] 75#75: *3765 upstream timed out (110: Operation timed out) while reading response header from upstream, client: 172.28.7.1, server: , request: "GET /api/Products HTTP/1.1", subrequest: "/_auth", upstream: "http://172.28.7.20:8081/validate", host: "127.0.0.1:8080"
```

Access logs record request status, upstream address/status and request time; the corresponding auth failures have outer status 500 and auth errors report an unexpected 504. E1 upstream failures were logged despite a final successful retry. The 502s have a separate recorded mechanism: `/socket.io/` response-header timeouts, temporary upstream disabling, and `no live upstreams` for application requests (for example `/` at 14:58:33). These logs support a shared upstream-pool failure, but do not establish that the identity fault caused the polling timeouts. The experiment therefore includes browser-traffic interference, not a clean isolation of identity-only effects.

The preserved `probe-setup-network-failure.csv` and `probe-baseline-previous.csv` concern earlier setup; they are not the successful baseline used in this analysis. E2/E3 log copies are cumulative snapshots, not isolated experiment logs.
