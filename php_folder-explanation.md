# Every Folder & Subfolder Explained (For PHP Developers)

In Next.js, the folder structure **IS** the routing system. There's no `routes.php` or `.htaccess` -- if you create a folder with a `route.ts` or `page.tsx` inside it, that folder path becomes a URL. Think of it like putting a PHP file at `api/weather/index.php` and it automatically responds at `example.com/api/weather/`.

---

## Root Directory: `/`

These are the project-level config files. In PHP, this is like your `composer.json`, `.htaccess`, and `php.ini` all living at the project root.

| File | What it does | PHP Equivalent |
|---|---|---|
| `package.json` | Lists all dependencies and scripts (`npm run dev`, `npm run build`). | `composer.json` |
| `package-lock.json` | Locks exact dependency versions. | `composer.lock` |
| `tsconfig.json` | TypeScript compiler settings (TypeScript = PHP with strict typing everywhere). | `php.ini` type settings |
| `next.config.mjs` | Next.js framework configuration. | `.htaccess` or framework config |
| `tailwind.config.ts` | CSS framework (Tailwind) settings. | No direct PHP equivalent -- styling config |
| `postcss.config.mjs` | CSS post-processing config. | No direct PHP equivalent |
| `eslint.config.mjs` | Linting rules (code style enforcement). | Like PHP_CodeSniffer config |
| `components.json` | shadcn/ui component library config. | No direct equivalent |
| `.env.example` | Example environment variables file. | `.env.example` (same concept!) |
| `.gitignore` | Files excluded from git. | `.gitignore` (identical concept) |

---

## `/app` -- The Core Application (Your `public_html/` or `www/`)

This is the **App Router** directory. In Next.js, every folder inside `app/` maps to a URL route. This is the equivalent of your entire `public_html/` directory in PHP where you'd have `index.php`, `about.php`, etc.

### `/app/layout.tsx`
**Path:** `app/layout.tsx`
**URL:** Wraps ALL pages (no URL of its own)
**PHP equivalent:** Your master template -- `header.php` + `footer.php` combined.

This file defines the `<html>`, `<head>`, and `<body>` tags that wrap every page. It loads the global CSS (`globals.css`) and fonts. Every page on the site is rendered inside the `{children}` placeholder here -- exactly like how in PHP you'd do:
```php
include('header.php');
echo $page_content;
include('footer.php');
```

### `/app/page.tsx`
**Path:** `app/page.tsx`
**URL:** `/` (the homepage)
**PHP equivalent:** `index.php` -- **this is the starting point of the app.**

When a user visits the root URL, this file renders. It displays two main components side by side:
- `<Assistant />` -- the chat window (70% width)
- `<ToolsPanel />` -- a settings sidebar (30% width)

It also handles an OAuth redirect check: if the URL has `?connected=1`, it resets the conversation to pick up newly connected Google integration settings.

### `/app/globals.css`
**Path:** `app/globals.css`
**URL:** None (imported by `layout.tsx`)
**PHP equivalent:** Your main `style.css` file.

Contains Tailwind CSS directives and CSS custom properties (variables) for theming -- colors, border radius, etc. Defines both light and dark mode color schemes.

### `/app/fonts/`
**Path:** `app/fonts/`
**Contents:** `GeistVF.woff`, `GeistMonoVF.woff`
**PHP equivalent:** A `fonts/` folder in your `assets/` directory.

Contains the custom Geist font files (variable-weight web fonts). These are loaded in `layout.tsx` using Next.js's built-in font optimization -- no `<link>` tags needed.

---

## `/app/api/` -- Backend API Routes (Your `api/` folder with PHP scripts)

This is the **server-side** part of the app. Every subfolder here with a `route.ts` file becomes an API endpoint. This is almost identical to how PHP works: `api/weather/index.php` → `GET /api/weather`.

The key difference from PHP: in Next.js, you export functions named after the HTTP method (`GET`, `POST`, `PUT`, `DELETE`) instead of checking `$_SERVER['REQUEST_METHOD']`.

---

### `/app/api/turn_response/`
**Path:** `app/api/turn_response/route.ts`
**URL:** `POST /api/turn_response`
**PHP equivalent:** `api/chat.php` -- your main chat processing endpoint.

