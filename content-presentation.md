# Content Presentation Skill Specification

This document defines a reusable content presentation skill. It is intentionally provider-agnostic, editor-agnostic, browser-agnostic, and machine-agnostic.

The skill can be implemented by any capable LLM agent that can open files with a local application and write HTML to a temporary location. No LLM API, SDK, or provider dependency is required.

## Skill Identity

Name: `content-presentation` (may also be published under the local alias `show-content`).

Purpose: when the user asks to see, show, view, present, open, or look at content, route the content to the right surface automatically:

1. if the content is source code or a plain text / markdown / log / configuration file that already exists on disk, open it in a text editor;
2. otherwise (data pulled from a database, an API, an agent, a summary, a digest, a report, a table, or anything the agent just assembled), render a polished HTML presentation and open it in a browser.

Use this skill for any "show me", "see", "view", "present", "open", or "display" request. The skill replaces the anti-pattern of pasting long content back into the chat transcript.

## Non-Negotiable Constraints

- Do not depend on any specific text editor, browser, operating system, LLM, or network service.
- Do not paste the full content back into the chat transcript when the same content is being opened in an editor or browser. The editor or browser is the delivery.
- Do not ask the user which surface to use. The routing rule decides.
- Do not write persistent files outside a system temp directory unless the user explicitly requested persistence.
- Do not overwrite user files. When rendering HTML, use a fresh path per invocation (e.g. include a slug and a date).
- Do not render content the user never asked to see. The skill surfaces existing or just-produced content; it does not invent, scrape, or summarize new material unless that is the user's request.
- Strip or redact secrets from any content the skill opens or writes, including logs and assembled data.

## Required Behavior

### Trigger

Activate when the user's request contains any of these intents, explicitly or implicitly:

- "show me", "see", "view", "open", "look at", "present", "display", "render", "pull up", "let me see";
- a direct instruction to open a file;
- a follow-up like "nice html presentation of that", "in Safari", "in my editor".

Default to this skill for any such request instead of inlining content in chat.

### Routing rule

| Content                                                                                     | Surface         |
|---------------------------------------------------------------------------------------------|-----------------|
| File on disk with extension `.py .js .ts .tsx .jsx .go .rs .swift .kt .java .c .cpp .h .sh .zsh .bash .rb .php .lua .toml .yaml .yml .json .xml .sql .html .css .scss .less .conf .ini .env.example .dockerfile .makefile` | Text editor     |
| File on disk with extension `.md .mdx .txt .log .rst .org .tex`                             | Text editor     |
| Any file the user names that already exists on disk with one of the above extensions        | Text editor     |
| Structured data pulled from a database, API, agent, CLI tool                                | HTML → browser  |
| A summary, digest, research finding, report, recommendation, meeting recap                  | HTML → browser  |
| Tables, lists, comparisons, timelines, dashboards                                           | HTML → browser  |
| Anything the agent itself just assembled in this turn                                       | HTML → browser  |
| Ambiguous or mixed                                                                          | HTML → browser  |

If the user's intent is to review raw source, route to editor even if the extension is missing. If the user's intent is to "see" structured information presented well, route to HTML even if it originated from a text file.

### Path 1 — Text editor

Open the file in a text editor via the operating system's file-open mechanism. Preferred editors, in order of user preference if known, otherwise in order of availability:

1. The user's configured editor, if the skill has access to that preference (e.g. CotEditor, Sublime Text, VS Code, BBEdit, Zed, Kate, Gedit).
2. The system default for the file's extension.
3. A generic fallback such as `xdg-open` on Linux or `open` on macOS.

Examples:

- macOS, preferred editor CotEditor: `open -a CotEditor /absolute/path/to/file`
- macOS, default: `open /absolute/path/to/file`
- Linux: `xdg-open /absolute/path/to/file`
- Windows: `start "" "C:\\absolute\\path\\to\\file"`

Multiple files: repeat for each so the editor opens each in a tab or window.

Do NOT also copy the file's content into the chat response.

### Path 2 — HTML presentation in a browser

1. Choose a slug derived from the topic (e.g. `night-watch-digest`, `q1-revenue-report`, `peggy-diary-recap`).
2. Write the HTML to a fresh path under the system temp directory:
   - macOS/Linux: `/tmp/<slug>-YYYY-MM-DD.html` (append a short hash or time if the same slug is reused).
   - Windows: `%TEMP%\\<slug>-YYYY-MM-DD.html`.
