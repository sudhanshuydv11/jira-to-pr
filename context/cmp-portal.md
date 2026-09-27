---
name: cmp-portal
source: https://github.com/bct-adidas/cfccm-cmp-portal
branch: main
commit: 1881fbd785722bafe26eeeed9862b81b5fdb03d5
lastUpdated: "2026-08-24T13:21:29Z"
generated: "2026-08-26"
paths:
  - force-app/main/default/aura/CM_CampaignCalendarReact
  - force-app/main/default/aura/CM_Mini_Calendar
  - force-app/main/default/aura/CM_Campaign_Timelines
  - force-app/main/default/staticresources
---

# cmp-portal Context

Salesforce DX project (`sourceApiVersion: 58.0`, `name: cmp-portal`) that hosts this
repo's build output. Description: "This repository contains the salesforce code for
Campaign Portal".

## How it consumes this repo

Two Aura components load the `CampaignCalendar` static resource via
`<lightning:container>`:

- `force-app/main/default/aura/CM_CampaignCalendarReact/CM_CampaignCalendarReact.cmp`
- `force-app/main/default/aura/CM_Mini_Calendar/CM_Mini_Calendar.cmp`

Both use the identical pattern:

```html
<lightning:container aura:id="CampaignCalendar" class="container"
    src="{!$Resource.CampaignCalendar + '/index.html'}"
    onmessage="{!c.handleMessage}"
    onerror="{!c.handleError}"/>
```

**Note:** as of the synced commit, `force-app/main/default/staticresources/` does
**not** currently contain a `CampaignCalendar` (or `CampaignCalendar.resource-meta.xml`)
entry — only `CampaignCalendarTimelines` (a separate bundle, unrelated to this repo,
consumed by `CM_Campaign_Timelines`). The static resource is presumably deployed by the
CI pipeline (per this repo's `Jenkinsfile`) rather than committed directly; this
repo's own `Jenkinsfile` does not reference `CampaignCalendar` or
`calendar-visualisation` at all, so the deploy step lives entirely in the
calendar-visualisation-ui pipeline, not here.

## postMessage contract (Aura ⇄ React)

- **React → Aura ready signal:** React calls `LCC.sendMessage({ method: 'reactLoaded' })`.
  Both controllers (`CM_CampaignCalendarReactController.js`,
  `CM_Mini_CalendarController.js`) respond to `method == 'reactLoaded'` by sending an
  initial payload via `component.find("CampaignCalendar").message(msg)`:
  ```js
  { name: "InitialLoad", value: <campaignInformation JSON>, labelIndex, filters: <filtersInformation JSON> }
  ```
- **Aura → React ongoing messages**, all sent the same way
  (`component.find("CampaignCalendar").message(msg)`), keyed by `name`:
  - `"LoadOnChange"` — re-sends `campaignInformation`/`labelIndex`/`filters` on change.
  - `"labelIndex"` — sent when `sendLabelIndex` fires (event param `labelIndex`).
  - `"Closed"` — `{ name: "Closed", value: "Closed" }`, a no-op signal (matches this
    repo's `value === 'Closed'` check in `App.js`).
- **React → Aura inbound methods** handled in `handleMessage` via
  `message.payload.method`, dispatched with a long `if/else if` chain (both
  CM_CampaignCalendarReactController.js and CM_Mini_CalendarController.js implement a
  similar/overlapping set). Known methods (non-exhaustive, extend as new payload types
  are added on either side):
  - `toggleLeftBar(value)`, `showDetailsModal(value)`, `showExportSpinner(value)`
  - `imageClick(campaignId)` — opens `/lightning/r/CM_CUS_Campaign__c/<id>/view`
  - `updateRecords(objectsList)` — updates `calendarPhaseWrapper`
  - `clone(mianPhaseObject)`, `addToMyCalendar(value)`, `openNotifications(id, notificationsAvailable)`
  - `openDetailsEdit(id, calendarId, isOwnCampaign)`, `createCalendarTouchpoint(value)`
  - `reorderTouchpoints(value.updatedTouchpoints, value.deletedTouchpoints, value.calendarId)`
  - `createCalendar(value)`, `createCampaign(value)`, `addMoment(value)`
  - `openCalendarNotifcations(value)`, `toggleFullscreen(calendarId, isFullscreen)`
  - `milestoneTimeline(calendarId)` — navigates to timeline screen
  - `acceptNewCampaign(notificationId, calendarId)`, `rejectNewCampaign(notificationId)`
  - `searchCampaign(value.location, value.calendarId)`, `editCalendar(calendarId)`
  - `editCalendarYearlyView(calendarId)` — comma-split into primary/secondary calendar IDs
  - `viewCalendar(calendarId)`, `openCommsProductDetail(CampaignRecId)`
  - `openLaunchPhaseDetail(launchPhaseId)`, `addToCalendarOnMoment(value.momentId)`

  **Breaking-change constraint:** any rename/removal of a `method` string or its
  expected `payload` shape on the React side requires a matching update in both
  `CM_CampaignCalendarReactController.js` and `CM_Mini_CalendarController.js` (and any
  other Aura consumer), and vice versa.

## Other integration points

- `force-app/main/default/aura/CM_Campaign_Timelines/` is a **separate, unrelated**
  Aura component/bundle (`CampaignCalendarTimelines` static resource) — not produced
  by this repo. Don't conflate the two when reasoning about "the calendar".
- No `package.json` dependency on this repo — cmp-portal only consumes it as a
  Salesforce static resource, never via npm.

## Local Notes

(none yet)
