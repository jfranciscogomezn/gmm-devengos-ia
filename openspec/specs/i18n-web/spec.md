## Purpose
Allow users to choose and persist the web application language between es-CO and en-US.

## Requirements

### Requirement: Users can choose the web application language
The system SHALL allow authenticated and unauthenticated users to select between `es-CO` and `en-US` from the web UI. The selected locale SHALL persist across browser sessions until changed.

#### Scenario: Default locale on first visit
- **WHEN** a user opens the application without a stored locale preference
- **THEN** the UI renders in `es-CO`

#### Scenario: User switches language
- **WHEN** the user selects `en-US` from the language control
- **THEN** all frontend-managed labels, buttons, and local messages update without a full page reload beyond i18n refresh

#### Scenario: Preference persists
- **WHEN** the user reloads the browser after choosing a locale
- **THEN** the same locale is applied

### Requirement: Frontend UI copy is externalized
The system SHALL not hardcode user-visible strings in React components except as translation keys. Labels, buttons, placeholders, table headers, and local validation or empty-state messages SHALL come from locale resource files.

#### Scenario: Login form in Spanish
- **WHEN** the active locale is `es-CO`
- **THEN** the login form labels and submit button appear in Spanish

### Requirement: Navigation menu labels are translated in the frontend
The system SHALL display sidebar menu labels using stable menu node codes mapped in frontend locale files. The API-provided label SHALL be used only as a fallback when no translation exists.

#### Scenario: Menu item by code
- **WHEN** the login menu includes an item with code `MY_TIME` and locale `es-CO`
- **THEN** the sidebar shows the Spanish label from the frontend catalogue, not necessarily the English database label

### Requirement: Locale-aware formatting
The system SHALL format dates, times, and currency amounts according to the active locale (`es-CO` or `en-US`).

#### Scenario: Money in Colombian locale
- **WHEN** locale is `es-CO` and a report displays earnings
- **THEN** amounts use Colombian Spanish number formatting

### Requirement: API requests advertise the user locale
The system SHALL send an `Accept-Language` header matching the user's selected locale on security and business API requests.

#### Scenario: Header on authenticated request
- **WHEN** the user locale is `en-US` and the frontend calls a business API
- **THEN** the request includes `Accept-Language: en-US`

### Requirement: Backend data and structures remain unchanged
The system MUST NOT alter database schema, menu catalogue data, entity fields, or API response shapes for i18n in the frontend increment.

#### Scenario: Menu API unchanged
- **WHEN** the frontend loads the login menu
- **THEN** the API response shape and stored labels are unchanged from the pre-i18n contract
