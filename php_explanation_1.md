# How This App Works: A Guide for PHP Developers

## The "index.php" Equivalent: Where Execution Starts

In PHP, everything starts at `index.php` -- the web server receives a request, hits `index.php`, and that file bootstraps the whole application. In this **Next.js (React)** app, there are **two entry points** that work together -- one for the frontend (what the user sees) and one for the backend (API routes). Think of it as if PHP had a built-in separation between your views and your API controllers, all in one project.

---

## 1. Frontend Entry Point: `app/layout.tsx` + `app/page.tsx`

### `app/layout.tsx` -- The Master Template (like your `header.php` + `footer.php`)

This is the **root wrapper** for every page. In PHP terms, it's your master layout file that wraps `<html>`, `<head>`, and `<body>` around all page content.

```
app/layout.tsx  -->  Wraps ALL pages with <html>, <body>, fonts, CSS
    |
    +--  {children}  -->  The specific page content gets inserted here
```

Think of it like:
```php
// PHP equivalent
include('header.php');   // <html><head>...</head><body>
echo $page_content;      // The actual page -- this is {children}
include('footer.php');   // </body></html>
```

### `app/page.tsx` -- The Homepage (like `index.php` itself)

This is the **actual homepage** component. When a user visits `/` (the root URL), Next.js renders this file inside the layout. This is your closest equivalent to `index.php`.

It renders two main UI panels:
- **`<Assistant />`** (70% width) -- The chat interface
- **`<ToolsPanel />`** (30% width) -- A sidebar for configuring tools (web search, file search, functions, MCP, etc.)

---

## 2. Backend Entry Point: `app/api/turn_response/route.ts`

### The API Route (like a PHP controller/endpoint)

In PHP, you might have something like `api/chat.php` that receives `$_POST` data. Here, `app/api/turn_response/route.ts` is the equivalent. The file-based routing works like this:

| File Path                          | URL it handles         | PHP Equivalent              |
|------------------------------------|------------------------|-----------------------------|
| `app/api/turn_response/route.ts`   | `POST /api/turn_response` | `api/turn_response.php`  |
| `app/api/functions/get_weather/`   | `GET /api/functions/get_weather` | `api/functions/get_weather.php` |

The `route.ts` file exports a `POST()` function -- this is like writing:

```php
// PHP equivalent
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $data = json_decode(file_get_contents('php://input'), true);
    $messages = $data['messages'];
    // ... call OpenAI API, stream response back
}
```

What `route.ts` does:
1. Receives the conversation messages + tool settings from the frontend
2. Creates an OpenAI client and calls the Responses API with streaming enabled
3. Streams the response back to the browser as **Server-Sent Events (SSE)** -- like PHP's `echo` with `flush()` in a loop

---

## 3. The Full Request Lifecycle

Here's how a user message flows through the entire app, compared to PHP:

```
USER TYPES A MESSAGE AND HITS ENTER
         |
         v
[components/chat.tsx]          <-- Like your HTML form / JS AJAX handler
  Captures input, calls onSendMessage()
         |
         v
[components/assistant.tsx]     <-- Like a PHP "controller" action
  Packages user message, calls processMessages()
         |
         v
[lib/assistant.ts]             <-- Like your PHP "service" or "model" layer
  handleTurn() sends POST fetch("/api/turn_response")
         |
         v
[app/api/turn_response/route.ts]  <-- Like api/chat.php (your backend endpoint)
  - Reads the POST body
  - Builds tool configuration
  - Calls OpenAI Responses API with streaming
  - Streams SSE events back
         |
         v
[lib/assistant.ts]             <-- Back on the frontend, reads the stream
  processMessages() reads each SSE event and updates the UI:
  - "response.output_text.delta"  -->  Append text to chat (like typing effect)
  - "response.output_item.added"  -->  New tool call or message appeared
  - "response.output_item.done"   -->  Tool call finished, maybe run a local function
  - "response.completed"          -->  All done
         |
         v
[stores/useConversationStore.ts]  <-- Like PHP $_SESSION or a database
  Updates the chat state (messages array) so React re-renders the UI
         |
         v
[components/chat.tsx]          <-- Re-renders with new messages
  User sees the assistant's response appear
```

---

## 4. State Management (like `$_SESSION` in PHP)

PHP uses `$_SESSION` to keep track of user data across requests. This app uses **Zustand stores** -- think of them as in-memory `$_SESSION` variables that live in the browser:

| Store File | Purpose | PHP Equivalent |
|---|---|---|
| `stores/useConversationStore.ts` | Holds chat messages & conversation history | `$_SESSION['messages']` |
| `stores/useToolsStore.ts` | Holds tool toggles (web search on/off, etc.) | `$_SESSION['settings']` |

---

## 5. Tool/Function Calls (like PHP helper functions)

The app can call "tools" -- these are functions the AI can invoke. They're defined in two places:

- **`config/tools-list.ts`** -- Declares the tool schemas (name, description, parameters). This tells OpenAI *what tools exist*. Like registering routes in a PHP framework.
- **`config/functions.ts`** -- The actual function implementations. When the AI says "call `get_weather`", this file maps that name to the real function that hits `/api/functions/get_weather`. Like the PHP function body.
- **`lib/tools/tools-handling.ts`** -- The dispatcher. Looks up the function name in `functionsMap` and executes it. Like a PHP `switch` statement or `call_user_func()`.

---

## 6. Key File Map

| File/Folder | Role | PHP Analogy |
|---|---|---|
| `app/layout.tsx` | Root HTML wrapper | `header.php` + `footer.php` |
| `app/page.tsx` | Homepage / main view | `index.php` |
| `app/api/turn_response/route.ts` | Chat API endpoint | `api/chat.php` |
| `app/api/functions/*` | Utility API endpoints | `api/helpers/*.php` |
| `components/assistant.tsx` | Chat controller logic | A PHP controller action |
| `components/chat.tsx` | Chat UI rendering | A Blade/Twig template |
| `lib/assistant.ts` | Core message processing & streaming | Service/model layer |
| `stores/useConversationStore.ts` | Chat state | `$_SESSION['messages']` |
| `stores/useToolsStore.ts` | Tool settings state | `$_SESSION['config']` |
| `config/constants.ts` | App constants (model name, prompts) | `config.php` |
| `config/functions.ts` | Tool function implementations | Helper functions |
| `config/tools-list.ts` | Tool schema definitions | Route/function declarations |

---

## Summary

- **`app/layout.tsx`** is the outermost shell (like your master PHP template).
- **`app/page.tsx`** is the homepage -- the closest thing to `index.php`. It's where the user lands and where the UI tree begins.
- **`app/api/turn_response/route.ts`** is the backend API endpoint -- like a PHP file that processes `$_POST` data, calls the OpenAI API, and streams the response back.
- The **frontend** (`components/`, `lib/assistant.ts`, `stores/`) handles rendering and state, similar to how a PHP MVC framework separates views, controllers, and models -- except it all runs in the browser instead of on the server.
