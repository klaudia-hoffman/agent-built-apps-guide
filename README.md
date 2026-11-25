# Building apps with an AI agent: principles and guidelines

Nov 25, 2026 · @Klaudia Hoffman

These are the principles behind the talk *My plants took months to die. The app took an afternoon to grow*, written so you can use them with any coding agent.

The agent pipeline and dashboard from Part 4 are not included. They run many model calls and cloud deploys without anyone watching, and a misconfigured run can cost real money. Everything below works with one agent and you in the loop, which is the safer place to start.

## The principles

One rule drives the rest: a side project gets the minimum time, effort and money that still produces something good.

1. **Outsource your weak spots.** Hand off what you are not good at. For me that was design (Stitch) and sign-in (Google OAuth).
2. **Keep the decisions you are good at.** The agent types; you own the API design and the data model.
3. **Plan before code.** Ask for a task list first and approve it before any code gets written.
4. **Review evidence, not diffs.** The agent shows that the code runs: screenshots of the app and test output.
5. **Write the contract down.** The design system and the plan live in files the agent reads, not in your head or in chat history.
6. **Pick a boring stack.** One language, one container, few frameworks. Less for the agent to get wrong, and less for you to review.
7. **Ask models for strict JSON.** When your app calls a model, demand a fixed set of fields, and handle the "could not answer" case.
8. **No keys in code.** The app runs as a service account; the secrets it does need live in a secret manager.
9. **Pay nothing when idle.** Use hosting that scales to zero, so an app you open twice a week costs almost nothing.
10. **A human decides what ships.** Agents can build and test, but a person approves the deploy.
11. **The design is never frozen.** After every change, check the live app against the design again.

## Directing your agent

The order that worked for me: design first, then a written plan, then one feature at a time.

1. **Start from a design.** Describe the app in plain language in [Stitch](https://stitch.withgoogle.com), explore a few variants, then export DESIGN.md (or connect the agent to Stitch's MCP server so it reads the live design).
2. **Give the agent a project instructions file.** The file your tool reads on every session (rules, CLAUDE.md or AGENTS.md). Put in it the stack, how to run the app locally, how to test, and "verify changes by running the app, not just the tests".
3. **Ask for a plan and edit it.** Data model, API endpoints, one page per screen. Save it as a file in the project and correct it before anything is built.
4. **Build one feature at a time.** One feature per request, each ending with proof that it works.
5. **Send it back with one comment.** If the result is wrong, say what is wrong and ask it to re-plan, rather than patching it yourself.
6. **Deploy on your word only.** Deploying should be one command the agent runs when you say "deploy it", never something it decides to do.

If you later split the work across several agents, these rules kept mine predictable:

- Each agent has one job and may only write its own files.
- Everyone builds against the same written plan, so parallel agents never need to talk to each other.
- Fixes happen after the parallel work finishes, so there is never more than one writer on a file.
- A check confirms the frontend only calls endpoints that exist in the plan.

## Checking the agent's work

Don't accept "done" until the agent has run the app and shown you what it saw.

- **Run it locally.** The agent starts the app on your machine, the same way you would.
- **Look at it.** The agent opens each page, clicks through it and saves screenshots at phone and desktop sizes. Most agents can read those screenshots and compare them with the design. This needs a browser the agent can drive (see the table below).
- **Call every endpoint.** Including the unhappy paths: signed out, a missing item, a bad upload.
- **Click through it yourself before it ships.** The screenshots narrow down what to check; they don't replace you.

Screenshots only work if the agent has a browser tool. Without one it can still run your tests, but it can't see the app. Playwright is one option, not a requirement:

| Your setup | How the agent gets screenshots | What you set up |
| --- | --- | --- |
| Antigravity app, personal Google account | Its built-in browser agent | Nothing |
| Antigravity IDE, CLI, or business account | Its browser agent drives your Chrome | Chrome 144+, Node.js, and remote debugging on at chrome://inspect/#remote-debugging ([docs](https://antigravity.google/docs/browser)) |
| Any other coding agent (Claude Code, Cursor, Copilot…) | A browser MCP server | Add [Playwright MCP](https://github.com/microsoft/playwright-mcp) or Chrome DevTools MCP |
| No MCP at all | Plain headless Chrome from the command line | Chrome installed; the agent runs `chrome --headless --screenshot=page.png --window-size=390,844 <url>` |

The last row is what my pipeline's drift check uses: no Playwright, just Chrome. It takes static screenshots but can't click or fill forms.

A prompt you can adapt at the end of each feature:

> Start the app locally. Open every page in a headless browser at phone and desktop sizes and save screenshots. Compare them with DESIGN.md and list any differences. Call every API endpoint, including error cases. Report what failed, fix it, and try again up to three times, then stop and show me the screenshots and the test output.

## Keeping costs safe

Set the limits before you automate anything: agents that loop, retry or deploy on their own are where surprise bills come from. My plant app costs under a dollar a month, but only because of the limits below.

- **Budget alert first.** Create a billing budget with email alerts before your first deploy. Alerts warn you; they don't stop spending.
- **Cap every loop.** Fix-and-retry cycles get a fixed number of attempts, then stop and ask a human.
- **Cap model calls in your app.** A daily limit per user and a maximum output length on every call.
- **Start with a Flash-class model.** Move up to a bigger model only for the task that actually fails.
- **Scale to zero, with a ceiling.** Minimum instances 0, and a low maximum, so a traffic spike or a bug can't scale you up.
- **Restrict your keys.** Each API key only allows the APIs it needs, lives in a secret manager and never in the repository.
- **Don't leave agents running unattended,** and never let one deploy without a human approving it.
- **Check the billing report after the first week,** while a mistake is still small.

## A first project

Pick a problem only you have, small enough for one afternoon: one photo or text in, one model call, one list of saved results.

- [ ] Describe the app in Stitch and pick a variant you like
- [ ] Export DESIGN.md and write the project instructions file
- [ ] Ask the agent for a plan; fix the data model and API yourself
- [ ] Build the first screen and check it with screenshots
- [ ] Set a budget alert, then deploy with one command
- [ ] Use it for a week before adding the next feature
