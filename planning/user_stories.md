# Chronoscope: Master User Story Document

## 1. Application Core

*   **Application Name:** Chronoscope
*   **Core Purpose:** A high-performance web application that enables users to connect to large-scale analytical databases, automatically discover the schema of time series data, and conduct rapid, interactive visual analysis.
*   **Key Asset:** The primary unit of work in Chronoscope is the "Chrono," a shareable, editable dashboard linked to a specific database table.

### 1.1. System Configuration Parameters
*   `MAX_VIEW_TABS`: The maximum number of view tabs a user can have open. **Default: 16**
*   `HIGH_CARDINALITY_THRESHOLD`: The number of unique values a dimension must exceed to be considered "high-cardinality" for filtering purposes. **Default: 256**
*   `SPLITBY_CARDINALITY_LIMIT`: The maximum number of unique values a dimension can have to be available in the "Breakdown By" dropdowns. **Default: 48**

## 2. User Roles

*   **Editor:** A user with rights to modify a Chrono's configuration, permissions, and labels.
*   **Viewer:** A user with read-only access to a Chrono.
*   **Owner (of a Chrono or Label):** The user who created a specific asset.
*   **Admin:** A system administrator who configures data sources.

## 3. User Stories

### Group 1: Authentication & Chrono Management

#### US101: Developer Login
> As a Developer (in a test environment),
> I want to instantly log in as any user by simply providing their email/ID,
> so that I can quickly test different user roles and permissions.

**Acceptance Criteria:**
1.  **AC1:** Given the application is running with `AUTH_MODE=developer`, when a user navigates to the root URL and is not logged in, they are redirected to a simple login page.
2.  **AC2:** Given the developer login page is visible, it must show a single text box for "User Email/ID" and a "Log In" button.
3.  **AC3:** Given a developer enters an ID (e.g., "editor@test.com") and clicks "Log In", when the action completes, the system creates a mock session for that user (without checking a password).
4.  **AC4:** Given the session is created, the user is redirected to the root page (US104) as that user.
5.  **AC5:** Given the application needs to check permissions (e.g., for the "Edit Config" button, per US601), it treats the user as an "Editor" or "Viewer" based on their mock ID.

#### US102: Enterprise SSO Login
> As a User (in an enterprise environment),
> I want to log in via my company's single sign-on (SSO) provider,
> so that I can securely access Chronoscope without needing a separate password.

**Acceptance Criteria:**
1.  **AC1:** Given the application is running with `AUTH_MODE=oidc`, when a user navigates to the root URL and is not logged in, they are immediately redirected to the configured enterprise Identity Provider (IdP) login page.
2.  **AC2:** Given the user successfully authenticates with the IdP, when the IdP redirects them back to the Chronoscope application, the app validates their secure token.
3.  **AC3:** Given the token is valid, when the app reads it, it creates a real user session and (if configured) gets their "Editor" or "Viewer" role from the token's groups.
4.  **AC4:** Given the session is created, the user is redirected to the root page (US104) as that authenticated user.

#### US102a: Personal Login
> As a User (in a personal or small team environment),
> I want to log in simply by providing my email address,
> so that I can access the application without needing SSO or a pre-configured account.

**Acceptance Criteria:**
1.  **AC1:** Given the application is running with `AUTH_MODE=personal`, when a user navigates to the root URL and is not logged in, they are redirected to a simple login page.
2.  **AC2:** Given the personal login page is visible, it must show a single text box for "Email Address" and a "Log In" button.
3.  **AC3:** Given a user enters a valid email and clicks "Log In", the system creates a secure session for that user identity.
4.  **AC4:** Given the session is created, the user is redirected to the root page (US104) as that authenticated user.
5.  **AC5:** Given this is the very first time *any* user has logged into this instance of the application, that first user is automatically granted full "Admin" and "Editor" privileges.
6.  **AC6:** Given a new user logs in and they are not the first-ever user, they are granted "Viewer" permissions by default. An existing Admin or Editor must manually grant them higher permissions (per US505).

#### US103: Logout
> As a User,
> I want to log out of the application,
> so that I can securely end my session.

**Acceptance Criteria:**
1.  **AC1:** Given a user is logged in, when they click the "User Icon" in the top navigation bar, a menu appears.
2.  **AC2:** Given the menu is open, when the user clicks the "Logout" button, their local application session is destroyed.
3.  **AC3:** Given the session is destroyed, the user is redirected to the login page.
4.  **AC4 (Enterprise Only):** Given `AUTH_MODE=oidc`, when the user is redirected (AC3), the application also redirects them to the IdP's logout endpoint to terminate the SSO session.

#### US104: View Chrono List
> As a User,
> I want to see a list of all Chronos I have access to when I open the application,
> so that I can quickly choose one to load or decide to create a new one.

**Acceptance Criteria:**
1.  **AC1:** Given a user lands on the root page, when the page loads, then they are shown a list of all Chronos they have access to.
2.  **AC2:** Given a Chrono is displayed in the list, when the user views it, then they must see the Chrono's "friendly name".
3.  **AC3:** Given a Chrono is displayed in the list, when the user views it, then they must see a "Load" button for that Chrono.
4.  **AC4:** Given a Chrono is displayed in the list AND the user is an "Editor" for that Chrono, when the user views it, then they must see an active (clickable) "Delete" button.
5.  **AC5:** Given a Chrono is displayed in the list AND the user is not an "Editor" for that Chrono, when the user views it, then they must see a disabled (greyed out) "Delete" button.
6.  **AC6:** Given a user has no Chronos in their list, when the page loads, then a message (e.g., "Welcome!") must be displayed.
7.  **AC7:** A "Create a Chrono" button must always be visible on the Chrono List page for any logged-in user.

#### US105: Create New Chrono
> As a User,
> I want to create a new Chrono,
> so that I can start a new analysis.

**Acceptance Criteria:**
1.  **AC1:** Given a user is on the Chrono List (US104) page, when they click "Create a Chrono", then they are presented with a dialog.
2.  **AC2:** Given the dialog is open, when the user selects a Data Source, then they are shown a list of available tables from that source.
3.  **AC3:** Given the user selects a table and clicks "Create", the "Create" button must enter a disabled/loading state and a background job is initiated to create the Chrono and discover its schema (per US610).
4.  **AC4:** Given the schema discovery job succeeds, the user is then redirected to the new Chrono's analysis page.
5.  **AC5:** Given the schema discovery job fails, the user remains on the dialog, the "Create" button becomes active again, and a clear error message is displayed (e.g., "Error: No timestamp column found in table").
6.  **AC6:** Given the new Chrono is created, it must immediately be discoverable and appear in the user's Chrono List (US104).
7.  **AC7:** Given a user successfully creates a new Chrono, they are designated as its Owner and are granted Editor permissions for it.

#### US106: Delete a Chrono
> As an Editor,
> I want to delete a Chrono I am an editor for,
> so that I can remove old or irrelevant analyses.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor is on the Chrono List (US104), when they click the "Delete" button for a Chrono, then they are shown a confirmation modal.
2.  **AC2:** Given the confirmation modal, when the user confirms the action, then the Chrono is permanently deleted.
3.  **AC3:** Given the Chrono is deleted, then it is removed from all users' lists.
4.  **AC4:** Given the Chrono is deleted, all Labels associated with that Chrono are also permanently deleted.

#### US107: Load a Chrono
> As a User,
> I want to load a Chrono from the Chrono List,
> so that I can begin my analysis.

