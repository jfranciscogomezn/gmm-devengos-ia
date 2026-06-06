## Purpose
Return localized API error messages based on the client Accept-Language header.

## Requirements

### Requirement: API errors respect Accept-Language
The system SHALL resolve the request locale from the `Accept-Language` header. When the header is absent, the default locale SHALL be `es-CO`.

#### Scenario: Spanish default
- **WHEN** a client calls an API without `Accept-Language`
- **THEN** error `message` fields are returned in Spanish (Colombia)

#### Scenario: English requested
- **WHEN** a client sends `Accept-Language: en-US` and receives a 403 error
- **THEN** the `message` field is in English (US)

### Requirement: Known HTTP errors are localized
The system SHALL translate standard authorization, not-found, conflict, and server error messages via message bundles. The JSON response shape MUST remain unchanged.

#### Scenario: Forbidden operation
- **WHEN** an authenticated user lacks permission for an endpoint
- **THEN** the 403 response `message` is localized, not a hardcoded English string in the handler

#### Scenario: User limit reached
- **WHEN** tenant user creation exceeds `max_users`
- **THEN** the 409 `message` is localized and still identifiable by the frontend user-limit handler

### Requirement: Response contract unchanged
The system MUST NOT add or remove fields on existing error DTOs for localization.

#### Scenario: Incomplete report error
- **WHEN** a report is blocked by incomplete records
- **THEN** the response still includes `incompleteDates` and a localized `message`