**This is the most important backend file.** It:
1. Receives the conversation history + tool settings as JSON POST body
2. Initializes the OpenAI client (`new OpenAI()` -- reads `OPENAI_API_KEY` from env)
3. Calls OpenAI's Responses API with streaming enabled
4. Pipes each streaming event back to the browser as Server-Sent Events (SSE)

In PHP terms, this is like:
```php
$data = json_decode(file_get_contents('php://input'));
$response = $openai->chat($data->messages);
while ($chunk = $response->read()) {
    echo "data: " . json_encode($chunk) . "\n\n";
    ob_flush(); flush();
}
```

---

### `/app/api/functions/` -- Custom Tool Endpoints

These are simple API endpoints that the AI can call as "tools." Think of them as utility PHP scripts.

#### `/app/api/functions/get_weather/`
**Path:** `app/api/functions/get_weather/route.ts`
**URL:** `GET /api/functions/get_weather?location=NYC&unit=celsius`
**PHP equivalent:** `api/get_weather.php`

Fetches real weather data by:
1. Geocoding the location name → latitude/longitude (via OpenStreetMap Nominatim API)
2. Getting the temperature from Open-Meteo's free weather API
3. Returning the current temperature as JSON

#### `/app/api/functions/get_joke/`
**Path:** `app/api/functions/get_joke/route.ts`
**URL:** `GET /api/functions/get_joke`
**PHP equivalent:** `api/get_joke.php`

Fetches a random programming joke from JokeAPI and returns it as JSON. Simple proxy endpoint.

---

### `/app/api/vector_stores/` -- File Search / Knowledge Base Management

These endpoints manage OpenAI's **Vector Stores** -- think of them as a searchable document database. The AI can search through uploaded files to answer questions. In PHP terms, this is like having CRUD endpoints for managing a document library.

#### `/app/api/vector_stores/create_store/`
**Path:** `app/api/vector_stores/create_store/route.ts`
**URL:** `POST /api/vector_stores/create_store`
**PHP equivalent:** `api/create_store.php`

Creates a new vector store (searchable file collection) via the OpenAI API. Receives a `name` in the POST body.

#### `/app/api/vector_stores/upload_file/`
**Path:** `app/api/vector_stores/upload_file/route.ts`
**URL:** `POST /api/vector_stores/upload_file`
**PHP equivalent:** `api/upload.php` (like handling `$_FILES`)

Receives a base64-encoded file in the POST body, converts it to a buffer, and uploads it to OpenAI's file storage. This is the equivalent of a PHP file upload handler, except the file comes as base64 JSON instead of `multipart/form-data`.

#### `/app/api/vector_stores/add_file/`
**Path:** `app/api/vector_stores/add_file/route.ts`
**URL:** `POST /api/vector_stores/add_file`
**PHP equivalent:** Part of `api/upload.php`

Links an already-uploaded file to a specific vector store. This is a two-step process: first upload the file (above), then attach it to a store (this endpoint). Like inserting a row into a `files_stores` junction table.

#### `/app/api/vector_stores/list_files/`
**Path:** `app/api/vector_stores/list_files/route.ts`
**URL:** `GET /api/vector_stores/list_files?vector_store_id=vs_xxx`
**PHP equivalent:** `api/list_files.php?store_id=123`

Lists all files in a given vector store. Simple GET request that proxies to the OpenAI API.

#### `/app/api/vector_stores/retrieve_store/`
**Path:** `app/api/vector_stores/retrieve_store/route.ts`
**URL:** `GET /api/vector_stores/retrieve_store?vector_store_id=vs_xxx`
**PHP equivalent:** `api/get_store.php?id=123`

Retrieves metadata about a specific vector store (name, file count, etc.).

---

### `/app/api/container_files/` -- Code Interpreter File Downloads

#### `/app/api/container_files/content/`
**Path:** `app/api/container_files/content/route.ts`
**URL:** `GET /api/container_files/content?file_id=xxx&container_id=yyy&filename=zzz`
**PHP equivalent:** `api/download.php?file_id=xxx`

When the AI uses the Code Interpreter tool and generates output files (like charts, CSVs, etc.), this endpoint lets the user download those files. It fetches the file from OpenAI's container storage and streams it back as a download. Like a PHP `readfile()` proxy with proper `Content-Disposition` headers.

---

### `/app/api/google/` -- Google OAuth Integration

