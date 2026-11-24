# Accessibility checks in Playwright e2e tests

The crawler audits the pages listed in `a11y.config.json`. If the client already has a Playwright test suite, it already reaches the screens that matter: logged in, with data, mid-flow. `checkA11y` audits those screens from inside the tests and writes the same results format as the crawler. The VS Code tree view, report, checklist suggestions, VPAT and remediation plan then work with these results too.

## Install

Install the helper from npm as a dev dependency:

```sh
npm install -D @nikodinovicf/a11y-audit
```

It needs `@playwright/test` in the project, which an existing suite already has. Use the same version as the extension, so both write the same results format.

## Use in a test

```ts
import { test, expect } from '@playwright/test';
import { checkA11y } from '@nikodinovicf/a11y-audit/playwright';

test('checkout', async ({ page }) => {
  await page.goto('/cart');
  await checkA11y(page, 'Cart');

  await page.getByRole('button', { name: 'Checkout' }).click();
  await page.getByRole('dialog').waitFor();
  await checkA11y(page, 'Cart', { state: 'Checkout dialog' });
});
```

`checkA11y(page, label, options?)` audits the page as it is at that moment, records the result and returns it.

| Option | Description |
|---|---|
| `state` | Shown after the label, like a config state: `Cart · Checkout dialog`. |
| `viewport` | Shown after the state, e.g. `"mobile"` for a mobile project. |
| `outputDir` | Where fragments go. Default: `.a11y/results` in the working directory. |

By default `checkA11y` only records findings, so the test does not fail. To make the test fail on violations, assert on the result:

```ts
const result = await checkA11y(page, 'Cart');
expect(result.findings.filter((f) => f.kind === 'violation')).toEqual([]);
```

## Deeper checks in e2e tests

`checkA11y` can run the same deeper checks as the config's `"audit"` block:

```ts
import { checkA11y, installA11yRecorder, startA11yRecording } from '@nikodinovicf/a11y-audit/playwright';

test.beforeEach(async ({ context }) => {
  await installA11yRecorder(context); // records status messages from page load
});

test('profile save message is announced', async ({ page }) => {
  await page.goto('/profile');
  await startA11yRecording(page);        // messages from here on are checked
  await page.getByRole('button', { name: 'Save' }).click();
  await checkA11y(page, 'Profile · saved', { deep: true });
});

test('settings widgets work with the keyboard', async ({ page }) => {
  await page.goto('/settings');
  // Last in the test: it presses keys and leaves the page in another state.
  await checkA11y(page, 'Settings', { deep: { keyboard: true, interactiveComponents: true, focusVisibility: true } });
});
```

- `deep: true` runs the checks that do not press keys: accessibility tree, live regions and static component checks.
- `keyboard` and `focusVisibility` must be asked for by name. They move focus, open and close widgets and do not reload the page between tests (an e2e state cannot be rebuilt), so call them at the end of a test. Requests that could change data are blocked while the keyboard journey runs, and afterwards the page is loaded again with the block still on, so nothing the journey started (an autosave timer, a retry) can send data later. Do not rely on page state after `checkA11y` with `keyboard: true`.
- Requests a service worker sends are not seen by that block. If a service worker is registered for the page, the keyboard journey is skipped with a note. To run it, create the context with service workers blocked: `test.use({ serviceWorkers: 'block' })` in Playwright Test, or `browser.newContext({ serviceWorkers: 'block' })`.
- Screenshots and tree snapshots go to `.a11y/evidence/e2e-<run>/`, next to `.a11y/results/`, and the merged results link to them.

## Merge the results

Each call writes one small fragment file, so parallel workers never write to the same file. Merge the fragments into one results file in one of two ways.

**Automatically, after every run.** Add a global setup and teardown:

```ts
// a11y.setup.ts
import { startA11yRun } from '@nikodinovicf/a11y-audit/playwright';
export default function () {
  startA11yRun();
}
```

```ts
// a11y.teardown.ts
import { mergeE2eResults } from '@nikodinovicf/a11y-audit/playwright';
export default async function () {
  await mergeE2eResults();
}
```

```ts
// playwright.config.ts
export default defineConfig({
  globalSetup: './a11y.setup.ts',
  globalTeardown: './a11y.teardown.ts',
  // ...
});
```

**Manually, in VS Code.** Run **A11y: Import E2E Results**. It merges the fragments in `.a11y/results` and loads them into the A11y Results view.

Without `startA11yRun`, every run writes into `.a11y/results/e2e-latest`, so fragments from runs that were not merged collect there and end up in one result.

## Where the results go

Results are written to `.a11y/results/<time>-e2e.json`. The tool also writes `.a11y/.gitignore`, because the results contain page HTML from the client's app. Open the workspace in VS Code to see the results in the **A11y Results** view, or run **A11y: Export Report**.

The e2e helper and the crawler use the same axe-core rules and WCAG 2.2 A/AA tags.
