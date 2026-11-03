# WonderWorld AI Chatbot

A full-stack customer-support chatbot for a fictional theme park, built with React, TypeScript, Express, Bun, and the OpenAI API.

The assistant uses a local park information document to answer questions about tickets, attractions, opening hours, dining, and accommodation. This portfolio project explores how a chat interface, a backend API, and contextual AI instructions work together.

## Features

- Conversational interface with distinct user and assistant messages.
- Markdown rendering, automatic scrolling, and an animated typing indicator.
- Keyboard submission with Enter; Shift + Enter inserts a new line.
- Form validation that rejects empty and whitespace-only messages.
- Server-side validation of prompts and conversation UUIDs with Zod.
- Conversation continuity through OpenAI response IDs.
- A separate prompt template and Markdown knowledge document.
- Loading and error feedback in the chat interface.
- One command to start the frontend and backend together.

## Tech stack

- **Frontend:** React, TypeScript, Vite, Tailwind CSS, shadcn/ui with Base UI, React Hook Form, Axios, and React Markdown.
- **Backend:** Express, TypeScript, Bun, Zod, dotenv, and the OpenAI SDK.
- **AI:** OpenAI Responses API using `gpt-4o-mini`.
- **Tooling:** Bun workspaces, concurrently, Prettier, Husky, and lint-staged.

## Architecture

The project is organized as a monorepo with separate client and server packages.

```text
packages/
├── client/
│   └── src/
│       └── components/
│           ├── chat/
│           │   ├── ChatBot.tsx
│           │   ├── ChatInput.tsx
│           │   ├── ChatMessages.tsx
│           │   └── TypingIndicator.tsx
│           └── ui/
└── server/
    ├── index.ts
    ├── routes.ts
    ├── controllers/chat.controller.ts
    ├── services/chat.service.ts
    ├── repositories/conversation.repository.ts
    └── prompts/
        ├── chatbot.txt
        └── WonderWorld.md
```

The browser sends a request to `POST /api/chat`. During development, Vite proxies `/api` requests to Express on port 3000.

1. The **route** forwards the request to the controller.
2. The **controller** validates the input and handles the HTTP response.
3. The **service** sends the prompt, park information, and previous response ID to OpenAI.
4. The **repository** stores the latest response ID for each conversation in an in-memory Map.
5. The frontend displays the returned message.

## Getting started

### Prerequisites

- Bun installed locally.
- Node.js available for the Vite frontend tooling.
- An OpenAI API key with access to the configured model and available API quota. API requests may incur charges.

### Install dependencies

From the repository root:

```bash
bun install
```

### Configure the backend

Create `packages/server/.env` and add your own key:

```dotenv
OPENAI_API_KEY=your_openai_api_key_here
```

Keep this file local. Do not commit API keys or expose them in frontend code.

The backend defaults to port 3000. If you change its port, also update the proxy target in `packages/client/vite.config.ts`.

### Start development

From the repository root:

```bash
bun run dev
```

- Frontend: http://localhost:5173
- Backend: http://localhost:3000

If port 5173 is occupied, use the frontend URL printed by Vite. Stop the application with **Ctrl + C**. Ctrl + Z suspends processes and can leave ports occupied.

Try asking:

- “How much are tickets?”
- “Which attractions are suitable for a five-year-old?”
- “What time does the park open?”

## API example

`POST /api/chat` accepts a non-empty prompt of up to 1,000 characters and a conversation UUID.

```bash
curl http://localhost:3000/api/chat \
  -H 'Content-Type: application/json' \
  -d '{"prompt":"How much are tickets?","conversationId":"550e8400-e29b-41d4-a716-446655440000"}'
```

A successful response has this shape:

```json
{
  "message": "The assistant response appears here."
}
```

Reuse the conversation ID for follow-up questions. Invalid input returns HTTP 400; service failures return HTTP 500.

## Customizing the assistant

Edit `packages/server/prompts/chatbot.txt` to change the assistant instructions. Edit `packages/server/prompts/WonderWorld.md` to change the supplied park information.

The service replaces `{{parkInfo}}` in the template with the Markdown document. This supplies context at request time; it does not train the model. Restart the backend after editing these files to ensure the updated content is loaded.

## Development commands

From the repository root:

```bash
bun run format
```

From `packages/client`:

```bash
bun run lint
bun run build
```

The build command checks the client TypeScript project and builds the frontend. It does not deploy the backend. Husky and lint-staged run Prettier on configured staged file types before commits.

## Current limitations

- Conversation mappings are held in memory and are lost when the backend restarts. Refreshing the frontend starts a new conversation.
- The park document is static sample data. The application does not retrieve live wait times, weather, availability, or bookings.
- Prompt instructions encourage relevant, factual answers but do not guarantee accuracy.
- There is no persistent database, user authentication, or rate limiting.
- Responses are returned as complete messages; the typing indicator is a loading animation, not token streaming.
- No automated test suite or production deployment workflow is currently included.

## Learning and attribution

Built while following Code with Mosh’s **Build AI-Powered Apps** course. The WonderWorld scenario and starting implementation come from the course. This repository documents my hands-on implementation, debugging, and practice with frontend components, REST APIs, layered backend organization, validation, and prompt engineering.

The project is a learning foundation for future work on Scan&Serve, rather than the Scan&Serve application itself.
