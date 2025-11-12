# API Addendum: Technical Implementation Examples

This document provides illustrative technical examples for the architectural approaches detailed in the main `README.md`. These include example API requests, response payloads, and conceptual data models to guide implementation.

---

## Approach 1: Folder-Level Embedding

This approach focuses on retrieving a specific folder's `EmbedUrl` from the Panopto REST API and rendering it within an iframe.

### 1. API Request: Retrieve Folder Details

The Workday component first identifies the target folder's `id` and queries the Panopto API.

```
GET /api/v1/folders/{id}
Authorization: Bearer <AUTH_TOKEN>`
```

### 2. API Response: Folder Details Payload

The API returns a JSON object containing the folder's metadata. The critical data point is the `Urls.EmbedUrl` property.

[... Example JSON Response ...]

### 3. HTML Rendering: Embedded Iframe

The Workday UI uses the retrieved `EmbedUrl` as the `src` for an `iframe`, rendering the Panopto folder view.

[... Example HTML iframe snippet ...]

---

## Approach 2: Custom API-Driven Search

This approach involves two distinct API interactions: a server-side REST API call for searching and a client-side Embed API call for player control.

### 1. Real-Time Search Query

The custom Workday search UI queries the Panopto search endpoint directly. The `includeFields=Context` parameter is essential for retrieving time-stamped results.

#### API Request: Real-Time Search Query

[... Example HTTP GET request with query parameters ...]

#### API Response: Search Results with Context

The API returns a JSON payload. The `Context` array provides the specific timestamps (in seconds) where the search term was found, enabling "Smart Search" functionality.

[... Example JSON Response with search results ...]

### 2. Client-Side Player Control

After rendering these results, the user can click a specific time. This action triggers a client-side JavaScript call using the Panopto Embed API to control the video player.

#### Client-Side Method Call: Player `seekTo`

[... Example JavaScript function for player.seekTo() ...]

---

## Approach 3: Federated Search Model

This approach does not involve real-time API calls to Panopto for search. Instead, it relies on an intermediary indexing service (like the `panopto-index-connector`) to populate an external search index.

### 1. Conceptual Data: Federated Index Payload

The indexing service extracts all relevant data from Panopto and formats it for the enterprise search index. This conceptual JSON object illustrates the data that would be indexed.

[... Example JSON payload for the search index ...]

### 2. Conceptual API Call: Querying the Enterprise Index

The Workday search bar queries this internal enterprise index, not Panopto. The search index is responsible for security trimming based on the `AllowedUsers` / `AllowedGroups` fields and the authenticated user's identity.

[... Example API POST request to an enterprise search index (e.g., MS Graph) ...]
