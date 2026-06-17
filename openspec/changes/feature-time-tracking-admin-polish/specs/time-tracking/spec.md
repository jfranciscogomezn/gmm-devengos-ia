## ADDED Requirements

### Requirement: Admin can view earnings breakdown in the time records list
The admin time records screen SHALL allow an administrator to optionally show the calculated earnings breakdown alongside raw time records for any employee. The earnings view SHALL support a capped/uncapped toggle. The admin SHALL NOT need to navigate away from the time records screen to verify per-day devengado values.

#### Scenario: Admin enables earnings view
- **WHEN** an admin selects an employee, sets a date range, and enables the "Show earnings" toggle
- **THEN** the time records table is replaced with the earnings-enriched report table showing classified minutes, earnings, and highlight indicators

#### Scenario: Admin toggles uncapped view
- **WHEN** the admin enables the "Uncapped view" sub-toggle while earnings are shown
- **THEN** the table displays uncapped classified minutes and uncapped earnings per the existing uncapped report contract

#### Scenario: Earnings unavailable for open record
- **WHEN** a time record has status OPEN or INCOMPLETE and the earnings view is enabled
- **THEN** the earnings columns display a dash indicator and no calculation error is raised

#### Scenario: Earnings view disabled by default
- **WHEN** an admin opens the time records screen
- **THEN** the plain time records table is shown without earnings columns and no report query is issued
