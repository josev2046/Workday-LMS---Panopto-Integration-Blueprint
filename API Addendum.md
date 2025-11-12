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

```
{
  "Id": "b36925af-fa27-4ec2-a64c-b39201022610",
  "Name": "Course Videos - Module 1",
  "Urls": {
    "EmbedUrl": "https://<your-panopto-site>/Panopto/Pages/EmbeddedList.aspx?folderID=b36925af-fa27-4ec2-a64c-b39201022610&embedded=1",
    "SharingUrl": "..."
  },
  "State": 0
}
```

### 3. HTML Rendering: Embedded Iframe

The Workday UI uses the retrieved `EmbedUrl` as the `src` for an `iframe`, rendering the Panopto folder view.

```
<iframe
  src="https://<your-panopto-site>/Panopto/Pages/EmbeddedList.aspx?folderID=b36925af-fa27-4ec2-a64c-b39201022610&embedded=1"
  width="770"
  height="433"
  style="border: 0;">
</iframe>
```
---

## Approach 2: Custom API-Driven Search

This approach involves two distinct API interactions: a server-side REST API call for searching and a client-side Embed API call for player control.

### 1. Real-Time Search Query

The custom Workday search UI queries the Panopto search endpoint directly. The `includeFields=Context` parameter is essential for retrieving time-stamped results.

#### API Request: Real-Time Search Query

```
GET /api/v1/sessions/search?searchQuery=integration&includeFields=Context
Authorization: Bearer <AUTH_TOKEN>
```

#### API Response: Search Results with Context

The API returns a JSON payload. The `Context` array provides the specific timestamps (in seconds) where the search term was found, enabling "Smart Search" functionality.

```
{
  "Results": [
    {
      "SessionId": "a1a30113-1b91-47a7-8998-ab0c014798b5",
      "SessionName": "Integration Blueprint Review",
      "SessionUrl": "...",
      "Context": [
        {
          "Timestamp": 124.5,
          "Text": "...discussing the integration approach..."
        },
        {
          "Timestamp": 331.0,
          "Text": "...this integration is key..."
        }
      ]
    }
  ],
  "TotalHits": 1
}
```

### 2. Client-Side Player Control

After rendering these results, the user can click a specific time. This action triggers a client-side JavaScript call using the Panopto Embed API to control the video player.

#### Client-Side Method Call: Player `seekTo`

```
// Assumes 'player' is an initialized Panopto Embed API object
// and 'timeInSeconds' is 331.0 from the API response

function onResultClick(timeInSeconds) {
  // Use the Embed API's seekTo method to jump to the timestamp
  player.seekTo(timeInSeconds);
}
```

---

## Approach 3: Federated Search Model

This approach does not involve real-time API calls to Panopto for search. Instead, it relies on an intermediary indexing service (like the `panopto-index-connector`) to populate an external search index.

### 1. Conceptual Data: Federated Index Payload

The indexing service extracts all relevant data from Panopto and formats it for the enterprise search index. This conceptual JSON object illustrates the data that would be indexed.

```
{
  "ObjectID": "panopto-a1a30113-1b91-47a7-8998-ab0c014798b5",
  "Provider": "Panopto",
  "Title": "Integration Blueprint Review",
  "Description": "A meeting to review the Workday integration...",
  "Url": "https://<your-panopto-site>/Panopto/Pages/Viewer.aspx?id=a1a30113-1b91-47a7-8998-ab0c014798b5",
  "EmbedUrl": "https://<your-panopto-site>/Panopto/Pages/Embed.aspx?id=a1a30113-1b91-47a7-8998-ab0c014798b5",
  
  "Content": "integration approach integration is key...",
  "Transcripts_ASR": "hello and welcome today we are discussing the integration approach...",
  "OnScreenText_OCR": "Integration Architecture Diagram REST API...",
  "SlideContent": "Key Considerations Approach 1 Approach 2...",
  
  "AllowedUsers": [
    "workday-user@your-org.com",
    "another-user@your-org.com"
  ],
  "AllowedGroups": [
    "workday-group-id-for-all-staff",
    "workday-group-id-for-it-dept"
  ]
}
```

### 2. Conceptual API Call: Querying the Enterprise Index

The Workday search bar queries this internal enterprise index, not Panopto. The search index is responsible for security trimming based on the `AllowedUsers` / `AllowedGroups` fields and the authenticated user's identity.

```
POST /v1.0/search/query
Content-Type: application/json

{
  "requests": [
    {
      "entityTypes": ["externalItem"],
      "query": {
        "queryString": "integration"
      },
      "from": 0,
      "size": 10
    }
  ]
}
```
