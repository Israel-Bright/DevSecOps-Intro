# STRIDE analysis

Threat Model v1.0 — CPS 5981 01, Week 02

## How to read the results

The worksheet automatically labels unticked categories "Checked, nothing found" and ticked
categories "Applies." Those labels have been clarified below to match the evidence in the notes.
"No confirmed finding; testing not established" is not a completed negative test or proof of safety.
"Potential threat" identifies a concern for analysis, not a demonstrated vulnerability.
Observations, unperformed checks, and architectural assumptions are described for each component.

## Customer (external entity, User's browser)

| Category | Result |
| --- | --- |
| Spoofing | No confirmed finding; testing not established — see notes |
| Tampering | No confirmed finding; testing not established — see notes |
| Repudiation | No confirmed finding; testing not established — see notes |
| Information disclosure | No confirmed finding; testing not established — see notes |
| Denial of service | No confirmed finding; testing not established — see notes |
| Elevation of privilege | No confirmed finding; testing not established — see notes |

**What I found, and what I looked for and did not find:** Found: I could search for products and view their names, prices, descriptions, and reviews while signed in and after logging out. Looked for and did not find: a sign-in requirement for viewing those product details. Not checked: impersonation, unauthorized changes, additional permissions, service disruption, or audit records.


## Browser Interface (process, User's browser)

| Category | Result |
| --- | --- |
| Spoofing | No confirmed finding; testing not established — see notes |
| Tampering | No confirmed finding; testing not established — see notes |
| Repudiation | No confirmed finding; testing not established — see notes |
| Information disclosure | **Potential threat — Information disclosure**; unauthorized exposure not established |
| Denial of service | No confirmed finding; testing not established — see notes |
| Elevation of privilege | No confirmed finding; testing not established — see notes |

**What I found, and what I looked for and did not find:** Found: Product reviews displayed email-shaped reviewer identifiers after logout, raising a potential information-disclosure concern. When I searched for “banana,” the recorded traffic showed a product image request but no new product-data request, suggesting filtering of previously loaded data. Looked for and did not find: a sign-in requirement for viewing the reviewer identifiers. Not established: whether those identifiers are intentionally public or whether returned content is rendered safely.

## Web application (process, Application server)

| Category | Result |
| --- | --- |
| Spoofing | No confirmed finding; testing not established — see notes |
| Tampering | **Potential threat — Tampering**; unauthorized modification not demonstrated |
| Repudiation | No confirmed finding; testing not established — see notes |
| Information disclosure | **Potential threat — Information disclosure**; unauthorized exposure not established |
| Denial of service | No confirmed finding; testing not established — see notes |
| Elevation of privilege | No confirmed finding; testing not established — see notes |

**What I found, and what I looked for and did not find:** Found: GET /rest/products/search?q= returned 200 OK from the network. A product-response preview contained names, descriptions, prices, and image filenames. Tampering with catalog information and disclosure of nonpublic fields are potential threats, not demonstrated vulnerabilities. Looked for and did not find: a new product-data request during the observed banana search. Not checked: catalog write permissions, response-field access controls, input validation, or audit logging.

## Product database (data store, Data store)

| Category | Result |
| --- | --- |
| Spoofing | No confirmed finding; testing not established — see notes |
| Tampering | **Potential threat — Tampering**; unauthorized modification not demonstrated |
| Repudiation | No confirmed finding; testing not established — see notes |
| Information disclosure | **Potential threat — Information disclosure**; unauthorized exposure not established |
| Denial of service | No confirmed finding; testing not established — see notes |
| Elevation of privilege | No confirmed finding; testing not established — see notes |

**What I found, and what I looked for and did not find:** Found: Product records appeared in the application’s response, but this does not establish the underlying database design. My model assumes a product store; unauthorized changes and unauthorized reading are potential Tampering and Information disclosure threats. Not checked: database queries, storage permissions, write access, or audit records. Neither threat has been confirmed, and the database boundary remains an assumption.
