# SDK Reference

TypeScript client library, React hooks, and type definitions for TMOS13.

---

## Installation

```bash
npm install @tmos13/sdk
```

The SDK provides three layers:
1. **`TMOS13Client`** — HTTP/WebSocket client for direct API access
2. **React hooks** — State-managed hooks for building frontends
3. **Types** — Full TypeScript definitions for all API shapes

---

## TMOS13Client

The core client for communicating with a TMOS13 engine instance.

### Constructor

```typescript
import { TMOS13Client } from "@tmos13/sdk";

const client = new TMOS13Client({
  baseUrl: "http://localhost:8000",
  token: "optional-jwt-token",
});
```

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `baseUrl` | `string` | Yes | Engine URL |
| `token` | `string` | No | JWT for authenticated requests |

### Chat

```typescript
// Send a message
const response = await client.chat("hello", {
  sessionId: "optional-session-id",
  userId: "optional-user-id",
  packId: "optional-pack-override",
});

// response.response — AI response text
// response.session_id — Session identifier
// response.state — Current session state
```

### WebSocket

```typescript
// Real-time connection
const ws = client.connectWebSocket({
  onMessage: (data) => console.log(data.message),
  onConnect: (data) => console.log("Connected:", data.session_id),
  onError: (error) => console.error(error),
});

// Send via WebSocket
ws.send("hello");

// Close
ws.close();
```

### Health

```typescript
const health = await client.health();
// health.status — "online"
// health.pack — Active pack info
// health.active_sessions — Session count
// health.infrastructure — Database, cache, monitoring status
```

### Packs

```typescript
// List available packs
const packs = await client.getPacks();

// Get specific pack details
const pack = await client.getPack("legal_intake");
```

---

## Authentication Methods

```typescript
// Sign up
await client.signUp({ email: "user@example.com", password: "..." });

// Sign in
const session = await client.signIn({ email: "user@example.com", password: "..." });
// session.access_token — JWT for subsequent requests

// OAuth
const { url } = await client.oauthStart("google");
// Redirect user to url, handle callback

// Get current user
const user = await client.getMe();

// Refresh token
await client.refreshToken();

// Sign out
await client.signOut();
```

---

## Audio

```typescript
// Transcribe audio (speech-to-text)
const result = await client.transcribe(audioBlob, {
  language: "en",
  format: "webm",
});
// result.text — Transcribed text

// Synthesize speech (text-to-speech)
const audioBlob = await client.synthesize("Hello, how can I help?", {
  voice: "alloy",
  speed: 1.0,
});

// Voice chat (audio in → text processing → audio out)
const response = await client.audioChat(audioBlob, {
  sessionId: "...",
});
// response.audio — Response audio blob
// response.transcript — What the user said
// response.response — AI response text

// List available voices
const voices = await client.getVoices();

// Audio system status
const status = await client.audioStatus();
```

---

## Files

```typescript
// Upload a file
const metadata = await client.uploadFile(file, {
  sessionId: "...",
  category: "document",
});
// metadata.id, metadata.filename, metadata.size, metadata.chunks

// Get file metadata
const file = await client.getFile("file-id");

// List files
const files = await client.getFiles({ sessionId: "..." });

// Get file chunks (for large files)
const chunks = await client.getFileChunks("file-id");

// Delete a file
await client.deleteFile("file-id");
```

---

## Notes

```typescript
// Create a note
const note = await client.createNote({
  title: "Meeting Notes",
  content: "Discussed project timeline...",
  tags: ["meeting", "planning"],
});

// Get a note
const note = await client.getNote("note-id");

// Update a note
await client.updateNote("note-id", { content: "Updated content" });

// Delete a note
await client.deleteNote("note-id");

// Search notes
const results = await client.searchNotes("project timeline");

// Get context-relevant notes
const context = await client.getNoteContext("What was decided about the timeline?");

// Summarize a conversation
const summary = await client.summarizeConversation(messages);

// Export notes
const exported = await client.exportNotes({ format: "markdown" });
```

---

## Feed Portal

```typescript
// Query the feed (natural language → structured data)
const card = await client.feedQuery("What's the weather in NYC?");
// card.type — "weather", "stock", "news", etc.
// card.data — Structured result data
// card.connector — Which connector handled it

// Query history
const history = await client.feedHistory();

// Available connectors
const connectors = await client.feedConnectors();
```

---

## React Hooks

All hooks manage their own state and provide loading/error handling.

### `useTMOS13`

Core hook for chat interactions.

```tsx
import { useTMOS13 } from "@tmos13/sdk";

function Chat() {
  const {
    sendMessage,    // (text: string) => Promise<void>
    messages,       // Message[]
    sessionState,   // SessionState | null
    isLoading,      // boolean
    error,          // Error | null
    sessionId,      // string | null
    resetSession,   // () => void
  } = useTMOS13({
    baseUrl: "http://localhost:8000",
    packId: "my_pack",           // optional
    userId: "user-123",          // optional
    onMessage: (msg) => {},      // optional callback
    onStateChange: (state) => {},// optional callback
  });

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id} className={m.role}>
          {m.text}
        </div>
      ))}
      {isLoading && <Spinner />}
      <input onSubmit={(text) => sendMessage(text)} />
    </div>
  );
}
```