3. Open the file in the browser:
   - macOS, preferred browser: `open -a Safari /tmp/<slug>-YYYY-MM-DD.html` (or `open -a "Google Chrome"`, `open -a Firefox`).
   - macOS, default: `open /tmp/<slug>-YYYY-MM-DD.html`.
   - Linux: `xdg-open /tmp/<slug>-YYYY-MM-DD.html`.
   - Windows: `start "" "%TEMP%\\<slug>-YYYY-MM-DD.html"`.
4. In the chat, return a one-line summary of what was produced and the path. Do not dump the full content.

## HTML Template

Reuse this template verbatim across presentations so they feel like one visual family. Swap the body content only; keep the `<style>` block identical.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>TITLE HERE</title>
<style>
  :root {
    --bg: #0b0e12; --panel: #11161d; --panel-2: #161d27;
    --ink: #e6edf3; --muted: #8b96a5;
    --accent: #7cc4ff; --accent-2: #ffb86b;
    --good: #7bd88f; --warn: #ffb86b; --bad: #ff6e6e;
    --rule: #1f2731; --code: #0d1218;
  }
  * { box-sizing: border-box; }
  html, body { margin: 0; padding: 0; background: var(--bg); color: var(--ink);
    font-family: -apple-system, BlinkMacSystemFont, "SF Pro Text", "Helvetica Neue", Arial, sans-serif;
    font-size: 15px; line-height: 1.55; -webkit-font-smoothing: antialiased; }
  a { color: var(--accent); text-decoration: none; word-break: break-all; }
  a:hover { text-decoration: underline; }
  code { background: var(--code); padding: 1px 6px; border-radius: 4px;
    font-family: "SF Mono", Menlo, Consolas, monospace; font-size: 0.9em; color: #ffd08a; }
  .wrap { max-width: 960px; margin: 0 auto; padding: 48px 28px 80px; }
  header { border-bottom: 1px solid var(--rule); padding-bottom: 20px; margin-bottom: 32px; }
  .kicker { color: var(--muted); font-size: 12px; letter-spacing: 2px; text-transform: uppercase; }
  h1 { font-size: 30px; margin: 6px 0 4px; font-weight: 650; letter-spacing: -0.3px; }
  .sub { color: var(--muted); font-size: 14px; }
  .card { background: var(--panel); border: 1px solid var(--rule); border-radius: 10px;
    padding: 22px 26px; margin: 22px 0; }
  .card h2 { font-size: 19px; margin: 0 0 2px; font-weight: 600; letter-spacing: -0.2px; }
  .card h3 { font-size: 14px; letter-spacing: 1.2px; text-transform: uppercase;
    color: var(--muted); margin: 24px 0 10px; font-weight: 600; }
  .tag { display: inline-block; font-size: 10.5px; letter-spacing: 1.2px; text-transform: uppercase;
    padding: 3px 8px; border-radius: 999px; margin-right: 8px; vertical-align: middle;
    background: rgba(124, 196, 255, 0.13); color: var(--accent); }
  .tag.alt { background: rgba(255, 184, 107, 0.13); color: var(--accent-2); }
  .signal { background: var(--panel-2); border-left: 3px solid var(--accent);
    padding: 14px 18px; border-radius: 6px; margin: 10px 0 18px; }
  .card.alt .signal { border-left-color: var(--accent-2); }
  ul { margin: 8px 0 0; padding-left: 22px; } li { margin: 8px 0; }
  .src { color: var(--muted); font-size: 12.5px; display: block; margin-top: 4px; }
  footer { color: var(--muted); font-size: 12.5px; margin-top: 40px; text-align: center;
    border-top: 1px solid var(--rule); padding-top: 20px; }
  hr.sep { border: none; border-top: 1px solid var(--rule); margin: 28px 0; }
  table { border-collapse: collapse; width: 100%; margin: 10px 0; font-size: 14px; }
  th, td { border-bottom: 1px solid var(--rule); padding: 8px 10px; text-align: left; }
  th { color: var(--muted); font-weight: 600; font-size: 12px; letter-spacing: 1px; text-transform: uppercase; }
  .good { color: var(--good); } .warn { color: var(--warn); } .bad { color: var(--bad); }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <div class="kicker">KICKER</div>
    <h1>TITLE</h1>
    <div class="sub">SUBTITLE / CONTEXT</div>
  </header>

  <section class="card">
    <div><span class="tag">CATEGORY</span></div>
    <h2>Section heading</h2>
    <div class="signal">Executive signal or top takeaway in one or two sentences.</div>
    <h3>Findings</h3>
    <ul>
      <li>Point 1 <span class="src">source or citation</span></li>
    </ul>
    <h3>Actions</h3>
    <ul>
      <li><b>Owner:</b> Action.</li>
    </ul>
  </section>

  <footer>Rendered YYYY-MM-DD</footer>
</div>
</body>
</html>
```

### Variants

- **Two-color category cards**: use `.card` plus `.card.alt` with `<span class="tag alt">` to alternate accent colors for mixed-domain reports.
- **Tabular data**: use `<table>`; styles are already included.
- **Status rows**: use `<span class="good">`, `<span class="warn">`, `<span class="bad">` inline.
- **Long transcripts**: wrap alternating speakers in `.card` blocks with the speaker as the tag.
- **Dashboards**: repeat `.card` blocks; use `hr.sep` between sections.

Keep the `<style>` block unchanged unless the user explicitly asks for a different look. Consistency across presentations is a feature.

## Output Conventions

- Always write to a temp directory unless persistence is requested.
- Name files `<slug>-YYYY-MM-DD.html` where `slug` is derived from the topic.
- If the same slug was already used today, append a short hash or HH-MM so old renders are not overwritten.
- When the content is regenerated from a live source, include a small footer line stating when it was rendered and where the data came from.

## Redaction

Before writing HTML or opening any file, strip or mask:

- private key blocks;
- environment variable values whose names include `key`, `token`, `secret`, `password`, or `credential`;
- common API key, access token, session token, SSH key, cloud credential, and OAuth token formats;
- URLs containing embedded credentials;
- personal data the user has asked not to see in reports.

Use `[REDACTED]` or a similarly obvious placeholder. Redaction applies to both editor-opened files (if the skill is generating them) and rendered HTML.

## What NOT to Do

- Do not paste the full rendered content into the chat response. The whole point is to move it out of the terminal.
- Do not invent a new stylesheet per invocation. Reuse the template so presentations stay in one visual family.
- Do not ask the user which surface to use.
- Do not write Markdown files "as a compromise" between editor and browser. The routing is binary.
- Do not render HTML for content whose primary use is in a code editor (e.g. a single `.py` file the user wants to edit).
- Do not open files the user did not ask to see.

## Example Invocations

- "show me the logs from last night" → gather logs, render HTML with expandable sections, open in browser.
- "open founder-tools/emma-tet-ai/server.py" → open in text editor.
- "let me see what the research agent found" → render HTML with one card per finding, open in browser.
- "show me the diff" → open the diff file in the text editor, or render a colorized HTML if the diff is multi-file and large.
- "present the Q1 numbers" → render HTML table in browser.
- "view my CLAUDE.md" → open in text editor.
- "can you see what's in the quarterly recap?" → if it is already a markdown file, open in editor; if the agent just assembled the recap, render HTML.

## Installation Recipe (implementation-agnostic)

An LLM installing this skill on a fresh system should:

1. Create a skill or prompt entry that the harness surfaces when the user
   requests content. Use the equivalent skill, prompt, tool, or agent-config
   location for that harness.
2. Ship the routing rule, the HTML template, and the open-command selection in that skill file so the agent can act on it without additional lookups.
3. Detect the user's preferred editor and browser if that information is available (environment variables, known user preferences, or a simple config file) and record the choice. Fall back to the OS default when unknown.
4. Optionally include a tiny shell wrapper that an LLM can invoke instead of composing the `open` command each time:
   - `show-content edit <path>` → open in the preferred editor.
   - `show-content html <path>` → open in the preferred browser.
5. Document the skill's trigger patterns, routing rule, output conventions, and redaction requirements in the skill file so every agent behaves identically.
6. If the system has organization-wide LLM instruction files, add a short
   reference pointing to this skill so agents discover it without being told.

## Optional Backward-Compatibility Notes

- The skill was first published under the alias `show-content`. Implementations may keep that alias as an internal name. The canonical generalized name is `content-presentation`.
- On systems where `/tmp` is cleared frequently, persist renders under the user's cache directory instead (`~/.cache/content-presentation/` or equivalent) if the user expects to re-open the artifact later.

## Smoke Test

The skill should support a deterministic smoke test:

- input: a known fixture file (for editor path) and a known fixture HTML payload (for browser path);
- output: a confirmed `open` call was issued for the correct path with the correct application, and the HTML file exists, is valid, and uses the template's `<style>` block unchanged.

The smoke test must not require network access. It verifies routing, template integrity, and file-path handling only.
