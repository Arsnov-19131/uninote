# Project Status

**Project name:** Uninotes  
**Goal (1-2 lines):** Build clean, minimal Zustand stores with persistence and reliable cloud sync across pages and components.  
**Tech stack:** Next.js (App Router), React, TypeScript, Tailwind CSS, Supabase, Zustand  
**GitHub repo:** Local repository  
**Last updated:** 2026-11-17  

## Current State  
Task 12.4 complete: converted network status tracking to standard `useSyncExternalStore` and deferred client-only environment mounting via microtasks, eliminating all 7 `react-hooks/set-state-in-effect` errors. `npm run lint` passes with 0 errors and 0 warnings, `tsc --noEmit` passes with 0 errors, and unit tests pass 33/33 (`npm test`). Ready for user testing (Task 12.5).

## Build Order / Task List  
- [x] 1. Core Academic Stores (`src/store/subjects.ts`, `src/store/topics.ts`)  
- [x] 2. Planner & Notification Stores (`src/store/tasks.ts`, `src/store/notifications.ts`)  
- [x] 3. Content & Study Stores (`src/store/notes.ts`, `src/store/resources.ts`)  
- [x] 4. Interactive Feature Stores (`src/store/quizzes.ts`, `src/store/chat.ts`, `src/store/progress.ts`)  
- [x] 5. Build Verification & Typecheck  
- [x] 6. Cloud Sync Stable Stringify (`src/lib/cloud-sync.ts`)  
- [x] 7. Store Mutation `updatedAt` Consistency Fix across all stores & types  
- [x] 8. Subject Cascade Deletion & Task Disconnection (`src/store/subjects.ts`, `tasks.ts`, `quizzes.ts`, `progress.ts`, `chat.ts`)  
- [x] 9. Cloud Sync Merge List Data Loss Protection (`src/lib/cloud-sync.ts`)  
- [x] 10. Settings Page Dead Control Pruning (`src/app/dashboard/settings/page.tsx`)  
- [x] 11. Code-review fixes (all except RLS on unused 001 tables and dropping 001, skipped on purpose)  
- [x] 12.1 Remove Unnecessary Files & Dead Dependencies (`001_core_tables.sql`, `@supabase/ssr`)  
- [x] 12.2 Prune Dead Code & Unused Imports / Variables  
- [x] 12.3 Fix Syntax, Entities & Type Cleanups  
- [x] 12.4 Resolve React Effect Synchronization Errors  
- [ ] 12.5 Verification & Validation (User Testing)  

## Next Task  
Task 12.5: Verification & Validation (User Testing — left to user per request).

## Last Commit  
not committed yet  

## Important Files  
- `src/types/index.ts`: TypeScript data model and shared types  
- `src/lib/cloud-sync.ts`: cloud sync with canonical JSON stringify comparison and unkeyed row retention  
- `src/store/subjects.ts`: central cascade dispatcher on `deleteSubject`  
- `src/store/tasks.ts`: task management and `disconnectBySubject`  
- `src/store/topics.ts`, `notes.ts`, `resources.ts`, `quizzes.ts`, `progress.ts`, `chat.ts`: stores with `deleteBySubject` handlers  
- `src/app/dashboard/settings/page.tsx`: Settings page with functional controls only (Appearance, Timezone, Cloud sync, Local backup)  
- `src/app/dashboard/roadmap/page.tsx`: Roadmap UI triggering subject and topic deletion  

## Decisions Made  
- Removed inert controls from Settings (fake email backup without Resend, fake push checkbox, inert "Save Preferences", hardcoded display name) per YAGNI / Lazy Senior Dev rules.  
- Retained clean read-only device timezone information block.  
- Chat style/reading level remains directly selectable on the Chat screen (`src/app/dashboard/chat/page.tsx`).  
- In `mergeList`, preserve unkeyed or non-string-id rows safely with content-based deduplication (`stableStringify`) and console warning, preventing local state wipeouts upon `applySnapshot`.  
- Centralized subject deletion cascade in `useSubjectStore.deleteSubject` so that any caller routes through the same cleanup logic.  
- Planner tasks have their subject and topic references cleared (`subjectId: null, topicId: null`) and `updatedAt` bumped when a subject is deleted, honoring the UI dialog contract ("disconnect associated tasks").  
- All other dependent entities (topics, soft-deleted notes, resources, quizzes + attempts/answers, progress snapshots/suggestions, chat sessions + messages/keywords) are deleted via `deleteBySubject`.  
- All mutations bump `updatedAt: now()` to ensure deterministic cloud sync merges.  

## Known Issues / Notes  
- Skipped on purpose (project is not public): RLS missing on `quiz_questions`, `attempt_answers`, `chat_messages`, `keywords` in migration 001, and the unused 001 schema.
- Skipped: AI rate limits are still per-instance memory (a real fix needs Upstash or a Supabase table); no CSP header (needs nonce setup, untestable here). Other security headers were added.
- Run the new migration `003_limit_app_data_size.sql` in Supabase (5 MB cap on the sync row).
- Task 11 decisions: reminder ids are `taskId|offset|sendMs` so devices converge; sent reminders older than 30 days are pruned; passwords must be 8+ characters (also raise Supabase's minimum password length in the dashboard); fonts use `next/font` (needs network at build time).