These three endpoints handle the OAuth 2.0 flow for connecting the user's Google account (Gmail + Calendar). In PHP, you'd typically use a library like `league/oauth2-client` for this.

#### `/app/api/google/auth/`
**Path:** `app/api/google/auth/route.ts`
**URL:** `GET /api/google/auth`
**PHP equivalent:** `api/google/login.php`

**Step 1 of OAuth:** Generates a PKCE code challenge, stores state/verifier in secure cookies, and redirects the user to Google's consent screen. This is the "Login with Google" redirect.

#### `/app/api/google/callback/`
**Path:** `app/api/google/callback/route.ts`
**URL:** `GET /api/google/callback`
**PHP equivalent:** `api/google/callback.php`

**Step 2 of OAuth:** Google redirects back here after the user approves. This endpoint:
1. Validates the state parameter (CSRF protection)
2. Exchanges the authorization code for access + refresh tokens
3. Saves tokens in both memory and secure httpOnly cookies
4. Redirects to `/?connected=1` so the frontend knows the connection succeeded

#### `/app/api/google/status/`
**Path:** `app/api/google/status/route.ts`
**URL:** `GET /api/google/status`
**PHP equivalent:** `api/google/status.php`

Simple check: "Is Google OAuth configured (env vars present)? Is the user currently connected (has tokens)?" Returns JSON booleans. The Tools Panel calls this on load to show/hide the Google integration toggle.

---

## `/components/` -- UI Components (Your `views/` or `templates/` folder)

In PHP MVC, this is your `views/` directory. Each file here is a reusable UI component (like a Blade partial or a Twig template). They only handle rendering -- no database queries, no business logic.

### `/components/assistant.tsx`
**PHP equivalent:** A controller action for the chat page.

The "brain" that connects the chat UI to the backend. It:
- Captures user messages from the `<Chat />` component
- Packages them into the correct format
- Calls `processMessages()` to send them to the API
- Also handles MCP tool approval responses

### `/components/chat.tsx`
**PHP equivalent:** The main chat template (`chat.blade.php`).

Renders the full chat interface: message list + text input + send button. Maps over the `items` array and renders the appropriate component for each item type (message, tool call, MCP tools list, MCP approval request). Also handles keyboard events (Enter to send).

### `/components/message.tsx`
**PHP equivalent:** A message partial template (`_message.blade.php`).

Renders a single chat message (user or assistant). Handles markdown rendering for assistant messages.

### `/components/tool-call.tsx`
**PHP equivalent:** A tool-call partial template.

Renders a tool call result in the chat -- shows the tool name, arguments, status (in progress / completed), and output. Handles different tool types: web search, file search, function calls, MCP calls, and code interpreter.

### `/components/annotations.tsx`
**PHP equivalent:** A citations/references partial.

Renders source annotations (citations) that appear below assistant messages -- like footnotes showing which files or web sources were used.

### `/components/tools-panel.tsx`
**PHP equivalent:** A settings sidebar template.

The right-side panel that lets users toggle tools on/off:
- File Search (with vector store setup)
- Web Search (with location config)
- Code Interpreter
- Functions (local tool calls)
- MCP (remote tool server)
- Google Integration (Gmail + Calendar)

### `/components/file-search-setup.tsx`
**PHP equivalent:** A file upload form partial.

UI for creating vector stores and uploading files for the File Search tool. Includes drag-and-drop file upload.

### `/components/file-upload.tsx`
**PHP equivalent:** A drag-and-drop file upload widget.

Reusable file upload component using `react-dropzone`. Reads files as base64 and passes them up.

### `/components/websearch-config.tsx`
**PHP equivalent:** A web search settings form.

Lets the user configure their approximate location (country/city/region) for web search results.

### `/components/functions-view.tsx`
**PHP equivalent:** A functions list display.

Shows the list of available custom functions (from `config/tools-list.ts`) with their names, descriptions, and parameters.

### `/components/mcp-config.tsx`
**PHP equivalent:** An MCP server configuration form.

Form for configuring a remote MCP (Model Context Protocol) server: server label, URL, allowed tools, and whether to skip approval for tool calls.

### `/components/mcp-tools-list.tsx`
**PHP equivalent:** A tools list display partial.

Renders the list of tools discovered from a connected MCP server.

### `/components/mcp-approval.tsx`
**PHP equivalent:** A confirmation dialog partial.

