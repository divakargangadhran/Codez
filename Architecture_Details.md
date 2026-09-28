# System Architecture Details
**Project:** Codez - AI Agent

This document provides a detailed breakdown of the system architecture for the Codez project, based on the application's dependencies and configuration.

---

## 1. High-Level Architecture Overview

Codez is built as a modern, Edge-native, AI-powered development environment. The architecture is primarily divided into four main layers:

1.  **Client-Side (Frontend Editor & UI)**
2.  **Server-Side / Edge (Backend & API Routes)**
3.  **Execution Environment (In-Browser & Native)**
4.  **AI Integration Layer (LLM Routing)**

Below is a Mermaid diagram illustrating the flow of data and interaction between these components:

```mermaid
graph TD
    %% Define styles
    classDef frontend fill:#e3f2fd,stroke:#1565c0,stroke-width:2px;
    classDef execution fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef backend fill:#fff3e0,stroke:#ef6c00,stroke-width:2px;
    classDef ai fill:#f3e5f5,stroke:#6a1b9a,stroke-width:2px;
    classDef infra fill:#eceff1,stroke:#455a64,stroke-width:2px;

    %% Components
    subgraph Client [Client-Side UI]
        UI[React UI / Radix / Tailwind]:::frontend
        Editor[CodeMirror 6 Editor]:::frontend
        Terminal[Xterm.js Terminal]:::frontend
        State[Zustand / Nanostores / IndexedDB]:::frontend
    end

    subgraph Execution [Local Execution Engine]
        WebContainer[WebContainer API <br/>(In-browser Node.js)]:::execution
    end

    subgraph Backend [Edge Backend Services]
        Remix[Remix Server]:::backend
        MCP[Model Context Protocol SDK]:::backend
    end

    subgraph AILayer [AI Integration Layer]
        AISDK[Vercel AI SDK]:::ai
        LLMs[(Multiple LLM Providers <br/> OpenAI, Anthropic, etc.)]:::ai
    end

    subgraph Infrastructure [Deployment & Hosting]
        Cloudflare[Cloudflare Pages / Workers]:::infra
        Docker[Docker Containers]:::infra
        Electron[Electron Desktop App]:::infra
    end

    %% Connections
    UI <--> State
    Editor <--> State
    Terminal <--> WebContainer
    Editor --> WebContainer

    State <--> Remix
    WebContainer <--> Remix

    Remix <--> MCP
    Remix <--> AISDK
    AISDK <--> LLMs

    Remix -.-> Cloudflare
    Remix -.-> Docker
    Remix -.-> Electron
```

---

## 2. Detailed Component Breakdown

### 2.1. Client-Side (Frontend Editor & UI)
The frontend is responsible for providing a responsive, real-time IDE experience. It is built using:
*   **React 18 & Remix:** The core UI framework and server-side rendering engine.
*   **CodeMirror 6:** The code editor component, heavily customized with syntax highlighting (`@codemirror/lang-*`) and autocomplete suitable for various languages (JS, TS, Python, C++, etc.).
*   **Xterm.js:** Provides a fully functional, integrated terminal directly within the browser UI.
*   **Styling & UI Components:** 
    *   **TailwindCSS & UnoCSS:** For utility-first styling.
    *   **Radix UI:** For accessible, unstyled UI primitives (Dialogs, Menus, Tooltips).
    *   **Framer Motion:** For smooth micro-animations and transitions.
*   **State & Storage:** Utilizing **Zustand** (for global application state), **Nanostores** (for tiny, isolated stores), and **IndexedDB** (for local data persistence).

### 2.2. Execution Environment (In-Browser Node.js)
A critical differentiator of Codez is its ability to run code locally without a traditional backend server, or packaged natively.
*   **WebContainer API:** This technology enables running a full Node.js environment *inside the browser*. It allows the application to execute terminal commands, run development servers, and handle file system operations securely within the client.
*   **Electron:** For the desktop version, Electron wraps the Remix application, providing native OS capabilities and improved filesystem access.

### 2.3. Server-Side / Edge (Backend logic)
The backend primarily acts as a secure proxy to AI services and handles server-side rendering.
*   **Remix Server:** Handles routing, data loading, and server mutations.
*   **Cloudflare Workers / Pages (`@remix-run/cloudflare`):** The application is designed to be deployed to the edge via Cloudflare, ensuring extremely low latency and high availability globally.
*   **MCP (Model Context Protocol):** Integrates `@modelcontextprotocol/sdk` to allow AI models to deeply understand the context of the user's local workspace.

### 2.4. AI Integration Layer
Codez acts as an aggregator for multiple Large Language Models (LLMs), allowing developers to choose the best model for their task.
*   **AI SDK (`ai` package):** Utilizes the Vercel AI SDK to unify the interfaces for different providers.
*   **Supported Providers:**
    *   `@ai-sdk/openai` (GPT-4o, etc.)
    *   `@ai-sdk/anthropic` (Claude 3.5 Sonnet, etc.)
    *   `@ai-sdk/google` (Gemini Pro, etc.)
    *   `@ai-sdk/mistral`, `@ai-sdk/deepseek`, `@ai-sdk/cohere`, `@ai-sdk/amazon-bedrock`, `@ai-sdk/cerebras`
    *   Additionally supports OpenRouter and Ollama for open-source/local models.

---

## 3. Data Flow Example: AI Code Generation

1.  **User Action:** The user types a prompt in the Codez chat interface requesting a new feature.
2.  **Context Gathering:** The frontend gathers the current file context (via CodeMirror state) and file system structure (via WebContainer/Local FS) using the Model Context Protocol (MCP).
3.  **Request Handling:** The prompt and context are sent to the Remix Edge backend.
4.  **AI Routing:** The Vercel AI SDK routes the request to the currently selected LLM provider (e.g., Anthropic Claude).
5.  **Streaming Response:** The LLM generates the code, which is streamed back to the frontend in real-time.
6.  **Application:** The UI updates, and the user can click to apply the code changes, which directly modifies the files within the WebContainer or local filesystem.

---

## 4. Deployment Architecture

The application is highly versatile in its deployment capabilities, controlled via scripts in `package.json`:

*   **Cloud (Edge):** `wrangler pages deploy` pushes the compiled Remix application to Cloudflare Pages.
*   **Desktop:** Electron builder (`electron-builder`) compiles the app into standalone executables for Mac (`--mac`), Windows (`--win`), and Linux (`--linux`).
*   **Containerized (Docker):** A multi-stage Dockerfile allows the application to be deployed as a standard container (`codez-ai-live`), ensuring consistency across different server environments.
