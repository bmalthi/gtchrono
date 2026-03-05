### 7.1. Component Hierarchy Tree

```
App
├── <Router>
│   ├── LoginPage (Handles US101, US102)
│   ├── ChronoListPage (US104)
│   │   ├── ChronoListItem
│   │   │   ├── EditButton (Visible if Editor)
│   │   │   └── DeleteButton (Enabled if Editor)
│   │   ├── CreateChronoModal (US105)
│   │   └── Header (UserIcon, LogoutButton - US103)
│   ├── ChronoViewPage (The main analysis view for a single Chrono)
│   │   ├── Header
│   │   │   ├── ChronoTitle
│   │   │   ├── SaveLabelButton (US501)
│   │   │   ├── LoadLabelDropdown (US503)
│   │   │   ├── TopDateControls (US304 - ComparisonOffset, etc.)
│   │   │   ├── EditConfigButton (Visible if Editor - US601)
│   │   │   └── UserIcon / LogoutButton
│   │   ├── LeftSidebar
│   │   │   ├── DateFilterPanel (US301-US303)
│   │   │   │   ├── DatePresetSelector
│   │   │   │   └── DateAggregationDropdown
│   │   │   ├── DimensionFilterPanel (US305)
│   │   │   │   ├── DimensionFilterItem (One for each dimension)
│   │   │   │   └── ApplyChangesButton
│   │   │   └── TimeSeriesLegendPanel (US402)
│   │   ├── MainContentArea
│   │   │   ├── TabBar (US203)
│   │   │   │   └── TabItem (Default, Pinned, User-created)
│   │   │   └── GraphGrid (US705 - Responsive Grid)
│   │   │       └── GraphComponent (The core visualization unit)
│   │   │           ├── GraphHeader
│   │   │           │   ├── GraphTitle / Subtitle
│   │   │           │   └── GraphIcons (Pin, Configure, Expand, etc. - US206, US401, US409)
│   │   │           ├── EChartsWrapper (Wraps the Apache ECharts instance)
│   │   │           │   ├── XAxis, YAxis
│   │   │           │   ├── TimeSeriesLine / Bar
│   │   │           │   └── Tooltip (US407)
│   │   │           ├── GraphStateOverlay (Handles Loading, Error, No Data states - US701, 702, 703)
│   │   │           └── GraphContextMenu (For Drill Down - US207)
│   │   └── EditConfigModal (US601 and related stories)
│   │       ├── Tabs (Permissions, Metrics, Dimensions, etc.)
│   │       ├── UserPermissionsPanel (US505)
│   │       ├── MetricEditorPanel (US604, US605)
│   │       └── DimensionEditorPanel (US606, US607)
│   └── AccessDeniedPage (US603)
└── GlobalErrorHandler (US710)
```

### 6.2. Key Component Responsibilities
*   **`ChronoViewPage`:** The most complex container component. It is responsible for fetching the main `Chrono` configuration, managing the state for all view tabs (US308), and orchestrating API calls to the `/data` endpoint when filters are applied.
*   **`LeftSidebar`:** This component will hold the global filter state for the *active* view tab. When the user switches tabs, `ChronoViewPage` will pass the new tab's filter state down to the sidebar, causing it to re-render with the correct values.
*   **`GraphGrid`:** Manages the layout and rendering of all `GraphComponent` instances for the active view tab. It receives an array of graph configurations from `ChronoViewPage`.
*   **`GraphComponent`:** A self-contained unit responsible for a single graph. It receives its configuration (metric, filters, etc.) and data as props. It does **not** fetch its own data. This makes it highly reusable and performant.
*   **`EChartsWrapper`:** This presentational component's sole job is to interact with the Apache ECharts library. It receives structured data and chart options as props and translates them into the specific format ECharts requires, isolating the library-specific logic.