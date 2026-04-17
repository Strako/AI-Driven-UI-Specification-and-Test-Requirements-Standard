# AI-Driven UI Specification and Testing Standard

![License](https://img.shields.io/badge/license-Apache%202.0-blue)
![Status](https://img.shields.io/badge/status-active-brightgreen)
![AI](https://img.shields.io/badge/AI-powered-purple)
![Contributions](https://img.shields.io/badge/contributions-welcome-orange)
![Version](https://img.shields.io/badge/version-1.1.0-blue)

A flexible, open framework that uses AI to define, document, and validate UI requirements and tests — enabling scalable and traceable UI design for automated workflows.

---

## How It Works

1. **Central Template Repository** — This repository is the single source of truth for screen specifications. Each view is documented in a structured Markdown file following `TEMPLATE.md`.

2. **Unique ID Generator** — Every entity (view, component) receives a human-readable prefix followed by a system-generated UUID, guaranteeing uniqueness across branches and teams. Format: `<<readable-name-uuid>>`.

3. **Structured Specification** — Each screen is described with well-defined sections: identification, components, field validations, screen states, related views, business rules, actions, and a detailed flow narrative.

4. **AI-Powered Test Generation** — From a filled specification, the `test-generation` skill automatically produces data-agnostic test cases and a separate test data template — covering happy paths, smoke tests, functional validations, edge cases, and exploratory scenarios.

5. **AI-Powered Test Execution** — The `test-execution` skill takes the generated test cases and test data, hydrates placeholders with concrete values, executes every test against the live application via Playwright MCP, and produces a structured execution report with evidence.

6. **Traceability** — Test results are linked to the same documented IDs, enabling auditing, comparison across runs, and full traceability from requirement to evidence.

---

## Syntax Convention

### ID Format

All entities — views and components — are identified using:

```
<<readable-name-uuid>>
```

| Part            | Description                                                               | Example                                    |
| --------------- | ------------------------------------------------------------------------- | ------------------------------------------ |
| `readable-name` | Human-assigned, descriptive, kebab-case label                             | `login-screen`, `auth-form`, `email-input` |
| `uuid`          | System-generated short UUID, ensures uniqueness across branches and teams | `f3a9c1b2`                                 |

**Examples:**

| Entity    | ID                          |
| --------- | --------------------------- |
| View      | `<<login-screen-f3a9c1b2>>` |
| Component | `<<auth-form-c4d7e2a1>>`    |

### Dynamic Placeholder Format

All user-interactable elements are referenced using:

```
${field-name}
```

| Usage                    | Example              |
| ------------------------ | -------------------- |
| Text input               | `${email}`           |
| Password input           | `${password}`        |
| Dropdown selection       | `${country}`         |
| Radio button             | `${payment-method}`  |
| Button / action trigger  | `${submit}`          |
| Link / navigation action | `${forgot-password}` |

> `${field-name}` represents both **what the field is** and **what the user provided or triggered**. All user interactions collapse into this single unified reference regardless of the underlying UI element type.

---

## Screen Description Template

> **One spec file = one view.** Each file defines exactly one `<<view-id>>`. Components and view-level fields are contained within that single view. The full template is defined in [`TEMPLATE.md`](TEMPLATE.md).

### Screen Identification

```markdown
- **View ID**: `<<view-name-uuid>>`
- **Name**: (Descriptive name of the view)
- **Version**: (Design or template version)
- **Route**: /path
```

The **Route** is a relative path. The full URL is resolved by combining `BASE_URL` from [`vars.md`](vars.md) with this route. Never define the domain or protocol in the spec — only the path.

### Origin Context

```markdown
- **Previous View**: `<<previous-view-name-uuid>>`
- **Start Flow**: (Brief description of the flow that leads to this view)
```

### Components

Components are the named UI sections of a view (forms, panels, tables, modals). Each field, button, and interactive element belongs to a component whenever possible.

```markdown
- **Component**: `<<component-name-uuid>>`
  - **Name**: (Human-readable name)
  - **Role**: (What this component does)
  - **Component Validation**: _(optional)_
    - **Rule**: (Component-level rule)
    - **Message**: (Error or feedback message)
    - **Condition**: _(optional)_
  - **Fields**:
    - **Field**: `${field-name}`
      - **Type**: (text | password | email | number | dropdown | radio | checkbox | button | link | file | textarea | other)
      - **Placeholder**: (hint text)
      - **Required**: (yes | no)
      - **Validation**: _(omit if none)_
        - **Rule**: (Validation rule)
        - **Message**: (Error message)
        - **Condition**: _(optional)_
```

### View-Level Fields

For interactive elements that exist directly in the view and do not belong to any component (e.g. a global error banner, a page-level action button).

```markdown
- **Field**: `${field-name}`
  - **Type**: (text | button | link | other)
  - **Required**: (yes | no)
  - **Validation**: _(omit if none)_
    - **Rule**: (...)
    - **Message**: (...)
    - **Condition**: _(optional)_
```

### Screen States

```markdown
- **State**: (state name — e.g. idle, loading, error, success, empty)
  - **Transition to**: `<<view-name-uuid>>` or (state name)
  - **Triggered by**: `${field-name}`
  - **Change Conditions**: (What must happen for the view to enter this state)
```

### Related Views

Views or external services whose flows are required to generate a complete set of test cases.

**Spec File** — for views with their own specification:

```markdown
- **Related View**: `<<view-name-uuid>>` → `./relative/path/to/spec.md`
  - **Relationship**: (Why this view is needed)
  - **Test Context**: (Steps that happen in this external view as part of a cross-view test)
```

**External Service** — for third-party systems with no local spec:

```markdown
- **External Service**: (Service name)
  - **Description**: (Role in the flow)
  - **Test Note**: Not directly tested. Mock or stub in tests.
```

### Business Rules

```markdown
- **Rule**: `<<rule-name-uuid>>`
  - **Description**: (Clear description)
  - **Condition**: (When it applies)
  - **Action**: (What happens — success and/or violation path)
```

### Actions and Transitions

```markdown
- **User Action**: `${field-name}`
  - **Transition**: `<<view-name-uuid>>` or (state name)
  - **Expected Reaction**: (What should happen after this action)
```

### Detailed Flow Description

A step-by-step narrative referencing `<<view-ids>>` for navigation and `${field-name}` for every user interaction, including real-time validations, state transitions, and cross-view scenarios.

---

## Template Usage Example

Below is a condensed example of a login screen specification following the template:

### Screen Identification

- **View ID**: `<<login-screen-f3a9c1b2>>`
- **Name**: Login Screen
- **Version**: 1.0
- **Route**: `/login`

### Origin Context

- **Previous View**: `<<landing-page-a1b2c3d4>>`
- **Start Flow**: The user arrives from the main page by triggering `${sign-in}`.

### Components

- **Component**: `<<auth-form-c4d7e2a1>>`
  - **Name**: Authentication Form
  - **Role**: Captures and submits user credentials
  - **Component Validation**:
    - **Rule**: `${submit}` is enabled only when both `${email}` and `${password}` pass their validation rules
    - **Message**: N/A
  - **Fields**:
    - **Field**: `${email}`
      - **Type**: email
      - **Placeholder**: you@example.com
      - **Required**: yes
      - **Validation**:
        - **Rule**: Must be a valid email address format
        - **Message**: "Please enter a valid email"
    - **Field**: `${password}`
      - **Type**: password
      - **Placeholder**: ••••••
      - **Required**: yes
      - **Validation**:
        - **Rule**: Must not be empty and must contain at least 6 characters
        - **Message**: "Password must be at least 6 characters"
    - **Field**: `${submit}`
      - **Type**: button
      - **Required**: no
    - **Field**: `${forgot-password}`
      - **Type**: link
      - **Required**: no

### Screen States

- **State**: `start`
  - **Transition to**: `validating`
  - **Triggered by**: `${submit}`
  - **Change Conditions**: Both `${email}` and `${password}` pass validation

- **State**: `validating`
  - **Transition to**: `login-success` / `login-failed`
  - **Triggered by**: API response from `<<auth-api-e5f6a7b8>>`
  - **Change Conditions**: Waiting for server response

- **State**: `login-success`
  - **Transition to**: `<<dashboard-screen-b2c3d4e5>>`
  - **Triggered by**: Valid authentication token received
  - **Change Conditions**: Successful authentication

- **State**: `login-failed`
  - **Transition to**: `start`
  - **Triggered by**: Invalid token or server error
  - **Change Conditions**: Authentication failed, error message shown

### Related Views

**External Service:**

- **External Service**: Authentication API (`<<auth-api-e5f6a7b8>>`)
  - **Description**: Validates user credentials and returns a session token
  - **Test Note**: Not directly tested. Mock or stub this service in tests.

### Business Rules

- **Rule**: `<<rule-auth-access-d1e2f3a4>>`
  - **Description**: The user may only access the dashboard upon successful authentication
  - **Condition**: If the token returned by `<<auth-api-e5f6a7b8>>` is valid
  - **Action**: Redirect to `<<dashboard-screen-b2c3d4e5>>`; if invalid, remain on `<<login-screen-f3a9c1b2>>` and display an error

### Actions and Transitions

- **User Action**: `${submit}`
  - **Transition**: State `validating`
  - **Expected Reaction**: `<<auth-form-c4d7e2a1>>` sends `${email}` and `${password}` to `<<auth-api-e5f6a7b8>>`

- **User Action**: `${forgot-password}`
  - **Transition**: `<<forgot-password-screen-c3d4e5f6>>`
  - **Expected Reaction**: User navigates away from `<<login-screen-f3a9c1b2>>`

### Detailed Flow Description

The user arrives at `<<login-screen-f3a9c1b2>>` from `<<landing-page-a1b2c3d4>>` by triggering `${sign-in}`. The view renders `<<auth-form-c4d7e2a1>>` in its initial state. The user provides `${email}` and `${password}`. Validation is evaluated in real time. Once both fields are valid, `${submit}` becomes active. Upon triggering `${submit}`, the view transitions to `validating` and dispatches credentials to `<<auth-api-e5f6a7b8>>`. If the API returns a valid token, the user is redirected to `<<dashboard-screen-b2c3d4e5>>`. If the API returns an error, the view returns to `start` and displays an error message.

---

## LLM Instructions — Test Case Generation

The template includes a dedicated section with rules that govern how an LLM generates test cases from a filled specification. These rules are defined in full in [`TEMPLATE.md`](TEMPLATE.md) and enforced by the `test-generation` skill. The key principles are:

1. **Data-Agnostic** — Test cases reference `${field-name}` placeholders, never concrete values. Cross-view fields use `<<view-id>>.${field-name}`.
2. **Complete Coverage** — Every field validation, component validation, view-level validation, screen state, business rule, action, and cross-view scenario must be covered.
3. **Cross-View Tests** — For each Related View spec file, generate self-contained test cases that start in the related view and assert in the current view.
4. **Separate Test Data** — A standalone test data template is always produced alongside test cases, organized by scenario and grouped by source view.
5. **Independence** — Test cases and test data are never merged. The user fills the test data template before execution; the test cases themselves never change.

---

## Project Configuration — `vars.md`

The file [`vars.md`](vars.md) at the project root defines global variables used across all specifications and skills.

```markdown
BASE_URL = https://your-app-url.example.com
```

Currently it defines:

| Variable   | Purpose                                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| `BASE_URL` | The domain prepended to every `Route` defined in spec files to construct full URLs for navigation steps. |

Both skills (`test-generation` and `test-execution`) read `vars.md` before processing any specification or test case. Spec files define only the relative path (e.g. `/login`); the full URL is always resolved as `BASE_URL` + `Route`.

---

## AI Skills

This project includes two Kiro skills located in `.kiro/skills/`. Skills are AI-driven automation routines that can be triggered from the IDE to perform structured tasks.

### `test-generation`

**Location:** `.kiro/skills/test-generation/SKILL.md`

Generates a complete set of test cases and a test data template from a UI screen specification file.

**Trigger:** When the user asks to generate tests, test cases, or QA coverage for a view spec.

**What it does:**

1. Reads the UI spec file, all related view specs, and `vars.md`.
2. Applies the conventions from `TEMPLATE.md` — `${field-name}` placeholders, `<<view-id>>` references, full URL construction.
3. Determines optimal coverage across five test types: Happy Path, Smoke, Functional, Edge Case, and Exploratory.
4. Produces **two artifacts** in the same directory as the spec file:
   - `test-cases.md` — Data-agnostic test cases using only placeholders.
   - `test-data.md` — A fillable template where the user provides concrete values for each scenario.

### `test-execution`

**Location:** `.kiro/skills/test-execution/SKILL.md`

Executes all test cases against the live application using Playwright MCP and generates a structured execution report.

**Trigger:** When the user asks to run, execute, or validate test cases against a live application.

**What it does:**

1. Reads the test cases file, test data file, `vars.md`, and the UI spec (if present).
2. Hydrates `${field-name}` placeholders with concrete values from the test data and resolves `<<view-id>>` to full URLs.
3. Executes every test case sequentially via Playwright MCP — navigating, clicking, typing, uploading files, taking snapshots, and capturing screenshots.
4. Classifies each result as ✅ PASS, ❌ FAIL, or ⚠️ BLOQUEADO.
5. Produces a **structured execution report** (`test-report-{module}.md`) in the same directory, following a strict format:
   - Header with module name, URL, date, and tool
   - Resumen Ejecutivo table with per-category breakdown and success rate
   - Five mandatory sections: SMOKE TESTS, HAPPY PATH, FUNCTIONAL TESTS, EDGE CASES, EXPLORATORY TESTS
   - Detalle de Fallos with step, expected vs actual, cause, and evidence
   - Detalle de Tests Bloqueados with reason
   - Screenshots Capturados table

### Skill Workflow

The two skills form a pipeline:

```
UI Spec (.md)
    │
    ▼
┌──────────────────┐
│  test-generation  │  → test-cases.md + test-data.md
└──────────────────┘
    │
    ▼  (user fills test-data.md with concrete values)
    │
┌──────────────────┐
│  test-execution   │  → test-report-{module}.md + screenshots
└──────────────────┘
```

---

## Project Structure

```
.
├── TEMPLATE.md                              # The specification template standard
├── vars.md                                  # Global variables (BASE_URL)
├── README.md                                # This file
├── .kiro/
│   └── skills/
│       ├── test-generation/SKILL.md         # Test case generation skill
│       └── test-execution/SKILL.md          # Test execution skill
│
├── <module>/                                # One folder per module or view
│   ├── <view>-description.md                # (1) UI screen specification — input to test-generation
│   ├── test-cases.md                        # (2) Generated test cases — output of test-generation
│   ├── test-data.md                         # (3) Test data template — filled by user before execution
│   └── test-report-<module>.md              # (4) Execution report — output of test-execution
│
└── <module-with-submodules>/                # Modules can be nested for complex flows
    ├── <sub-module-a>/
    │   ├── <view>-description.md            # (1) UI screen specification
    │   ├── test-cases.md                    # (2) Generated test cases
    │   ├── test-data.md                     # (3) Test data template
    │   └── test-report-<sub-module-a>.md    # (4) Execution report
    └── <sub-module-b>/
        ├── <view>-description.md            # (1) UI screen specification
        ├── test-cases.md                    # (2) Generated test cases
        ├── test-data.md                     # (3) Test data template
        └── test-report-<sub-module-b>.md    # (4) Execution report
```

> Each module folder always contains the same four artifacts, produced and consumed in order: the spec drives test generation, the test data is filled by the user, and execution produces the report.

---

## VS Code Extension

> **Status: In Progress**

A VS Code extension to automatically validate `<<readable-name-uuid>>` IDs across the current branch — ensuring no duplicates or skipped references across teams — is currently under development and will be added to this repository once available.

---

## License

Copyright 2026 Armando Isai Hernandez Ibarra

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
