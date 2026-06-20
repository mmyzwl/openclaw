# Issue #92062 Analysis: sessions_history misses archived/reset transcripts

## Root Cause

`sessions_history` calls `chat.history` gateway method, which reads the current
active session transcript via `readRecentSessionMessagesAsync`. This function
uses `findExistingTranscriptHistoryPathAsync` which returns only the active
transcript when it exists — it does NOT merge prior archived/reset transcripts.

The `allowResetArchiveFallback: true` option only kicks in when the active
transcript file is MISSING (deleted or not yet created). When the active file
exists (normal case), archived `.jsonl.reset.*` files are never consulted.

## Affected Code

- `src/gateway/session-utils.fs.ts` — `findExistingTranscriptHistoryPathAsync()`
- `src/gateway/server-methods/chat.ts` — `handleChatHistoryRequest()`

## Recommended Fix

1. Add optional `includeArchived` parameter to `sessions_history` tool
2. When enabled, search for `.jsonl.reset.*` files in the session store
3. Read archived transcripts and merge them with the active transcript
4. Deduplicate overlapping messages across transcripts

This is tracked by related issues: #56131, #57139, #82334, #89760
