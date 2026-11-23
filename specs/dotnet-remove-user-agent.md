# Change specification: remove the .NET BookCatalog browser diagnostic

## Metadata

- **Owner:** Workshop application team
- **Status:** Draft
- **Source root:** `legacy-src/dotnet-bookcatalog/shared-legacy-app`
- **Related issue:** Not created

## Problem and outcomes

### Problem

The BookCatalog index action in
`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
reads `HttpContext.Request.UserAgent` and stores the result in
`ViewBag.UserAgent` (`BooksController.cs:13-20`). The index view at
`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`
renders that value below the catalog table as `YOUR BROWSER`
(`Index.cshtml:67-71`). This exposes request metadata and couples the catalog
index to browser request information even though the diagnostic is not catalog
business behavior.

Users who open `/` or `/Books/Index` are affected by the diagnostic. Repository
evidence does not identify an external consumer of the diagnostic. Whether it
has a known consumer remains an unresolved approval question.

### Desired outcomes

1. The index page continues to show active books in title order with existing
   table columns and CRUD links.
2. The page does not read or render the request `User-Agent` value or a
   `YOUR BROWSER` diagnostic.
3. Requests with a missing or unusual `User-Agent` header render the catalog
   normally.

### Success measures

- Source review finds no `HttpContext.Request.UserAgent`,
  `ViewBag.UserAgent`, or `YOUR BROWSER` reference in the two allowed files.
- A manual request with a synthetic `User-Agent` returns the catalog page
  without that value or label.
- The existing active-filter and title-order behavior remains unchanged.
- The allowed validation command is successful or a precise prerequisite
  blocker is recorded.

## Functional requirements

| ID | Requirement | Evidence or rationale |
| --- | --- | --- |
| FR-1 | `BooksController.Index` shall continue to select books where `IsActive` is true and order them by `Title`. | `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:13-20` |
| FR-2 | `BooksController.Index` shall not read `HttpContext.Request.UserAgent`. | Current behavior is explicit at `BooksController.cs:17`; removal is the requested outcome. |
| FR-3 | `BooksController.Index` shall not assign `ViewBag.UserAgent`. | Current behavior is explicit at `BooksController.cs:18`; removal is the requested outcome. |
| FR-4 | `Views/Books/Index.cshtml` shall not render a `YOUR BROWSER` label or browser diagnostic value. | Current behavior is explicit at `Index.cshtml:67-71`. |
| FR-5 | Missing or unusual `User-Agent` values shall not affect catalog rendering. | A request header must have no effect on the catalog index. |
| FR-6 | Existing routes, table columns, status badges, and Details, Edit, and Delete links shall remain unchanged. | `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/App_Start/RouteConfig.cs:7-16` and `Index.cshtml:1-65` |

## Constraints and out of scope

### Constraints

- **Allowed paths:**
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
  - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`
- **Primary validation command:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`
- **Source root:** Run the primary command from
  `legacy-src/dotnet-bookcatalog/shared-legacy-app`.
- **Compatibility:** Preserve the default route behavior, EF6 catalog query,
  active-only filtering, title ordering, table structure, status badges, and
  existing CRUD links.
- **Dependencies:** No package or dependency changes.
- **Data and security:** Use synthetic data and synthetic request headers.
  Do not expose any `User-Agent` value or other request metadata as replacement
  behavior. Do not connect the app to production or business data.

### Exclusions (out of scope)

- ASP.NET Core migration.
- EF6 or database context, initializer, schema, seed data, or connection
  changes.
- Package, project-import, configuration, public API, telemetry, logging, or
  data changes.
- Styling cleanup, including unused diagnostic CSS unless a separate approved
  scope includes that work.
- Changes to CRUD behavior, routes, models, filtering, ordering, table columns,
  or status badges except as required to remove the diagnostic.

## Edge cases

| Type | Condition | Expected behavior |
| --- | --- | --- |
| Normal | Active and inactive books exist | Only active books appear, ordered by title, with existing actions. |
| Failure | Database access or catalog rendering fails | Existing application failure behavior remains unchanged; no replacement diagnostic or error behavior is added. |
| Missing-header boundary | Request has no `User-Agent` header | The catalog renders normally and no browser diagnostic is shown. |
| Boundary | `User-Agent` is empty or unusually long | The value is never read or rendered and the catalog remains usable. |
| Compatibility | Request reaches `/` or `/Books/Index` | Existing route behavior continues and the catalog list remains compatible. |
| Compatibility | Existing Details, Edit, or Delete actions are used | Existing behavior and links remain unchanged. |

## Acceptance criteria

### AC-1: Catalog behavior is preserved

- **Given** active and inactive records exist.
- **When** a user opens `/` or `/Books/Index`.
- **Then** only active books appear, ordered by title, with existing table
  columns and CRUD links.
- **Verification:** Manual IIS Express/LocalDB smoke check and source review.
  The exact runtime result or missing prerequisite shall be recorded.

### AC-2: Browser diagnostic and request access are removed

- **Given** the allowed source files are the implementation target.
- **When** source review searches them for `HttpContext.Request.UserAgent`,
  `ViewBag.UserAgent`, and `YOUR BROWSER`.
- **Then** all three references are absent.
- **Verification:** Search/diff review of the two allowed files.

### AC-3: Synthetic request headers do not affect rendering

- **Given** a synthetic request has no `User-Agent` header.
- **When** the index page is requested.
- **Then** the catalog renders normally and no browser diagnostic appears.
- **Verification:** Manual browser or HTTP smoke check, if IIS Express/LocalDB
  prerequisites are available.

### AC-4: Unusually long or distinctive request headers are not rendered

- **Given** a synthetic request contains a distinctive or unusually long
  `User-Agent` value.
- **When** `/` or `/Books/Index` is requested.
- **Then** the value and `YOUR BROWSER` label do not appear in the response.
- **Verification:** Manual browser or HTTP response inspection and source diff.

### AC-5: Validation evidence is recorded accurately

- **Given** the primary validation command is run from
  `legacy-src/dotnet-bookcatalog/shared-legacy-app`.
- **When** the Visual Studio web prerequisites are available.
- **Then** restore and build complete successfully and the exit status is
  recorded.
- **Verification:** Capture `msbuild BookCatalog.sln -restore -p:Configuration=Release`
  and its exit status. If the command cannot run, record the exact missing
  prerequisite and perform no state-changing workaround.

## Non-functional requirements

| ID | Quality | Testable requirement | Verification |
| --- | --- | --- | --- |
| NFR-1 | Privacy | The implementation shall not read or expose the request `User-Agent` in the index action or view. | Source search for `HttpContext.Request.UserAgent`, `ViewBag.UserAgent`, and `YOUR BROWSER`. |
| NFR-2 | Maintainability | The implementation diff shall contain changes only in the two allowed files. | `git diff --name-only` review. |
| NFR-3 | Reliability | Catalog filtering and title ordering shall remain unchanged. | Source review and manual catalog smoke check. |
| NFR-4 | Compatibility | No package, configuration, public API, database, schema, or route change is introduced. | Diff and project-file review; no unrelated files changed. |

## Assumptions, dependencies, and risks

| Item | Type | Validation or mitigation |
| --- | --- | --- |
| The diagnostic has no known external or operational consumer. | Unresolved assumption | A maintainer must confirm no consumer before approval. If a consumer is found, implementation stops. |
| Visual Studio web tooling, IIS Express, and LocalDB may be available on the runtime machine. | Dependency | The primary build command and manual smoke check are used if available; otherwise record the exact missing prerequisite. |
| The route `/Books/Index` is reachable through the existing default route. | Assumption | Verify from `RouteConfig.cs` and runtime routing behavior if IIS Express is available. |
| Database failure behavior is implementation-agnostic. | Assumption | Preserve existing database context and error handling; make no database or exception-handling change. |

## Plan-mode review checkpoint

### Evidence

- **Current behavior:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:13-20`
  reads the active-book set, orders it by title, reads
  `HttpContext.Request.UserAgent`, and assigns `ViewBag.UserAgent`.
