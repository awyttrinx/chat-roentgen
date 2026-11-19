<!-- Owner: @awyttrinx (documentation). Last checked against the code: 19 Nov 2026. -->

# Contributing to tea

This guide is for developers who want to fix a bug, improve the docs, or add a feature to tea. It takes you from your first idea to a merged pull request.

Two things to know before you start:

- tea makes a privacy promise: the chat stays in the browser, and only a pseudonymized sample leaves it, after the user agrees. The [privacy rules](#privacy-rules) below protect that promise. A change that breaks one of them will not be merged.
- For how the app works, read [Concept.md](Concept.md). For setup, follow [Getting started](getting-started.md).

## Contents

1. [Ways to contribute](#ways-to-contribute)
2. [Set up](#set-up)
3. [Where the code lives](#where-the-code-lives)
4. [Privacy rules](#privacy-rules)
5. [Code style](#code-style)
6. [Branches and commits](#branches-and-commits)
7. [Open a pull request](#open-a-pull-request)
8. [Who owns what](#who-owns-what)

## Ways to contribute

| You want to | Do this |
|---|---|
| Report a bug | Open an [issue](https://github.com/awyttrinx/chat-roentgen/issues) with the steps to reproduce it, what you expected, what happened, and your browser and OS. **Never paste real chat content.** If the date format matters, remove the names and paste only those lines. |
| Suggest a feature | Open an issue. Describe the user problem first and your idea second. |
| Fix the docs | Edit the file in `docs/` and open a pull request. Small fixes like typos and broken links are always welcome. |
| Change the code | Open an issue first if the change is bigger than a small fix, so we can agree on the approach before you write code. |

## Set up

Follow [Getting started](getting-started.md). In short:

```bash
git clone https://github.com/awyttrinx/chat-roentgen.git
cd chat-roentgen/roentgen
npm install
cp .env.example .env
npm run dev
```

The app runs at `http://localhost:5173`. By default it runs in **fixture mode**: the AI reads load sample results from `public/fixtures/`, so you need no API key. Most contributions can be built and tested in fixture mode.

The commands you will use:

| Command | What it does |
|---|---|
| `npm run dev` | Starts the Vite dev server with hot reload |
| `npm run lint` | Runs ESLint over the project |
| `npm run build` | Runs a strict TypeScript check and builds the app into `dist/` |
| `npm run preview` | Serves the built app locally |

## Where the code lives

```
roentgen/
├── api/              Serverless functions: the AI proxy, checkout, Stripe webhook
├── public/fixtures/  Sample AI results for fixture mode
├── server/           Local dev proxy for the api/ functions
└── src/
    ├── parser/       WhatsApp export → parsed chat
    ├── analysis/     Hard Facts and their text templates
    ├── ai/           Evidence, pseudonymization, prompts, analyzer
    ├── auth/         Supabase session
    ├── credits/      Credit balance and Stripe client
    ├── store/        Chats and results saved in the browser
    └── components/   Screens, charts, modals
```

Where a new piece of code goes:

- A new statistic → `src/analysis/hardFacts.ts`
- A new chart → `src/components/charts/`
- A new screen → a component in `src/components/` and a new `Stage` in `App.tsx`
- A parser change → `src/parser/whatsapp.ts`, plus an example in the pull request that shows the new behavior
- A new AI read → a prompt in `src/ai/prompts.ts`, an orchestrator next to `profile.ts`, and a fixture in `public/fixtures/`. Open an issue first, this is a big change.

## Privacy rules

These rules are the product, not a style choice.

1. **Never log chat content.** Not in `console.log`, not on the server, not in analytics, not in error messages. Describe the type of error, not the data.
2. **Never send raw chat text anywhere.** The only data that leaves the browser is the pseudonymized evidence packet, and only after the user agrees on the consent screen.
3. **Never send real names to the model.** Run `pseudonymizeDeep` on everything that goes to `api/analyze.ts` or `api/analyze-relationship.ts`, and `restoreNamesDeep` on everything that comes back.
4. **Never store chats on the server.** Supabase holds accounts and credits only. Chats stay in the user's browser.
5. **Never skip the consent screen.** It is the user's last chance to say no.

If your feature seems to need one of these rules broken, open an issue before you write code. There is usually a way to keep the rule.

## Code style

- **TypeScript is strict.** Avoid `any`. Use `unknown` and narrow it. Use `import type` for type imports.
- **Prefer pure functions.** The parser, the Hard Facts and the pseudonymizer are pure. Keep them that way.
- **React:** function components only, Tailwind classes instead of new CSS files.
- **Text:** every new user-facing string goes into `src/i18n.ts` in English and German.
- **Comments say why, not what.** The code already says what it does. A good comment explains a decision, a rule, or something that would surprise the next reader. For good examples, see `src/ai/pseudonymize.ts` and `api/analyze.ts`.

## Branches and commits

- `main` must always build and run. Do not commit to it directly.
- Name your branch after the type of change: `feat/<topic>`, `fix/<topic>` or `docs/<topic>`.
- Write commit messages in the imperative and keep each commit to one change, for example `fix(parser): handle narrow no-break spaces in AM/PM exports`.

```bash
git checkout -b fix/am-pm-parsing
# make your change, then commit
git fetch origin
git rebase origin/main
git push -u origin fix/am-pm-parsing
```

Then open the pull request on GitHub.

## Open a pull request

Check these before you ask for a review:

- [ ] `npm run lint` passes
- [ ] `npm run build` passes
- [ ] The change works in fixture mode, without an API key
- [ ] All five [privacy rules](#privacy-rules) still hold
- [ ] No real chat content in the code, the commits or the pull request
- [ ] If you changed `src/ai/`: names are replaced and restored correctly, also for names with two parts like `Max Müller`
- [ ] New text exists in English and German
- [ ] The description says what changed and why, with a link to the issue if there is one

The reviewer checks the privacy rules first, then whether the change works in fixture mode, then the code style.

## Who owns what

| Area | Owner |
|---|---|
| Documentation in `docs/` | [@awyttrinx](https://github.com/awyttrinx) |
| App code in `roentgen/` | [@Nils43](https://github.com/Nils43) |

If you are not sure who to ask, open an issue and tag the owner of the area.
