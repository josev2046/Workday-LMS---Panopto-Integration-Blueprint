# Workday LMS - Panopto Integration Blueprint

## Overview

This repository provides a high level architectural blueprint for integrating the Panopto video platform within the Workday Learning Management System (LMS).

The primary objective is to create a seamless user experience, allowing Workday users to access, browse, and search their organisation's Panopto video library directly from the Workday interface. This document assumes that a Single Sign-On (SSO) solution between the two platforms is already in place.

## Key capability: folder-level embedding

While the Panopto API facilitates the embedding of individual videos, a more scalable and user-centric approach involves embedding entire Panopto folders.

<img width="972" height="348" alt="image" src="https://github.com/user-attachments/assets/8578e65a-9ef2-4b32-9400-d59bace4421c" />


The Panopto REST API provides specific endpoints to achieve this. An integration component within Workday can be developed to perform the following sequence:

1.  **Identify user:** The component first identifies the authenticated Workday user.

2.  **Find folders:** It then queries the Panopto API (e.g., using `GET /api/v1/folders/search` or `GET /api/v1/folders/creator`) to retrieve a list of folders relevant to that specific user.

3.  **Get embed URL:** For each folder identified, the component makes a subsequent call to `GET /api/v1/folders/{id}`.

4.  **Render content:** The JSON response from this call contains an `Urls.EmbedUrl` property. This URL is then used as the `src` for an `iframe`, dynamically rendering the complete Panopto folder view within the Workday UI.

This method is superior for browsing as it presents users with their entire, permission-trimmed library or course-specific collections, rather than requiring developers to manage links to individual videos.

## Search functionality: Architectural approaches

A significant benefit of this integration is exposing Panopto's "Smart Search," which indexes spoken words, on-screen text (OCR), and slide content. There are two primary approaches to enabling this search functionality for users.

### Approach 1: In-built folder search (Recommended)

This approach requires no additional development effort beyond the folder-embedding process described above.

The `EmbedUrl` provided by the API renders the standard Panopto folder page. This page **already contains the full-featured "Smart Search" bar**. Users can type queries directly into this search bar, and Panopto will handle the search, returning results with timecodes within the embedded `iframe`.

### Approach 2: Custom API-driven search (Advanced)

This approach involves building a *native* search bar within the main Workday UI, separate from the embedded Panopto folder.

Critically, this approach **must retain the full power of "Smart Search"**. The `GET /api/v1/sessions/search` endpoint queries the complete index, including spoken words (ASR), on-screen text (OCR), and slide content.

This is however a more complex integration, as the custom search bar would need to:

1.  **Capture query:** Capture the user's query from the Workday UI.

2.  **Pass query to API:** Pass this query to the Panopto API using the `GET /api/v1/sessions/search` endpoint.
    * To retrieve the specific timecodes for matches, the `includeFields=Context` parameter must be included in the API call.

3.  **Receive and parse results:** Receive the JSON-formatted search results from the API. The developer is responsible for parsing this data, which will include the list of matching sessions and, if requested, the specific timecode contexts.

4.  **Render custom UI:** Render these results (a list of matching videos and their timecodes) natively within the Workday interface.

5.  **Control player:** When a user clicks a timecode-specific result, the custom UI must use the separate **Panopto Embed API** (a client-side JavaScript library) to command the embedded video player to `seekTo()` that exact point in time.

While this method provides the most deeply integrated feel, it represents a significantly larger development undertaking, as it requires building a custom search results UI and handling the video player-seeking logic.

## Relevant links

* **Panopto REST API Documentation:** <https://demo.hosted.panopto.com/Panopto/api/docs/index.html> (Note: Replace `demo.hosted.panopto.com` with your organisation's Panopto URL).

* **Panopto Index Connector Example:** <https://github.com/Panopto/panopto-index-connector>

* **Panopto embed API:** <https://github.com/josev2046/Panopto-Embed-API-Lab>

* **About smart search:** <https://support.panopto.com/s/article/smart-search-faq>

* **About folders:** <https://support.panopto.com/s/article/How-to-Embed-a-Folder-into-a-Webpage>
  

