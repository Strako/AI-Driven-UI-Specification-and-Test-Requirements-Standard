---
name: test-execution
description: Execute all test cases from a test-cases file using Playwright MCP via the playwright-testing power, hydrating steps with test-data values, capturing evidence, and generating a structured execution report. Use when the user asks to run, execute, or validate test cases against a live application.
---

## Test Execution

You are acting as a QA automation engineer. Execute every test case defined in a `test-cases` file against the live application using the **playwright-testing** power (Playwright MCP), then produce a structured execution report.

---

### Step 0 — Activate the Playwright power

Before any browser interaction, activate the `playwright-testing` power:

1. Call `action="activate"`, `powerName="playwright-testing"`.
2. Review the returned `toolsByServer` to identify the server name and available tools.
3. Use these tools for all subsequent browser interactions (navigate, click, fill, screenshot, snapshot, etc.).

---

### Step 1 — Read required files

Read the following files **in order** before executing anything:

1. **Test cases file** — the file the user indicated (e.g. `test-cases-vacantes.md`, `test-cases.md`). Parse every test case extracting: ID, Type, Description, Preconditions, Steps, and Expected Result.
2. **Test data file** — located in the same directory, named `test-data.md` (or the variant the user indicates). For each TC ID, extract the concrete values that replace `${field-name}` placeholders.
3. **`vars.md`** at the project root — extract `BASE_URL`. This is the domain prepended to every route when navigating.
4. **UI spec file** (if present in the same directory) — read it to understand `<<view-id>>`, component structure, field types, and validation messages. This helps you locate elements in the DOM.

---

### Step 2 — Hydrate test cases with test data

For every test case that has a matching entry in the test data file:

- Replace each `${field-name}` placeholder in the Steps and Preconditions with the concrete value from the test data.
- Replace each `<<view-id>>` with the corresponding URL constructed as `BASE_URL` + route from the spec.
- Keep the original TC ID, Type, and Description unchanged.
- If a test case has **no matching test data entry** and its steps require input values, mark it as `⚠️ BLOQUEADO` with reason: "Sin datos de prueba definidos para este caso".

---

### Step 3 — Execute test cases

Execute each test case sequentially using Playwright MCP tools. Follow these rules strictly:

#### 3.1 — Execution rules

- Execute **every** test case. Do not skip any.
- Follow the steps **exactly** as defined. Do not modify, assume, or extend scenarios.
- Before each test case, navigate to the required URL from the preconditions (use `BASE_URL` + route).
- For each step, use the appropriate Playwright tool:
  - **Navigate**: `browser_navigate`
  - **Click**: `browser_click`
  - **Fill/Type**: `browser_type`
  - **Select dropdown**: `browser_click` on the dropdown, then `browser_click` on the option
  - **Upload file**: `browser_file_upload` (if available) or the appropriate tool
  - **Observe/Assert**: `browser_snapshot` to read the current DOM state, then verify the expected result
  - **Screenshot**: `browser_take_screenshot` for evidence capture
  - **Resize viewport**: `browser_resize` for responsive tests

#### 3.2 — Evidence capture

- Take a **screenshot** at the end of every test case execution.
- For **failed** test cases, take an additional screenshot at the exact step where the failure occurred.
- Name screenshots using the pattern: `{TC-ID}-{short-description}.png` (e.g. `TC-SMK-01-pagina-cargada.png`).
- Save all screenshots in the same directory as the test cases file.

#### 3.3 — Result classification

Classify each test case result as:

| Status    | Emoji | Condition                                                                    |
| --------- | ----- | ---------------------------------------------------------------------------- |
| PASS      | ✅    | All steps completed and expected result matches observed behavior            |
| FAIL      | ❌    | One or more steps did not produce the expected result                        |
| BLOQUEADO | ⚠️    | The test cannot be executed due to environment, data, or tooling limitations |

For **FAIL** results, record:

- The **exact step** where the error occurred
- The **expected result** (from the test case)
- The **actual result** (what was observed)
- The **probable cause** of the failure
- The **screenshot filename** as evidence

For **BLOQUEADO** results, record:

- The **reason** why the test could not be executed

---

### Step 4 — Generate the execution report

After all test cases have been executed, generate a report file named `test-report-{module-name}.md` in the same directory as the test cases file.

The report **MUST** follow this exact structure, sections, style, and level of detail. No sections may be omitted, renamed, or restructured.

