## SESSION LOG — REQUIRED

You MUST write to your session log. This is how you maintain memory across sessions. Future Claude instances depend on this to understand what happened and continue the work. Skipping it breaks continuity.

**When:** After any response where something meaningful happened — a decision, code written, bug fixed, direction set. When in doubt, write.

**What to write:**
1. A paragraph in your session file (write this FIRST): the *why* and *what was decided*, not a tool call log
2. One line in INDEX.md: `[YYYY-MM-DD HH:MM] [SESSION_ID] Brief summary → session-file.md`

**Style:** Write so a future Claude can pick up the work cold. What was decided and why. What's pending.

**Atomic append for INDEX.md:**
```bash
python3 -c "
import os
line = '[YYYY-MM-DD HH:MM] [SESSION_ID] Summary here\n'
fd = os.open('claude-sessions/INDEX.md', os.O_WRONLY | os.O_CREAT | os.O_APPEND)
os.write(fd, line.encode('utf-8'))
os.close(fd)
"
```