**Acceptance Criteria:**
1.  **AC1:** Given a user is on the Chrono List (US104), when they click the "Load" button for a Chrono (or navigate to its direct base URL), then they are redirected to that Chrono's main analysis page.
2.  **AC2:** Given the analysis page loads, when the page renders, then it must load the "Default" view (per US201) for that Chrono.
3.  **AC3:** Given a user attempts to load a Chrono's URL they do not have access to, then the Analysis View must not load, and an access-denied error message must be displayed (per US603).
4.  **AC4:** Given a user attempts to load a Chrono URL with an ID that does not exist, the system must respond with the same access-denied error message as defined in AC3.

### Group 2: Core Analysis View & Navigation

#### US201: View Default Dashboard
> As a User,
> I want to see a pre-built "Default" dashboard when I load a Chrono,
> so that I can get an immediate overview without any setup.

**Acceptance Criteria:**
1.  **AC1:** Given a user loads a Chrono (per US107) or resets their view, when the page loads, then a view tab labeled "Default" is displayed and active.
2.  **AC2:** Given the "Default" view loads, it must display one graph for each of the Chrono's "base metrics" (per US605).
4.  **AC4:** The graphs must be displayed in the order their corresponding metrics appear in the Chrono's configuration file.

#### US202: Load Chrono from URL Parameters
> As a User,
> I want to load a Chrono and a specific view state directly from a URL,
> so that I can share my exact analysis with others or bookmark it.

