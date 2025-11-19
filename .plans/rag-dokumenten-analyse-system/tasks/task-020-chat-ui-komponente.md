# Task-020: Chat-UI-Komponente

## Metadata
- **ID**: task-020
- **Status**: pending
- **Priority**: must
- **Estimate**: 5 Story Points
- **Labels**: frontend, react, ui, chat
- **Assignee**: frontend-developer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Erstelle die Chat-Interface-Komponente für die RAG-Anwendung. Nutzer können Fragen eingeben, Antworten werden in einer Message-Liste angezeigt mit Quellenangaben, Loading-States und Error-Handling.

**User Story**: Als Anna möchte ich eine Frage zu meinen Dokumenten stellen und die Antwort mit Quellenangaben sehen (US-3.1, US-3.2).

## Acceptance Criteria

- [ ] Chat-Input-Feld mit Auto-Focus
- [ ] Message-Liste (User-Messages + AI-Responses)
- [ ] Quellenangaben clickable (Dokumentname, Seitenzahl)
- [ ] Loading-Indikator während Query-Verarbeitung
- [ ] Error-Handling (API-Error, Rate-Limit, Timeout)
- [ ] Scroll-to-Bottom bei neuer Nachricht
- [ ] Responsive Design (Desktop & Tablet)
- [ ] Accessibility (Keyboard-Navigation, Screen-Reader)
- [ ] Empty-State ("Stelle eine Frage zu deinen Dokumenten...")
- [ ] Character-Limit-Indikator (max 500 Zeichen pro Query)

## Dependencies

- **Requires**:
  - task-015 (RAG-Pipeline funktional)
  - task-018 (Query-API-Endpoint verfügbar)
- **Blocks**:
  - task-025 (End-to-End-Integration benötigt Chat-UI)

## Agent Recommendation

**Recommended Agent**: `frontend-developer`

**Rationale**: React-Komponenten-Entwicklung mit komplexem State-Management, Real-Time-Updates und Accessibility-Requirements.

## Implementation Notes

### Component-Struktur

```typescript
// src/components/Chat/Chat.tsx
import React, { useState, useRef, useEffect } from 'react';
import { Message, QueryResponse } from '@/types';
import { ChatInput } from './ChatInput';
import { MessageList } from './MessageList';
import { useChat } from '@/hooks/useChat';

export const Chat: React.FC = () => {
  const [messages, setMessages] = useState<Message[]>([]);
  const { sendQuery, isLoading, error } = useChat();
  const messagesEndRef = useRef<HTMLDivElement>(null);

  const handleSendMessage = async (text: string) => {
    // Add user message
    const userMessage: Message = {
      id: Date.now(),
      type: 'user',
      text,
      timestamp: new Date()
    };
    setMessages(prev => [...prev, userMessage]);

    // Send query to API
    try {
      const response: QueryResponse = await sendQuery(text);

      // Add AI response
      const aiMessage: Message = {
        id: Date.now() + 1,
        type: 'ai',
        text: response.answer,
        sources: response.sources,
        confidence: response.confidence,
        timestamp: new Date()
      };
      setMessages(prev => [...prev, aiMessage]);
    } catch (err) {
      // Add error message
      const errorMessage: Message = {
        id: Date.now() + 1,
        type: 'error',
        text: 'Fehler beim Verarbeiten deiner Frage. Bitte versuche es erneut.',
        timestamp: new Date()
      };
      setMessages(prev => [...prev, errorMessage]);
    }
  };

  // Auto-scroll to bottom
  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  return (
    <div className="chat-container">
      <MessageList
        messages={messages}
        isLoading={isLoading}
      />
      <div ref={messagesEndRef} />
      <ChatInput
        onSend={handleSendMessage}
        disabled={isLoading}
        maxLength={500}
      />
      {error && <ErrorBanner message={error} />}
    </div>
  );
};
```

### Message-List mit Sources

