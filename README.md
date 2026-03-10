# Integration Guide

This guide explains how to embed SK8 components into your application, both on the frontend (as web components) and backend (via npm package).

## Prerequisites

Before beginning, ensure you have:
- A backend server (Node.js)
- Ability to serve static files from your backend

---

## Step 1: Obtain Your Access Token

1. Ask SK8 support to provide an API key

### 🔐 Security Notes
- **Never expose this token in client-side code** (browser, mobile apps, etc.)
- Store the token in environment variables or a secure secrets manager

---

## Step 2: Install the Backend SDK

Install the SK8 npm package on your backend server:

```npm i npm-middleware-repo```

### Create the Middleware Endpoint

The SDK provides a middleware function that proxies requests from your embedded components to the SK8 API.
This SDK tries to be as much framework-agnostic as possible by using standard Node APIs (`res.statusCode`, `res.setHeader`, `res.end`) and can run in any Node-based framework. Examples use Express because it is the most common setup.

Add an additional middleware before the SK8 middleware that gets
the requesting client identifier. Cliend identifier can be anything
you use in order to uniquely identify users in you application,
but it must be the same client identifier that you provided during SK8 tenant creation.
This middleware must set `clientId` attribute on the request object.

```
// Example: Express.js implementation
const express = require('express');

import { initializeSK8Middleware } from 'npm-middleware-repo';

const app = express();

app.use(req => {
  req.clientId = <your-cliend-id-provider>;
  next();
});

// Initialize the middleware with your API key
const sk8Middleware = initializeSK8Middleware({
  apiKey: process.env.SK8_API_KEY,
  baseUrl: "https://app-dev.sk8.ai/api"
});

app.use(express.json()) 
app.post('/api/sk8-embedded', sk8Middleware);
app.use(errorHandler)

// Full order (preferred): body parser (mandatory) → cliendId extractor middleware (mandatory) → SK8 middleware → error handler (optional)
```

**Important:**
- `baseUrl` should point to `https://app-dev.sk8.ai/api`

**Note:** The middleware automatically:
- Forwards requests to the SK8 API
- Attaches your API key for authentication

If you don't use Express then make sure that these properties are set for `req` and `res`:

**`req` properties:**
- **url** – request path including query parameters
- **method** – request method (GET, POST, etc.)
- **headers** – request headers; The middleware does not forward incoming headers except `Content-Type`. Authentication is handled via your SK8 API key.
- **body** – request body should be already parsed as JSON (e.g. via `express.json()` or another body parser); required for methods that send a body (POST, PUT, PATCH, etc.)

**`res` properties** (standard Node `ServerResponse`):
- **statusCode** – set by the middleware for the response status
- **setHeader** – standard Node API to set headers
- **end** – standard Node API to send the response

**`next`**
- Typical 'next' function in a middleware chain.
This function will be called only on error response from the SK8 API because typically this embedded middleware should be the last in the chain before the error middleware.
If 'next' is not a function then on SK8 API error response a generic
500 error will be returned as a response so there are no hanging requests left.
---

## Step 3: Serve the Component Script

Download the latest component build and serve it from your backend:

1. **Download** the component scripts:
   ```https://github.com/uladzislau-smirnou/script-documentation```

2. **Save** to the directory of your choise, for example:
   ```
   # Example directory structure
   public/
   ├── embed/
   │   └── pipelines-embed.js
   └── index.html
   ```

3. **Configure** your server to serve static files:
   ```
   // Express example
   app.use('/public', express.static('public'));
   ```
---

## Step 4: Embed the Component in Your Frontend

### Load the Script
Include the component script in your HTML:

```
<head>;
  <!-- Load the SK8 component script -->
  <script src="/public/embed/pipelines-embed.js"></script>;
</head>
```

### Add the Web Component
Place the web component anywhere in your application.

```
<pipelines-embed
  base-api="https://your-backend.example.com/api/sk8-embedded"
/>
```

### Framework-Specific Examples

**React:**
```
function MyComponent() {
  return (
    <div>
      <pipelines-embed
        base-api="https://your-backend.example.com/api/sk8-embedded"
      />
    </div>
  );
}
```

**Vue:**
```
<template>
  <div>
    <pipelines-embed
      :base-api="apiUrl"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      apiUrl: 'https://your-backend.example.com/api/sk8-embedded'
    };
  }
};
</script>
```

**Vanilla JavaScript:**
```
// Dynamically add component
const sk8Component = document.createElement('pipelines-embed');
sk8Component.setAttribute('base-api', 'https://your-backend.example.com/api/sk8-embedded');
document.getElementById('container').appendChild(sk8Component);
```

---

## Configuration Options

### Component Attributes

| Attribute | Required | Type | Default | Description |
|-----------|----------|------|---------|-------------|
| `base-api` | **Yes** | string | - | Full path to your backend endpoint with SK8 middleware |

---