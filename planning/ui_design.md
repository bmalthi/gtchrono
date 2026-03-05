Here is the content converted to Markdown.

---

# Page 1: Chrono List Screen

This is the main landing page for a logged-in user. It allows them to see all the "Chronos" (dashboards) they can access and provides the main navigation paths to load or create a new one.

## 1. Overall Page Layout
- **Structure**: A simple, clean, single-column layout that is centered in the browser.
- **Background**: A light neutral background color for the main page body.
- **Content Container**: The main content (title, button, and list) should be placed within a wide, centered container to give it margins on a desktop view.

## 2. Component: Main Header / Navigation
This component is fixed at the top of the page.
- **Left Side**:
  - **App Logo**: An icon for the application.
  - **App Name**: The text "Chronoscope."
- **Right Side**:
  - **Navigation Links**: Text links for "Chronos" (this should be in an *active* visual state) and "Docs."
  - **User Icon**: A "User Profile" icon on the far right. This should be designed to look like a dropdown menu trigger.

## 3. Component: Content Header
This component sits below the Main Header and titles the content area.
- **Left Side**:
  - **Page Title**: A clear, large-font heading with the text "Available Chronos".
- **Right Side**:
  - **Primary Button**: A high-contrast primary action button (e.g., blue).
  - **Button Text**: "Create new Chrono".
  - **Alignment**: This button should be vertically aligned with the Page Title.

## 4. Component: Chrono List Container
This component sits below the Content Header and holds the list of Chrono items.
- **Visuals**: A container (like a card with a border or a subtle shadow) that visually groups all the list items.
- **Component States**: This container needs two states:
  - **List State (Default)**: This state displays the list of Chrono Items.
  - **Empty State**: This state is shown when there are no Chronos in the list.

## 5. Sub-Component: Chrono List Item
This is the repeating component inside the List Container. Each row represents one Chrono.
- **Layout**: A single-row item with elements arranged horizontally. The entire row should have a clear hover state.
- **Required Elements (per row)**:
  - **Friendly Name (Left Side)**: The main text element for the row (e.g., "Sales Analytics").
    - **Standardized Action**: Clicking this text loads the Chrono.
  - **Action Group (Right Side)**:
    - **"Load" Button**: A clear call-to-action button (secondary or "ghost" button).
      - **Standardized Action**: Clicking this button loads the Chrono.
    - **"Delete" Button**: A "delete" icon button (e.g., a trash can icon). This is a secondary, destructive action.
- **Visual States**: Must have an *Active State* (clickable) and a *Disabled State* (greyed-out, not clickable).

## 6. Sub-Component: Empty State View
This is the visual that appears inside the Chrono List Container when there are no items to show.
- **Visuals**: Design a simple, centered message.
- **Text**: Include a welcoming message, for example: "Welcome! There are no Chronos available to display.".
- **Context**: The "Create new Chrono" button in the Content Header will still be visible.

---

# Page 2: Create New Chrono

This is a full-page screen where a user goes after clicking the "Create new Chrono" button.

## 1. Overall Page Layout
- **Structure**: The layout should be identical to the "Chrono List Screen": a fixed Main Header at the top, followed by the main page content centered in a single column.

## 2. Component: Main Header / Navigation
- **Visuals**: This component is identical to the header on the "Chrono List Screen".
- **State**: The "Chronos" link should remain in the *active* visual state.

## 3. Standardized Component: Content Header
This component sits below the Main Header.
- **Left Side**:
  - A large-font heading: "Create New Chrono".
- **Right Side**:
  - **Standardized Back Link**: A simple text link.
  - **Text**: "Back to Chronos List".
  - **Action**: Returns the user to the Chrono List Screen.

## 4. Component: Form Container
This is the main content of the page, centered below the Content Header.
- **Visuals**: A card-like container with a border or shadow, visually grouping the form elements.
- **Form Elements**:
  - **Data Source Dropdown**: (Label: "Data Source:")
  - **Table Dropdown**: (Label: "Table:")
  - **Helper Text**: Include a small text area below the form fields.
    - **Text**: "Creating a new Chrono will scan the table schema. This may take a moment."
  - **Standardized Primary Button (Simple Form)**: A high-contrast, primary action button centered inside the card at the bottom.
    - **Text**: "Create Chrono".

## 5. Required States for the Form
Visual mockups are required for the following states:
- **Default State**: "Data Source" is enabled. "Table" is disabled. "Create Chrono" button is disabled.
- **Table Selection State**: "Data Source" is selected. "Table" is enabled. "Create Chrono" button is still disabled.
- **Ready State**: "Data Source" and "Table" are both selected. "Create Chrono" button is enabled and in its active primary color.
- **Loading State**: User has clicked "Create Chrono." Both dropdowns are disabled. The "Create Chrono" button shows a loading state (e.g., spinner, text changes) and is disabled.
- **Error State**: An error message (e.g., in a red-colored box or text) must appear above the "Create Chrono" button. All form elements return to the *Ready State*.

