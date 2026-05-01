🧩 Repository— solana-developer-onboarding-bot

1. Repository Overview

Name: solana-developer-onboarding-bot  
Purpose:  
A GitHub App that automates onboarding for new Solana developers by creating starter issues, assigning tasks, generating repo setup steps, granting permissions, and sending welcome messages.

This bot becomes your DevRel assistant inside GitHub.

---

2. Folder Structure

`text
solana-developer-onboarding-bot/
  .github/
    workflows/
      ci.yml
      onboarding-test.yml
  src/
    index.ts
    config.ts
    github/
      client.ts
      issue-service.ts
      permission-service.ts
      repo-setup-service.ts
      welcome-service.ts
    templates/
      welcome.md
      first-issue.md
      checklist.md
    webhooks/
      router.ts
      handlers/
        installation.ts
        member_added.ts
        repository_created.ts
  docs/
    architecture.md
    onboarding-flow.md
    templates.md
  test/
    issue-service.test.ts
    permission-service.test.ts
  app.yml
  package.json
  tsconfig.json
  README.md
  .eslintrc.cjs
  .gitignore
`

---

3. README.md

`markdown

Solana Developer Onboarding Bot

The Solana Developer Onboarding Bot is a GitHub App that automates the onboarding experience for new developers joining Solana-based repositories.

What it does

- Sends a personalized welcome message
- Creates a "First Task" issue with a checklist
- Sets repository permissions for new contributors
- Generates a starter guide based on repository metadata
- Optionally provisions Solana CLI + Anchor setup instructions
- Supports per-repo configuration via .solana-onboarding.yml

Example onboarding flow

1. A new developer is added to a repository or organization.
2. The bot:
   - Sends a welcome message
   - Creates a "Getting Started" issue
   - Assigns the issue to the new developer
   - Adds them to the correct team or permission level
   - Posts a setup checklist

Configuration

Create .solana-onboarding.yml:

`yaml
welcome:
  enabled: true
  template: "welcome.md"

first_issue:
  enabled: true
  template: "first-issue.md"

permissions:
  default: "triage"
  teams:
    solana-devs: "write"

checklist:
  enabled: true
  template: "checklist.md"
`

Events

The bot listens to:

- installation.created
- membership.added
- repository.created

Local development

`bash
pnpm install
pnpm dev
`

License

MIT
`

---

4. app.yml

`yaml
name: Solana Developer Onboarding Bot
url: https://github.com/apps/solana-developer-onboarding-bot
hook_attributes:
  url: https://your-domain.com/webhooks/github
redirect_url: https://your-domain.com/app/callback
callback_urls:
  - https://your-domain.com/app/callback
public: false
default_permissions:
  contents: read
  metadata: read
  issues: write
  members: write
  administration: write
default_events:
  - installation
  - membership
  - repository
`

---

5. GitHub Actions: ci.yml

`yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: pnpm install
      - run: pnpm lint
      - run: pnpm test
`

---

6. GitHub Actions: onboarding-test.yml

`yaml
name: Onboarding Flow Test

on:
  workflow_dispatch:

jobs:
  simulate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Simulate onboarding event
        run: echo "Simulating onboarding flow..."
`

---

7. src/index.ts

`ts
import { createNodeMiddleware, Webhooks } from "@octokit/webhooks";
import { App } from "@octokit/app";
import { createServer } from "http";
import { router } from "./webhooks/router";

const appId = process.env.APP_ID!;
const privateKey = process.env.PRIVATE_KEY!;
const webhookSecret = process.env.WEBHOOK_SECRET!;

const app = new App({ appId, privateKey });
const webhooks = new Webhooks({ secret: webhookSecret });

router(webhooks, app);

const middleware = createNodeMiddleware(webhooks);

const port = process.env.PORT || 3001;
createServer(middleware).listen(port, () => {
  console.log(Solana Developer Onboarding Bot running on :${port});
});
`

---

8. Webhook Router

`ts
import type { Webhooks } from "@octokit/webhooks";
import type { App } from "@octokit/app";
import { handleInstallation } from "./handlers/installation";
import { handleMemberAdded } from "./handlers/member_added";
import { handleRepositoryCreated } from "./handlers/repository_created";

export function router(webhooks: Webhooks, app: App) {
  webhooks.on("installation.created", (event) =>
    handleInstallation(event, app)
  );

  webhooks.on("membership.added", (event) =>
    handleMemberAdded(event, app)
  );

  webhooks.on("repository.created", (event) =>
    handleRepositoryCreated(event, app)
  );
}
`

---

9. Example Handler: installation.ts

`ts
import type { WebhookEvent } from "@octokit/webhooks-types";
import type { App } from "@octokit/app";
import { WelcomeService } from "../../github/welcome-service";

export async function handleInstallation(
  event: WebhookEvent<"installation.created">,
  app: App
) {
  const installationId = event.payload.installation.id;
  const octokit = await app.getInstallationOctokit(installationId);

  const service = new WelcomeService(octokit);
  await service.sendWelcomeMessage(event.payload.installation);
}
`

---

10. Templates

templates/welcome.md
`markdown
👋 Welcome to the Solana ecosystem!

This repository uses the Solana Developer Onboarding Bot to help you get started quickly.

Your first task has been created — check the Issues tab.
`

templates/first-issue.md
`markdown

Your First Task

Welcome aboard! Here’s your first guided task to get familiar with the repository.

- [ ] Install Solana CLI
- [ ] Install Anchor
- [ ] Clone the repository
- [ ] Run anchor build
- [ ] Run anchor test

Ask questions anytime — we’re here to help.
`

---

