# Netflix High-Level Design (HLD)

This document contains the architecture diagrams and system design flows for a scalable, subscription-based video streaming platform modeled after Netflix. 

## 1. High-Level Design Overview
The core architecture illustrating the client layer, load balancing, stateless API backend, PostgreSQL database, object storage, and CDN edge delivery.

![HLD Overview](HLD%20overview.png)

---

## 2. System Requirements
The functional and non-functional requirements driving the architectural decisions and component scaling limits.

![Functional & Non-Functional Requirements](Functional%20&%20non%20Functional.png)

---

## 3. Back-of-the-Envelope Estimation
Capacity planning and order-of-magnitude estimates for API traffic, video bandwidth, and storage footprint based on active user assumptions.

![Back of the Envelope Estimation](Back%20of%20the%20envelope%20estimation.png)

---

## 4. Components Used
A breakdown of the specific responsibilities for each major building block in the architecture.

![Components Used](components%20used.png)

---

## 5. Browse & Search Flow
The data path for user catalog requests, highlighting the fast-path Redis cache hit and the fallback to PostgreSQL on a cache miss.

![Browse and Search Flow](browse%20&%20search%20flow.png)

---

## 6. Playback & Streaming Flow
The authorization path demonstrating how the backend grants access while the CDN delivers the high-bandwidth video data directly to the client.

![Playback & Streaming Flow](playback%20&%20streaming.png)

---

## 7. Admin Upload & Encoding Flow
The asynchronous process for uploading original source files, transcoding them into multiple adaptive bitrates, and publishing them to the catalog.

![Admin Upload Flow](admin%20upload.png)

---

## 8. Watch Progress & Resume Flow
The consistency pattern for saving playback progress at regular intervals and allowing users to seamlessly resume across different devices.

![Watch Progress Flow](watch%20progress.png)