### `useAuth`

Authentication state management.

```tsx
import { useAuth } from "@tmos13/sdk";

const {
  user,           // UserProfile | null
  isAuthenticated,// boolean
  isLoading,      // boolean
  signIn,         // (email, password) => Promise<void>
  signUp,         // (email, password) => Promise<void>
  signOut,        // () => Promise<void>
  oauthStart,     // (provider) => Promise<void>
} = useAuth({ baseUrl: "http://localhost:8000" });
```

### `useAudio`

Voice interaction with MediaRecorder integration.

```tsx
import { useAudio } from "@tmos13/sdk";

const {
  isRecording,    // boolean
  isProcessing,   // boolean
  startRecording, // () => void
  stopRecording,  // () => Promise<TranscriptionResult>
  synthesize,     // (text) => Promise<Blob>
  audioChat,      // () => Promise<AudioChatResult>
  voices,         // Voice[]
} = useAudio({ baseUrl: "http://localhost:8000" });
```

### `useFiles`

File upload with drag-and-drop support.

```tsx
import { useFiles } from "@tmos13/sdk";

const {
  files,          // FileMetadata[]
  isUploading,    // boolean
  upload,         // (file: File) => Promise<FileMetadata>
  remove,         // (id: string) => Promise<void>
  dragProps,      // Spread onto a drop zone element
} = useFiles({ baseUrl: "http://localhost:8000" });
```

### `useNotes`

Notes CRUD with search.

```tsx
import { useNotes } from "@tmos13/sdk";

const {
  notes,          // Note[]
  isLoading,      // boolean
  create,         // (data) => Promise<Note>
  update,         // (id, data) => Promise<Note>
  remove,         // (id) => Promise<void>
  search,         // (query) => Promise<Note[]>
} = useNotes({ baseUrl: "http://localhost:8000" });
```

### `useFeed`

Natural language data queries.

```tsx
import { useFeed } from "@tmos13/sdk";

const {
  query,          // (text: string) => Promise<FeedCard>
  cards,          // FeedCard[]
  connectors,     // ConnectorInfo[]
  isQuerying,     // boolean
} = useFeed({ baseUrl: "http://localhost:8000" });
```

### `useFeatureFlags`

Runtime feature flag access.

```tsx
import { useFeatureFlags } from "@tmos13/sdk";

const {
  flags,          // Record<string, boolean>
  isEnabled,      // (flag: string) => boolean
  isLoading,      // boolean
} = useFeatureFlags({ baseUrl: "http://localhost:8000" });
```

### `usePrivacy`

Privacy controls and consent management.

```tsx
import { usePrivacy } from "@tmos13/sdk";

const {
  consent,        // ConsentSettings
  updateConsent,  // (settings) => Promise<void>
  exportData,     // () => Promise<ExportData>
  deleteAccount,  // () => Promise<void>
  redactMemories, // (keys: string[]) => Promise<void>
  auditLog,       // AuditEntry[]
} = usePrivacy({ baseUrl: "http://localhost:8000" });
```

### `useNotifications`

In-app notification management.

```tsx
import { useNotifications } from "@tmos13/sdk";

const {
  notifications,  // InAppNotification[]
  unreadCount,    // number
  markRead,       // (id: string) => void
  markAllRead,    // () => void
  dismiss,        // (id: string) => void
} = useNotifications({ baseUrl: "http://localhost:8000" });
```

---

## Types

The SDK exports full TypeScript definitions for all request/response shapes:

```typescript
import type {
  // Core
  ChatRequest,
  ChatResponse,
  SessionState,
  HealthResponse,

  // Auth
  UserProfile,
  SignInRequest,
  SignUpRequest,

  // Audio
  TranscriptionResult,
  SynthesisResult,
  AudioChatResult,
  Voice,

  // Files
  FileMetadata,
  FileChunk,
  FileCategory,

  // Notes
  Note,
  NoteSearchResult,
  ConversationSummary,

  // Feed
  FeedCard,
  FeedQuery,
  ConnectorInfo,

  // Privacy
  ConsentSettings,
  ExportData,
  AuditEntry,

  // Packs
  PackInfo,
  PackMetadata,
} from "@tmos13/sdk";
```

---

## Configuration

### Environment Detection

The SDK auto-detects the environment and adjusts behavior:

| Environment | WebSocket | Auth | Default Base URL |
|------------|-----------|------|-----------------|
| Browser | Native WebSocket | Cookie + Bearer | Window origin |
| Node.js | `ws` package | Bearer only | Must be specified |
| React Native | Native WebSocket | Bearer + SecureStore | Must be specified |

### Error Handling

All client methods throw typed errors:

```typescript
import { TMOS13Client } from "@tmos13/sdk";

try {
  await client.chat("hello");
} catch (error) {
  if (error.status === 401) {
    // Token expired — refresh and retry
  } else if (error.status === 429) {
    // Rate limited — back off
  }
}
```

---

## Next Steps

- [Getting Started](getting-started.md) — Quick setup guide
- [API Reference](api-reference.md) — Raw REST endpoints the SDK wraps
- [Pack Development](pack-development.md) — Build your own vertical