**Acceptance Criteria:**
1.  **AC1:** Given a URL contains a `label_id`, when the page loads, then it must find and load the matching Label (per Group 5 stories).
2.  **AC2:** Given a URL contains `tsN` parameters (e.g., `ts1=...`, `ts2=...`), when the page loads, then it must construct a custom view with graphs defined by those parameters.
3.  **AC3:** Given a URL contains both a valid `label_id` and `tsN` parameters, when the page loads, then the `label_id` takes priority, and the `tsN` parameters are ignored.
4.  **AC4:** Given a URL contains a `date_options` parameter, when the page loads, then it must load the Chrono using those date options.
5.  **AC5:** Given a URL contains both a valid `label_id` and `date_options`, when the page loads, then the `date_options` from the URL override the date options saved in the Label.
6.  **AC6:** Given a URL contains a `label_id`, `tsN` parameters, and `date_options`, when the page loads, it must load the graphs defined by the `label_id` (ignoring `tsN`) and apply the `date_options` from the URL (overriding the Label's dates).
7.  **AC7:** Given a URL contains an invalid or inaccessible `label_id`, when the page loads, then it must fall back to any `tsN` parameters in the URL. If neither is valid, it must load the Default view (US201).
8.  **AC8:** Given an invalid `label_id` was provided (AC7), when the view loads, then a non-blocking "Label not found" overlay or message must be displayed.
9.  **AC9:** Given the URL contains an invalid parameter key outside of a `ts` value (e.g., `?rgion=1`), then the invalid parameter must be silently ignored.
10. **AC10:** Given a `tsN` parameter's value contains an invalid fragment (e.g., `ts1=rgion:1;channel:5`), then the invalid fragment is silently ignored, while all valid fragments (`channel:5`) are still applied.

**Technical Notes:**
*   The `tsN` value format is `dim_name:id1,id2;other_dim:id3`. A comma (`,`) creates an "IN" list. A semicolon (`;`) creates an "AND" condition.

#### US203: Manage Views (Tabs)
> As a User,
> I want to work with multiple views in a tabbed interface,
> so that I can conduct multiple lines of analysis in parallel.

**Acceptance Criteria:**
1.  **AC1:** Given a user is on the analysis page, they must see a list of views with very minimal ui at the top of the main view, probably above the graphs but in the graphs container
2.  **AC2:** The tab bar must always contain: The "Default" view (US201) and the "Pinned" view (US206).
3.  **AC3:** The "Default" and "Pinned" tabs cannot be closed or renamed.
4.  **AC4:** Any user-created tabs (e.g., from "Drill" or "Selection") must appear to the right of the "Pinned" tab.
5.  **AC5:** Given a new view is created by an action (e.g., "Drill Down"), it is given a descriptive name (e.g., "Drill: Region") and is automatically set as the active view.
6.  **AC6:** Given the View List contains the maximum number of views (as defined by `MAX_VIEW_TABS`), when a new view is created, then the oldest closable view is automatically removed from the list (FIFO).

#### US204: Deleted

#### US205: Close a View
> As a User,
> I want to close a view tab I no longer need,
> so that I can clean up my workspace.

**Acceptance Criteria:**
1.  **AC1:** Given a user-created tab, when the user views it, then a "Close" (X) icon must be visible on the tab.
2.  **AC2:** Given the user clicks the "Close" icon, then the tab is removed from the tab bar.
3.  **AC3:** Given a user closes the currently active view, when the view is removed, the "Default" view must automatically become the new active view.
4.  **AC4:** The "Default" and "Pinned" tabs cannot be closed.

#### US206: Pin a Graph
> As a User,
> I want to "pin" a graph from any view,
> so that I can collect all the graphs I care about in one place.

**Acceptance Criteria:**
1.  **AC1:** Given a user views any graph, a "Pin" icon must be visible.
2.  **AC2:** Given a user clicks the "Pin" icon, a *reference* to that graph is added to the "Pinned" view. The pinned graph is not a static copy.
3.  **AC3:** Given a graph has been pinned, its icon in its original view must change to a persistent "Pinned" state (e.g., a solid, colored pin) that is always visible.
4.  **AC4:** Given a user modifies a graph that is pinned (e.g., changes its metric via US401), the changes are instantly reflected in the version of the graph displayed in the "Pinned" view.
5.  **AC5:** Given a user is on the "Pinned" view OR the graph's original view, when they hover over a pinned graph, the icon must change to an "Unpin" icon.
6.  **AC6:** Given a user clicks "Unpin" from either location, the reference to the graph is removed from the "Pinned" view, and the icon on the source graph reverts to the hover-only "Pin" icon (per AC1).

#### US207: Drill Down on a Graph
> As a User,
> I want to drill down on a graph's dimension,
> so that I can see a more granular breakdown either within that graph or across multiple new graphs.

**Acceptance Criteria:**
1.  **AC1:** Given a user right-clicks on a graph, a context menu must appear, showing a "Split by.." sub-menu.
2.  **AC2:** The "Split by.." sub-menu must list all **visible** dimensions (e.g., "Region", "Product").
3.  **AC3:** Given a user selects a dimension, a new sub-menu appears with two options: "Single Graph Split" and "Multigraph Split".
4.  **AC4 (Single Graph Split):** Given the source graph displays a single time series (has no "Split By" dimension), when the user selects "Split in this graph", then the graph instantly reloads, displaying one time series line for each value of the selected dimension (similar to US401). This action does not create a new view.
5.  **AC5 (Single Graph Split):** Given the source graph already has a "Split By" dimension applied, the "Split in this graph" option in the sub-menu (per AC3) must be disabled (greyed out).
6.  **AC6 (Single Graph Split):** Given a dimension's unique value count exceeds the `SPLITBY_CARDINALITY_LIMIT`, the "Split in this graph" option in the sub-menu (per AC3) must be disabled (greyed out).
7.  **AC7 ("Multigraph Split):** Given a user selects "Split into new graphs", a new sub-menu appears with "Top 10", "Bottom 10", and "Normal".
8.  **AC8 (Split - Normal):** Given a user selects "Normal", a new view tab is created (per US203), displaying N graphs, where N is the number of unique values for that dimension. Each graph shows the original metric(s) filtered for one of those values.
9.  **AC9 (Split - Top 10):** Given a user selects "Top 10", a new view tab is created, displaying 10 graphs, one for each of the top 10 values of that dimension (ranked by the current metric).
10. **AC10 (Cardinality Limit):** Given a dimension's unique value count exceeds the `SPLITBY_CARDINALITY_LIMIT`, when the drill menu appears (per AC7), then the "Normal" option must be disabled.
11. **AC11 (Ranking Limit):** Given a source graph contains multiple metrics, when the user opens the drill sub-menu for splitting (per AC7), then the "Top 10" and "Bottom 10" options must be disabled.

#### US208: Synchronize View State with URL
> As a User,
> I want the browser URL to automatically update as I modify my view,
> so that I can reliably share my exact analysis at any time by copying the link.

**Acceptance Criteria:**
1.  **AC1 (Trigger):** The URL must be regenerated and updated in the browser's address bar immediately after any action that successfully reloads the graphs in the active view. This includes:
    *   Applying dimension filters (US305).
    *   Changing date options (US301).
    *   Applying a time series transformation (US402).
    *   Creating a new view via "Drill Down" or "Selection" (US207, US209).
2.  **AC2 (Detaching from Label):** If the view was originally loaded from a saved Label (`label_id` is in the URL), the moment any modification (per AC1) is made, the `label_id` parameter **must be removed** from the URL.
3.  **AC3 (Encoding the View):** The updated URL must encode the entire state of the active view by constructing a set of `tsN` parameters. Each `tsN` parameter will define a single graph, including its metric, breakdown dimension, and any specific filters applied to it (e.g., from Focus Mode).
4.  **AC4 (Encoding Date Options):** In addition to the `tsN` parameters, a `date_options` parameter must be added to the URL **if and only if** all graphs in the active view share the identical date configuration. If the date options are in a "Various" state (per US307), this parameter is omitted.
5.  **AC5 (Handling Mixed Views):** If a view contains graphs with different filters (i.e., the sidebar shows "Various" per US306), the URL will still update. The `tsN` parameters (per AC3) will accurately encode each graph's specific filters, ensuring the shared view is an exact replica. This is P2. I'm not sure this can be done well due to URL size limites

#### US209: Create a View from Graph Selection
> As a User,
> I want to select multiple graphs from my current view,
> so that I can create a new, temporary view containing only those specific graphs for a more focused analysis.

**Acceptance Criteria:**
1.  **AC1:** Given a user is on the Analysis View, when they click on a graph (but not on an interactive icon), then that graph enters a "selected" state.
2.  **AC2:** Given a graph is "selected," when the user views it, then it must have a clear visual indicator (e.g., a colored border).
3.  **AC3:** Given a user has selected one graph, when they click another graph in the same view, then that graph also enters the "selected" state.
4.  **AC4:** Given at least one graph is selected, when the selection is active, then an unobtrusive overlay must appear (e.g., bottom-right) with a "View" button and an "Unselect" button.
5.  **AC5:** Given the user clicks the "View" button in the overlay, when the click happens, then:
    *   A new view is created (e.g., "Sel 1") and added to the View List (as per US203).
    *   The user is automatically switched to this new view, which contains only the graphs that were selected.
    *   The selection state is cleared.
6.  **AC6:** Given the user clicks the "Unselect" button in the overlay, then all currently selected graphs are deselected.
7.  **AC7:** Given a user has selected one or more graphs in a view, when they switch to a different view tab, then the selection state is canceled.
8.  **AC8:** Given a user-created selection view already exists (e.g., "Sel 1"), when a new selection view is created, then it must be named sequentially (e.g., "Sel 2").

### Group 3: Data Interaction & Filtering

#### US301: Modify Date Options
> As a User,
> I want to change the date range and comparison options,
> so that I can see how my metrics change over time and have the display granularity adjust automatically.

**Acceptance Criteria:**
1.  **AC1:** Given a user changes any date-related control (Date Preset, Start/End Date, Date Aggregation, IsGrowth, IsPastPeriod, ComparisonOffset), when the control is changed, then all graphs in the active view must instantly reload.
2.  **AC2:** Given an instant reload is triggered by a date change, when the graphs reload, then any "pending" dimension filter changes (per US305) must also be applied at the same time.
3.  **AC3:** Given an instant reload is triggered while there are pending dimension filters (per AC2), when the reload completes, then the "Apply Changes" button must return to its default state.
4.  **AC4:** Given a user views the left sidebar, then they must see the DatePresetSelector (US302), Start/End Date inputs, and Date Aggregation dropdown (US303).
5.  **AC5:** Given a user views the top control bar, then they must see the `IsGrowth` checkbox, `IsPastPeriod` checkbox, and `ComparisonOffset` dropdown (US304).
6.  **AC6 (New):** Given a user action changes the date range to a new duration, if the currently selected Date Aggregation (per US303) is more granular than a sensible default for that duration (e.g., 'Daily' is selected for a 2-year range), the system must automatically override it. The new granularity will be set to the "Auto" equivalent, and the "Date Aggregation" dropdown in the UI will update to reflect this change.
6.  **AC7 (New):** Will need a debounce or other check. eg if the system updates both start and end date field we don't want the system to refresh the data twice

**Technical Notes:**
*   The "sensible default" logic for the automatic granularity adjustment in AC6 should be based on the length of the time period. For example:
    *   **< 90 days:** Defaults to "Daily"
    *   **90 - 730 days:** Defaults to "Weekly"
    *   **> 730 days:** Defaults to "Monthly"
*   This automatic adjustment only occurs if the user's current selection is finer-grained than the new default. For example, if the user has "Monthly" selected and changes the range to 30 days, their "Monthly" selection will be respected.

#### US302: Use Date Presets
> As a User,
> I want to select from a list of common date presets,
> so that I can quickly set my date range.

**Acceptance Criteria:**
1.  **AC1:** Given a user modifies the Start Date or End Date manually, when the dates no longer match a preset, then the `DatePresetSelector` dropdown in the left sidebar must display the text 'Custom'.
2.  **AC2:** Given a user selects a preset (e.g., "Last 28 Days"), when the selection occurs, then the Start Date and End Date inputs must update, and all graphs must reload (per US301).

#### US303: Set Default Date Aggregation
> As a User,
> I want to set the default date aggregation (granularity),
> so that I can view all graphs as "Daily", "Weekly", or "Monthly".

**Acceptance Criteria:**
1.  **AC1:** Given a user is in the left sidebar, when they view it, then they must see a dropdown for "Date Aggregation".
2.  **AC2:** The dropdown must contain "Auto", "Daily", "Weekly", "Monthly", "Quarterly", "Yearly".
3.  **AC3:** Given the user selects a new granularity, when the change occurs, then all graphs instantly reload (per US301) using the new time grouping.

#### US304: Set Comparison Offset
> As a User,
> I want to set the "Comparison Offset" period,
> so that I can compare the current period to a period other than the previous one.

**Acceptance Criteria:**
1.  **AC1:** Given a user is in the top control bar, when they view it, then they must see a dropdown labeled `ComparisonOffset` (e.g., "vs. 28 days ago").
2.  **AC2:** The dropdown must contain common offsets: "Previous Period", "1 day ago", "7 days ago", "28 days ago", "365 days ago".
3.  **AC3:** Given the user selects a new offset, when the `IsPastPeriod` checkbox (US301) is active, or `isGrowth` then all graphs must reload to show the new comparison period.
4.  **AC4:** Given `IsGrowth` AND `IsPastPeriod` are both set to "false" (unchecked), when the user views the `ComparisonOffset` dropdown, then it must be visible but disabled (greyed out).
5.  **AC5:** Given either `IsGrowth` OR `IsPastPeriod` is set to "true" (checked), when the user views the `ComparisonOffset` dropdown, then it must be enabled (clickable).

#### US305: Modify and Apply Dimension Filters
> As a User,
> I want to filter my dashboard by one or more dimension values,
> so that I can narrow my analysis to specific segments.

**Acceptance Criteria:**
1.  **AC1:** Given a user is on the analysis page, when they view the left sidebar, then they must see a list of all **visible** dimensions (per US606).
2.  **AC2:** Given a user has no pending dimension filter changes, the "Apply Changes" button must be in a default state (e.g., grey or low-contrast).
3.  **AC3:** Given a dimension is low-cardinality (unique values <= `HIGH_CARDINALITY_THRESHOLD`), when the user clicks it, then it must display a dropdown with a list of all values as checkboxes.
4.  **AC4:** Given a user checks or un-checks one or more dimension values, when the change is made, then the change is "pending", graphs do not reload, and the "Apply Changes" button must immediately change to an active, high-contrast state (e.g., blue).
5.  **AC5:** Given a dimension is high-cardinality (unique values > `HIGH_CARDINALITY_THRESHOLD`), when the user clicks it, then it must display a dropdown with a search box.
6.  **AC6:** Given the user types into the live search box, the dropdown must display a list of matching values (~5) and must also display any already selected values at the top of the list.
7.  **AC7:** Given a user checks a new value from the search results, that value is added to the "selected" list at the top of the dropdown.
8.  **AC8:** Given a user closes and re-opens a high-cardinality dropdown that has selected items, the dropdown must display an empty search box and the list of only the currently selected items.
9.  **AC9:** Given a user single-clicks a value in a low-cardinality list, when they click, then the checkbox is toggled.
10. **AC10:** Given a user double-clicks a value in a low-cardinality list, when they double-click, then that value is selected, and all other values for that dimension are deselected.
11. **AC11:** Given the user has pending changes, when they click the "Apply Changes" button, then all graphs in the active view must reload with the new filters, and the "Apply Changes" button must return to its default state.
12. **AC12:** Given the user has pending changes, when they click the "Cancel" button (next to "Apply Changes"), then all pending dimension filter changes are discarded, and the dimension filter UI reverts to reflect only the currently applied filters.
13. **AC13:** Given the "Cancel" button is clicked, the "Apply Changes" button must immediately return to its default state.
14. **AC14:** Given a user has pending dimension filter changes in the active view, when they navigate to a different view tab, then the pending changes are discarded, and the "Apply Changes" button reverts to its default state.
15. **AC15:** Given one or more values for a dimension are selected, when the dimension's dropdown is closed, then the dimension's entry in the sidebar must display a summary of the selection (e.g., "Americas|APAC" or "Americas...").

#### US306: Handle Conflicting Dimension Filters
> As a User,
> I want the dimension filter UI to clearly show when my graphs have different filters, and I want to be able to override them all at once.

**Acceptance Criteria:**
1.  **AC1:** Given the active view contains graphs with different filters for the same dimension (e.g., after loading a Label or using Focus Mode per US409), then the filter summary for that dimension in the left sidebar must display the text "Various".
2.  **AC2:** Given the scenario in AC1, when the user clicks the dropdown for that dimension, then all selected values must be shown in a "half selected" state (e.g., a light blue checkbox).
3.  **AC3:** Given the user opens and then closes the dropdown (per AC2) without making any changes, when the dropdown closes, then the dimension's state remains "Various" and no changes are applied.
4.  **AC4:** Given the scenario in AC1, when a user makes a new selection in that dimension's dropdown, this new selection overrides all previous filters for that dimension on all graphs in the view.
5.  **AC5:** Given the override in AC4, when the user clicks "Apply Changes" (per US305), then all graphs in the view reload with the new, unified dimension filter.

#### US307: Handle Conflicting Date Options
> As a User,
> I want the date options UI to clearly show when my graphs have different date settings, and I want to be able to override them all at once.

**Acceptance Criteria:**
1.  **AC1:** Given the active view contains graphs with different date options, then the `DatePresetSelector` in the left sidebar must display the text "Various".
2.  **AC2:** Given the scenario in AC1, when a user selects a new preset from the `DatePresetSelector`, then all graphs in the view must instantly reload (per US301) with this new, unified date setting.
3.  **AC3:** Given the scenario in AC1, when a user manually changes the `DisplayedDatePeriod` selectors, then all graphs in the view must instantly reload with this new, unified date period.

#### US308: Maintain Independent View State
> As a User,
> I want each view tab to maintain its own date and filter settings,
> so that I can switch between different lines of analysis without losing my context.

**Acceptance Criteria:**
1.  **AC1:** Each view tab (Default, Pinned, and user-created) must maintain its own independent state for all Date Options (per US301) and all Dimension Filters (per US305).
2.  **AC2:** Given a user modifies a filter or date option in "View A", the change must only apply to "View A". No other view tabs are affected.
3.  **AC3:** Given a user switches from "View A" to "View B", all of the control components in the UI (e.g., date pickers, dimension filter dropdowns in the sidebar) must instantly update to reflect the saved state of "View B".
4.  **AC4:** Given a user switches back to "View A", the UI controls must again update to show the previously configured state of "View A".

### Group 4: Graph-Specific Interactions

#### US401: Removed

#### US402: Apply Time Series Transformations
> As a User,
> I want to apply transformations to my time series,
> so that I can change the data display (e.g., to see percent of total).

**Acceptance Criteria:**
1.  **AC1:** Given the Analysis View is loaded, a **"Time Series Legend" panel must be visible in the left sidebar**, below the Date Aggregation control.
2.  **AC2:** The legend must display a list of all unique time series present in the active view. Each series must be represented by a **small colored square and its corresponding label - the label will only be visible on hover** (e.g., "Region: Americas").
3.  **AC3:** Below the list of time series, the text actions `% of total`, `% of this`, and `unset %` must be displayed.
4.  **AC4:** A user can select a single time series by clicking its square, which enters a "selected" state.
5.  **AC5:** The `% of this` action must be disabled if no time series square is selected.
6.  **AC6:** The `% of this` action must be disabled if the active view contains a state of ambiguity, defined as either:
    *   a) Graphs having different "Breakdown By" dimensions.
    *   b) Graphs having different graph-specific filters (from US409 Focus Mode) that cause identically named series to represent different underlying data (e.g., a "Mobile" series for `USA` and a "Mobile" series for `Canada`).
