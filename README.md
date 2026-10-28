# deployai

The DeployAI command line tool. It checks a site built with an AI editor, explains what has to change, and publishes it to DeployAI.

The output is short and structured for your AI agent. The results you may read (check, deploy, status and the rest) are in your language when you start from the DeployAI site, or with `--lang he|en`; the instructions for the agent stay English.

Website: https://deployai.co.il. For developers: https://deployai.co.il/developers.

## Quick start

The easiest start is on https://deployai.co.il:
1. Choose a sample or "Go live now".
2. Copy the one line it gives you into an AI agent that can run commands in your folder (Claude Code, Cursor, GitHub Copilot in VS Code, Codex in the ChatGPT app, or Google Antigravity).

The line is `npx deployai@latest start <id>`. It carries every choice you made there, prepares the folder, and prints the steps for the agent.

Already have a site in this folder? Ask your agent:

```
Run npx deployai@latest check and fix what it lists, then npx deployai@latest deploy --guest. Explain every step to me in simple words and ask me before anything goes live.
```

If `npx` is missing, install Node.js LTS from https://nodejs.org first.

## Install

```
npx deployai check
# or
pnpm dlx deployai check
# or install it once
npm i -g deployai
```

Node 20 or newer. No other setup: the CLI talks to https://deployai.co.il.

## Commands

| Command | What it does |
|---------|--------------|
| `deployai init` | Creates `DEPLOYAI.md` (rules for the AI agent: the stack to build with, what to avoid, how to keep existing data), adds one line pointing to it in `AGENTS.md` and `CLAUDE.md`, and `.deployai/project.json`. Run it first in a new project, even an empty folder. Never edits your code and does not link a project (use `link`). `--force` overwrites `DEPLOYAI.md`, `--no-agent-files` leaves `AGENTS.md` and `CLAUDE.md` alone. |
| `deployai inspect` | Shows framework, package manager, language, database, storage and the number of env files (values are never read). `--json` for machine output. |
| `deployai check` | Lists blockers and warnings with file and line. It answers only whether the project can run on DeployAI. More below. |
| `deployai login` | Logs in through the browser (device code). The token lasts 30 days and is stored in `~/.deployai/config.json` with the API address that issued it; it is never sent to another address. |
| `deployai logout` | Cancels the token on DeployAI and removes it from this computer. |
| `deployai whoami` | Shows the logged in user. |
| `deployai link [project]` | Links this folder to a project (slug, id or name). `--create "<name>"` makes a new project. With no name it never links on its own: in a terminal it asks first, an agent gets the exact `link <slug>` command. |
| `deployai deploy -m "<message>"` | Runs the check, packs the source and uploads it. Prints the version, the queue position and the preview address. |
| `deployai versions` | Lists versions: number, live or not, status ("replaced by a newer upload" for one that never built), date, size, message, and a dashboard link when its SQL waits for approval. |
| `deployai rollback v5` | Puts version 5 back live, with no rebuild. |
| `deployai promote v5` | Puts version 5 live. |
| `deployai status` | Site state, address, live version, the newest version when it is not live yet (with the promote command once it is ready), and queue. |
| `deployai deploy --guest` | No login needed: puts the site live as a temporary site with a claim link. More below. |
| `deployai pull` | Downloads the code of the linked site's live version into this empty folder. Needs login. |
| `deployai claim` | Prints this folder's temporary site keep link again. `--email <address>` mails it, with a reminder before the delete. |
| `deployai samples [name] [folder]` | Lists the six sample sites, or downloads one. More below. |
| `deployai start <id>` | The one line copied on the DeployAI site. Prepares this folder, then prints the steps for your agent. No login needed. More below. |
| `deployai speed <url>` | Google PageSpeed test of a page: lab results, real visitor data when Google has it and the biggest opportunities. `--desktop` tests a desktop load. |
| `deployai addons` | The marketplace add-ons of the linked site: which it has, which are done and which is next. Works once the DeployAI marketplace is open; the server checks that the site has each add-on. |
| `deployai addon next` | Prints the next add-on of the site for your agent. `deployai addon run <name>` prints one by name. |
| `deployai addon done <runId>` | Saves the report of a finished add-on (`--status done\|failed\|skipped`, `--report .deployai/addon-report.json`) and prints the next one. |
| `deployai telemetry status\|on\|off` | Shows exactly what is sent, or turns it on or off. |

