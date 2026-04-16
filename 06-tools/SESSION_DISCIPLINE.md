# Session Discipline

This skill explains how to use Hermes sessions correctly and how to manage content within each session.

## Why session stats matter
- Stats (total sessions, total messages, source breakdown, database size) help you understand the shape of your session history before it becomes invisible clutter.
- Use stats as a first health check; deeper insights exist in Hermes’ separate insights tooling.

## Sessions vs session_search
- A **session** is the stored conversation itself (the archive).
- **session_search** is the retrieval layer that helps Hermes look back across past conversations (the recall mechanism).
- Good session organization improves both your manual workflow and Hermes’ ability to retrieve useful past context later.

## What new Hermes users should do first (5 habits)
1. **Resume ongoing work** instead of starting fresh every time.
   - Use: `hermes -c` or `hermes -c "project name"`
2. **Name important sessions early**.
   - Use: `/title my project`
3. **Use one session per meaningful workstream**.
   - Do not mix unrelated tasks into one endless thread.
4. **Branch when exploring alternatives**.
   - Use: `/branch`
5. **List and prune occasionally**.
   - Use: `hermes sessions list` and `hermes sessions prune`

## Final takeaway
Sessions are Hermes’ working memory structure — they let Hermes act like an environment you return to, not a blank page you reopen. Learning how to create, name, resume, branch, export, prune, and organize sessions is core operational habit that makes Hermes feel powerful.