---

# Page 3: Analysis View Screen

This is the "workbench" for data analysis, featuring a persistent set of controls on the left and a main content area for graphs on the right.

## 1. Overall Page Layout
- **Structure**: A two-column layout.
  - **Left Sidebar**: A fixed-width (e.g., ~280-320px) column on the left.
  - **Main Content Area**: The primary, flexible-width column on the right.
- **Header**: The Main Header is present and fixed at the very top.

## 2. Component: Chrono Header
This component sits at the top of the Main Content Area, just below the Main Header.
- **Layout**: A single horizontal bar.
- **Left Side**:
  - **App Icon**: The app's logo icon.
  - **Chrono Title**: The "friendly name" of the Chrono.
- **Right Side (Action Group)**:
  - **Date Checkboxes**: "Growth" and "Past Period".
  - **Comparison Dropdown**: (e.g., "vs 7 Days Ago"). This should be disabled if both checkboxes are unchecked.
  - **Label Controls**: "Load Label" (dropdown) and "Save Label" (button).
  - **Edit Button**: An "Edit" (or gear) icon button. This button is only visible for users with "Editor" permissions.

## 3. Component: Left Sidebar
Contains all filtering controls.
1.  **Date Filter Panel**:
    - "Date Range" Dropdown (e.g., "Last 7 Days").
    - Start/End Date pickers.
2.  **Date Aggregation Panel**:
    - "Date Aggregation" Dropdown (e.g., "Day," "Week").
3.  **Time Series Legend Panel**:
    - **Legend**: A list of active time series (small colored squares. Maybe up to 10 wide before overflowing to the next row).
    - **Actions**: Text actions "% of total", "% of this", and "unset %". The "% of this" action must have a *Disabled State*.
4.  **Dimensions Panel**:
    - **Section Title**: "Dimensions."
    - **Dimension Items**: A list of all available dimensions (e.g., "Channel," "Region").
5.  **Action Button (Sticky Footer)**:
    - **"Apply Changes" Button**: A primary, high-contrast button at the bottom of the sidebar (sticky on overflow).
    - **Button States**: Must have a *Default State* (disabled/grey) and an *Active State* (high-contrast/blue, shown when a filter is changed).

## 4. Component: Main Content Area
1.  **View Tab Bar**:
    - A tab bar must appear above the graphs.
    - **Tabs**: "Default," "Pinned," and any user-created tabs (which need a "Close" X icon).
2.  **Graph Grid**:
    - A responsive grid (1-4 columns) that holds all the graph cards.

## 5. Sub-Component: Graph Card
A self-contained visualization.
1.  **Graph Header**:
    - **Title**: The graph's main title (e.g., "Revenue").
    - **Subtitle (Optional)**: Contextual text.
2.  **Icon Tray**:
    - A cluster of icon buttons in the top-right.
    - **Required Icons**: "Pin," "Expand," "Edit Metadata," and "Save" (dropdown for Download Image/CSV, See SQL).
3.  **Chart Area**:
    - The main area for the chart.
4.  **Required States for Chart Area**:
    - **Loading State**: A loading indicator (e.g., spinner).
    - **Error State**: An error icon and message.
    - **No Data State**: A clear "No data available" message.

---

# Page 4: Label List Screen

This page displays all saved "Labels" (views) for a single Chrono.

## 1. Overall Page Layout
- **Structure**: A full page, single-column, centered layout (identical to Page 1).
- **Background**: A light neutral background color.

## 2. Component: Main Header / Navigation
- **Visuals**: This component is identical to the header on the previous screens.
- **State**: The "Chronos" link should remain in the *active* visual state.

## 3. Standardized Component: Content Header
This component sits below the Main Header.
- **Left Side (Page Title)**:
  - **Primary Title**: "Available Labels"
  - **Subtitle**: A subtitle showing context (e.g., "For: Sales Analytics").
- **Right Side (Navigation)**:
  - **Standardized Back Link**: A simple text link.
  - **Text**: "Back to Analysis View"
  - **Alignment**: This link should be vertically aligned with the Page Title.

## 4. Component: Label List Container
This component sits below the Content Header.
- **Visuals**: A card-like container, identical to the "Chrono List Screen" container.
- **Component States**:
  - **List State (Default)**: Displays the list of "Label Items."
  - **Empty State**: This state is shown when there are no saved Labels.