Global options:
- `--api <url>` (or env `DEPLOYAI_API_URL`)
- `-C, --dir <path>` to run on another folder
- `--lang he|en` for the language of the results (or env `DEPLOYAI_LANG`; `start` saves the one the owner chose)

### Command details

**`deployai check`**
- Lists blockers and warnings with file and line, and writes `.deployai/last-check.json`.
- Prints a prompt for your AI editor and copies it to the clipboard.
- Exit code 1 when there are blockers.
- `--json` for machine output, `--no-clipboard` to skip the copy.
- It answers only whether the project can run on DeployAI.

**`deployai deploy --guest`**
- No login needed: puts the site live as a temporary site with a claim link, deleted after the days DeployAI sets unless you claim it.
- A temporary site may have a small database, but no file storage or environment variables.
- A folder linked to a site in an account refuses: run `login`, then `deploy`.
- For a database sample it prints the site's owner keys once ("shown once, save them now"). They are never stored, so save them right away.

**`deployai samples [name] [folder]`**
- Lists the six sample sites, or downloads one.
- Without a folder it uses this folder when empty. Otherwise it asks in a terminal (a new folder, this folder or cancel), and an agent gets a new folder named after the sample.
- A named folder must be new or empty (`--force` writes into one that has files).
- The next steps end with `deployai deploy --guest` when you are logged out, or `deployai deploy -m "first version"` when logged in.

**`deployai start <id>`**
- The one line copied on the DeployAI site.
- Prepares this folder with every choice made there: a new folder for a new site when this one has files, the sample and its install, the owner's details, the link to the site. Then prints the steps for your agent.
- No login needed.
- It saves the owner's language, so later results print in it (D-266; `--lang he|en` or `DEPLOYAI_LANG` override it).
- `--here` keeps this folder, `--no-install` skips the install.

## Help for every finding

Every blocker and warning has a code (for example `fs_write`) and a help page, which `check` prints:
- Hebrew: https://deployai.co.il/help/fs-write
- English: https://deployai.co.il/en/help/fs-write
- All codes: https://deployai.co.il/help

Each page says what the finding means, why it blocks or warns, how to fix it, and gives a prompt to copy into your agent.

## API address

The default is `https://deployai.co.il`. The order is: `--api <url>`, then env `DEPLOYAI_API_URL`, then the address saved at login, then the default. For local development of DeployAI itself, use `--api http://localhost:3000` or `DEPLOYAI_API_URL=http://localhost:3000`.

## What deploy uploads

The source folder, packed as a `.tar.gz`. The upload is capped at 50 MB.

Never uploaded:
- `node_modules`, `.git`, and tool caches like `.next`, `.astro`, `.svelte-kit`, `.wrangler`
- `dist`, `build`, `out` at the project root when `package.json` has a build script (DeployAI builds them again). Folders like `assets/build/` are uploaded.
- `.deployai/`, `DEPLOYAI.md`, `CLAUDE.md`, `AGENTS.md`, `.claude/`, `.cursor/`, `.vscode/` and other dotfiles (except `.well-known/` and build config like `.babelrc`). DeployAI never serves these either, however they arrive.
- Any `.env*` file
- Anything in the root `.gitignore` or in `.deployaiignore` (gitignore style, read last, so `!file` brings a file back)
- Files over 10 MB

## Telemetry

Sent when on:

- Install id (random, made on this computer) and CLI version
- After each command: command name and result (success, blocked, error category)
- Linked project id, detected framework and version
- After each check: compatible or not, blocker and warning codes (for example `fs_write`)
- OS, architecture and Node version (in the request header)
- Your user id, only after login

Never sent: source code or file contents, `.env` values or any secret, file paths or file names.

Turn it off with `deployai telemetry off`, or set `DEPLOYAI_TELEMETRY=0` or `DO_NOT_TRACK=1`. Off stops everything except the deploy API calls themselves.

## Development

```
pnpm --filter ./packages/cli test
pnpm --filter ./packages/cli typecheck
pnpm --filter ./packages/cli build
pnpm --filter ./packages/cli start -- check
```

- `@deployai/shared` and `zod` are bundled into `dist/index.js` by tsup, so the published package depends only on `commander`.
- The package ships only `dist` and this README (check with `npm pack --dry-run`).
- Publish from this folder with `pnpm publish`: `prepublishOnly` builds and runs the tests first.
- On npm as `deployai`: run it with `npx deployai@latest <command>`.