7.  **AC7:** Given a user clicks any of the text actions ('% of total', '% of this', or 'unset %'), all graphs on the page must immediately begin reloading.
8.  **AC8:** The reload must NOT include any pending (un-applied) dimension filter changes from US305.
9.  **AC9:** Given a user clicks `% of total`, all graphs reload, displaying each time series as a percentage of the total for each data point.
10. **AC10:** Given a user clicks `% of this` while a square is selected, all graphs reload, displaying each time series as a percentage of the selected time series.
11. **AC11:** Given a user clicks `unset %` while a percentage transformation is active, all graphs reload, displaying their original absolute values.

#### US403: Change Graph Type
> As a User,
> I want to change the visualization type for a graph,
> so that I can view the data as a line chart or bar chart.

**Acceptance Criteria:**
1.  **AC1:** Given a user opens the "Configure" popover (per US401), when the popover is open, then they must see options (e.g., icons) to select "Line" or "Bar" chart.
2.  **AC2:** Given the user selects a new chart type, when the selection occurs, then the graph instantly re-renders using the new visualization type.

#### US404: Edit Graph Metadata
> As a User,
> I want to edit the title, subtitle, and time series labels for a graph,
> so that I can create a more readable and customized visualization.

**Acceptance Criteria:**
1.  **AC1:** Given a graph is displayed, an "Edit Metadata" icon is visible in the top-right corner.
2.  **AC2:** Given the user clicks the "Edit Metadata" icon, then a modal must open.
3.  **AC3:** The modal must contain: A text field for "Title", a text field for "Subtitle", a list of editable text fields for each "Time Series Label", and a "Save" button.
4.  **AC4:** Given the "Title" field has not been customized, it defaults to the metric's friendly name.
5.  **AC5:** Given the "Subtitle" field has not been customized, it defaults to a string of all dimension filters common to all time series on the graph, plus the displayed date range.
6.  **AC6:** Given a time series label has not been customized, it defaults to a string of all dimension filters that are unique to that time series.
7.  **AC7:** Given the user clicks "Save", the modal closes, and the graph updates to show the new metadata.
8.  **AC8:** Given the user saves or updates a Label (per Group 5 stories) after making these changes, these custom values are saved as part of the Label.
9.  **AC9:** Given a data refresh is triggered (e.g., by changing a global filter), a previously customized **Title** must persist, but any customized **subtitles or time series labels** are overwritten with new, auto-generated values based on the new context.

