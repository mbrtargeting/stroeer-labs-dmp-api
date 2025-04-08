# Contextual Data API - v1
## Overview

The IAB Classifier API provides content categorization services based on the IAB Content Taxonomy v2.2. This API allows clients to submit URLs for analysis and receive predicted IAB categories for the content at that location. The service uses a request-polling model where clients may need to make multiple requests to retrieve results, especially for previously unanalysed URLs that require crawling.

## Services Offered

- **Content Classification**: Submit URLs to receive IAB category predictions
- **Cached Results**: Immediate results for previously analyzed URLs
- **Asynchronous Processing**: For new URLs, the system initiates crawling and analysis in the background
- **Polling Mechanism**: Clients check back periodically until classification is complete

## Authentication

Our API uses token-based authentication to secure access. Here’s what you need to know:

### How It Works

**Authentication Flow**:  
To access our API, you must first obtain a token from an external **Auth API**. If you have not received your credentials yet, please [contact us via our website](https://osdatasolutions.de/#kontakt). Use these credentials to authenticate with the Auth API.

**Token Provision**:  
Tokens are obtained by sending your provided credentials to the Auth API and must be used as received.

- **Auth API Endpoint**:  
  ```bash
  POST https://api.osdata.solutions/auth
  ```

- **Request Payload**:
  ```json
  {
    "email": "foo@osdata.solutions",
    "password": "your-password"
  }
  ```

- **Response Payload**:
  ```json
  {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6...",
    "expiresIn": 3600 // In seconds
  }
  ```
*Security Tip*: Keep your email and password safe—do not store them in code or version control.

### How to Include the Token

- **Request Header**: Add the token in the `Authorization` header when making API requests.
  - **Example Header Format**:  
  ```bash  
  Authorization: Bearer {token}
  ```  
  Replace **{token}** with the token provided to you.

**Security Note**:
 For security reasons, our API does not accept HTTP requests. All requests must be made over **HTTPS**.

### Common Authentication Errors

- **401 Unauthorized**: This error occurs when the token is missing or incorrect.

### Security Recommendations

**Use HTTPS**: Always use HTTPS to encrypt your requests.

**Keep Your Token Secure**: Do not share your token publicly or embed it in publicly accessible code.

**Monitor for Unauthorized Use**: If you suspect your token has been compromised, contact support immediately.

## Request

### Base URL

- **API Base URL**: `TBD`

### HTTP Method

- **Method**: `POST`

### Request Body

- **Format**: `JSON`

All relevant information for the request is sent as fields in the JSON body.

### Fields Overview

| Field Name |  Data Type | Description  | Requirement |  
| ---- |  ---- | ----  | ---- |  
| `url` | String | An absolute URL including a scheme (http:// or https://), or a MAID in case "trafficType": "MAID".  Note that setting this field to a MAID with a different traffic type will result in HTTP 400 response and using a URL when the traffic type is MAID will always result in a response with no contextual information provided. | **Required** |
| `trafficType` | Enum | Possible values: <ul><li>`SITE` - normal page opened in a browser regardless of device</li><li>`MAID` - mobile app traffic where URL cannot be determined/shared and therefore the mobile app ID is provided for contextual classification</li><li>`CTV` - traffic from smart/internet-connected TV</li></ul> | **Optional (SITE by default)** |

**Example JSON Payloads:**

```json
// For Site traffic
{
  "trafficType": "SITE",
  "url": "https://www.sports.com"
}

// For App traffic
{
  "trafficType": "MAID",
  "url": "wetter.at_app"
}
```

### Headers & Authentication
- **Headers:**
  - `Content-Type: application/json`
  - `Authorization: Bearer token_provided`
- **Authentication:** Include the token in the Authorization header.

### Examples & Code Snippets

- **Successful Request Example** (Omitting `trafficType` - default **SITE**)  
```bash
curl -X POST "TBD" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${CONTEXTUAL_API_TOKEN}" \
  -d '{
        "url": "https://www.sports.com"
      }'
```

- **Successful Request Example** (Setting `trafficType` explicitly)  
```bash
curl -X POST "TBD" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${CONTEXTUAL_API_TOKEN}" \
  -d '{
        "url": "https://www.sports.com",
        "trafficType": "SITE"
      }'
```

- **Error Request Example:**  (MAID traffic expected)
```bash
curl -X POST "TBD" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${CONTEXTUAL_API_TOKEN}" \
  -d '{
        "url": "https://www.sports.com",
        "trafficType": "MAID"
      }'
```

## Response

**All responses are returned in JSON format.**
There are two types of responses:

### 1. Successful Response (HTTP 200)
When the HTTP status code is **200**, the response follows this structure:

```json
{
  "status": 10,
  "segments": ["69abe"],
  "ttl": 213456
}
```

Below is a table describing each field in the successful response:

| Field |  Data Type | Description  |
| ---- |  ---- | ----  |
| `status` | Number | The Status of the response. This indicates when and if another request should be made. For more details, refer to [Status Codes](#status-codes). |
| `segments` | String Array | A list of IAB categories. Current version used is [IAB Tech Lab Content Taxonomy 2.2](https://github.com/InteractiveAdvertisingBureau/Taxonomies/blob/main/Content%20Taxonomies/Content%20Taxonomy%202.2.tsv). |
| `ttl` | Number | The time to live (TTL) in seconds. These are closely tied to the `status` field as they indicate for how long the data will be retained in our system for a Success status or how long before another request can be made for a record in Pending status. For details, see [Time To Live Values](#time-to-live-values). |

**Status codes are grouped into four ranges:**

| Status Code Range |  Category | Description  |  
| ---- |  ---- | ----  |  
| `10–19` | SUCCESS | The url was correctly classified. |
| `20–29` | IN-PROGRESS | Url classification in progress. |
| `30–39` | IGNORED | The url was ignored. |
| `40–49` | FAILURE | Url classification failed. |

**Example of a FAILURE Response (HTTP 200):**

```json
{
  "status": 42,
  "segments": [],
  "ttl": 213456
}
```

### 2. Error Response (HTTP 4xx/5xx)  

When the HTTP status code is not **200**, the response uses the following schema:

```json
{
  "message": "No URL was provided"
}
```

Below is a table describing each field in this error response:

| Field |  Data Type | Description  |  
| ---- |  ---- | ----  |  
| `message` | String | A descriptive error message explaining why the request failed. |


### Status Codes

For HTTP 200 responses, the `status` field returns codes grouped into four categories. Each category is defined below.

**Success Codes (10–19)**  

These codes indicate that the operation has completed successfully.  

| Status Code |  Description  |  
| ---- |  ----  |  
| `10` | Url Successful with categories. |  

---  

**In-Progress Codes (20-29)**

These codes signify that the url is currently being processed.

| Status Code |  Description  |  
| ---- |  ----  |  
| `20` | Url accepted; processing has begun. |

--- 

**Ignored Codes (30-39)**

These codes indicate that the url was intentionally not classified for some reason.

| Status Code |  Description  |  
| ---- |  ----  |  
| `30` | Url was ignored. |

--- 

**Error Codes (40-49)**

These codes indicate that there was a problem while trying to classify the url. Further description is explained below.

| Status Code |  Description |  Message |  
| ---- |  ----  |  ---- |  
| `40` | Url failed to be processed. | Some description of the error |  


### Time To Live Values

| Status |  Default |  
| ---- |  ---- |  
| `SUCCESS` | <ul><li>86400 (1 day) for homepages / Second-level domains</li><li>2592000 (30 days) for subpages / article pages and MAIDs</li></ul> |  
| `IN-PROGRESS` | 300 (5 minutes) |  
| `IGNORED` | 86400 (1 day) |  
| `FAILURE` | `MAX_INT` |  
