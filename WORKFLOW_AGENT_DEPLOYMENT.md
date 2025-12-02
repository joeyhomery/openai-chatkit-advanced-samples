# Deploying a Workflow Builder agent to another site

This guide explains how to take an agent you created with OpenAI's Workflow Builder and expose it on any site using the ChatKit SDKs. It mirrors the production-ready patterns already used in this repo's examples, so you can copy/paste confidently.

## 1) Gather credentials
- **OpenAI API key** (`OPENAI_API_KEY`).
- **Domain allowlist key** (`domain_pk_...`). Use any placeholder for local work, but generate a real key for your production domain at the [domain allowlist](https://platform.openai.com/settings/organization/security/domain-allowlist) page.
- **Workflow Builder agent identifier** (from the Workflow UI). You'll pass this to the SDK when invoking your agent.

## 2) Stand up a backend endpoint
1. Install backend deps (FastAPI + ChatKit Python SDK) in your API project.
2. Create a FastAPI app and expose a single ChatKit endpoint. The sample customer-support server shows the exact wiring: it builds a `ChatKitServer` subclass, adds CORS, and exposes `POST /support/chatkit` that streams responses when the SDK asks for them.【F:examples/customer-support/backend/app/main.py†L211-L242】
3. Inside your server class, load your Workflow agent and any domain-specific tools. The customer-support example initializes its agent (`support_agent`), thread title helper, and converters on startup.【F:examples/customer-support/backend/app/main.py†L83-L122】
4. Host this API where your website can reach it (e.g., `https://api.example.com/chatkit`).

## 3) Plug ChatKit into your website
1. Add the ChatKit React SDK (`@openai/chatkit-react`) or vanilla SDK to your frontend bundle.
2. Point the SDK at your backend endpoint and domain key. The customer-support panel shows the minimal config: `useChatKit` is given the API `url`, `domainKey`, theme, and callbacks, then rendered via `<ChatKit control={chatkit.control} />`.【F:examples/customer-support/frontend/src/components/ChatKitPanel.tsx†L1-L66】
3. Keep your API URLs and domain key in environment-driven config. The example uses `VITE_SUPPORT_CHATKIT_API_URL` and `VITE_SUPPORT_CHATKIT_API_DOMAIN_KEY`, with fallbacks for local development.【F:examples/customer-support/frontend/src/lib/config.ts†L5-L26】
4. When you deploy your site, set those environment variables so the client points at your hosted backend and uses the production domain key.

## 4) End-to-end checklist
- ✅ Backend responds at `/chatkit` (or similar) and streams SSE for ChatKit.
- ✅ CORS allows your website origin.
- ✅ Frontend loads `@openai/chatkit-react`, passes the backend URL + domain key, and renders `<ChatKit />`.
- ✅ Production domain is allowlisted and uses the `domain_pk_...` key.

Use any of the example projects in `examples/*` as a reference implementation; the customer-support sample pairs a FastAPI backend with a React frontend and is closest to a multi-tenant support experience.