#### US406: Export Graph Image, SQL, and Data
> As a User,
> I want to be able to download an image of a graph, see its underlying SQL, or download its raw data,
> so that I can use the graph's information in other contexts.

**Acceptance Criteria:**
1.  **AC1:** Given a graph is displayed, a "Save" icon must be visible in the top-right corner.
2.  **AC2:** Given the user clicks the "Save" icon, a menu must display three options: "Download Image", "See SQL", and "Download as CSV".
3.  **AC3 (Image):** Given the user selects "Download Image", the browser must download a PNG or JPG image file of the graph's complete current visual state, including title, subtitle, and legend.
4.  **AC4 (Image Filename):** The downloaded image filename must be descriptive, using the pattern `{chrono_name}_{graph_title}_{date}.png` (e.g., `Sales-Analytics_Revenue-by-Region_2025-12-01.png`).
5.  **AC5 (SQL):** Given the user selects "See SQL", a modal must appear, displaying the read-only, non-formatted SQL query used to generate the graph's data.
6.  **AC6 (CSV):** Given the user selects "Download as CSV", the browser must download a CSV file containing the graph's raw data.
7.  **AC7 (CSV Format):** Given the downloaded CSV is opened, each time series column must be labeled with the full string of all its dimension filters (e.g., `region=Americas;vehicle=Car`).

#### US407: View Graph Data on Hover
> As a User,
> I want to hover my mouse over a graph,
> so that I can see the specific data values for each time series at that point in time.

**Acceptance Criteria:**
1.  **AC1:** Given a user hovers their mouse over the plot area of a graph, a vertical line (crosshair) must appear, tracking the mouse's X-position.
2.  **AC2:** A tooltip must appear, displaying the Date/Time for the crosshair's position, as well as the friendly name and specific value for every time series on the graph at that point.
3.  **AC3:** All values must be formatted according to their defined type (e.g., "Dec 1, 2025", "$1,234.50", "15.2%").
4.  **AC4:** Given `IsPastPeriod` is on, the tooltip must clearly label the values for each period (e.g., "Revenue (2025): $1.2M", "Revenue (2024): $1.0M").

#### US408: Re-order Graphs in Custom Views
> As a User,
> I want to drag-and-drop graphs within my "Pinned" or user-created views,
> so that I can visually organize my analysis for comparison or saving.

**Acceptance Criteria:**
1.  **AC1:** Given a user is viewing the "Pinned" view or a user-created view (e.g., "Drill: Region"), they must be able to re-order graphs by clicking and dragging them.
2.  **AC2:** Given a user is viewing the "Default" view, graph re-ordering must be disabled.
3.  **AC3:** Given a user re-orders graphs, the new order must be saved as part of a Label (per Group 5 stories).

#### US409: Focus on a Single Graph for Detailed Viewing and Editing
> As a User,
> I want to expand any graph to "Focus" on it,
> so that I can see it in more detail or edit its configuration and filters in an isolated view.

**Acceptance Criteria:**
1.  **AC1 (UI and Availability):** Given a user views any graph in any view (including "Default"), a new "Expand" icon must be visible in the top-right corner, to the left of the other icons.
2.  **AC2 (Enter Focus Mode):** Given the user clicks the "Expand" icon, the view transitions to "Focus Mode":
    *   The selected graph expands to fill the entire graph grid area.
    *   All other graphs in the view are hidden.
    *   The "Expand" icon on the focused graph changes to a "Minimize" icon.
3.  **AC3 (Isolated Filtering):** Given the user is in Focus Mode, any changes made to the Date Options (US301) or Dimension Filters (US305) apply **only** to this single, focused graph.
4.  **AC4 (Isolated Configuration):** Given the user is in Focus Mode, any changes made to the graph's configuration (e.g., changing the metric via US401) apply only to this graph.
5.  **AC5 (Exit Focus Mode):** A user can exit Focus Mode by either:
    *   Clicking the "Minimize" icon on the graph.
    *   Clicking on the active view's tab in the tab bar.
6.  **AC6 (Return to Grid View):** Given the user exits Focus Mode, the UI returns to the multi-graph grid view. The graph that was edited now renders with its specific changes applied for the current session.
7.  **AC7 (UI State on Return):** Given a graph now has a filter that conflicts with the view's other graphs, the relevant dimension in the sidebar must display the "Various" state, as defined in US306. All sidebar controls revert to controlling the global view state.
8.  **AC8 (Global Filter Precedence):** Given a graph has a specific filter applied via Focus Mode (e.g., `Region: EMEA`), if the user then applies a new *global* filter for that same dimension from the sidebar (e.g., `Region: APAC`), this global action **overwrites** the specific filter. The graph will immediately reload with the new global `APAC` filter.
9.  **AC9 (Saving State):** Given a user saves or updates a Label in a customizable view (per US501/US502), any specific graph configurations made via Focus Mode are saved as part of the Label. When the Label is reloaded, these configurations are restored.

### Group 5: Saving & Sharing (Labels & Permissions)

#### US501: Save a New Label
> As a User,
> I want to save my current view (filters, graphs, etc.) as a new "Label",
> so that I can easily return to this specific analysis later or share it with others via its URL.

**Acceptance Criteria:**
1.  **AC1:** Given a user is viewing a state that is not associated with a saved Label, a "Save" button is visible in the main control bar.
2.  **AC2:** Given the user clicks "Save", they are prompted to enter a `label_name` in a dialog.
3.  **AC3:** The save dialog must include text clarifying that only the currently active view will be saved (e.g., "Saving a new Label for the 'Drill: Region' view").
4.  **AC4:** Given a user provides a `label_name` and confirms, the system must:
    *   Generate a short, random string `label_id` (e.g., 5 characters).
    *   Save the current view state of the currently active view tab only.
5.  **AC5:** The saved state must include: current Date Options, applied Dimension Filters, and the configuration of all graphs in the active view (e.g., metrics, time series, graph order, metadata, and any specific configurations from US409).
6.  **AC6:** Given the save is successful, the browser's URL must be updated to include the new parameter (e.g., `.../chrono-url?label_id=[new_id]`).
7.  **AC7:** A confirmation message must be displayed (e.g., "Label saved.").
8.  **AC8:** Given the save operation fails, an error message must be displayed (e.g., "Error: Could not save label").

