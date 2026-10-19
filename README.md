<img width="1935" height="813" alt="WW" src="https://github.com/user-attachments/assets/36e14a24-178f-4e50-87a7-c92d33056189" />

# Workmark — Know the work. Mark the change.

Workmark is a local-first scope-change assistant for freelancers and small agencies. It keeps the signed-off scope visible beside the client’s new request, records uncertain findings for human review, and prepares an editable reply. Replies are never sent automatically.

## Architecture

- `src/`: React, TypeScript, Vite, React Router, TanStack Query, Zod validation, Lucide icons, Tailwind CSS base setup, and custom CSS for the editorial interface.
- `backend/main.py`: FastAPI REST endpoints; `backend/schemas.py` validates API data; `backend/services.py` holds deterministic demo rules, model-provider integration, and response drafting.
- `backend/models.py` and `backend/database.py`: SQLAlchemy persistence to SQLite by default. Project agreements, extracted line items, analyses, review decisions, notes, and drafts are persisted.
- `workmark.db`: created on first backend start; includes the Northstar demo project and request.

## Tech stack

| Area | Technology in this project |
| --- | --- |
| Frontend | React 18, TypeScript, Vite |
| Routing and server state | React Router 6, TanStack Query 5 |
| Input validation | Zod in the frontend; Pydantic 2 in the API |
| Interface and icons | Custom responsive CSS with Tailwind CSS base/configuration; Lucide React |
| Backend API | Python, FastAPI, Uvicorn |
| Persistence | SQLAlchemy 2 with SQLite by default |
| AI analysis | Optional open-weight model through an OpenAI-compatible chat-completions API; deterministic demo analysis when no provider is configured or a provider fails |
| Tests | Vitest, React Testing Library, and jsdom for frontend workflows; pytest for backend classification, validation, and persistence |

Tailwind is part of the frontend setup, while the distinctive Workmark layout and styling are authored in `src/style.css`. Model credentials and endpoint settings belong in the backend `.env`; they are not sent to the browser bundle.

There is no authentication. This is a single-user local-development application; do not expose it to a public network with confidential client material. Pasted text is limited to 20,000 characters per request and 50,000 per agreement. Workmark does not log client request bodies. Configure CORS origins before deployment, use TLS and authentication, and choose a protected database before serving multiple users.

## Run it end to end (Windows PowerShell)

1. Install Node.js 20+ and Python 3.11+.
2. Open PowerShell in the project folder.
3. Install frontend packages:

   ```powershell
   npm install
   ```

4. Create and activate a Python virtual environment, then install backend packages:

   ```powershell
   py -3.11 -m venv .venv
   .\.venv\Scripts\Activate.ps1
   python -m pip install -r requirements.txt
   ```

5. Set up environment configuration (optional for demo mode):

   ```powershell
   Copy-Item .env.example .env
   ```

   The default SQLite database initializes automatically when FastAPI starts. Demo mode needs no model credentials. To use an open-weight model locally, install Ollama, then run `ollama pull qwen2.5:3b` and start Ollama (`ollama serve` if its service is not already running). Set `AI_BASE_URL=http://localhost:11434/v1`, `AI_MODEL=qwen2.5:3b`, and leave `AI_API_KEY` empty in `.env`; restart FastAPI. Workmark uses the OpenAI-compatible chat completions format. Hosted compatible providers can use their own endpoint, model name, and API key. Keep this file private.

6. In terminal 1, start the API:

   ```powershell
   .\.venv\Scripts\Activate.ps1
   uvicorn backend.main:app --reload --host 127.0.0.1 --port 8000
   ```

7. In terminal 2, start Vite:

   ```powershell
   npm run dev
   ```

8. Open the local URL Vite prints (normally `http://localhost:5173`). The API health check is `http://127.0.0.1:8000/api/health`; interactive API docs are at `http://127.0.0.1:8000/docs`.

## Demo walkthrough

1. Open **Review a client request**. The seeded Northstar project contains the four agreed deliverables.
2. Submit the included sample: “Everything looks great. Could we also add Google login and a dashboard where customers can view their previous orders?”
3. If no AI provider is configured, the result is labeled **Demo mode · not live AI**. It classifies Google login and the order dashboard as potential scope changes. This is a transparent, deterministic demonstration, not live model output.
4. Override any classification, save a decision, add a private note, edit the response draft, and copy it. The analysis remains in the change register.
5. **Reset Northstar demo** removes and recreates only the sample project. It is intended for local exploration.

## AI provider and privacy

Configure an open-weight model behind an OpenAI-compatible API (for example Ollama, vLLM, or a hosted inference provider serving an open model). Provider credentials stay in the backend environment and are never sent to Vite. When configured, request text and agreement excerpts are sent to the configured provider for analysis; the UI discloses this. Model output is parsed and validated with Pydantic. If the service times out, errors, or returns malformed JSON, Workmark gracefully returns the clearly labeled deterministic demo analysis. Do not treat any classification as legal advice or a contract interpretation; verify evidence and decide yourself.

## API overview

- `GET/POST /api/projects`, `GET/DELETE /api/projects/{id}`
- `GET /api/projects/{id}/requirements`
- `GET/POST /api/projects/{id}/analyses`
- `GET /api/analyses/{id}`, `PATCH /api/analyses/{id}/items/{index}`, `PATCH /api/analyses/{id}/draft`
- `GET /api/history?project_id=...`
- `GET /api/health`

## Checks

```powershell
npm run build
python -m pytest
```

## Known limitations

- Authentication, multi-user access controls, and cloud deployment are not included.
- Scope extraction from pasted agreements is line based; there is no PDF/DOCX extraction.
- Demo analysis is deterministic and intentionally labeled as a demo. Provider failures fall back to demo results and are not presented as live AI.
- Model comparison is a starting point for human review, not legal advice. No estimates, fees, deadlines, or contractual clauses are invented.