When an MCP tool call requires approval, this shows an approve/deny dialog with the tool name and arguments.

### `/components/google-integration.tsx`
**PHP equivalent:** A Google account connection panel.

Shows a "Connect Google Account" button that initiates the OAuth flow, or shows "Connected" status if already linked.

### `/components/panel-config.tsx`
**PHP equivalent:** A collapsible section wrapper.

Reusable wrapper component for each tool section in the Tools Panel. Provides a title, tooltip, and enable/disable toggle switch.

### `/components/loading-message.tsx`
**PHP equivalent:** A loading spinner partial.

Shows an animated loading indicator while waiting for the assistant's response.

### `/components/country-selector.tsx`
**PHP equivalent:** A country dropdown partial.

A searchable country dropdown for the web search location config.

### `/components/ui/` -- Base UI Primitives
**PHP equivalent:** A UI component library (like Bootstrap partials).

These are low-level, reusable UI building blocks from **shadcn/ui** (a popular React component library). They're pre-styled, accessible components:

| File | What it renders |
|---|---|
| `ui/button.tsx` | Styled button with variants (primary, secondary, destructive, etc.) |
| `ui/input.tsx` | Styled text input field |
| `ui/textarea.tsx` | Styled multi-line text area |
| `ui/dialog.tsx` | Modal dialog / popup window |
| `ui/popover.tsx` | Floating popover (like a tooltip with content) |
| `ui/tooltip.tsx` | Hover tooltip |
| `ui/switch.tsx` | Toggle switch (on/off) |
| `ui/command.tsx` | Command palette / searchable list (used in country selector) |

---

## `/lib/` -- Business Logic & Utilities (Your `includes/` or `src/` folder)

In PHP, this is where you'd put your service classes, helpers, and utility functions -- the `includes/` or `src/` directory.

### `/lib/assistant.ts`
**PHP equivalent:** Your main service class (`ChatService.php`).

The largest and most important library file. Contains:
- **`handleTurn()`** -- Sends a POST request to `/api/turn_response` and reads the SSE stream. Like a PHP cURL call that reads a streaming response.
- **`processMessages()`** -- The main message processing loop. Reads each streamed event and updates the UI state based on event type:
  - `response.output_text.delta` → Append text to the current assistant message (typing effect)
  - `response.output_item.added` → A new item appeared (message, function call, web search, etc.)
  - `response.output_item.done` → An item finished; if it was a function call, execute the function locally and send the result back for another turn
  - `response.function_call_arguments.delta/done` → Stream function call arguments as they arrive
  - `response.completed` → Handle MCP tool lists and approval requests

Also defines TypeScript interfaces (`MessageItem`, `ToolCallItem`, `McpListToolsItem`, etc.) -- these are like PHP class definitions or data transfer objects (DTOs).

### `/lib/connectors-auth.ts`
**PHP equivalent:** `GoogleOAuthService.php`

Handles Google OAuth client setup:
- **`getGoogleClient()`** -- Discovers Google's OAuth endpoints and configures the client (cached). Like `new Google_Client()` in PHP.
- **`getRedirectUri()`** -- Returns the OAuth callback URL.
- **`getFreshAccessToken()`** -- Checks if the access token is expired/expiring and refreshes it using the refresh token. Like `$client->fetchAccessTokenWithRefreshToken()`.

### `/lib/session.ts`
**PHP equivalent:** `session.php` or `SessionManager.php`

Manages user sessions using cookies + an in-memory Map:
- **`getOrCreateSessionId()`** -- Creates a session ID cookie if one doesn't exist. Exactly like `session_start()` in PHP.
- **`saveTokenSet()` / `getTokenSet()`** -- Store/retrieve OAuth tokens for a session. Like `$_SESSION['tokens'] = $tokens`.
- **`clearSession()`** -- Destroys a session. Like `session_destroy()`.

Note: Uses an in-memory `Map` for demo purposes -- in production you'd use Redis, a database, etc. (just like PHP sessions can be stored in files, memcached, or DB).

### `/lib/utils.ts`
**PHP equivalent:** A small `helpers.php` file.

Contains one utility function: `cn()` -- merges Tailwind CSS class names intelligently. No PHP equivalent; it's a CSS utility.

### `/lib/tools/` -- Tool Configuration & Execution