#### US502: Update or "Save As" an Existing Label
> As a User,
> I want to either update an existing Label with my changes or save my changes as a new Label,
> so that I can manage my saved analyses without accidentally overwriting important ones.

**Acceptance Criteria:**
1.  **AC1:** Given a user has loaded a view from a Label (i.e., `label_id` is in the URL), the "Save" button in the main control bar must be replaced with a split button.
2.  **AC2:** The primary action on the split button must be "Update Label".
3.  **AC3:** A dropdown on the split button must contain the action "Save As...".
4.  **AC4 (Update):** Given a user is the **'Owner' of the selected Label**, when they click "Update Label", they are shown a confirmation. The button must be disabled for any user who is not the Label Owner.
5.  **AC5 (Update):** Given the user confirms the update, the system must overwrite the label document (specified by the `label_id` in the URL) with the new, current view state of the currently active view tab only. A confirmation message must be displayed (e.g., "Label updated.").
6.  **AC6 (Save As):** Given the user clicks "Save As...", the "Save a New Label" workflow (per US501) is initiated, allowing them to save the current view under a new name and `label_id`.

#### US503: Load, Rename, and Delete Labels
> As a User,
> I want to access a list of saved Labels,
> so that I can load, rename, delete, or change the visibility of my saved views.

**Acceptance Criteria:**
1.  **AC1:** Given a user clicks the "Load Label" button, a dropdown appears.
2.  **AC2:** The list must display all labels associated with the current Chrono that were either created by the user OR created by other users and marked as "world visible".
3.  **AC3 (Load):** Given a user clicks a Label from the list, they are navigated to the Analysis View with that specific label loaded (as per US202).
4.  **AC4 (Rename UI):** Given a user is the **'Owner' of the Label**, when they click on the text of a Label's name, then the text must become an editable input field.
5.  **AC5 (Rename Action):** Given the user types a new name and clicks out (or hits 'Enter'), then the new name is saved. This does not change the saved view state.
6.  **AC6 (Delete UI):** A "delete" icon must be visible next to every Label for which the user is the **'Owner'**.
7.  **AC7 (Delete Action):** Given the Label Owner clicks the "delete" icon, when they confirm the action, then the Label is permanently removed.
8.  **AC8 (Visibility UI):** A "Toggle World Visible" button is active if the user is the **'Owner' of the Label** and disabled otherwise.
9.  **AC9 (Visibility Action):** Given the Label Owner clicks the active "Toggle World Visible" button, the label's "world visible" status (true/false) is toggled.
10. **AC10:** A "world visible" Label is only discoverable by users who already have at least 'Viewer' access to the parent Chrono.

#### US505: Manage User Permissions (Editor)
> As an Editor,
> I want to manage who has Viewer or Editor access to my Chrono,
> so that I can control who can see or modify the analysis.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor enters the "Edit" mode (per US601), they can access a "User Permissions" management panel.
2.  **AC2:** The panel must display a list of all users who currently have access and their permission level ("Viewer" or "Editor").
3.  **AC3:** The panel must contain a search box to find and add new users by email.
4.  **AC4:** Given an Editor adds a user, they must be able to assign them a role of either "Viewer" or "Editor".
5.  **AC5:** Given an Editor views a user in the list, they must see a dropdown to change that user's role (e.g., from "Viewer" to "Editor").
6.  **AC6:** Given an Editor views a user in the list, they must see a "Remove" button.
7.  **AC7:** Given the Editor clicks "Remove", when they confirm, then that user's access to the Chrono is revoked.
8.  **AC8:** The system must prevent an Editor from removing or demoting the last remaining Editor for a Chrono. An attempt to do so must be blocked with an error message (e.g., "Cannot remove the last editor.").

### Group 6: Configuration & Admin

#### US601: Access Editor Functions
> As an Editor,
> I want to click an "Edit" button on the main analysis page,
> so that I can access the settings panels for metrics, dimensions, and permissions.

**Acceptance Criteria:**
1.  **AC1:** Given a user with the "Editor" role loads a Chrono, then they must see a small "Edit" button (or icon) on the main analysis view.
2.  **AC2:** Given a user with the "Viewer" role loads a Chrono, then the "Edit" button must not be visible.
3.  **AC3:** Given an Editor clicks the "Edit" button, then the UI reveals the entry points to access:
    *   User Permission Management (US505)
    *   Composite Metric Management (US604)
    *   Base Metric Management (US605)
    *   Dimension Management (US606)
    *   Low-Cardinality Value Management (US607)
    *   Chrono Rename (US602)
    *   Access Denied Message (US603)
    *   Date Preset Management (US608)
4.  **AC4:** Given an Editor clicks the "Edit" button again (or a "Done" button), then the settings panels are hidden, and the UI returns to the standard analysis view.

#### US602: Rename a Chrono
> As an Editor,
> I want to change the "friendly name" of a Chrono,
> so that I can make it more descriptive than the default table name.

**Acceptance Criteria:**
1.  **AC1:** Given a new Chrono is created (per US105), its "friendly name" must default to the name of its base table (e.g., "sales_data_v3").
2.  **AC2:** Given an Editor has clicked the "Edit" button (per US601), they must see a text input field displaying the Chrono's current friendly name.
3.  **AC3:** Given the Editor types a new name into the field and saves, then the Chrono's friendly name is updated for all users.
4.  **AC4:** Given the friendly name is updated, it must be reflected in the Chrono List (US104) and the browser tab title.

#### US603: Set Access Denied Message
> As an Editor,
> I want to set a custom message for users who are denied access to a Chrono,
> so that I can direct them to the correct person to request access (e.g., "For access, please contact@company.com").

**Acceptance Criteria:**
1.  **AC1:** Given an Editor has clicked the "Edit" button (per US601), they must see an optional text area to set an "Access Denied Message".
2.  **AC2:** Given a user attempts to load a Chrono URL for which they do not have "Viewer" permissions (per US107), when the page loads, then they must be shown a generic "Access Denied" page.
3.  **AC3:** Given an Editor has set a custom message (per AC1), when a user is denied access (per AC2), then the custom message (e.g., "For access, ask Dave") must be displayed on the page.

#### US604: Create and Manage Composite Metrics
> As an Editor,
> I want to define new "Composite Metrics" using formulas,
> so that I can analyze derived data (e.g., 'Revenue per User').

**Acceptance Criteria:**
1.  **AC1:** Given an Editor accesses the editor functions (per US601), they can access a "Composite Metrics" management panel.
2.  **AC2:** The panel must allow an Editor to "Create New" or "Edit" an existing composite metric.
3.  **AC3:** The create/edit view must show: `friendly_name`, `metric_name` (atomic name), `description`, and a `formula` input.
4.  **AC4:** The `formula` input must allow basic arithmetic, including the use of parentheses `()` for order of operations (e.g., `(metric_a + metric_b) / metric_c`).
5.  **AC5:** The system must validate the formula and only allow saving if:
    *   The referenced `metric_names` are valid base or composite metrics.
    *   The `metric_name` for the new composite metric is unique.
6.  **AC6:** The `format_type` and `aggregation` fields must not be editable; they are inherited from the base metrics.
7.  **AC7:** Given a composite metric formula is '`metric_a` / `metric_b`', when the query is generated, it must use the saved default aggregation for each base metric (e.g., '`SUM(metric_a)` / `SUM(metric_b)`').
8.  **AC8:** Given a composite metric formula involves division, the generated SQL must use a safe division function (e.g., `SAFE_DIVIDE()` in BigQuery, or `NULLIF(..., 0)`) to prevent division-by-zero errors.
9.  **AC9:** Given a composite metric is saved, it must become available in the "Metric" selector (US401) and appear in the "Default" view (US201) just like a base metric.

