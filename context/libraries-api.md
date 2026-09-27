---
name: libraries-api
description: API reference and usage patterns for all runtime and key dev dependencies in package.json
lastUpdated: "2026-08-24"
source: package.json
---

# Library API Context

Quick-reference for the libraries used in this project. Use this when implementing
features to ensure correct API usage without guessing.

---

## React 19 (peer dependency)

- Hooks: `useState`, `useReducer`, `useEffect`, `useLayoutEffect`, `useRef`, `useCallback`, `useMemo`, `useContext`, `useId`, `useTransition`, `useDeferredValue`, `useSyncExternalStore`, `useOptimistic`, `useActionState`, `use`
- `useReducer(reducer, initialArg, init?)` — returns `[state, dispatch]`
  - `reducer(state, action)` must be pure, return next state
  - `dispatch(action)` — action is typically `{ type: string, payload?: any }`
- `useEffect(setup, deps?)` — cleanup returned from setup runs on unmount or before re-run
- `useCallback(fn, deps)` — memoize callback reference
- `useMemo(() => value, deps)` — memoize computed value
- `useRef(initialValue)` — `.current` persists across renders
- `React.memo(Component)` — skip re-render if props are shallowly equal
- `forwardRef((props, ref) => JSX)` — forward ref to child DOM/component
- `createContext(defaultValue)` + `useContext(Ctx)` — context pattern
- `lazy(() => import('./Component'))` + `<Suspense fallback={...}>` — code splitting
- React 19 additions: `use(promise | context)`, `useFormStatus()`, `useOptimistic(state, updateFn)`
- Server components not applicable (this is a client-side SPA in Salesforce Lightning Container)

## prop-types ^15.6.2

- `PropTypes.string`, `.number`, `.bool`, `.array`, `.object`, `.func`, `.node`, `.element`
- `.shape({ key: PropTypes.string })`, `.arrayOf(PropTypes.number)`
- `.oneOf(['a', 'b'])`, `.oneOfType([PropTypes.string, PropTypes.number])`
- `.isRequired` suffix for required props
- Declare as `Component.propTypes = { ... }` and `Component.defaultProps = { ... }`

## dayjs ^1.11.21

Replaces moment.js in new code. Immutable, chainable.

- `dayjs()` — current time
- `dayjs(dateStr)`, `dayjs(timestamp)`, `dayjs(Date)`
- `.format('YYYY-MM-DD')`, `.format('HH:mm:ss')`
- `.add(7, 'day')`, `.subtract(1, 'month')`
- `.startOf('week')`, `.endOf('month')`
- `.diff(otherDayjs, 'day')` — returns number
- `.isBefore(d)`, `.isAfter(d)`, `.isSame(d, 'day')`
- `.toDate()` — convert to native Date
- `.unix()` — Unix timestamp in seconds
- `.valueOf()` — Unix timestamp in milliseconds
- Plugins: `import isBetween from 'dayjs/plugin/isBetween'` then `dayjs.extend(isBetween)`
- Common plugins used: `isBetween`, `isSameOrBefore`, `isSameOrAfter`, `customParseFormat`, `weekOfYear`

## axios ^1.8.3

- `axios.get(url, config?)` → `Promise<{ data, status, headers }>`
- `axios.post(url, body, config?)`, `.put()`, `.patch()`, `.delete()`
- `axios.create({ baseURL, headers, timeout })` — instance factory
- Interceptors: `instance.interceptors.request.use(fn)`, `.response.use(fn, errFn)`
- Cancel: `const controller = new AbortController(); axios.get(url, { signal: controller.signal })`
- Error shape: `err.response?.status`, `err.response?.data`, `err.message`

## lightning-container ^1.0.0

Salesforce Lightning Container messaging protocol.

- `import LCC from 'lightning-container'`
- `LCC.sendMessage(messageObject)` — post message to Salesforce Aura parent
- `LCC.addMessageHandler(callbackFn)` — register handler for messages FROM Salesforce
- Message format: `{ name: string, value: any }` (both directions)
- Must call `addMessageHandler` once on mount; no built-in cleanup — manual removal needed if component unmounts
- Messages are serialized via `postMessage`; only JSON-serializable data

## @ag-grid-community/react ^32.3.5

