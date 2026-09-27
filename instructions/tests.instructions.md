---
applyTo: "test/**/*.js,src/**/*.spec.js"
---

# Test Claude Instructions

Follow these guidelines when writing or modifying tests.

## Testing Stack
- **Runner**: Jest (`jest.config.js`).
- **Library**: Enzyme with `@wojtekmaj/enzyme-adapter-react-17`. Setup lives in `enzyme.config.js`.
- **Environment**: `jest-environment-jsdom`.
- **Canvas**: `jest-canvas-mock` is auto-loaded for components that touch canvas (PDF/PPT export).
- **Setup files**: `jest.setup.js`, `enzyme.config.js`. Add new global mocks there, not per-file.

## Layout and Naming
- **Specs live in**: `test/specs/` — mirror the `src/` folder structure where possible.
- **Naming**: Test files end in `.spec.js`. Match the source filename (`Calendar.js` → `Calendar.spec.js`).

## Writing Tests
- **Describe blocks**: One top-level `describe` per module under test. Nest `describe` for sub-behaviours, use `it('should ... when ...')` for assertions.
- **Mounting**: Prefer `shallow` for unit-level component tests; use `mount` only when you need full lifecycle / DOM behaviour (e.g. measuring layout, integration with `ag-grid-react`).
- **Selectors**: Use semantic selectors where possible (`find(MyComponent)`, `find('button')`). Avoid coupling tests to deep CSS class chains.
- **Async**: Use `await` with explicit ticks (`await new Promise(setImmediate)`) when the component depends on resolved promises; or wrap in `act()` for state-update flushes.

## Mocking
- **API**: Mock `axios` (or the relevant `src/api/` module) per-test with `jest.mock` / `jest.spyOn`. Don't make real network calls.
- **i18n**: Mock `i18next` only if it interferes with assertions; otherwise let it pass through.
- **Date / time**: Use `jest.useFakeTimers()` and a fixed `Date` when tests depend on "now".
- **Canvas / chart libs**: `jest-canvas-mock` covers most cases. For `pptxgenjs` / `jspdf`, mock the module to assert calls without producing real binaries.

## Best Practices
- **Test behaviour, not implementation**: Assert on rendered output and user-visible state changes rather than internal helper invocations.
- **Isolation**: Each test should set up and tear down its own state. Don't rely on order between tests.
- **Coverage**: `npm test` runs the full suite; add tests for new behaviour and for the bug you're fixing. The repo doesn't enforce a coverage threshold, but new code should be covered.
- **Run before pushing**: `npm test` and `npm run lint:test`.

## Keeping This File Up to Date
If a change to `package.json` touches the test stack (runner, adapter, mocking lib, jsdom, canvas mock), update this file **in the same PR**.