- **Diagnostic view:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml:67-71`
  renders `YOUR BROWSER` and `ViewBag.UserAgent`.
- **Routing:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/App_Start/RouteConfig.cs:7-16`
  defines the default controller/action route.
- **Existing tests:** No automated test project was found under
  `legacy-src/dotnet-bookcatalog`; the repository guidance identifies a manual
  IIS Express/LocalDB smoke check.
- **Baseline validation evidence:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`
  was run from the source root and failed with `MSB4019`; the environment
  lacks `Microsoft.WebApplication.targets` at
  `C:\Program Files\dotnet\sdk\11.0.100\Microsoft\VisualStudio\v18.0\WebApplications\Microsoft.WebApplication.targets`.
  This failure is a prerequisite blocker, not a passing baseline.

### Unresolved questions

- [ ] Has a maintainer confirmed that no external or operational consumer
  depends on the browser diagnostic?
- [ ] Will the runtime environment provide Visual Studio web tooling, IIS
  Express, and LocalDB so the build and manual smoke check can run?
- [ ] Is `/` and `/Books/Index` behavior confirmed in an available runtime
  environment?
- [ ] Who is the human reviewer, what is the approval date, and what is the
  approved specification commit?

The specification is not ready for approval until questions that can change
scope, acceptance criteria, allowed paths, or validation are resolved.

### Boundary and validation review

- [ ] Evidence citations were checked against the repository.
- [ ] Allowed paths include all anticipated implementation changes and nothing
  unrelated.
- [ ] The primary validation command is safe, non-interactive, and runnable
  from the stated source root.
- [ ] Acceptance criteria cover normal, missing-header, unusual-header,
  database-failure, and compatibility behavior.
- [ ] Exclusions prevent unrelated modernization or cleanup.

### Stop conditions

Stop and request human guidance if:

- repository evidence contradicts this specification;
- an unresolved question could materially change behavior or scope;
- satisfying a criterion requires a path outside the two allowed files;
- the baseline or primary validation fails for a reason caused by the proposed
  change;
- Visual Studio web tooling, IIS Express, LocalDB, or another required
  prerequisite is missing;
- a known consumer of the browser diagnostic is found;
- a package, project import, configuration, public API, schema, database,
  telemetry, logging, secret, credential, production resource, destructive
  action, or dependency change is required but not explicitly approved;
- implementation or validation cannot satisfy the acceptance criteria without
  expanding the approved contract;
- source or runtime validation reveals a routing, filtering, ordering, CRUD,
  or database behavior change outside the requested diagnostic removal.

### Human approval

- [ ] I reviewed the evidence, unresolved questions, allowed paths, acceptance
  criteria, validation command, exclusions, risks, and stop conditions.
- **Reviewer:** Not yet recorded
- **Approval date:** Not yet recorded
- **Approved specification commit:** Not yet recorded

The specification remains **Draft**. Planning approval will authorize issue
preparation only, not implementation.