- `import { AgGridReact } from '@ag-grid-community/react'`
- `import { ClientSideRowModelModule } from '@ag-grid-community/client-side-row-model'`
- `<AgGridReact rowData={rows} columnDefs={cols} modules={[ClientSideRowModelModule]} />`
- Column def: `{ field: 'name', headerName: 'Name', sortable: true, filter: true, width: 150 }`
- Grid API via `onGridReady={(params) => params.api}` — `api.setRowData()`, `api.refreshCells()`
- Events: `onCellClicked`, `onRowSelected`, `onSortChanged`, `onFilterChanged`

## react-calendar-timeline ^0.30.0-beta.18

- `import Timeline from 'react-calendar-timeline'`
- Props: `groups`, `items`, `defaultTimeStart`, `defaultTimeEnd`, `visibleTimeStart`, `visibleTimeEnd`
- `groups`: `[{ id, title, rightTitle?, ...custom }]`
- `items`: `[{ id, group, title, start_time, end_time, ...custom }]`
- Events: `onItemMove`, `onItemResize`, `onItemSelect`, `onItemClick`, `onTimeChange`, `onCanvasClick`
- `onTimeChange(visibleTimeStart, visibleTimeEnd, updateScrollCanvas)` — call `updateScrollCanvas(start, end)` to allow
- `itemRenderer` prop for custom item rendering
- `groupRenderer` prop for custom group rendering

## reactstrap ^9.2.3

Bootstrap 5 React components (no jQuery).

- `import { Modal, ModalHeader, ModalBody, ModalFooter, Button, Input, Row, Col, Popover, PopoverBody, Tooltip, Dropdown, DropdownToggle, DropdownMenu, DropdownItem, Nav, NavItem, NavLink, TabContent, TabPane } from 'reactstrap'`
- `<Modal isOpen={bool} toggle={fn}>` — controlled modal
- `<Popover target="id" isOpen={bool} toggle={fn} placement="bottom">`
- `<Tooltip target="id" isOpen={bool} toggle={fn}>`
- `<Button color="primary" onClick={fn}>` — color: primary, secondary, success, danger, warning, info
- Grid: `<Row><Col sm="6">...</Col><Col sm="6">...</Col></Row>`

## @hello-pangea/dnd ^18.0.1

Fork of `react-beautiful-dnd` (maintained). Drag and drop.

- `import { DragDropContext, Droppable, Draggable } from '@hello-pangea/dnd'`
- `<DragDropContext onDragEnd={result => ...}>`
- `<Droppable droppableId="list">{(provided) => <div ref={provided.innerRef} {...provided.droppableProps}>...</div>}</Droppable>`
- `<Draggable draggableId="item-1" index={0}>{(provided, snapshot) => <div ref={provided.innerRef} {...provided.draggableProps} {...provided.dragHandleProps}>...</div>}</Draggable>`
- `onDragEnd({ source: { droppableId, index }, destination: { droppableId, index } | null })`

## react-day-picker ^9.14.0

- `import { DayPicker } from 'react-day-picker'`
- `<DayPicker mode="single" selected={date} onSelect={setDate} />`
- `<DayPicker mode="range" selected={range} onSelect={setRange} />` — range: `{ from: Date, to: Date }`
- Props: `disabled`, `fromDate`, `toDate`, `modifiers`, `modifiersStyles`
- `mode`: `"single"`, `"multiple"`, `"range"`

## rc-slider ^11.1.9

- `import Slider from 'rc-slider'`
- `<Slider min={0} max={100} value={val} onChange={setVal} />`
- Range: `<Slider range defaultValue={[20, 50]} />`
- Props: `step`, `marks`, `dots`, `vertical`, `disabled`, `tipFormatter`

## interactjs ^1.10.27

Low-level drag/resize/gesture library.

- `import interact from 'interactjs'`
- `interact('.draggable').draggable({ onmove: event => ... })`
- `interact('.resizable').resizable({ edges: { left: true, right: true, bottom: true, top: true } })`
- Event props: `event.dx`, `event.dy`, `event.rect.width`, `event.rect.height`
- `.unset()` — remove all listeners

## pptxgenjs ^4.0.1

PowerPoint generation.

- `import PptxGenJS from 'pptxgenjs'`
- `const pptx = new PptxGenJS()`
- `pptx.defineLayout({ name: 'A4', width: 10, height: 7.5 })`
- `const slide = pptx.addSlide()`
- `slide.addText('Hello', { x: 1, y: 1, fontSize: 24, color: '363636' })`
- `slide.addImage({ path: url, x: 0, y: 0, w: 10, h: 7.5 })`
- `slide.addTable(rows, { x, y, w, colW, border })` — rows: `[[cell1, cell2], ...]`
- `pptx.writeFile({ fileName: 'export.pptx' })` — triggers download
- `pptx.write('base64')` — for programmatic use

