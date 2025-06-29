# Blog Entry Aggregation Method

This BIP defines the core specification for BEAM (Blog Entry Aggregation Method), a simple JSON-based protocol for blog content syndication and aggregation.

## Abstract

BEAM is a modern alternative to RSS/Atom feeds that uses JSON format for better readability, easier parsing, and enhanced extensibility. The protocol defines a standardized structure for blog feeds and individual entries, making it simple for developers to implement both publishers and consumers.

## Motivation

Existing syndication formats like RSS and Atom are based on XML, which can be verbose and complex to parse. BEAM addresses these issues by:

- Using JSON for better developer experience
- Providing a minimal yet flexible structure
- Supporting modern web development practices
- Enabling easy extension for custom fields

## Specification

### Feed Structure

A BEAM feed is a JSON object with the following structure:

```json
{
  "version": "1.0",
  "title": "Blog Title",
  "description": "Blog description",
  "home_page_url": "https://example.com",
  "feed_url": "https://example.com/feed.json",
  "language": "en-US",
  "author": {
    "name": "Author Name",
    "email": "author@example.com",
    "url": "https://author.example.com"
  },
  "last_updated": "2025-06-29T12:00:00Z",
  "items": []
}
```

### Required Fields

#### Feed Level
- `version`: Protocol version string (MUST be "1.0")
- `title`: Feed title
- `feed_url`: URL of the feed itself
- `items`: Array of entry objects

#### Entry Level
- `id`: Unique identifier for the entry
- `title`: Entry title
- `url`: Permalink to the entry
- `published`: ISO 8601 timestamp of publication

### Optional Fields

#### Feed Level
- `description`: Feed description
- `home_page_url`: URL of the associated website
- `language`: Language code (ISO 639-1 format)
- `author`: Author information object
- `last_updated`: ISO 8601 timestamp of last feed update

#### Entry Level
- `content`: Full content (HTML or plain text)
- `summary`: Brief summary or excerpt
- `updated`: ISO 8601 timestamp of last update
- `author`: Entry-specific author information
- `tags`: Array of tag strings
- `category`: Primary category string
- `image`: URL of featured image
- `reading_time`: Estimated reading time in minutes

### Entry Structure

```json
{
  "id": "unique-entry-id",
  "title": "Entry Title",
  "content": "<p>Full content goes here...</p>",
  "summary": "Brief summary of the entry",
  "url": "https://example.com/entry/title",
  "published": "2025-06-29T10:00:00Z",
  "updated": "2025-06-29T11:00:00Z",
  "author": {
    "name": "Entry Author",
    "email": "entry-author@example.com"
  },
  "tags": ["tag1", "tag2", "tag3"],
  "category": "Technology",
  "image": "https://example.com/images/featured.jpg",
  "reading_time": 5
}
```

### Data Types

- **String**: UTF-8 encoded text
- **URL**: Valid HTTP/HTTPS URL
- **Timestamp**: ISO 8601 format (e.g., "2025-06-29T12:00:00Z")
- **Integer**: Whole number (for reading_time)
- **Array**: JSON array of specified type

### Content Guidelines

#### Entry IDs
Entry IDs MUST be unique within a feed and SHOULD remain stable across updates. Recommended formats:
- UUID: `550e8400-e29b-41d4-a716-446655440000`
- Slug-based: `my-blog-post-title`
- Date-based: `2025-06-29-post-title`

#### Timestamps
All timestamps MUST use ISO 8601 format with UTC timezone (Z suffix). Examples:
- `2025-06-29T12:00:00Z`
- `2025-06-29T12:00:00.000Z`

#### Content Format
The `content` field MAY contain:
- Plain text
- HTML markup
- Markdown (if specified by extension)

## HTTP Headers

BEAM feeds SHOULD be served with appropriate HTTP headers:

```
Content-Type: application/json; charset=utf-8
Cache-Control: public, max-age=3600
```

Publishers MAY include:
```
ETag: "feed-version-hash"
Last-Modified: Thu, 29 Jun 2025 12:00:00 GMT
```

