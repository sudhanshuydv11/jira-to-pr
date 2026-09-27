---
applyTo: "src/**/*.js"
---

# React Claude Instructions

Follow these guidelines when working on React components in this repository.

## Component Design
- **React Version**: React 17. Functional components with hooks are preferred for new code; existing class components remain — leave them as-is unless asked to migrate.
- **Fragments**: Use short syntax `<>...</>` for fragments.
- **Props**: Validate props with `prop-types` (already a devDependency). Match the validation style of neighbouring files.
- **Boolean Props**: Use `is{PropName}` or `has{PropName}` naming where it improves readability.
- **JSX file extension**: This repo uses `.js` (not `.jsx`) for files containing JSX — keep the convention.
- **Formatting**: Keep a blank line between groups of related JSX or logic blocks; defer to ESLint for the rest.

## File and Folder Conventions
- **Component folder**: A component lives in its own PascalCase folder, e.g. `Calendar/`, with:
  - `Calendar.js` — component file.
  - `Calendar.scss` — styles.
  - `index.js` — re-export.
  - `helpers.js` (optional) — component-local helpers.
- **Shared helpers**: Pure utility modules go in `src/helpers/` (`ObjectHelper.js`, `CampaignDataHelper.js`, `utils.js`).
- **Containers**: Top-level container components live in `src/containers/`.
- **Context providers**: Live in `src/context/`.
- **API layer**: All `axios` calls go through `src/api/`. Don't call `axios` directly from components.

## State and Data Fetching
- **Local state**: Prefer `useState` / `useReducer` for component-local state.
- **Shared state**: Use the existing React Context providers in `src/context/`. Do not introduce Redux, Zustand, or other state libraries.
- **Server state**: Use the API clients in `src/api/`. Cache / memoise at the call site if needed.

## Hooks and Lifecycle
- **Custom hooks**: Place hooks in the same folder as the component they serve, named `use{Behaviour}.js`. Promote to `src/helpers/` if reused across components.
- **Memoisation**: Use `useMemo` and `useCallback` for expensive derived values and for props passed to grids/timelines (`ag-grid-react`, `react-calendar-timeline`) where reference equality matters.
- **Effect cleanup**: Always return a cleanup function from `useEffect` when subscribing to events, timers, or DOM observers.

## UI Libraries
- Use [yarn-design-system-react-components](https://yarn.adidas.com/) for adidas-standard UI primitives.
- Use `reactstrap` for Bootstrap-based components already present in the codebase.
- Use `react-calendar-timeline` for the timeline view; use `ag-grid-react` for tabular data.
- Avoid introducing new UI libraries — match what neighbouring files use.

## Date and i18n
- **Dates**: Newer code should prefer `react-day-picker`'s utilities; existing code uses `moment` heavily — match what the surrounding file already does rather than mixing both.
- **Strings**: Wrap user-facing strings with `i18next` translations. Don't hard-code English strings in JSX.

## Performance
- The calendar can render hundreds of campaigns — be mindful of `.map` over large arrays in render. Memoise derived data, virtualise where the existing component already does, and avoid creating new object/array literals in props on every render.

## Keeping This File Up to Date
If a change to `package.json` touches the React-related stack (UI lib swap, state-management addition, date-lib change, React major upgrade), update this file **in the same PR**.
