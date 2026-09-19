# Abuse cases

Threat Model v1.0 — CPS 5981 01, Week 02

**Mission:** This system succeeds when customers can find products and make informed choices using accurate names, descriptions, and prices, without unnecessary exposure of information about other customers.

Each case names an actor, what they can already do, what they do with it, what stops being
true for the mission, and what that costs.

## Case 1

- **Actor:**   A visitor collecting reviewer identifiers
- **Capability:** can view product reviews and email-shaped reviewer identifiers without signing in, as observed after logout
- **Action sequence:** collect the displayed identifiers across product reviews and, if they are usable contact addresses, use them to send unwanted messages
- **Mission effect:** People who contribute product reviews could receive unwanted contact because their identifiers were exposed alongside their reviews
- **Impact:**  Time spent handling unwanted messages and complaints, and reduced customer willingness to contribute reviews

Because   A visitor collecting reviewer identifiers can collect the displayed identifiers across product reviews and, if they are usable contact addresses, use them to send unwanted messages, People who contribute product reviews could receive unwanted contact because their identifiers were exposed alongside their reviews occurs, costing  Time spent handling unwanted messages and complaints, and reduced customer willingness to contribute reviews.

## Case 2

- **Actor:** a person controlling a malicious browser extension
- **Capability:** Has an extension installed in the customer’s browser with permission to read and change the shop’s page; this is an assumed starting condition
- **Action sequence:** change product descriptions or displayed prices after the catalog data reaches the browser
- **Mission effect:** The affected customer could choose a product using information that differs from what the shop actually offers
- **Impact:**  Wasted customer time, disputed advertised prices, and staff time investigating complaints

Because a person controlling a malicious browser extension can change product descriptions or displayed prices after the catalog data reaches the browser, The affected customer could choose a product using information that differs from what the shop actually offers occurs, costing  Wasted customer time, disputed advertised prices, and staff time investigating complaints.

## Case 3

- **Actor:**  A person operating an automated client
- **Capability:** Can send repeated product-catalog requests to the publicly reachable catalog endpoint using an automated client.
- **Action sequence:** Send repeated catalog requests at a volume that exceeds processing capacity, if effective request limits are absent
- **Mission effect:** Other customers could be unable to retrieve product information in time to decide what to buy
- **Impact:**   Abandoned shopping visits, lost sales opportunities, and staff time restoring normal service

Because  A person operating an automated client can Send repeated catalog requests at a volume that exceeds processing capacity, if effective request limits are absent, Other customers could be unable to retrieve product information in time to decide what to buy occurs, costing   Abandoned shopping visits, lost sales opportunities, and staff time restoring normal service.

## Case 4

- **Actor:** A customer using the product catalog
- **Capability:** can send search requests to the catalog endpoint
- **Action sequence:** submit unexpected search input that the application processes differently from the normal search interface
- **Mission effect:** customers may receive incomplete or misleading product information
- **Impact:** incorrect purchasing decisions, complaints, and staff investigation time

Because A customer using the product catalog can submit unexpected search input that the application processes differently from the normal search interface, customers may receive incomplete or misleading product information occurs, costing incorrect purchasing decisions, complaints, and staff investigation time.

## Case 5

- **Actor:** an automated visitor without an account
- **Capability:** can view product reviews and email-shaped reviewer identifiers while logged out
- **Action sequence:** repeatedly open product pages and collect the displayed identifiers
- **Mission effect:** review contributors may no longer be able to participate without exposing identifying information to visitors
- **Impact:** privacy complaints, reduced willingness to submit reviews, and time spent responding to affected users

Because an automated visitor without an account can repeatedly open product pages and collect the displayed identifiers, review contributors may no longer be able to participate without exposing identifying information to visitors occurs, costing privacy complaints, reduced willingness to submit reviews, and time spent responding to affected users.

## Case 6

- **Actor:** A person who gains unauthorized access to the product-management function
- **Capability:** can submit requests that the application accepts for catalog changes; this access is assumed and not verified
- **Action sequence:** change a product’s price or description in the stored catalog
- **Mission effect:** customers may receive product information that no longer matches the shop’s intended catalog
- **Impact:** incorrect purchasing decisions, refunds or complaints, and loss of confidence in the catalog

Because A person who gains unauthorized access to the product-management function can change a product’s price or description in the stored catalog, customers may receive product information that no longer matches the shop’s intended catalog occurs, costing incorrect purchasing decisions, refunds or complaints, and loss of confidence in the catalog.
