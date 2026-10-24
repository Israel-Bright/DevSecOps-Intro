# Unit 07 — Injection Plan

Student: Israel Imhomoh
Environment: GitHub Codespaces, Docker Compose, project u07
Branch: feature/unit07
Target: http://127.0.0.1:8080

## Measured steady state

The 30-second baseline recorded 120 requests: 30 per endpoint.
All requests returned HTTP 200.

| Endpoint | Maximum response time |
| --- | --- |
| /edge-health | 0.007346 seconds |
| / | 0.041553 seconds |
| /api/Products | 0.105242 seconds |
| /rest/admin/application-configuration | 0.026444 seconds |

Evidence: results/probe-baseline.csv.
A preliminary check also confirmed traffic reached both Juice Shop replicas.

## Setup issue resolved before experiments

The legacy firewall FORWARD policy blocked communication between
containers. A rule restricted to the lab bridge br-269e00254237 and
source/destination subnet 172.28.7.0/24 restored communication.
The rule was restored after the Codespace restarted.
This was setup troubleshooting, not a planned fault experiment.

## Scope and safety

Only this nonproduction u07 Compose project is in scope.
The published port remains bound to 127.0.0.1.
Keep the Codespaces forwarded port private.
Inject one fault at a time.

Each experiment uses 30 seconds of steady state, approximately
60 seconds of fault, and 60 seconds of recovery.
Record probe CSVs, Docker events, and nginx logs.
Capture browser behavior during E2 and E3.

For every experiment, abort if any non-u07 container changes,
the host becomes unresponsive, or test traffic targets anything
other than the local lab. Restore the affected service immediately.
Do not begin another experiment until all four endpoints recover.
If recovery fails, stop testing and troubleshoot.

## E1 — Replica loss

Fault: docker compose stop juice-a
Restore: docker compose start juice-a

Hypothesis: The system should survive loss of one replica.
The homepage and API routes should continue through juice-b because
nginx retries upstream connection errors and timeouts.
A brief latency increase is possible during failover.
/edge-health should remain 200.

Detection: Docker stop/start events, nginx upstream errors,
and served_by values shifting to 172.28.7.12:3000.
A health-only monitor may not notice the replica loss.

## E2 — Identity provider stopped

Fault: docker compose stop idp
Restore: docker compose start idp

Hypothesis: /api/Products and /rest/admin/application-configuration
will fail closed, likely returning 500 after the auth subrequest fails.
Both replicas share this dependency, so redundancy will not help.
The homepage HTML and /edge-health should remain 200, although
the browser application may fail to load API-backed content.

Detection: Probe status changes, Docker events, auth_request errors,
and browser behavior. /edge-health alone should miss the outage.

## E3 — Identity provider paused

Fault: docker compose pause idp
Restore: docker compose unpause idp

Hypothesis: API requests will wait for the identity service and then
fail closed, likely with 500 responses after approximately the
10-second auth read timeout. This should be slower than E2.
The homepage HTML and /edge-health should remain available.

Detection: Increased API latency, nginx timeout/auth errors,
Docker pause/unpause events, and browser behavior.
Requests still pending when idp is unpaused may complete successfully.

## Recovery verification

After each restoration, confirm all containers are running and
all four probed endpoints return 200 with response times below
one second before proceeding.

## Pre-experiment commitment

Commit and successfully push this plan before issuing the first
planned stop or pause command. Retain the commit hash and timestamp.