---

#### 4.1 — Header

```markdown
# Reporte de Ejecución de Tests — {Nombre del módulo}

**URL:** `{full URL}`
**Fecha:** {YYYY-MM-DD}
**Ejecutado con:** Playwright MCP (`@playwright/mcp`)

---
```

#### 4.2 — Resumen Ejecutivo

A summary table with columns: Categoría, Total, ✅ Exitosos, ❌ Fallidos, ⚠️ Bloqueados.

- Include one row per test type (Smoke Tests, Happy Path, Functional Tests, Edge Cases, Exploratory Tests).
- Include a **TOTAL** row.
- Add a note if there are discrepancies in counting (e.g. sub-verifications).
- Calculate and display: **Tasa de éxito: X/Y (Z%)**

```markdown
## Resumen Ejecutivo

| Categoría         | Total | ✅ Exitosos | ❌ Fallidos | ⚠️ Bloqueados |
| ----------------- | ----- | ----------- | ----------- | ------------- |
| Smoke Tests       | N     | N           | N           | N             |
| Happy Path        | N     | N           | N           | N             |
| Functional Tests  | N     | N           | N           | N             |
| Edge Cases        | N     | N           | N           | N             |
| Exploratory Tests | N     | N           | N           | N             |
| **TOTAL**         | **N** | **N**       | **N**       | **N**         |

**Tasa de éxito: X/Y (Z%)**

---
```

#### 4.3 — Test sections

Create the following sections **ALWAYS**, even if all tests pass:

```
## SMOKE TESTS
## HAPPY PATH
## FUNCTIONAL TESTS
## EDGE CASES
## EXPLORATORY TESTS
```

Each section must:

- Show a summary line: `(X/Y — N estado)` or `(X/Y ✅)` if all pass.
- Include a table with columns: `| ID | Descripción | Resultado | Detalle |`
- The **Detalle** column must explain what was validated and what occurred — it must be clear, concise, and verifiable.

```markdown
## SMOKE TESTS (X/Y ✅)

| ID        | Descripción | Resultado | Detalle                        |
| --------- | ----------- | --------- | ------------------------------ |
| TC-SMK-01 | Description | ✅ PASS   | What was verified and observed |
```

#### 4.4 — Detalle de Fallos

Include **ONLY** if there are failed tests. For each failed test:

```markdown
## Detalle de Fallos

### {ID} — {Nombre del test}

**Paso donde ocurrió el error:** {Specific step}
**Resultado esperado:** {Expected behavior}
**Resultado obtenido:** {What actually happened}
**Motivo del fallo:** {Probable or technical cause}
**Evidencia:** {Screenshot filename}
```

#### 4.5 — Detalle de Tests Bloqueados

Include **ONLY** if there are blocked tests. For each blocked test:

```markdown
## Detalle de Tests Bloqueados

### {ID} — {Nombre del test}

**Razón:** {Clear explanation of why the test could not be executed}
```

#### 4.6 — Screenshots Capturados

A table listing all captured screenshots:

```markdown
## Screenshots Capturados

| Archivo        | Descripción                              |
| -------------- | ---------------------------------------- |
| `filename.png` | Description of what the screenshot shows |
```

---

### Step 5 — Consistency rules

The report **MUST** comply with these rules:

- Maintain titles, subtitles, and separators (`---`) exactly as shown.
- Use Markdown tables with correct alignment.
- Use status emojis consistently: ✅ PASS, ❌ FAIL, ⚠️ BLOQUEADO.
- Keep section names in UPPERCASE where specified.
- Include aggregate metrics (totals, success rate).
- Write in **technical Spanish**.
- TC IDs must follow the format from the test cases file (e.g. `TC-SMK-01`, `TC-HP-01`, `TC-001`).
- Descriptions must be clear, concise, and verifiable.
- Details must explain what was validated and what occurred.
- Use present tense verbs.
- Do not mix languages.
- Details must be verifiable (not ambiguous).
- Clearly explain causes in failures and blocked tests.
- Do **NOT** change section names, structure, summarize excessively, omit tables, or use free format.

---

### Step 6 — Report completion

Once the report is written, summarize:

- Path to the generated report file.
- Total test cases executed, broken down by result (✅, ❌, ⚠️).
- Success rate percentage.
- Number of screenshots captured.
- Key findings or issues discovered during execution (if any).