#### `/lib/tools/tools.ts`
**PHP equivalent:** `ToolsFactory.php` or a tools configuration builder.

**`getTools()`** -- Builds the tools array to send to the OpenAI API based on which tools the user has enabled. Reads the toggle states from the frontend and assembles the correct configuration objects for:
- Web Search (with optional location)
- File Search (with vector store ID)
- Code Interpreter
- Custom Functions (from `config/tools-list.ts`)
- MCP (remote tool server)
- Google Connectors (Calendar + Gmail, with fresh access token)

#### `/lib/tools/tools-handling.ts`
**PHP equivalent:** A function dispatcher (`call_user_func()`).

**`handleTool()`** -- When the AI says "call function `get_weather`", this looks up `get_weather` in the `functionsMap` and executes it. It's a simple dispatcher:
```php
// PHP equivalent
$result = call_user_func($functionsMap[$toolName], $parameters);
```

#### `/lib/tools/connectors.ts`
**PHP equivalent:** `GoogleConnectors.php`

**`getGoogleConnectorTools()`** -- Returns the MCP tool definitions for Google Calendar and Gmail connectors, injecting the user's OAuth access token. These tell OpenAI how to connect to Google's services on behalf of the user.

---

## `/stores/` -- State Management (Your `$_SESSION` equivalent)

These are **Zustand stores** -- client-side state containers that live in the browser's memory. They're the React equivalent of PHP's `$_SESSION` superglobal.

### `/stores/useConversationStore.ts`
**PHP equivalent:** `$_SESSION['chat']`

Holds:
- **`chatMessages`** -- Array of messages displayed in the chat UI (initialized with a welcome message)
- **`conversationItems`** -- Array of items sent to the OpenAI API (the full conversation history)
- **`isAssistantLoading`** -- Boolean flag for showing the loading spinner
- **`resetConversation()`** -- Clears the chat and starts fresh

### `/stores/useToolsStore.ts`
**PHP equivalent:** `$_SESSION['settings']`

Holds all tool toggle states and configurations, persisted to `localStorage` (survives page refreshes):
- Toggle booleans: `webSearchEnabled`, `fileSearchEnabled`, `functionsEnabled`, `codeInterpreterEnabled`, `mcpEnabled`, `googleIntegrationEnabled`
- Configuration objects: `vectorStore`, `webSearchConfig` (location), `mcpConfig` (server URL, label, etc.)

---

## `/config/` -- App Configuration (Your `config.php`)

### `/config/constants.ts`
**PHP equivalent:** `config.php` with `define()` constants.

Defines:
- **`MODEL`** -- Which OpenAI model to use (`"gpt-5.2"`)
- **`DEVELOPER_PROMPT`** -- The system instructions sent to the AI (like a system message)
- **`getDeveloperPrompt()`** -- Returns the prompt with today's date appended
- **`INITIAL_MESSAGE`** -- The first message shown in the chat ("Hi, how can I help you?")
- **`defaultVectorStore`** -- Default vector store config (empty by default)

### `/config/functions.ts`
**PHP equivalent:** `functions.php` -- the actual function implementations.

Defines the client-side functions that correspond to tool calls:
- **`get_weather()`** -- Calls `/api/functions/get_weather` with location and unit
- **`get_joke()`** -- Calls `/api/functions/get_joke`
- **`functionsMap`** -- Maps function names to their implementations (used by `tools-handling.ts`)

### `/config/tools-list.ts`
**PHP equivalent:** Tool schema declarations (like OpenAPI/Swagger definitions).

