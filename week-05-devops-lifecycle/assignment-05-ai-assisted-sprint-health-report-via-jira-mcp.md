# Assignment 5 — AI-Assisted Sprint Health Report via Jira MCP

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will connect Claude Code to your Jira board through an MCP server, the same way you connected it to GitHub in Week 2, and build a read-only `/sprint-health` skill. The skill reads your current sprint through Jira's API and reports sprint velocity, stories at risk of missing the sprint, and items missing an estimate — but it must never create, edit, comment on, or transition a single ticket itself. You will prove that boundary holds by making a real change on the board yourself and confirming the skill only ever reports, never acts.

---

# Task 1 — Create a Jira API Token

## Goal

Generate an API token from your Atlassian account that the MCP server will use to authenticate with your Jira site. Do not screenshot the token value itself.

### Evidence

#### Screenshot 1 — Jira API token creation confirmation page showing the token name, with the token value not visible

![Week 05 Screenshot](screenshots/week-05-screenshot-56.png)

![Week 05 Screenshot](screenshots/week-05-screenshot-57.png)

### Notes You Must Write (Very Important):

Why does the MCP server need your site URL and account email in addition to the token?

Jira needs your site URL, email, and API token because Basic Auth requires all three: the URL identifies your Jira instance, the email identifies your account, and the token authenticates you.

---

# Task 2 — Create .mcp.json at the Project Root

## Goal

Create or update `.mcp.json` at your project root with a Jira MCP server block, following the same shape as the GitHub MCP server you configured in Week 2.

### Evidence

#### Screenshot 2 — `.mcp.json` open in VS Code showing the Jira server configuration

![Week 05 Screenshot](screenshots/week-05-screenshot-59.png)

### Notes You Must Write (Very Important):

Compare this jira block to the github block from Week 2 Assignment 5. The GitHub server ran via npx (a Node.js package); this one runs via uvx (a Python package) — what stays exactly the same shape despite that difference, and why doesn't Claude Code care which language a given MCP server is written in?

The Jira MCP block and the GitHub MCP block look almost identical even though one runs with npx (Node.js) and the other runs with uvx (Python). Both use the same MCP structure: a command, arguments, and environment variables. The only differences are the command used and the specific credentials required.

Claude Code doesn’t care what language an MCP server is written in because MCP servers run as external processes. Claude only communicates with them through JSON over stdin/stdout, so any language works : Node, Python, Go, Rust, etc. What matters is the MCP protocol, not the programming language.

---

# Task 3 — Add Your Credentials to settings.local.json

## Goal

Add your Jira site URL, account email, and API token to `.claude/settings.local.json`, and confirm that file is listed in `.gitignore` so it is never committed.

### Evidence

#### Screenshot 3 — `settings.local.json` open in VS Code showing the `env` section, with the actual token value blurred or covered

![Week 05 Screenshot](screenshots/week-05-screenshot-60.png)

### Notes You Must Write (Very Important):

Why must JIRA_API_TOKEN live in settings.local.json and never in .mcp.json?

Your Jira API token must stay in settings.local.json because .mcp.json is a project file that can be committed to Git, while settings.local.json is a local‑only secret store designed specifically to keep your credentials safe.

---

# Task 4 — Verify the Connection with /mcp

## Goal

Restart Claude Code and confirm the Jira MCP server shows as connected.

### Evidence

#### Screenshot 4 — `/mcp` output showing `jira: connected`

![Week 05 Screenshot](screenshots/week-05-screenshot-61.png)

---

# Task 5 — Run a Live Query to Prove Real Board Data

## Goal

Ask Claude to list the issues in your current active sprint through the Jira MCP connection, and confirm the result matches what you see on your live board in the browser.

### Evidence

#### Screenshot 5 — Claude's response showing the live sprint issue list retrieved via Jira MCP

![Week 05 Screenshot](screenshots/week-05-screenshot-62.png)

### Notes You Must Write (Very Important):

How did you confirm this was real board data and not something Claude guessed?

the Jira MCP server retrieves live data from a Jira project rather than generating or inferring information. When the prompt is submitted, the request is forwarded by Claude to the Jira MCP server, which authenticates using the configured site URL, account email, and API token. The server then queries the Jira REST API for the active sprint. Jira returns the sprint details, including all issues, their statuses, assignees, story points, priorities, and an overall sprint summary. Because the data is obtained directly from Jira’s API, the response reflects the current state of the project.

---

# Task 6 — Build the /sprint-health Skill

## Goal