#### US605: Manage Base Metric Settings
> As an Editor,
> I want to adjust the settings for auto-discovered "Base Metrics",
> so that I can set correct formats and aggregations.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor accesses the editor functions (per US601), they can access a "Base Metrics" management panel.
2.  **AC2:** The panel must list all metrics discovered from the database (per US610).
3.  **AC3:** Given an Editor selects a metric, they must be able to edit its: `friendly_name`, `description`, `is_visible` (boolean), `format_type` ("$", "%", "#"), `time_aggregation` ("SUM", "AVG", "COUNT"), and `dim_aggregation` ("SUM", "AVG", "COUNT").
4.  **AC4:** Given the `time_aggregation` and `dim_aggregation` are set, when this metric is used, it must be aggregated using these methods by default. `time_aggregation` is used when grouping data by time (e.g., daily sums), while `dim_aggregation` is used when grouping by a dimension (e.g., the sum for a specific region).

#### US606: Manage Dimension Settings
> As an Editor,
> I want to adjust the settings for auto-discovered "Dimensions",
> so that I can hide irrelevant fields or set user-friendly names.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor accesses the editor functions (per US601), they can access a "Dimension" management panel.
2.  **AC2:** The panel must list all string/categorical columns discovered from the database (per US610).
3.  **AC3:** Given an Editor selects a dimension, they must be able to edit its: `friendly_name` and `is_visible` (boolean).
4.  **AC4:** Given `is_visible` is set to 'false', the dimension must be hidden from all user-facing UI, including the filter list (US305), drill-down menus (US207), and breakdown selectors (US401).

#### US607: Manage Low-Cardinality Dimension Values
> As an Editor,
> I want to edit the friendly names for the values of my low-cardinality dimensions,
> so that I can make raw data IDs appear as human-readable names in the Analysis View.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor is on the 'Edit Config' page (per US601) viewing the 'Dimensions' list, when they look at a low-cardinality dimension, then its list of scanned values is displayed.
2.  **AC2:** Each row in the value list must show the read-only Raw Value (e.g., '1') and an editable Friendly Name text field (e.g., 'Americas').
3.  **AC3:** Given an Editor clicks the Friendly Name text field for a value, when they type a new name and click away, the new friendly name for that value is saved.
4.  **AC4:** Given a Friendly Name has not been set by an Editor, it must default to the Raw Value.
5.  **AC5:** Given an Editor is viewing a high-cardinality dimension, no list of values is displayed.

#### US608: Manage Date Presets (Editor)
> As an Editor,
> I want to add, edit, and delete date presets,
> so that I can define the quick-select date range options for users in the Analysis View.

**Acceptance Criteria:**
1.  **AC1:** Given an Editor is on the 'Edit Config' page (per US601), they must see a "Date Presets" section with a list of all currently defined presets.
2.  **AC2:** Each preset row must show in-line editable fields for: Friendly Name (text), Type (dropdown: "Relative", "Fixed"), and Definition fields (e.g., Start/End Date for "Fixed"; Start/End Offset/Unit for "Relative").
3.  **AC3:** Given an Editor modifies any field for an existing preset, the change is saved immediately on blur.
4.  **AC4:** An "Add Preset" button must be visible.
5.  **AC5:** Given an Editor clicks "Add Preset", a new, blank, editable preset row is added to the list.
6.  **AC6:** A "Delete" icon must be visible on hover for each preset row.
7.  **AC7:** Given an Editor clicks the "Delete" icon, upon confirmation, the preset row is removed.

#### US609: Data Source Configuration
> As an Admin,
> I want to configure data source connection strings in an environment file,
> so that the application can securely connect to the analytical databases.

**Acceptance Criteria:**
1.  **AC1:** Given the application starts, it must read a configuration (e.g., YAML file) defining available data sources.
2.  **AC2:** Each source definition must include a `source_id`, `friendly_name`, `source_type` (e.g., SYSTEM_UPLOAD_DATASOURCE_ID), connection details, and policy controls (e.g., `caching_type`, `max_data_size`).
3.  **AC3:** Given the configuration is read, when a user creates a Chrono (US105), the list of `friendly_names` must be available to select from.

#### US610: Automatic Schema Discovery (Initial)
> As an Admin,
> I want to have the system automatically discover the schema when a Chrono is created,
> so that metrics and dimensions are set up without manual effort.

**Acceptance Criteria:**
1.  **AC1:** Given a new Chrono is created (per US105), a background job is triggered to scan the specified table.
2.  **AC2:** The scan must identify the primary "timestamp" column. Failure triggers an error ("No date/time field found").
3.  **AC3:** The scan must identify all numeric columns and register them as "Base Metrics" (per US605).
4.  **AC4:** The scan must identify all string/categorical columns and register them as "Dimensions" (per US606).
5.  **AC5:** The scan must identify and store all unique values for low-cardinality dimensions (<= 256 unique values).
6.  **AC6:** A JSON config file is generated and saved to Firestore.
7.  **AC7:** The config must set defaults:
    *   **friendly\_name (Chrono):** Set to table name (per US602).
    *   **friendly\_name (Fields):** Set to field name.
    *   **aggregation (Metrics):** Defaulted to SUM.
    *   **is\_visible:** True for all fields.
    *   **editors:** Creating user is added.
    *   **Default View:** DAY granularity, Last 28 Days.

#### US611: Trigger Manual Schema Rescan
> As an Admin,
> I want to trigger a manual schema rescan (e.g., via an API endpoint),
> so that I can add newly available columns to the Chrono without overwriting my existing configuration.

**Acceptance Criteria:**
1.  **AC1:** Given an Admin calls the `POST /chrono/{id}/rescan` API endpoint, a schema discovery job is triggered.
2.  **AC2:** The scan compares the schema from the source table with the Chrono's current configuration.
3.  **AC3:** By default, if the scan detects that a column corresponding to a configured Base Metric or Dimension is *missing* from the source table, the entire rescan process must fail and return an error message (e.g., "Error: Configured field 'total_revenue' no longer exists in source table.").
4.  **AC4:** By default, if the scan detects that a Base Metric required by a Composite Metric formula is missing from the source table, the entire rescan process must fail and return an error message (e.g., "Error: Rescan failed. Field 'total_revenue' is required by Composite Metric 'revenue_per_user' and is missing from the source table.").
5.  **AC5:** If the scan finds new columns in the source table that are not in the Chrono's configuration, they are added as new metrics or dimensions, set to `is_visible = True` by default.
6.  **AC6:** The scan updates the internal list of values for low-cardinality dimensions, adding any new values discovered.
7.  **AC7:** Given an Admin calls the rescan endpoint with a `force=true` parameter (e.g., `POST /chrono/{id}/rescan?force=true`), the checks in AC3 and AC4 are skipped. Any configured metrics or dimensions that are missing from the source table will be removed from the Chrono's configuration. This will also remove any Composite Metrics that become invalid as a result.

### Group 7: System, Graphing, & Error Handling

#### US701: Graph Loading State
> As a User,
> I want to see a loading indicator on a graph,
> so that I know it is fetching data.

**Acceptance Criteria:**
1.  **AC1:** Given a graph query is initiated (e.g., on page load, filter change), the graph's existing visual contents must be immediately cleared.
2.  **AC2:** The graph's container must maintain its size and position.
3.  **AC3:** The empty container must be replaced with a loading indicator (e.g., a spinner and/or text "Loading Chart Data").
4.  **AC4:** Given the data is successfully fetched, the loading indicator disappears, and the graph renders.

