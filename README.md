# deployai

The DeployAI command line tool. It checks a site built with an AI editor, explains what has to change, and publishes it to DeployAI.

The output is short and structured for your AI agent. The results you may read (check, deploy, status and the rest) are in your language when you start from the DeployAI site, or with `--lang he|en`; the instructions for the agent stay English.

Website: https://deployai.co.il. For developers: https://deployai.co.il/developers.

## Quick start

The easiest start is on https://deployai.co.il:
1. Choose a sample or "Go live now".
2. Copy the one line it gives you into an AI agent that can run commands in your folder (Claude Code, Cursor, GitHub Copilot in VS Code, Codex in the ChatGPT app, Google Antigravity, or Grok Build).

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

Node.js 22.12 or newer. No other setup: the CLI talks to https://deployai.co.il.

It stays up to date by itself. `npx deployai@latest` always runs the newest version. An installed copy checks npm at most once an hour, installs a newer version once into `~/.deployai/cli`, and runs every command on it. Set `DEPLOYAI_NO_UPDATE=1` to stay on the installed version; `CI` does the same.

## Commands

| Command | What it does |
|---------|--------------|
| `deployai init` | Creates `DEPLOYAI.md` (rules for the AI agent: the stack to build with, what to avoid, how to keep existing data), adds one line pointing to it in `AGENTS.md` and `CLAUDE.md`, and `.deployai/project.json`. Run it first in a new project, even an empty folder. Never edits your code and does not link a project (use `link`). `--force` overwrites `DEPLOYAI.md`, `--no-agent-files` leaves `AGENTS.md` and `CLAUDE.md` alone. |
| `deployai inspect` | Shows framework, package manager, language, database, storage and the number of env files (values are never read). `--json` for machine output. |
| `deployai check` | Lists blockers and warnings with file and line. It answers only whether the project can run on DeployAI. More below. |
| `deployai login` | Logs in through the browser (device code). The token lasts 30 days and is stored in `~/.deployai/config.json` with the API address that issued it; it is never sent to another address. |
| `deployai logout` | Cancels the token on DeployAI and removes it from this computer. |
| `deployai whoami` | Shows the logged in user. |
| `deployai hosts` | Lists the hosts the work in this folder needs and tries each from this machine: reachable, blocked by this machine or no answer. Inside a sandbox, ask whoever runs it to allow the blocked ones. |
| `deployai link [project]` | Links this folder to a project (slug, id or name). `--create "<name>"` makes a new project. With no name it never links on its own: in a terminal it asks first, an agent gets the exact `link <slug>` command. |
| `deployai deploy -m "<message>"` | Runs the check, packs the source and uploads it, then waits for the build. At a terminal it asks whether to put it live right away or show the preview first. `--live` puts it live after the build without asking (only on the owner's yes), `--no-wait` returns after the upload. |
| `deployai versions` | Lists versions: number, live or not, status ("replaced by a newer upload" for one that never built), date, size, message, and the `sql vN` command when its SQL waits for approval. |
| `deployai rollback v5` | Puts version 5 back live, with no rebuild. |
| `deployai promote v5` | Puts version 5 live. New database changes (SQL) are shown and run first, after a yes; SQL that removes data needs the database name typed. An agent passes `--approve-sql <code>` only after the owner said yes, and never for SQL that removes data. |
| `deployai sql v5` | The database changes version 5 runs before it goes live. |
| `deployai status` | Site state, address, live version, the newest version when it is not live yet (with the promote command once it is ready), and queue. When the live site is closed to the country you run this from, it says so and how to open it, or to use a VPN. |
| `deployai logs` | Live logs of the linked site's code: each request it answered, its console lines and errors, with the time, for up to 10 minutes (Ctrl+C stops). `--errors` keeps only errors, `--json` prints one JSON object a line. For the site's owner or an editor; the lines may hold your visitors' data, so they go only to your terminal and DeployAI never stores them. |
| `deployai deploy --guest` | No login needed: puts the site live as a temporary site with a claim link. More below. |
| `deployai pull` | Downloads the code of the linked site's live version into this empty folder. Needs login. |
| `deployai pull --history` | The same, with the site's history as git commits, one per version up to the live one, oldest first. |
| `deployai migrate scan <url>` | Scans the old site you are moving: every page's address, title and description, the platform, and on WordPress its posts as Markdown. Also the kind of site (content, store, bookings, members) and its features, with what the owner must connect before going live. Up to 300 pages, `--max <n>` up to 1,000. Writes into `.deployai/migration/<host>/` only. |
| `deployai migrate media` | Downloads the old site's files at their old paths, resumable (25 MB a file, 1 GB a move). |
| `deployai migrate pages` | Lists every page the move keeps in the dashboard's Pages, at its old address, with the old menu. |
| `deployai migrate legal` | Starts the dashboard's Documents and Accessibility pages from the scan: what to look at, never marked done. Asks nothing. |
| `deployai migrate redirects` | Writes `public/_redirects` only for old addresses that cannot stay. |
| `deployai migrate verify <preview>` | Checks every old address on the preview: moved, missing, an error, or a lost title or description. Also checks each feature the scan found: the store's add to cart, the booking form, the sign in form, the forms, and that a members only page shows nothing to a visitor who is not signed in. Then it measures each page shot into `.deployai/migration/before`, as `migrate compare` does. |
| `deployai migrate compare <preview>` | Shoots each page of `.deployai/migration/before` on the preview and compares it pixel by pixel at both widths, leaving out what moves by itself: the share changed, pass or fail (past 1%, or `--max-diff <percent>`), and a diff picture. `--json` prints the result. |
| `deployai migrate status <stage> [note]` | For a managed move: reports the stage (and a note like the preview address) to the request, so the board and the owner's status page update. |
| `deployai brand favicon\|icons\|images` | The site's icons and share image, made by code from the site's own name, colors and logo: `favicon` gives a site with none its old icon, its logo or a letter in its color; `icons` makes every size; `images` measures every image against where it shows and changes nothing. |
| `deployai blocks [inbox\|blog]` | Adds one of DeployAI's building blocks to an existing site, never over a file, with a guide in `.deployai/blocks/<key>.md`. With no key it lists them. |
| `deployai claim` | Prints this folder's temporary site keep link again. `--email <address>` mails it, with a reminder before the delete. |
| `deployai samples [name] [folder]` | Lists the six sample sites, or downloads one. More below. |
| `deployai start <id>` | The one line copied on the DeployAI site. Prepares this folder, then prints the steps for your agent. A site's line logs in first; a temporary site needs no login. More below. |
| `deployai speed <url>` | Google PageSpeed test of a page: lab results, real visitor data when Google has it and the biggest opportunities. `--desktop` tests a desktop load. |
| `deployai shot <url>` | Phone (390) and desktop (1440) pictures of a page, full page, with the Chrome or Edge already on the computer. `--webkit` uses Safari's engine. `--again` takes each picture twice, 4 seconds apart, for `migrate compare`. |
| `deployai optimize [folder]` | AVIF and WebP next to every JPG and PNG under public/, plus a phone width for wide images. Originals never change. |
| `deployai addons` | The marketplace add-ons of the linked site: which it has, which are done and which is next. The server checks that the site has each add-on. It runs what the site has even before the marketplace opens. |
| `deployai addon next` | Prints the next add-on of the site for your agent. `deployai addon run <name>` prints one by name. |
| `deployai addons run` | Runs the add-ons picked for the site one after another, with no question between them and one publish at the end. With more than one picked and no order yet, prints what is known of each, and the agent comes back with `--order`. `--parallel` (On my own only) runs them in waves, each lane in its own git worktree next to the folder. `--end preview`, `--end live` or `--end none` answers the question at the end ahead, for a run nobody watches; with `CI` set and no `--end`, it ends with a preview. |
| `deployai addons merge` | Brings the lanes of a wave back into the folder, in order, and removes the lane folders. Stops on a conflict and lists the files. |
| `deployai addon done <runId>` | Saves the report of a finished add-on (`--status done\|failed\|skipped`, `--report .deployai/addon-report.json`) and prints the next one. |
| `deployai telemetry status\|on\|off` | Shows exactly what is sent, or turns it on or off. |
| `deployai sound on\|off\|status\|test` | A sound on this computer when your site is ready. Off until you say yes. |
| `deployai sound agent on\|off` | Claude Code only so far: a dot in the tab title and a sound when it finishes or asks you, in this folder only. `sound on` already does this inside Claude Code. Sends nothing and never reads your prompts. |

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
- A line for a site opens only for its owner or an editor, so it logs in first if needed. A temporary site's line needs no login.
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

With env `DEPLOYAI_TOKEN` set, every command uses that token instead of the login saved on this computer, with no browser login and no question at the terminal. It is never saved, and it is sent only to `DEPLOYAI_API_URL` or the default address, never to a `--api` address.

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
- After each command: command name and result (success, blocked, error category). A start that has to wait for a login sends one more, before it waits
- Linked project id, detected framework and version
- After each check: compatible or not, blocker and warning codes (for example `fs_write`)
- OS, architecture and Node version (in the request header)
- Your user id, only after login
- With each deploy, which AI tool ran it (for example Claude Code or Codex), when the environment says so

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