Create a `/sprint-health` skill restricted to read-only Jira tools plus `Read`, with no issue-mutating tools and no `Write`. Run it and confirm it produces a report covering sprint velocity, at-risk stories, and items missing an estimate.

### Evidence

#### Screenshot 6 — `SKILL.md` frontmatter showing `allowed-tools` limited to read-only Jira tools plus `Read`, with `disable-model-invocation: true`

![Week 05 Screenshot](screenshots/week-05-screenshot-63.png)

#### Screenshot 7 — `/sprint-health` output showing the full triage report against your real sprint

![Week 05 Screenshot](screenshots/week-05-screenshot-64.png)

![Week 05 Screenshot](screenshots/week-05-screenshot-65.png)

### Notes You Must Write (Very Important):

1. Which Jira MCP tools does this skill's allowed-tools list include, and which mutating tools (create issue, update issue, transition issue, add comment) does it deliberately exclude?

Allowed Jira MCP Tools (Read‑Only)
These tools are included in the skill’s allowed‑tools list:

**mcp__jira__jira_search**  
Performs read‑only search queries across Jira issues.

**mcp__jira__jira_get_issue**  
Retrieves full details for a specific issue.

**mcp__jira__jira_get_sprint**  
Fetches sprint metadata and sprint‑level information.

**mcp__jira__jira_get_board**  
Retrieves board metadata and configuration details.


Excluded Jira MCP Tools (Mutating Operations)
These tools are deliberately not included because they modify Jira data:

**create_issue**  
Would create new issues.

**update_issue**  
Would modify existing issue fields.

**transition_issue**  
Would move issues between workflow states.

**add_comment**  
Would write comments to issues.

All write‑capable tools are excluded to ensure the skill remains strictly read‑only.

These tools only read data and do not modify Jira in any way.

2. Why does a Scrum Master need this restriction more than almost any other role in this course?

The Scrum Master requires strict read‑only MCP restrictions because the role is responsible for process facilitation, not backlog management or issue manipulation. Mutating Jira tools would violate Scrum role boundaries, distort accountability, and compromise sprint transparency.

---

# Task 7 — Prove the Skill Never Mutates the Board

## Goal

Manually update one ticket on your board in the browser (for example, move a story to "Done" or add a missing estimate), then run `/sprint-health` again and confirm the new report reflects your change — proving the skill only ever reads live state and never wrote to the board itself.

### Evidence

#### Screenshot 8 — Second `/sprint-health` run showing the report now reflects your manual board change

![Week 05 Screenshot](screenshots/week-05-screenshot-66.png)

![Week 05 Screenshot](screenshots/week-05-screenshot-67.png)

### Notes You Must Write (Very Important):

Map this assignment to Gather → Analyze → Human Act → Verify from Week 3 Assignment 6. Which step did you perform manually in the browser, and why must that step stay human?

The assignment maps cleanly to Gather → Analyze → Human Act → Verify. The system gathers configuration and credentials, the MCP server analyzes by querying Jira’s API, the human performs the required browser action of opening the Jira board, and the human verifies that the returned sprint data matches the live project. The browser step must remain human because accessing private Jira workspaces requires explicit human intent and cannot be automated by Claude.

---

# Submission Instructions

Complete all tasks in sequence.

Your submission must include:
- All 8 required screenshots
- All the required notes

---

# Completion Checklist

- [ ] Task 1: Jira API token created, value never screenshotted (Screenshot 1)
- [ ] Task 2: `.mcp.json` has the Jira server block (Screenshot 2)
- [ ] Task 3: Credentials stored in `settings.local.json`, token blurred, file gitignored (Screenshot 3)
- [ ] Task 4: `/mcp` shows the Jira server connected (Screenshot 4)
- [ ] Task 5: Live query returned real sprint data, verified against the browser (Screenshot 5)
- [ ] Task 6: `/sprint-health` skill created with correct read-only `allowed-tools`, and produced a full report (Screenshots 6–7)
- [ ] Task 7: A manual board change was reflected in a second `/sprint-health` run (Screenshot 8)
- [ ] Skill never created, edited, transitioned, or commented on any issue
- [ ] Reflection answered (Notes)
- [ ] No API token value exposed

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://dmi.pravinmishra.com?utm_source=github&utm_medium=readme  
- 🎓 University: https://university.pravinmishra.com?utm_source=github&utm_medium=readme  
- 💬 Discord Community: https://discord.pravinmishra.com?utm_source=github&utm_medium=readme  
- 📝 Blog: https://dmi.pravinmishra.com/blog?utm_source=github&utm_medium=readme  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