## Example Implementation

### Publishing a Feed

```javascript
function generateBeamFeed(blogData, entries) {
  return {
    version: "1.0",
    title: blogData.title,
    description: blogData.description,
    home_page_url: blogData.homeUrl,
    feed_url: blogData.feedUrl,
    language: blogData.language || "en-US",
    author: blogData.author,
    last_updated: new Date().toISOString(),
    items: entries.map(entry => ({
      id: entry.id,
      title: entry.title,
      content: entry.content,
      summary: entry.summary,
      url: entry.url,
      published: entry.publishedAt.toISOString(),
      updated: entry.updatedAt?.toISOString(),
      author: entry.author,
      tags: entry.tags,
      category: entry.category,
      image: entry.image,
      reading_time: calculateReadingTime(entry.content)
    }))
  };
}
```

### Consuming a Feed

```javascript
async function fetchBeamFeed(feedUrl) {
  const response = await fetch(feedUrl);
  const feed = await response.json();
  
  // Validate version
  if (feed.version !== "1.0") {
    throw new Error(`Unsupported BEAM version: ${feed.version}`);
  }
  
  return feed;
}

function processEntries(feed) {
  return feed.items.map(item => ({
    id: item.id,
    title: item.title,
    url: item.url,
    publishedAt: new Date(item.published),
    summary: item.summary || extractSummary(item.content),
    tags: item.tags || []
  }));
}
```

## Extensions

BEAM supports extensions through additional fields. Extension fields SHOULD be prefixed with an underscore to avoid conflicts:

```json
{
  "id": "example-post",
  "title": "Example Post",
  "_analytics": {
    "views": 1250,
    "shares": 45
  },
  "_monetization": {
    "premium": false,
    "price": null
  }
}
```

## Security Considerations

- Feed URLs SHOULD use HTTPS
- Consumers SHOULD validate JSON structure before processing
- Content fields SHOULD be sanitized when displaying HTML
- Publishers SHOULD implement rate limiting on feed endpoints

## Backwards Compatibility

This specification defines version "1.0" of the BEAM protocol. Future versions will maintain backwards compatibility where possible, with version negotiation handled through the `version` field.

## Reference Implementation

Reference implementations in multiple languages are available at:
- JavaScript: `https://github.com/beam-protocol/beam-js`
- Python: `https://github.com/beam-protocol/beam-python`
- Go: `https://github.com/beam-protocol/beam-go`

## Test Vectors

### Minimal Valid Feed
```json
{
  "version": "1.0",
  "title": "Test Blog",
  "feed_url": "https://test.example.com/feed.json",
  "items": [
    {
      "id": "test-post-1",
      "title": "Test Post",
      "url": "https://test.example.com/test-post",
      "published": "2025-06-29T12:00:00Z"
    }
  ]
}
```

### Complete Feed Example
```json
{
  "version": "1.0",
  "title": "Tech Blog",
  "description": "Articles about web development and technology",
  "home_page_url": "https://techblog.example.com",
  "feed_url": "https://techblog.example.com/feed.json",
  "language": "en-US",
  "author": {
    "name": "John Doe",
    "email": "john@techblog.example.com",
    "url": "https://johndoe.dev"
  },
  "last_updated": "2025-06-29T12:00:00Z",
  "items": [
    {
      "id": "intro-to-react-2025",
      "title": "Introduction to React in 2025",
      "content": "<p>React continues to evolve...</p>",
      "summary": "A comprehensive guide to getting started with React",
      "url": "https://techblog.example.com/intro-to-react-2025",
      "published": "2025-06-29T10:00:00Z",
      "updated": "2025-06-29T11:30:00Z",
      "author": {
        "name": "John Doe",
        "email": "john@techblog.example.com"
      },
      "tags": ["react", "javascript", "frontend"],
      "category": "Web Development",
      "image": "https://techblog.example.com/images/react-2025.jpg",
      "reading_time": 8
    }
  ]
}
```

## Authors

- Isaque Véras @isaqueveras

## Copyright

This document is in the public domain.
