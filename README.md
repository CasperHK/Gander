# Gander

**Gander** is a lightweight, high-performance web framework that pairs the memory-safe, heapless systems programming language **Goose** with a reactive **Vue 3** frontend engine. Designed for ultra-low latency, zero-allocation routing, and predictable resource usage, Gander gives you the speed of C-level execution alongside the developer experience of modern frontend tools.

---

## Key Features

* **Zero-GC Backend Engine:** Powered by Goose's compile-to-C pipeline and contiguous inline data layouts for deterministic, zero-garbage-collection server execution.
* **Reactive Vue 3 UI:** Seamless frontend integration with Vue 3, Vite, and Pinia out of the box.
* **Isolated Worker Concurrency:** Threading model utilizing safe `memcpy` passing over typed worker queues without locking overhead.
* **Embedded Static Assets:** Production builds compile static frontend artifacts directly into the Goose binary using native static array buffers.
* **Unified CLI Workflow:** Integrated tooling for development, hot reloading, and single-binary compilation.

---

## Architecture Overview

```
                          GANDER WEB SYSTEM
 ┌─────────────────────────────────────────────────────────────────┐
 │                       Vue 3 Frontend SPA                        │
 │           (Vite / Vue Router / Pinia State / Tailwind)          │
 └────────────────────────────────┬────────────────────────────────┘
                                  │  REST / WebSockets
 ┌────────────────────────────────▼────────────────────────────────┐
 │                      Goose Core Engine                          │
 │            (Compiled to C via Goose Compiler)                  │
 ├─────────────────────────────────────────────────────────────────┤
 │  • Zero-Allocation Router Frame                                 │
 │  • High-Throughput HTTP / Socket Server (libuv / mongoose)     │
 │  • Multi-Thread Queue Workers (Isolated memcpy messaging)       │
 └─────────────────────────────────────────────────────────────────┘

```

---

## Quick Start

### Prerequisites

* **Goose Compiler** (`goose` installed and available in `$PATH`)
* **C Compiler** (`gcc` or `clang` / `cmake`)
* **Node.js** (v18+) & `pnpm` / `npm`

### Installation

```bash
# Clone repository
git clone https://github.com/your-username/gander.git
cd gander

# Install frontend dependencies
pnpm install

```

### Development Server

Run the unified Gander development server with hot module replacement (HMR) for Vue and fast recompilation for Goose:

```bash
gander dev

```

The application will start at `http://localhost:8080`.

---

## Project Structure

```text
gander-app/
├── src/
│   ├── main.goose            # Goose HTTP backend entry point
│   ├── routes/               # API route definitions & handlers
│   └── workers/              # Isolated Goose thread queue workers
├── web/
│   ├── src/
│   │   ├── App.vue           # Main Vue 3 component
│   │   ├── main.ts           # Frontend entry point
│   │   └── components/       # Vue UI components
│   └── vite.config.ts        # Vite build configuration
├── gander.config.toml        # Framework & compilation settings
└── README.md

```

---

## Code Example

### Backend Handler (`src/main.goose`)

```goose
// Goose Core Endpoint Definition
extern fn gander_http_listen(port: i32, handler: fn(req: Request) -> Response);

struct Request {
    path: String,
    method: String,
    body: String,
}

struct Response {
    status: i32,
    content_type: String,
    body: String,
}

fn handle_request(req: Request) -> Response {
    if req.path == "/api/status" {
        return Response {
            status: 200,
            content_type: "application/json",
            body: "{\"status\": \"ok\", \"engine\": \"Gander/Goose Core\"}",
        };
    }

    return Response {
        status: 404,
        content_type: "text/plain",
        body: "404 Not Found",
    };
}

fn main() {
    println("Gander server listening on port 8080...");
    gander_http_listen(8080, handle_request);
}

```

### Vue 3 Frontend Component (`web/src/App.vue`)

```vue
<template>
  <main class="container">
    <header>
      <h1>🪿 Gander Framework</h1>
      <p>Lightweight. Precise. Fast.</p>
    </header>

    <section class="card">
      <button @click="checkStatus" :disabled="loading">
        {{ loading ? 'Checking...' : 'Check Server Status' }}
      </button>
      <p v-if="response" class="status-output">
        Response: <code>{{ response }}</code>
      </p>
    </section>
  </main>
</template>

<script setup id="gander-app">
import { ref } from 'vue';

const response = ref(null);
const loading = ref(false);

async function checkStatus() {
  loading.value = true;
  try {
    const res = await fetch('/api/status');
    const data = await res.json();
    response.value = JSON.stringify(data);
  } catch (err) {
    response.value = 'Failed to connect to Gander server.';
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.container {
  max-width: 640px;
  margin: 3rem auto;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
.card {
  padding: 1.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  background-color: #f8fafc;
}
button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #0284c7;
  color: #ffffff;
  font-weight: 600;
  cursor: pointer;
}
button:hover {
  background-color: #0369a1;
}
.status-output {
  margin-top: 1rem;
  word-break: break-all;
}
</style>

```

---

## Production Build

To bundle the Vue 3 frontend and compile the Goose code into a single, optimized native binary:

```bash
gander build --release

```

This will output a optimized native executable inside `./dist/gander-app`.

---

## License

MIT License. See [LICENSE](https://www.google.com/search?q=LICENSE) for details.