## 5. Sub-Component: Label Item
This is the repeating component inside the List Container.
- **Layout**: A single row with a clear hover state.
- **Required Elements (per row)**:
  - **Label Name (Left Side)**:
    - **Text**: The `label_name`.
    - **Standardized Action**: Clicking this text loads the Label in the Analysis View.
  - **Action Group (Right Side)**:
    - **Standardized "Load" Button**: A clear call-to-action button (secondary or "ghost" button).
      - **Standardized Action**: Clicking this button loads the Label.
    - **"Edit Name" Icon**: (e.g., a "pencil" icon).
      - **Visibility**: This icon is visible only to the user who owns the Label.
      - **Action**: Clicking this icon should make the Label Name text on the left transform into an editable text input field.
    - **"Toggle Visibility" Icon**: (e.g., an "eye" or "globe" icon).
      - **Visual States**: Must have an *Active State* (clickable by owner) and a *Disabled State* (greyed-out for non-owners).
    - **"Delete" Icon**: (e.g., a "trash can" icon).
      - **Visibility**: This icon is visible only to the user who owns the Label.

## 6. Sub-Component: Empty State View
- **Visuals**: A simple, centered message.
- **Text**: "No Labels have been saved for this Chrono. Go to the Analysis View and click 'Save Label' to create one."

---

# Page 5: Edit Chrono Screen

A dedicated settings page for a single Chrono, accessible via the "Edit" button on the Analysis View.

## 1. Overall Page Layout
- **Structure**: A simple, single-column layout, identical to the "Create Chrono" screen.
- **Header**: The Main Header is fixed at the top.
- **Content**: The main content is a single, large, centered Settings Card.

## 2. Standardized Component: Content Header
This component sits outside the Settings Card, just below the Main Header.
- **Left Side (Page Title)**:
  - **Primary Title**: "Edit Chrono"
  - **Subtitle**: Show the friendly name of the Chrono being edited (e.g., "For: Sales Analytics").
- **Right Side (Action)**:
  - **Standardized Secondary Link**:
    - **Button Text**: "Cancel"
    - **Action**: This button discards all changes (with no confirmation) and returns the user to the Analysis View.
  - **Standardized Primary Button (Complex Form)**: A high-contrast primary button.
    - **Button Text**: "Done"
    - **Action**: This button saves all changes and returns the user to the Analysis View.

## 3. Component: Settings Card
This is the main container for all settings, centered on the page.
- **Structure**: The card uses a horizontal tab navigation at the top, which swaps the content panel displayed below it.

### Sub-Component: Settings Tab Bar
- **Tabs**: "General," "Permissions," "Base Metrics," "Composite Metrics," "Dimensions," and "Date Presets".

## 4. Settings Panels (Content for each tab)

- **Panel 1: General (Default Tab)**
  - **Chrono Name**: Text input.
  - **Access Denied Message**: Multi-line text area.
  - **Helper Text**: (Subtle text below) "This message will be shown to users who try to access this Chrono but do not have permission."

- **Panel 2: Permissions**
  - **Add User**: "User Email" input, "Role" dropdown ("Viewer," "Editor"), and an "Add" button.
  - **Current Users**: A list of all users.
    - **Row Design**: Each row must contain User's Email/Name, Role Dropdown, and "Remove" (trash icon) button.
    - **Visual State**: The controls (dropdown, remove) for the last remaining Editor must have a *Disabled State*.

- **Panel 3: Base Metrics**
  - **UI**: A list of all metrics.
  - **Row Design**: A "disclosure" element.
    - ***Collapsed State***: Shows "Friendly Name" and a "Visible" / "Hidden" toggle switch.
    - ***Expanded State***: Shows inputs for `friendly_name`, `description`, `format_type`, `time_aggregation`, and `dim_aggregation`.

- **Panel 4: Composite Metrics**
  - **UI**: A list of user-created metrics, plus a "Create New" button.
  - **Row Design**: Identical to "Base Metrics" panel.
    - ***Expanded State***: Shows inputs for `friendly_name`, `metric_name` (ID), `formula`, and `description`.

- **Panel 5: Dimensions**
  - **UI**: A list of all dimensions.
  - **Row Design**: A disclosure element.
    - ***Collapsed State***: Shows "Friendly Name" and a "Visible" / "Hidden" toggle switch.
    - ***Expanded State***: Shows `friendly_name` text input and (if applicable) a two-column "Value Map" table.

- **Panel 6: Date Presets**
  - **UI**: A list of rows, with an "Add Preset" button at the bottom.
  - **Row Design**: Each row must contain all fields in-line: "Friendly Name" (input), "Type" (dropdown), dynamic "Definition Fields", and a "Delete" (trash icon) button.