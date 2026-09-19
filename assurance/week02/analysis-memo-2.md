# Analysis Memo 2

CPS 5981 01, Week 02 — 10 points, two pages maximum

## As a vulnerability

The finding: The application’s root response does not send a Content-Security-Policy header, so the browser receives no stated policy restricting which scripts may execute on the page.
Named as: Missing Content-Security-Policy header — CWE-693, Protection Mechanism Failure.

## As a risk

If an attacker gets script content to execute in a page viewed by a signed-in customer, then that script may act with the customer’s browser session, and the application may not be able to show what happened.

## Which one goes in front of the manager, and why

I would present the risk framing to the application security manager, who must decide whether to prioritize security-header remediation. The risk framing connects the missing header to possible session misuse and loss of accountability while stating what remains unverified. That gives the manager a decision basis without claiming that script injection or account compromise has already been demonstrated.

## What I am not sure about

I confirmed that the root response did not include a Content-Security-Policy header, but I did not test whether script injection is possible or whether other routes send a policy. I also did not verify the session cookie’s protections or determine whether the missing header directly leads to account misuse. I would review all relevant responses and test the application’s output handling in the authorized lab environment.