## lodash ^4.17.11

- Import individual functions: `import get from 'lodash/get'` (tree-shakeable)
- Commonly used: `get`, `set`, `cloneDeep`, `debounce`, `throttle`, `isEqual`, `isEmpty`, `groupBy`, `sortBy`, `uniqBy`, `merge`, `omit`, `pick`, `flatten`, `chunk`
- `_.get(obj, 'a.b.c', defaultVal)` — safe deep access
- `_.cloneDeep(obj)` — deep clone (avoid for large objects in hot paths)
- `_.debounce(fn, wait, { leading?, trailing? })` — returns debounced fn with `.cancel()`
- `_.isEqual(a, b)` — deep equality

## classnames ^2.2.6

- `import cn from 'classnames'`
- `cn('foo', 'bar')` → `'foo bar'`
- `cn('foo', { bar: true, baz: false })` → `'foo bar'`
- `cn(['foo', { bar: true }])` → `'foo bar'`

## yarn-design-system & yarn-design-system-react-components

adidas internal design system.

- Import components: `import { Button, Icon, ... } from 'yarn-design-system-react-components'`
- Import icons: `import { IconName } from 'yarn-design-system-icons'`
- SCSS variables/mixins from `yarn-design-system` available via `@import` in SCSS files
- Follow existing usage patterns in the codebase for component API specifics

---

## Testing Libraries

### Jest ^29.7.0

- `describe('...', () => { ... })`, `it('...', () => { ... })`, `test('...', () => { ... })`
- `expect(val).toBe(x)`, `.toEqual(x)`, `.toBeTruthy()`, `.toBeFalsy()`, `.toBeNull()`
- `.toHaveBeenCalledWith(args)`, `.toHaveBeenCalledTimes(n)`
- `jest.fn()` — mock function; `.mockReturnValue(x)`, `.mockResolvedValue(x)`, `.mockImplementation(fn)`
- `jest.spyOn(obj, 'method')` — spy without replacing
- `jest.useFakeTimers()`, `jest.advanceTimersByTime(ms)`, `jest.useRealTimers()`
- `beforeEach(fn)`, `afterEach(fn)`, `beforeAll(fn)`, `afterAll(fn)`

### @testing-library/react ^16.3.0

- `import { render, screen, fireEvent, waitFor, act } from '@testing-library/react'`
- `render(<Component />)` — renders into document
- `screen.getByText('...')`, `.getByRole('button', { name: '...' })`, `.getByTestId('...')`
- `screen.queryByText('...')` — returns null if not found (no throw)
- `screen.findByText('...')` — async, returns Promise
- `fireEvent.click(element)`, `fireEvent.change(input, { target: { value: 'new' } })`
- `waitFor(() => expect(...))` — polls until assertion passes
- `act(() => { ... })` — wrap state updates

### @testing-library/user-event ^14.6.1

- `import userEvent from '@testing-library/user-event'`
- `const user = userEvent.setup()`
- `await user.click(element)`, `await user.type(input, 'text')`, `await user.clear(input)`
- `await user.selectOptions(select, ['value'])`, `await user.keyboard('{Enter}')`
- Preferred over `fireEvent` for realistic user interaction simulation

---

## Build Tools

### Webpack 5 (^5.98.0)

- Config: `webpack.config.js` (dev/Lightning Container), `webpack.publish.config.js` (NPM package)
- Loaders: `babel-loader`, `sass-loader`, `css-loader`, `style-loader`, `postcss-loader`
- Plugins: `HtmlWebpackPlugin`, `MiniCssExtractPlugin`, `BundleAnalyzerPlugin` (optional)
- Code splitting: `import()` dynamic imports for lazy-loaded chunks
- Dev server: `webpack-dev-server` with HMR on port 8080

### ESLint 9 (^9.39.4) — Flat Config

- Uses flat config (`eslint.config.*`) with `@eslint/eslintrc` compatibility layer
- Extends: `eslint-config-adidas-babel`, `eslint-config-adidas-es9`, `eslint-config-adidas-react`
- Parser: `@babel/eslint-parser`
- Plugins: `eslint-plugin-react`, `eslint-plugin-react-hooks` (^5.2.0), `eslint-plugin-import`, `eslint-plugin-jsdoc`, `eslint-plugin-promise`, `eslint-plugin-n`

### stylelint ^16.26.1

- Extends: `stylelint-config-adidas`, `stylelint-config-standard-scss`
- Run: `npm run lint:style`
