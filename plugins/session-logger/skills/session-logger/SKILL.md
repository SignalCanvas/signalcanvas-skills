## Session Logger

Your session file: `claude-sessions/YYYY-MM-DD_SESSION_ID.md`  
Index file: `claude-sessions/INDEX.md`

**When to write:** When something meaningful happened — a decision was made, code was written, a task completed, a bug fixed. Skip pure back-and-forth discussion. When in doubt, write.

**What to write:**
1. A short paragraph in your session file (write this FIRST)
2. One line in INDEX.md: `[YYYY-MM-DD HH:MM] [SESSION_ID] Brief summary → session-file.md`

**Style:** Write the *why* and *what was decided*. Not a tool call log. A future Claude should be able to read this and continue the work.

**Atomic append for INDEX.md:**  
First run `mkdir -p claude-sessions`, then use:
```bash
python3 -c "
import os
line = '[YYYY-MM-DD HH:MM] [SESSION_ID] Summary here\n'
fd = os.open('claude-sessions/INDEX.md', os.O_WRONLY | os.O_CREAT | os.O_APPEND)
os.write(fd, line.encode('utf-8'))
os.close(fd)
"
```
