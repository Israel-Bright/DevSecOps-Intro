# Written analysis — Threat Model v1.0

CPS 5981 01, Week 02

**Fork:** _not answered_
**Commit:** _not answered_
**Visibility:** _not answered_
**Collaborators:** none
**Declared AI use:** none

## 1. Which of your abuse cases are supported by evidence you gathered, and which rest on reasoning about the design?

Case 1 is partly supported by evidence: after logging out, I could still view product reviews with email-shaped reviewer identifiers. The possibility that someone collects them and sends unwanted messages is a reasoned consequence; I did not test automated collection or send messages. Case 5 uses the same observation, but automated collection remains hypothetical. Case 2 is design reasoning based on catalog data reaching the browser; I did not observe a malicious extension or altered prices. Case 3 is partly supported by observing GET /rest/products/search?q= return 200 OK, but I did not test automated volume, capacity, or rate limits. Case 4 is design reasoning about unexpected search input; I did not test malicious input or confirm different server-side processing. Case 6 is design reasoning about unauthorized catalog editing; I did not inspect an editing function, permissions, or database write controls.

## 2. What does STRIDE not surface for this target?

STRIDE organizes threats by component, but it does not by itself show the business consequence of exposing reviewer identifiers or displaying inaccurate catalog information. It also does not show how a chain across components changes the outcome: catalog data reaches the browser, the browser displays it, and customers make decisions from it. STRIDE does not establish whether reviewer identifiers are intentionally public, how many people could be affected, or whether a potential issue would change customer behavior. It also does not replace testing of request volume, business rules, or operational recovery.

## 3. Which trust boundary are you least confident about, and what would settle it?

I am least confident about the Application server → Data store and Data store → Application server boundaries. I observed product records in a browser response, but I did not observe the database query or inspect the storage implementation. My model assumes a separate product store and assumes that lookup criteria and matching records cross that boundary. Reviewing the product-retrieval code and storage configuration, or tracing one catalog request through the application, would show whether the separation and access controls exist. If the application stores the catalog in memory or uses a different arrangement, I would revise the diagram.

## 4. If this system had twice as many users, which case moves up your list, and why?

Case 5 would move up because twice as many users could mean more reviewers and more visitors who can view the reviewer identifiers. The number of people potentially affected by the same exposure would increase, along with privacy complaints and the loss of trust in submitting reviews. This is a prioritization judgment, not evidence that the application currently exposes twice as many identifiers. Case 3 could also become more important because higher normal traffic would make it harder to distinguish automated catalog requests from ordinary use, but I did not measure capacity or rate limits.
