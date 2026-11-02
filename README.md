# Can Claude generate images, and how do I set that up?

Not by itself, but yes once you connect an image model over MCP. Add the Wireflow MCP server (`https://www.wireflow.ai/api/mcp`) as a custom connector in Claude, or run `claude mcp add` in Claude Code, sign in with your Wireflow account and ask for the picture in plain words. There is no API key to copy.

Wireflow is our product: this repo is made by the Wireflow team, so read [Other ways to do this](#other-ways-to-do-this) with that in mind. Every other option links to its own vendor page.

![A white ceramic mug on a pale oak table by a rainy window, made with Flux Schnell by this repo's workflow](examples/mug-flux-schnell.jpg)

Made by this repo's workflow and script on 3 Nov 2026: Flux Schnell, 1 credit, run `aa470946-a651-4d97-90f6-ea0215e82936`.

## Set it up

The steps below are copied from the Wireflow docs pages linked under each host, read on 3 Nov 2026. Every host uses the same server URL and the same Wireflow sign-in (OAuth), so there is no API key in any config file.

### Claude app (Claude, Cowork and Claude Desktop)

From [How do I use Wireflow in Claude to make images and videos?](https://www.wireflow.ai/docs/agents/claude-app) (page updated 26 Oct 2026):

1. Open **Customize > Connectors**.
2. Click **+ Add**, then **Add custom connector**.
3. Enter a name, such as `Wireflow`.
4. Add `https://www.wireflow.ai/api/mcp` as the remote MCP server URL, then click **Continue**.
5. Review the authentication settings Claude detected and click **Continue**. Add no API key under **Request headers**: Wireflow signs you in with OAuth.
6. Click **Add**, then sign in with your Wireflow account and approve the consent screen.

Shortcut: [Add Wireflow to Claude](https://claude.ai/customize/connectors?modal=add-custom-connector&connectorName=Wireflow&connectorUrl=https%3A%2F%2Fwww.wireflow.ai%2Fapi%2Fmcp) opens the dialog with the name and URL filled in. Custom connectors work on Claude's Free, Pro, Max, Team and Enterprise plans; on Team and Enterprise an Owner adds Wireflow once under **Organization settings > Connectors**. To let Claude use your own photos, also tick `workflows:write` on the consent screen (it also lets Claude create, edit and delete workflows).

Check it: in a chat, click **+**, choose **Connectors**, turn on Wireflow and paste `Use Wireflow. Tell me which Wireflow account you are connected as, and my credit balance and plan.` Claude should call `whoami` and `get_credit_balance`, which spend no credits.

### Claude Code

From [How do I set up an MCP server in Claude Code?](https://www.wireflow.ai/docs/generate-videos-in-claude-code) (page updated 29 Oct 2026):

1. Run `claude mcp add --transport http wireflow https://www.wireflow.ai/api/mcp` (add `--scope user` to load it in every project).
2. Inside Claude Code, run `/mcp`, choose `wireflow` and sign in with your Wireflow account in the browser.
3. Run `claude mcp list` and check that `wireflow` shows Connected.

Or use this repo's [`.mcp.json`](.mcp.json), the project-scope file: open the folder in Claude Code, approve the `wireflow` server when asked, then run `/mcp` to sign in. If you log in to Claude Code with a Claude.ai account that already has the Wireflow connector, it is there already.

### Cursor

From [How do I add an image and video generation MCP server to Cursor?](https://www.wireflow.ai/docs/generate-images-and-videos-in-cursor) (page updated 24 Oct 2026):

1. Put the `wireflow` entry under `mcpServers` in `~/.cursor/mcp.json` (every project) or `.cursor/mcp.json` (one project). This repo ships [`.cursor/mcp.json`](.cursor/mcp.json).
2. Open **Customize** in the Cursor sidebar and check that the server's toggle is on.
3. When Cursor opens the Wireflow sign-in, sign in and approve the consent screen.

Cursor asks before it uses an MCP tool. In Auto-review mode, keep the tools that spend credits (`run_workflow`, `run_app`) off the allowlist.

### Codex

From [How do I add an MCP server to Codex CLI?](https://www.wireflow.ai/docs/generate-images-and-videos-with-codex) (page updated 24 Oct 2026):

1. Run `codex mcp add wireflow --url https://www.wireflow.ai/api/mcp`.
2. Run `codex mcp login wireflow` and sign in with your Wireflow account.
3. Run `codex mcp list` to check, or type `/mcp` inside Codex.

Or copy [`.codex/config.toml`](.codex/config.toml): Codex reads a project's config in trusted projects only, and the same table works in `~/.codex/config.toml`. Its `default_tools_approval_mode = "writes"` line makes Codex ask before any tool that is not read-only, such as `run_workflow`.

Something not connecting? See [Why won't my AI agent connect to the Wireflow MCP server?](https://www.wireflow.ai/docs/agents/fix-mcp-connection-problems)

## Run it

### In the chat

Paste a prompt that names the job, asks for the cost and waits for your yes. This one is from the Claude app docs:

```text
Use Wireflow to make an image of a stack of blueberry pancakes on a white plate in soft morning light. Use the newest image model from list_models, tell me which model it is and its credit cost, and wait for my yes before you run it. When it finishes, give me the image link.
```

In Claude Code or Cursor, add where the file should go, for example `then save it to public/images/pancakes.png`: both run terminal commands, so they can download the file into your project. Claude looks up models and prices with `list_models` and saves a private draft with `create_workflow_from_prompt`, which spend nothing; `run_workflow` spends credits; `get_execution` returns the image links.

Want a ready-made skill instead? [ai-image-generation](https://github.com/wireflowINC/skills/tree/main/skills/ai-image-generation) in `wireflowINC/skills` writes the prompt, picks the aspect ratio and quotes before it runs. Install it with `npx skills add wireflowINC/skills --skill ai-image-generation`. Its app uses Flux Pro Ultra, which the [model list](https://www.wireflow.ai/api/v1/models) prices at 19 credits an image. For a list of prompts, use [bulk-image-generation](https://github.com/wireflowINC/skills/tree/main/skills/bulk-image-generation).

### From code (build scripts, many images, your own app)

The chat route needs no key. For a script, create an API key in Wireflow under **Settings > API Keys** with the `workflows:write` scope (to create the workflow) plus `workflows:execute` and `executions:read` (to run it and read the result), or **Full access**. Node 18 or newer, no packages.

```bash
export WIREFLOW_API_KEY="your key"          # PowerShell: $env:WIREFLOW_API_KEY = "your key"

# 1. Save this repo's workflow (one Text Input wired into Flux Schnell) as a private workflow in your account. Free.
node scripts/create-workflow.mjs workflows/text-to-image-flux-schnell.json

# 2. Run it with your prompt. It waits for the run and saves the image to ./wireflow-outputs/<date>/<run id>/
node scripts/wireflow-run.mjs --workflow <your workflow id> --input prompt="a ceramic mug on a pale oak table by a rainy window"
```

`--describe` lists a workflow's inputs without spending anything, `--app <slug>` runs an app you published instead, and `--poll <run id>` picks up a run that is still going. The script uses `POST /api/v1/workflows/{id}/run` and its poll route ([Run Workflows](https://www.wireflow.ai/docs/api/run), [run a workflow from your app](https://www.wireflow.ai/docs/trigger-a-workflow-from-your-app)). Swap the model on the canvas or in the JSON file and the script stays the same.

What it costs: Flux Schnell is listed at 1 credit per megapixel in the [model list](https://www.wireflow.ai/api/v1/models) (read 3 Nov 2026), and our 1024x768 test was charged 1 credit. API and MCP access are on every Wireflow plan and runs use credits; see [pricing](https://www.wireflow.ai/pricing).

## What's in this repo

| File | What it is |
|---|---|
| [`.mcp.json`](.mcp.json) | Claude Code project config for the Wireflow MCP server (`type: http`) |
| [`.cursor/mcp.json`](.cursor/mcp.json) | Cursor project config |
| [`.codex/config.toml`](.codex/config.toml) | Codex config table, asks before tools that write |
| [`workflows/text-to-image-flux-schnell.json`](workflows/text-to-image-flux-schnell.json) | The tested workflow: Prompt into Flux Schnell |
| [`scripts/create-workflow.mjs`](scripts/create-workflow.mjs) | Lints the workflow file, then saves it in your account and prints its id |
| [`scripts/wireflow-run.mjs`](scripts/wireflow-run.mjs) | Runs your workflow or app, waits, downloads the files, writes `manifest.json` |

## Other ways to do this

Facts below are from each vendor's own pages, as recorded in our tool fact files on the dates shown.

- **[Adobe for Claude](https://www.adobe.com/adobe-connectors/adobe-for-claude.html)**: brings Adobe Express, Photoshop, Firefly and other Adobe tools into a Claude chat, on web, desktop and Claude Code; some tools need an eligible paid Adobe plan (Adobe's page, checked 11 Oct 2026).
- **[Canva connector](https://claude.com/marketplace/connectors/canva)**: listed in Anthropic's connector directory; Claude can search, generate, autofill and export Canva designs, and some actions such as Brand Template search need a paid Canva plan (directory listing and Canva pricing, checked 11 Oct 2026).
- **[fal MCP](https://fal.ai/docs/documentation/setting-up/mcp)**: a remote Streamable HTTP server with OAuth that lets an assistant find models, check pricing, upload inputs and run generations across fal's 1,000+ models, paid per use (fal docs and pricing, checked 11 Oct 2026).
- **[Replicate MCP](https://replicate.com/docs/reference/mcp)**: a remote server at mcp.replicate.com or the replicate-mcp npm package, covering all operations in Replicate's HTTP API; models are billed per output or per second of hardware (Replicate docs and pricing, checked 11 Oct 2026).
- **[Black Forest Labs FLUX MCP](https://docs.bfl.ai/api_integration/mcp_integration)**: generates and edits images and video with FLUX models from any MCP client, with OAuth sign-in; you add credits before the first call (BFL docs, checked 11 Oct 2026).
- **[ChatGPT](https://chatgpt.com/)**: if you don't need Claude, ChatGPT creates images from a prompt in the chat, with limited image generation on the Free plan (ChatGPT pricing page, checked 12 Oct 2026).

## Tested

| | |
|---|---|
| Date | 3 Nov 2026 (2026-11-02 22:32 UTC) |
| Host | Wireflow REST API, called by `scripts/wireflow-run.mjs` on Node 24, Windows 11, after `scripts/create-workflow.mjs` saved the workflow |
| Model | Flux Schnell, one 1024x768 JPEG |
| Run id | `aa470946-a651-4d97-90f6-ea0215e82936` |
| Credits | 1, from the account's credit history |

The MCP setup steps are copied from the Wireflow docs on 3 Nov 2026; the test did not sign in over MCP. We re-check the steps, the model list and this block once a month.

## License

[MIT](LICENSE). The example image was made by the workflow in this repo.