```typescript
// src/components/Chat/MessageList.tsx
import { Message } from '@/types';
import { SourceCitation } from './SourceCitation';

export const MessageList: React.FC<{ messages: Message[] }> = ({ messages }) => {
  return (
    <div className="message-list">
      {messages.map(message => (
        <div
          key={message.id}
          className={`message message--${message.type}`}
        >
          {message.type === 'user' ? (
            <UserMessage text={message.text} />
          ) : (
            <AIMessage
              text={message.text}
              sources={message.sources}
              confidence={message.confidence}
            />
          )}
        </div>
      ))}
    </div>
  );
};

const AIMessage: React.FC<{
  text: string;
  sources?: Source[];
  confidence?: number;
}> = ({ text, sources, confidence }) => {
  return (
    <div className="ai-message">
      <div className="ai-message__text">{text}</div>

      {sources && sources.length > 0 && (
        <div className="ai-message__sources">
          <h4>Quellen:</h4>
          {sources.map((source, i) => (
            <SourceCitation
              key={i}
              document={source.document}
              page={source.page}
              relevanceScore={source.relevance_score}
            />
          ))}
        </div>
      )}

      {confidence && (
        <div className="ai-message__confidence">
          Konfidenz: {(confidence * 100).toFixed(0)}%
        </div>
      )}
    </div>
  );
};
```

### API-Integration Hook

```typescript
// src/hooks/useChat.ts
import { useState } from 'react';
import axios from 'axios';

export const useChat = () => {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const sendQuery = async (query: string): Promise<QueryResponse> => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await axios.post('/api/query', { query });
      return response.data;
    } catch (err) {
      if (axios.isAxiosError(err)) {
        if (err.response?.status === 429) {
          throw new Error('Rate-Limit erreicht. Bitte warte einen Moment.');
        } else if (err.response?.status === 401) {
          throw new Error('Nicht authentifiziert. Bitte melde dich an.');
        }
      }
      throw new Error('Fehler beim Verarbeiten der Anfrage.');
    } finally {
      setIsLoading(false);
    }
  };

  return { sendQuery, isLoading, error };
};
```

### Files to Create/Modify
- `src/components/Chat/Chat.tsx`
- `src/components/Chat/ChatInput.tsx`
- `src/components/Chat/MessageList.tsx`
- `src/components/Chat/SourceCitation.tsx`
- `src/hooks/useChat.ts`
- `src/components/Chat/Chat.test.tsx`
- `src/components/Chat/styles.module.css`

## Testing Strategy

### Unit Tests
- Message-Rendering (User-Message, AI-Message, Error-Message)
- Source-Citation-Display
- Input-Validation (max 500 chars)
- Loading-State-Display
- Error-Handling-Display

### Integration Tests
- End-to-End Chat-Flow (Input → API-Call → Response-Display)
- Source-Click-Behavior
- Auto-Scroll-Functionality
- Multiple-Messages-Handling

### Accessibility Tests
- Keyboard-Navigation (Tab, Enter)
- Screen-Reader-Kompatibilität (ARIA-Labels)
- Focus-Management (Input-Field after response)

## Notes

**Referenced PRD Sections**:
- US-3.1 (Frage stellen)
- US-3.2 (Quellenangaben)
- FR-5 (Chat-UI)
- NFR-15-18 (Accessibility)

**Design-Inspiration**:
- ChatGPT-Interface (minimalistisch, fokussiert)
- Perplexity.ai (Quellenangaben-Display)
- Claude.ai (Error-Handling)

**Potential Challenges**:
- Long-Responses-Rendering (performance)
- Source-Highlighting im Original-Dokument (Post-MVP)
- Mobile-Responsiveness für lange Chats

**Future Enhancements** (Post-MVP):
- Streaming-Responses (Server-Sent-Events)
- Chat-Historie persistent speichern (US-3.3)
- Export-Funktion (Markdown/PDF)
- Voice-Input (Speech-to-Text)
