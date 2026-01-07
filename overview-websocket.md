# Realtime Communication Hooks Standards

This document establishes the standards for implementing implementation of Server-Sent Events (SSE) and WebSockets in the application. All future AI-generated code regarding realtime communication MUST follow these patterns.

## 1. Server-Sent Events (SSE) Standard

Use the `useSimpleSSE` pattern for unidirectional data streaming (e.g., text generation, progress updates).

### Key Requirements
- **Core Technology**: Use native `fetch` API with `ReadableStream` and `TextDecoder`. Do not use `EventSource` unless absolutely necessary, to allow for custom header manipulation (Authorization).
- **Authentication**: 
  - Token must be retrieved from `auth-storage` cookie if available.
  - Header: `Authorization: Bearer <token>`.
- **Payload Support**: 
  - Must support `GET` (query params) and `POST` (JSON body) requests.
- **Data Parsing**:
  - Handle `data:` prefix parsing manually.
  - Buffer incomplete lines to handle stream chunking correctly.
  - Support `autoParseJson` config.
- **State Management**:
  - Track `status`: `'idle' | 'connecting' | 'connected' | 'error' | 'closed'`.
  - Expose `reconnect`, `closeConnection`, `data`, and `error`.
- **Cleanup**:
  - Use `AbortController` to cancel requests on unmount or manual close.

### Reference Implementation (`hooks/useSSE.ts`)
Users of this guideline should refer to `hooks/useSSE.ts` for the exact implementation logic regarding the `readStream` loop and `AbortController` usage.

## 2. WebSocket (Socket.IO) Standard

Use the `useSimpleWebsocket` pattern for bidirectional real-time communication (e.g., voice chat, interactive AI sessions).

### Key Requirements
- **Core Technology**: Use `socket.io-client`.
- **Configuration**:
  - Auto-connect should be `false` (manual control via `sio.connect()` or logic).
  - Reconnection enabled with `Infinity` attempts and standard intervals.
  - Transports: `['polling', 'websocket']`.
- **Authentication**:
  - Pass token in `auth` object: `{ token, "x-ajari-key": customToken }`.
  - Use environment variable `NEXT_PUBLIC_AI_ONBOARD_TOKEN` as fallback.
- **Audio/Voice Handling (Critical for AI features)**:
  - If handling audio data, integrate `AudioContext` management directly in the hook.
  - Use `safeResumeAudioContext`, `createSafeAudioContext` utilities.
  - Listen for `AUDIO_OUTPUT` events, decode Int16/Float32 audio chunks, and play them.
- **Event Structure**:
  - `CONNECT`: Emit `join` event immediately.
  - `DISCONNECT` / `ERROR`: Standard logging and callbacks.
  - Dynamic Custom Events: Allow passing `customEvents` array `{ name, handler }`.
- **Emitters**:
  - Wrap `emit` calls in `useCallback`.
  - Ensure socket is connected before emitting.

### Reference Implementation (`hooks/useSimpleWebsocket.tsx`)
Refer to `hooks/useSimpleWebsocket.tsx` for the specific AudioContext implementation and reconnection logic.

## 3. General Best Practices
- **Environment Variables**: Always use `process.env.NEXT_PUBLIC_API_URL` or `NEXT_PUBLIC_AI_ONBOARD_URL` for base URLs.
- **Typing**: Use TypeScript generics (`<T>`) for data payloads (e.g., `Emission` payloads, `Message` types).
- **Error Handling**: Both hooks must expose a dedicated `onError` callback and `error` state.

## 4. LiveKit Pattern (Video/Audio Streaming)

**Video/Audio Streaming**

```typescript
"use client";

import { LiveKitRoom } from "@livekit/components-react";
import "@livekit/components-styles";

export default function VideoRoom({
  token,
  serverUrl,
}: {
  token: string;
  serverUrl: string;
}) {
  return (
    <LiveKitRoom
      video={true}
      audio={true}
      token={token}
      serverUrl={serverUrl}
      connect={true}
    >
      {/* Video components */}
    </LiveKitRoom>
  );
}
```