#### US702: Graph Error State
> As a User,
> I want to see a clear error message on a specific graph if its query fails,
> so that I know one graph failed without the whole dashboard crashing.

**Acceptance Criteria:**
1.  **AC1:** Given a graph query fails (e.g., SQL error, timeout), the loading indicator (US701) is replaced with an error state.
2.  **AC2:** The error state must display a clear error icon and a user-friendly message (e.g., "An error occurred while loading this graph").

#### US703: Graph "No Data" State
> As a User,
> I want to see a clear message inside a graph component,
> so that I know my query was successful but returned no data.

**Acceptance Criteria:**
1.  **AC1:** Given a user applies filters or loads a view, when a graph's query executes successfully but returns 0 rows, then the graph area must display a clear, centered message (e.g., "No data available for this selection").
2.  **AC2:** Given a graph is in the "No Data" state, it must not display an error icon or loading spinner.

#### US704: Graph Rendering Requirements
> As a User,
> I want to see graphs rendered with clear, intelligently formatted axes,
> so that I can easily interpret the data.

**Acceptance Criteria:**
1.  **AC1 (Y-Axis Label):** The Y-axis must have a label, which is the `friendly_name` of the metric.
2.  **AC2 (Y-Axis Formatting):** The format of the Y-axis labels must match the metric's `format_type` (e.g., "$", "%", "#").
3.  **AC3 (Y-Axis Growth Override):** If the `IsGrowth` date option is enabled, the Y-axis format must be "Percent" (e.g., "+10%").
4.  **AC4 (Y-Axis Scaling):** The scale must be dynamic. Labels must be abbreviated (e.g., 10K, 25M, 1.2B).
5.  **AC5 (X-Axis Ticks):** The format of the X-axis tick labels must match (or be less granular than) the Date Aggregation (e.g., "Quarter" granularity shows "Q1 2025").
6.  **AC6 (Zero/Null Series):** Given a query returns data, but one of the time series consists entirely of null or zero values, that specific time series must be drawn as a line at the 0 (zero) value on the Y-axis.
7.  **AC7 ("Past Period" Alignment):** When `IsPastPeriod` is on, the past data (e.g., from 2024) must be plotted against the current period's X-axis (e.g., data for "Jan 17, 2024" is plotted at the "Jan 17, 2025" tick).
8.  **AC8 ("Past Period" Color):** The past period lines must use the same base color as their primary time series, but with progressively lighter/more transparent shades.
9.  **AC9 (Data Gaps - Line Chart):** Given a time series contains missing data points (e.g., data for Tuesday is missing between Monday and Wednesday), the line chart must render a visible break in the line, not interpolate across the gap.
10. **AC10 (Data Gaps - Bar Chart):** Given a time series for a bar chart contains missing data points, it must render a zero-height bar for that period.
11. **AC11 (Large Legends):** Given a graph's legend contains a large number of items, the legend container must become scrollable if its height exceeds a predefined maximum (e.g., 30% of the total graph component height).

#### US705: Responsive Graph Grid
> As a User,
> I want the graph grid to be responsive,
> so that I can have an optimal viewing experience on any screen size, from a narrow window to a wide monitor.

**Acceptance Criteria:**
1.  **AC1:** The main graph container must display graphs in a grid with a variable number of columns, from a minimum of 1 to a maximum of 4.
2.  **AC2:** The number of columns must be set based on predefined viewport width thresholds (e.g., 1 column for narrow, 2 for medium, 3 for wide, 4 for very wide).
3.  **AC3:** Given the user resizes the window across a column threshold, the number of columns in the grid must change, and the graphs must immediately reflow.
4.  **AC4:** On wider screens, the grid container must expand horizontally to fill the available screen real estate, causing the individual graph components within the columns to become wider.

#### US706: Handle Data Source Unavailability
> As a User,
> I want to see a clear error message if a Chrono's data source is offline,
> so that I know the problem is not with the application itself.

**Acceptance Criteria:**
1.  **AC1:** Given a user loads a Chrono (per US107), when the application fails to connect to the underlying data source, then the page must display a full-page error (e.g., "Data source '{source\_id}' is unavailable").
2.  **AC2:** This check must happen before any graphs attempt to load.

#### US707: Handle View with Missing Fields
> As a User,
> I want to be clearly notified when I load a Label that references fields that no longer exist,
> so that I understand why some graphs are broken.

**Acceptance Criteria:**
1.  **AC1:** Given a user loads a Label (per US202) that references a field (metric or dimension) that no longer exists (e.g., after a rescan in US611).
2.  **AC2:** A non-blocking popup must be displayed, listing the missing fields (e.g., "Error: The following fields are missing: 'old\_revenue'").
3.  **AC3:** Any graphs in the Label that do not depend on the missing field must render correctly.
4.  **AC4:** Any graphs that do depend on the missing field must display an error state (e.g., "Cannot render: field 'old\_revenue' not found").

#### US708: Handle View with Missing Dimension Values
> As a User,
> I want to be notified when a Label or URL filter references dimension values that no longer exist,
> so that I understand why my data may look different.

**Acceptance Criteria:**
1.  **AC1:** Given a user loads a view (via Label or URL) that contains a filter for a dimension value ID that no longer exists (e.g., after a rescan in US611).
2.  **AC2:** For each such load event, a non-blocking popup must be displayed, listing the invalid filters (e.g., "Warning: The value '1' for dimension 'region' no longer exists and will be ignored.").
3.  **AC3:** All graphs must render, but the invalid filter fragment is silently ignored during the query.

#### US709: System Logging
> As an Admin,
> I want the system to log important events,
> so that I can monitor system health and query performance.

**Acceptance Criteria:**
1.  **AC1:** All application errors must be logged to a cloud logging service.
2.  **AC2:** All data source connection failures (per US706) must be logged as "Critical" errors.
3.  **AC3:** All individual graph query failures (US702) must be logged.
4.  **AC4:** All successful graph queries must be logged with their execution time and `query_hash`.
5.  **AC5:** All query logs (AC3, AC4) must be published to a Google Pub/Sub topic and include the full SQL query text for performance analysis.
6.  **AC6:** Significant usage events must be logged, including (at a minimum) `timestamp`, `user_id`, `action_type` (e.g., "user\_login", "chrono\_create", "label\_save", "config\_edit"), and the `target_id`.

#### US710: Handle Unexpected Application Errors
> As a User,
> I want to see a clear, helpful error page if the application encounters a critical, unexpected error,
> so that I know the system has a problem and I'm not left with a broken or blank screen.

**Acceptance Criteria:**
1.  **AC1:** Given the application experiences a critical, unhandled frontend error, the entire UI must be replaced by a full-page error message.
2.  **AC2:** The error page must display a user-friendly message (e.g., "Sorry, something went wrong.") and a unique error ID.
3.  **AC3:** The error page must provide a "Reload Application" button to allow the user to try and restart their session.

#### US711: Secure Application Startup
> As an Admin,
> I want the application to fail to start if the authentication mode is not explicitly and correctly configured,
> so that the system never runs in an unknown or insecure state.

**Acceptance Criteria:**
1.  **AC1:** Given the `AUTH_MODE` environment variable is not set, when the application attempts to start, it must log a fatal error (e.g., "FATAL: AUTH_MODE is not configured") and exit immediately.
2.  **AC2:** Given the `AUTH_MODE` is set to an invalid value (e.g., "sso"), when the application attempts to start, it must log a fatal error (e.g., "FATAL: Invalid AUTH_MODE 'sso' specified. Must be 'developer' or 'oidc'.") and exit immediately.