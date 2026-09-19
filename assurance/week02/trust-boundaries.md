# Trust boundaries

Threat Model v1.0 — CPS 5981 01, Week 02

Boundaries are derived from the zone each element sits in. A flow whose endpoints are in
different zones crosses one. Direction matters: the two directions between the same pair of
zones are separate boundaries, because the assumption being made differs.

Crossing flows: 4  —  distinct boundaries: 4

## 1. User's browser → Application server

**What crosses:** Product catalog request

**Why this is a real boundary:**  The product catalog request moves from the customer-controlled browser to the application server. The server cannot assume that incoming requests were produced only through the intended interface. It must decide which requested information may be returned and handle request parameters appropriately. This separates customer-controlled requests from server-side processing.

**Confidence, and what would settle it:** I observed a GET request to /rest/products/search?q= with an empty query value. This supports the browser-to-server catalog request in my model. Later, searching for “banana” changed the displayed results while the Network panel recorded only a product-image request. This suggests that the browser filtered previously loaded data rather than sending that search term to the server. I have not verified request validation or access controls. Capturing the initial request after a reload and reviewing its server-side handler would increase my confidence.

## 2. Application server → User's browser

**What crosses:** Product catalog data

**Why this is a real boundary:** Product catalog data moves from the application server into the customer-controlled browser. The interface relies on this data to display products and may filter it locally. The server must release only information intended for the recipient, while the interface must handle returned content safely. Once delivered, the customer can inspect the data, including fields not displayed on the page.

**Confidence, and what would settle it:** I inspected a product-response preview containing names, descriptions, prices, deluxe prices, image filenames, and timestamps. The selected request showed 304 Not Modified and Memory Cache, so the preview represented previously retrieved data rather than a newly transferred response body. This supports the catalog-data flow but does not establish that every returned field should be public or that rendering is safe. A fresh response captured with caches disabled, followed by a review of how the interface uses these fields, would increase confidence.

## 3. Application server → Data store

**What crosses:** Product lookup criteria

**Why this is a real boundary:** My model places a logical boundary between application processing and stored product information. Product lookup criteria cross this boundary. The store relies on the application to request appropriate operations, while storage access controls determine what the application may read or change. I have not yet established whether this separation is enforced in the running system.

**Confidence, and what would settle it:** My confidence is low because I have not observed a database query or inspected storage permissions. Browsing products does not establish how the application retrieves them. Reviewing the product-search code and database configuration would clarify how requests become queries and what access the application has. If no meaningful trust separation exists, I would revise the zone assignment.

## 4. Data store → Application server

**What crosses:** Matching product records

**Why this is a real boundary:** In my proposed model, matching product records move from storage into application processing. The application relies on those records when preparing search results. The trust question is whether stored values are accurate, have an appropriate source, and can be used safely. Being stored does not by itself establish that data is trustworthy. This boundary remains an architectural assumption.

**Confidence, and what would settle it:** My confidence is low because I have not traced a returned product record from storage to the application. Reviewing the product retrieval code, who can modify those records, and how returned fields are checked would help establish the trust assumptions. If the store and application share the same trust context without a meaningful separation, I would revise both database boundary entries.
