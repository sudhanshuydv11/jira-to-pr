---
applyTo: "src/**/*.scss,src/**/*.css"
---

# Styles Claude Instructions

Follow these guidelines when writing or modifying styles.

## Tooling
- **Preprocessor**: SCSS via `sass` + `sass-loader`.
- **Linter**: stylelint with `stylelint-config-adidas`. Run `npm run lint:style`.
- **Design tokens**: Source colours, spacing, and typography from `yarn-design-system` whenever possible — avoid one-off literals.

## File Layout
- **Co-located**: Component styles live next to the component, named after it: `Calendar/Calendar.scss`, `Timeline/Timeline.scss`.
- **Imports**: Components import their `.scss` file directly (`import './Calendar.scss';`). The webpack pipeline (`mini-css-extract-plugin` in production, `style-loader` in dev) handles the rest.

## Authoring Rules
- **Selectors**: Scope styles to the component's root class (`.calendar`, `.timeline`) to avoid leaking into sibling components. Salesforce hosts the bundle, so global selectors will collide with the host page.
- **No `!important`** unless overriding a third-party rule (`reactstrap`, `react-calendar-timeline`) and there is no other route. Add a comment explaining why.
- **Variables**: Reuse SCSS variables from `yarn-design-system` and shared partials. Define new component-local variables at the top of the component's `.scss` file.
- **Nesting**: Keep nesting depth ≤ 3. Deeper nesting is a sign the markup needs a class instead.
- **Specificity**: Match the lowest specificity that achieves the override. Don't chain class names where one would do.
- **Colours**: Prefer design-system tokens over hex literals. If a one-off colour is required, define it as a variable at the top of the file with a comment explaining the reason.

## Responsiveness
- The calendar is consumed inside Salesforce Lightning panes of varying widths. Use relative units (`rem`, `%`) where layout adapts; avoid fixed `px` widths on containers.

## Working with Third-Party Components
- `reactstrap`, `ag-grid-react`, `react-calendar-timeline`, `react-day-picker` ship their own CSS. When overriding:
  - Scope your overrides under the component's root class.
  - Prefer adding a CSS class to the third-party element via its `className` prop over selecting it by tag/attribute.
  - Document non-obvious overrides with a one-line comment naming the third-party class being overridden.

## Keeping This File Up to Date
If a change to `package.json` touches the styling stack (preprocessor, design-system version, stylelint config, new third-party UI lib whose CSS leaks in), update this file **in the same PR**.
