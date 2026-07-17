# LiveKit Agent Worker — Runtime Test Checklist

Follow these steps to validate the LiveKit Agent Worker MVP locally.

Prereqs:

- Node.js backend running (orchestrator)
- MongoDB running and reachable by `MONGODB_URI`/worker `.env`
- Node.js 18+ and `pnpm` (for Node LiveKit Agents worker)
- LiveKit server running (`docker-compose up -d livekit` for local dev)
- `OPENAI_API_KEY` and optionally `DEEPGRAM_API_KEY`, `LIVEKIT_*` in worker env

1. Start backend (orchestrator)

```bash
# from repo root
pnpm --filter @elyntix/orchestrator dev
```

2. Start web app (dashboard)

```bash
pnpm --filter @elyntix/web dev
```

3. Start worker (Node LiveKit Agents)

```bash
cd apps/agent-worker-node
pnpm install
pnpm build
export OPENAI_API_KEY=...
export DEEPGRAM_API_KEY=...   # optional for streaming STT
export LIVEKIT_URL=...
export LIVEKIT_API_KEY=...
export LIVEKIT_API_SECRET=...
pnpm dev
```

4. Create or use an agent in the dashboard and click "Test Call" (this creates a browser test call session).

5. Verify the orchestrator created a `CallSession` document (collection `callsessions`) with `status: pending` and `livekitRoomName`.

6. No manual step needed — the orchestrator automatically dispatches the worker into the room via LiveKit's explicit `AgentDispatchClient` (agent name `elyntix-voice-agent`), passing `{ callSessionId, agentId, workspaceId }` as job metadata.

7. Worker behavior to observe:

- Worker uses LiveKit Agents to join the room and stream audio.
- Worker uses Deepgram (if provided) to transcribe incoming audio to text.
- Worker persists `TranscriptEvent` documents for incoming user speech and agent responses.
- Worker calls OpenAI LLM for responses using `agent.instructions` as system prompt.
- Worker uses the SDK-backed `generateReply()` / `say()` flow with OpenAI TTS so the browser hears audio.

8. During/after the call:

- End the call from dashboard or call the orchestrator `POST /v1/calls/:id/end`.
- Worker should update call session to `completed` and set `endedAt`, `durationSeconds`, and `estimatedCost`.

9. Validation queries (Mongo shell / Compass):

- `db.callsessions.find({_id: ObjectId("<callId>")})`
- `db.transcriptevents.find({callSessionId: ObjectId("<callId>")}).sort({timestamp:1})`

10. API validation:

- `GET /v1/calls` should list the call (public-safe fields)
- `GET /v1/calls/:id` should show safe detail (no LiveKit secrets)

Notes & Known Limitations:

- This worker uses the official LiveKit Agents Node framework and wires Deepgram + OpenAI for STT, LLM, and TTS through the SDK-managed session APIs.
- If provider defaults change, adjust the OpenAI/Deepgram constructor options in `apps/agent-worker-node/src/index.ts` and re-run `pnpm build`.
- Failure reasons are persisted to `callsessions.failureReason` on job errors.
