# canvas-mcp

An MCP server that connects Claude to Canvas LMS. Ask about your courses, deadlines, rubrics, and instructor feedback in plain conversation, and submit work without leaving the chat.

**Ask Claude things like:**

- "What's due this week across all my courses?"
- "Show me the rubric for my English essay."
- "What feedback did I get on my last chemistry lab?"
- "Do I have any overdue assignments?"
- "Read the instructions in the .docx attached to this assignment."

It runs on your computer with your own Canvas token. Nothing goes through a third-party server.

## Quick start

Needs Node.js 18 or newer.

### 1. Get a Canvas API token

1. Log in to Canvas.
2. Click your profile picture, then **Settings**.
3. Scroll to **Approved Integrations** and click **+ New Access Token**.
4. Name it (for example, "Claude"), set an expiry date, and click **Generate Token**.
5. Copy the token. Canvas shows it only once.

### 2. Build the server

```bash
git clone https://github.com/Alejandro-Valadez/canvas-mcp.git
cd canvas-mcp
npm install
npm run build
```

### 3. Connect it to Claude

**Claude Desktop.** Open the config file:

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Add this block (merge it if `mcpServers` already exists):

```json
{
  "mcpServers": {
    "canvas": {
      "command": "node",
      "args": ["/FULL/PATH/TO/canvas-mcp/dist/index.js"],
      "env": {
        "CANVAS_API_TOKEN": "YOUR_TOKEN_HERE",
        "CANVAS_BASE_URL": "https://your-school.instructure.com"
      }
    }
  }
}
```

Replace the path, the token, and the URL. Your Canvas URL is the address in your browser when you're logged in, with no trailing slash. Then quit Claude Desktop completely and reopen it.

**Claude Code.**

```bash
claude mcp add canvas \
  -e CANVAS_API_TOKEN=YOUR_TOKEN_HERE \
  -e CANVAS_BASE_URL=https://your-school.instructure.com \
  -- node /FULL/PATH/TO/canvas-mcp/dist/index.js
```

You can also put both values in a `.env` file (copy `.env.example`) in the directory you start the server from.

## Tools

20 tools in 7 groups.

| Group | Tool | What it does |
|---|---|---|
| Courses | `list_courses` | List your enrolled courses |
| | `get_course` | Course details and syllabus |
| Assignments | `list_assignments` | List assignments, with filters |
| | `get_assignment` | Full assignment details and rubric |
| | `get_rubric` | Grading rubric for an assignment |
| Submissions | `get_submission` | Your submission, grade, and feedback |
| | `submit_assignment` | Submit text or a URL |
| | `upload_file` | Upload a file for a submission |
| Modules | `list_modules` | Browse course modules |
| | `list_announcements` | Course announcements |
| Discussions | `list_discussions` | Discussion topics |
| | `get_discussion_entries` | Read discussion posts |
| | `post_discussion_entry` | Post to a discussion |
| | `reply_to_discussion` | Reply to a post |
| Search | `find_assignments_by_due_date` | Assignments in a date range |
| | `get_upcoming_assignments` | Work due in the next N days |
| | `get_overdue_assignments` | Past-due work |
| | `search_course_content` | Search modules and assignments |
| | `get_all_upcoming_work` | Upcoming work across every course |
| Files | `read_docx_file` | Read the text of a `.docx` attachment |

Four tools write to Canvas as you: `submit_assignment`, `upload_file`, `post_discussion_entry`, and `reply_to_discussion`. Keep tool approval on in your Claude client so you confirm each one before it runs.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Canvas tools don't appear | Fully quit and reopen Claude Desktop. Check the path to `dist/index.js` and that the config is valid JSON. |
| `401 Unauthorized` | The token is wrong or expired. Make a new one in Canvas settings. |
| `403 Forbidden` | You don't have access to that course or item. The course may have ended. |
| Connection refused or network errors | Check `CANVAS_BASE_URL`: it needs `https://` and no trailing slash. |

## Security

Your token has the same access as your Canvas account: it can read your grades and submit work.

- Never commit it. `.env` is in `.gitignore`.
- Give it an expiry date.
- Revoke tokens you no longer use (Canvas **Settings > Approved Integrations**).

## Development

```bash
npm install
npm run build   # compile TypeScript to dist/
npm run dev     # rebuild on change
npm start       # run the server
```

```
src/
├── index.ts           # MCP server entry point
├── canvas-client.ts   # Canvas REST API wrapper
├── tools/             # courses, assignments, submissions, modules,
│                      # discussions, search, files
└── types/canvas.ts    # Canvas API types
```

Built with the [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk), [zod](https://zod.dev), and [mammoth](https://github.com/mwilliamson/mammoth.js) for `.docx` text.

Issues and pull requests are welcome.

## License

[MIT](LICENSE). Not affiliated with Instructure or Anthropic.
