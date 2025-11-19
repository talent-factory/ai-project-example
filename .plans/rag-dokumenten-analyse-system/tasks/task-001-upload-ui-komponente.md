# Task-001: Upload-UI-Komponente mit Drag-and-Drop

## Metadata
- **ID**: task-001
- **Status**: pending
- **Priority**: must
- **Estimate**: 3 Story Points
- **Labels**: frontend, react, ui, upload
- **Assignee**: frontend-developer
- **Created**: 2025-11-18
- **Updated**: 2025-11-18

## Description

Erstelle eine React-Komponente für den Datei-Upload mit Drag-and-Drop-Funktionalität. Die Komponente soll PDF- und DOCX-Dateien akzeptieren, Format- und Größenvalidierung durchführen und einen Upload-Progress-Indikator anzeigen.

**User Story**: Als Anna möchte ich ein PDF/Word-Dokument hochladen, damit ich später Fragen dazu stellen kann.

## Acceptance Criteria

- [ ] Drag-and-Drop-Zone implementiert (visuelles Feedback beim Hover)
- [ ] File-Input-Fallback für Click-to-Upload
- [ ] Format-Validierung (nur .pdf, .docx akzeptiert)
- [ ] Größen-Validierung (max 20MB)
- [ ] Upload-Progress-Bar mit Prozent-Anzeige
- [ ] Erfolgsbestätigung mit Dokumentenname
- [ ] Fehlerbehandlung (zu große Datei, falsches Format, Network-Error)
- [ ] Responsive Design (Desktop & Tablet)
- [ ] Accessibility (Keyboard-Navigation, Screen-Reader-kompatibel)

## Dependencies

- **Requires**: None (kann sofort starten)
- **Blocks**: task-005 (Dokumenten-Übersicht benötigt Upload-Funktionalität)

## Agent Recommendation

**Recommended Agent**: `frontend-developer`

**Rationale**: React-Komponenten-Entwicklung mit komplexem State-Management, File-Handling und Accessibility-Requirements. Frontend-Developer-Agent hat Expertise in modernen React-Patterns und UI-Best-Practices.

## Implementation Notes

### Technical Approach
- React Component mit Hooks (useState, useCallback)
- File-Drop-Event-Handling (onDrop, onDragOver)
- FormData für Multipart-Upload
- Axios/Fetch für Upload mit Progress-Tracking
- Error-Boundaries für robuste Fehlerbehandlung

### Design Specifications
- Drag-Zone: Gestrichelte Border, Highlight bei Hover
- Akzeptierte Formate-Anzeige: "PDF, DOCX (max 20MB)"
- Progress-Bar: Linear-Progress-Component (Material-UI/shadcn)
- Error-Messages: Toast/Alert-Component mit Retry-Option

### Files to Create/Modify
- `src/components/DocumentUpload/DocumentUpload.tsx`
- `src/components/DocumentUpload/DocumentUpload.test.tsx`
- `src/components/DocumentUpload/styles.module.css`
- `src/hooks/useFileUpload.ts` (Custom Hook für Upload-Logik)

### API-Integration
```typescript
POST /api/documents
Content-Type: multipart/form-data

Request:
- file: File (PDF/DOCX)

Response:
{
  "id": "doc-123",
  "filename": "report.pdf",
  "status": "processing",
  "size": 5242880
}
```

## Testing Strategy

### Unit Tests
- File-Drop-Event-Handler
- Format-Validierung (accept/reject verschiedene Formate)
- Größen-Validierung (< 20MB pass, > 20MB fail)
- Upload-Progress-Tracking
- Error-Handling (Network-Error-Simulation)

### Integration Tests
- End-to-End Upload-Flow (File-Select → Upload → Success)
- API-Integration mit Mock-Backend
- Error-Recovery-Flow (Retry nach Fehler)

### Accessibility Tests
- Keyboard-Navigation (Tab, Enter)
- Screen-Reader-Kompatibilität (ARIA-Labels)
- Focus-Management

## Notes

**Referenced PRD Sections**:
- User Story US-1.1 (Upload PDF-Dokument)
- User Story US-1.2 (Upload Word-Dokument)
- FR-1 (Dokumenten-Upload)
- NFR-14 (Mobile-responsive)
- NFR-15-18 (Accessibility WCAG 2.1 AA)

**Potential Challenges**:
- Large-File-Upload-Performance (20MB über langsame Verbindungen)
- CORS-Issues bei Cross-Origin-Uploads
- Browser-Kompatibilität für File-API

**Future Enhancements** (Post-MVP):
- Bulk-Upload (mehrere Dateien gleichzeitig)
- Pause/Resume für große Uploads
- Thumbnail-Preview nach Upload
