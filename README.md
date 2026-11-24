# A11y Audit: documentation and feedback

[A11y Audit](https://marketplace.visualstudio.com/items?itemName=nikodinovicf.a11y-audit) is a VS Code extension that audits live web pages, including pages behind SSO, against WCAG 2.2 Level A and AA. It runs axe-core in your own Google Chrome, shows the findings in VS Code, and walks you from automated results through a manual checklist to a VPAT® 2.5 Accessibility Conformance Report and a remediation plan.

This repository holds the documentation and is the place for feedback. The source code is not public.

## Install

- **Extension:** search for **A11y Audit** in the VS Code Extensions view, or install it from the [Marketplace](https://marketplace.visualstudio.com/items?itemName=nikodinovicf.a11y-audit).
- **Playwright e2e helper:** `npm install -D @nikodinovicf/a11y-audit`. See [docs/e2e.md](docs/e2e.md).

## Documentation

- [AI Insights (advisory)](docs/ai-insights.md)
- [Remediation Tracker](docs/remediation-coach.md)
- [Accessibility checks in Playwright e2e tests](docs/e2e.md)
- [SSO setup](docs/sso-setup.md)
- [Manual testing](docs/manual-testing.md)

The full feature guide is the extension's README, shown on the Marketplace page.

## Feedback

- **Bugs and feature requests:** open an [issue](https://github.com/nikodinovicf/a11y-audit-docs/issues). Use test data only: no client URLs, page content, screenshots or saved sessions.
- **Security problems:** do not open a public issue. Report them privately, as described in [SECURITY.md](SECURITY.md).

## License

A11y Audit 0.7.0 and later, and this documentation, are licensed under the [Business Source License 1.1](LICENSE). Production use is free for individuals and organizations with up to 5 employees; larger organizations need a commercial license. Each version becomes MIT on its Change Date. Version 0.6.1 and earlier were released under the MIT License and stay MIT.

VPAT® is a registered service mark of the Information Technology Industry Council (ITI).
