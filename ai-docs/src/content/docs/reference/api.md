---
title: API Reference
description: API documentation for AI services.
---

# API Reference

## Endpoints

### `/api/v1/generate`

Generates text based on prompt.

**Method:** `POST`

**Request:**
```json
{
  "prompt": "Hello world"
}
```

**Response:**
```json
{
  "text": "Hello! How can I help you?"
}
```
