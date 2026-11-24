# Security policy

## Supported versions

Only the latest release gets security fixes. Update to it from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=nikodinovicf.a11y-audit). The e2e helper is updated with `npm install -D @nikodinovicf/a11y-audit@latest`.

## Reporting a vulnerability

Do not open a public issue for a security problem. Report it privately on GitHub: open the [Security tab](https://github.com/nikodinovicf/a11y-audit-docs/security) of the a11y-audit-docs repository and choose **Report a vulnerability**.

Please include:

- the version of A11y Audit and of VS Code;
- what an attacker can do, and what they need first (for example, a crafted page that the user audits, or a crafted results file in the workspace);
- steps to reproduce, with test data only.

Never include real credentials, saved sessions, client data or screenshots of a client's app.

You will get an answer within 7 days. Once a fix is released, the advisory is published and the reporter is credited, unless they prefer otherwise.

## Scope

In scope: the extension and the e2e helper (`@nikodinovicf/a11y-audit`), including how it stores sessions and API keys, what it writes to `.a11y/`, what it sends to AI providers, and how it renders reports, the tree and webviews.

Out of scope: vulnerabilities in the audited web pages, in Google Chrome, in Playwright or axe-core (report those to their projects), and findings that need an attacker who already controls the user's machine or VS Code profile.