Defines the **schema** for each custom function tool -- name, description, and parameter types. This tells the OpenAI API what functions are available and what arguments they accept. It does NOT contain the actual function code (that's in `functions.ts`).

---

## `/public/` -- Static Assets (Your `public/` or `assets/` folder)

### `/public/openai_logo.svg`
**Path:** `public/openai_logo.svg`
**URL:** `/openai_logo.svg` (served directly)
**PHP equivalent:** `public/images/logo.svg`

The OpenAI logo used as the browser favicon. Files in `public/` are served as-is at the root URL, exactly like PHP's `public/` or `htdocs/` directory.

---

## Visual Folder Tree Summary

```
openai-responses-starter-app/
├── app/                          # Main application (like public_html/)
│   ├── layout.tsx                # Master HTML template (header+footer)
│   ├── page.tsx                  # Homepage = "index.php"
│   ├── globals.css               # Global stylesheet
│   ├── fonts/                    # Font files
│   │   ├── GeistVF.woff
│   │   └── GeistMonoVF.woff
│   └── api/                      # Backend endpoints (like api/*.php)
│       ├── turn_response/        # POST /api/turn_response -- main chat endpoint
│       │   └── route.ts
│       ├── functions/            # Utility function endpoints
│       │   ├── get_weather/      # GET /api/functions/get_weather
│       │   │   └── route.ts
│       │   └── get_joke/         # GET /api/functions/get_joke
│       │       └── route.ts
│       ├── vector_stores/        # File/knowledge base management
│       │   ├── create_store/     # POST -- create a vector store
│       │   │   └── route.ts
│       │   ├── upload_file/      # POST -- upload a file to OpenAI
│       │   │   └── route.ts
│       │   ├── add_file/         # POST -- attach file to a store
│       │   │   └── route.ts
│       │   ├── list_files/       # GET -- list files in a store
│       │   │   └── route.ts
│       │   └── retrieve_store/   # GET -- get store metadata
│       │       └── route.ts
│       ├── container_files/      # Code interpreter file downloads
│       │   └── content/          # GET -- download a generated file
│       │       └── route.ts
│       └── google/               # Google OAuth flow
│           ├── auth/             # GET -- redirect to Google login
│           │   └── route.ts
│           ├── callback/         # GET -- handle OAuth callback
│           │   └── route.ts
│           └── status/           # GET -- check connection status
│               └── route.ts
├── components/                   # UI components (like views/templates/)
│   ├── assistant.tsx             # Chat controller
│   ├── chat.tsx                  # Chat UI template
│   ├── message.tsx               # Single message display
│   ├── tool-call.tsx             # Tool call result display
│   ├── annotations.tsx           # Source citations display
│   ├── tools-panel.tsx           # Settings sidebar
│   ├── panel-config.tsx          # Collapsible section wrapper
│   ├── file-search-setup.tsx     # Vector store setup UI
│   ├── file-upload.tsx           # Drag-and-drop file upload
│   ├── websearch-config.tsx      # Web search location settings
│   ├── functions-view.tsx        # Functions list display
│   ├── mcp-config.tsx            # MCP server config form
│   ├── mcp-tools-list.tsx        # MCP discovered tools list
│   ├── mcp-approval.tsx          # MCP tool approval dialog
│   ├── google-integration.tsx    # Google account connection panel
│   ├── country-selector.tsx      # Searchable country dropdown
│   ├── loading-message.tsx       # Loading spinner
│   └── ui/                       # Base UI primitives (shadcn/ui)
│       ├── button.tsx
│       ├── command.tsx
│       ├── dialog.tsx
│       ├── input.tsx
│       ├── popover.tsx
│       ├── switch.tsx
│       ├── textarea.tsx
│       └── tooltip.tsx
├── lib/                          # Business logic (like includes/src/)
│   ├── assistant.ts              # Core chat processing & streaming
│   ├── connectors-auth.ts        # Google OAuth client & token refresh
│   ├── session.ts                # Session management (like session_start())
│   ├── utils.ts                  # CSS utility helper
│   └── tools/                    # Tool configuration & execution
│       ├── tools.ts              # Builds tools array for OpenAI API
│       ├── tools-handling.ts     # Function dispatcher (call_user_func)
│       └── connectors.ts         # Google connector tool definitions
├── stores/                       # State management (like $_SESSION)
│   ├── useConversationStore.ts   # Chat messages & history
│   └── useToolsStore.ts          # Tool toggles & config (persisted)
├── config/                       # App configuration (like config.php)
│   ├── constants.ts              # Model name, prompts, defaults
│   ├── functions.ts              # Function implementations
│   └── tools-list.ts             # Function schemas/declarations
├── public/                       # Static files served as-is
│   └── openai_logo.svg           # Favicon
├── package.json                  # Dependencies (composer.json)
├── tsconfig.json                 # TypeScript config
├── next.config.mjs               # Next.js config
├── tailwind.config.ts            # Tailwind CSS config
├── postcss.config.mjs            # PostCSS config
├── eslint.config.mjs             # Linting config
└── components.json               # shadcn/ui config
```
