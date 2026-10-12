---
name: my-merge-request
version: 1.0.0
description: |
  Basic instructions for creating a branch, pushing changes, and opening a merge
  request or pull request.
compatibility: claude-code cursor codex gemini-cli opencode
argument-hint: <ticket_id>
arguments: [ticket_id]
---

# Create Merge Request

1. First checkout main, stash any existing changes, and pull latest from origin. \
2. Commit the finished changes to a new branch using conventional commit message (no scope).
3. Open a merge request. 
   - Set the merge request title to the commit message. 
   - The description should be a brief, bullet-point summary of changes.
   - Do not include testing instructions.
4. Respond with merge request URL.