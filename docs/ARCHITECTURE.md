# Architecture

WooCommerce Review Importer is a modular WordPress plugin for importing WooCommerce product reviews from CSV files. The architecture separates admin presentation, upload handling, parsing, import orchestration, product matching, review creation, rating updates, logging, and settings.

The importer must be able to process large files without loading the full CSV into memory and without relying on a single long-running request.

## Design Goals

- Keep importer logic independent from the admin UI.
- Use WordPress and WooCommerce APIs whenever possible.
- Support large CSV imports through AJAX batch processing.
- Keep classes small and focused on one responsibility.
- Use namespaces and dependency injection where practical.
- Make future import sources possible without rewriting the CSV importer.
- Never modify WordPress core, WooCommerce core, themes, or database schema.

## Plugin Bootstrap Design

The main plugin file is responsible only for bootstrapping the plugin.

Responsibilities:

- Define WordPress plugin headers.
- Define core constants such as plugin version, path, URL, basename, and minimum requirements.
- Register activation and deactivation hooks.
- Load the autoloader.
- Instantiate the main plugin class.
- Start plugin initialization on the appropriate WordPress hook.

The bootstrap should not contain importer logic, admin rendering logic, CSV parsing logic, or WooCommerce review creation logic.

Expected bootstrap sequence:

```text
WordPress loads main plugin file
↓
Main plugin file defines constants
↓
Main plugin file loads autoloader
↓
Main plugin file checks runtime requirements
↓
Main plugin file creates WCRI\Plugin
↓
WCRI\Plugin registers services and hooks
```

Activation responsibilities:

- Verify PHP version requirement.
- Verify WordPress version requirement where possible.
- Set default plugin options.
- Prepare any upload/log directories needed by the plugin.
- Schedule no recurring jobs for v1.0 unless explicitly required later.

Deactivation responsibilities:

- Stop active transient runtime state where appropriate.
- Leave user settings and import logs intact unless uninstall behavior later says otherwise.
- Do not delete imported reviews.

Uninstall behavior should be explicit and conservative. A future uninstall file may remove plugin-owned options, temporary files, and logs, but must never remove WooCommerce reviews unless the user explicitly chooses such behavior in a future feature.

## Namespace Structure

Namespace root: `WCRI`.

Suggested directory and namespace layout:

```text
wc-review-importer.php
includes/
  Plugin.php                         WCRI\Plugin
  Admin/
    AdminMenu.php                    WCRI\Admin\AdminMenu
    ImportPage.php                   WCRI\Admin\ImportPage
    AssetManager.php                 WCRI\Admin\AssetManager
    TemplateController.php           WCRI\Admin\TemplateController
  Ajax/
    ImportAjaxController.php         WCRI\Ajax\ImportAjaxController
  CSV/
    CsvUploadHandler.php             WCRI\CSV\CsvUploadHandler
    CsvValidator.php                 WCRI\CSV\CsvValidator
    CsvParser.php                    WCRI\CSV\CsvParser
    CsvRow.php                       WCRI\CSV\CsvRow
  Importer/
    ImportController.php             WCRI\Importer\ImportController
    ImportJob.php                    WCRI\Importer\ImportJob
    ImportJobRepository.php          WCRI\Importer\ImportJobRepository
    ImportResult.php                 WCRI\Importer\ImportResult
    BatchProcessor.php               WCRI\Importer\BatchProcessor
  Product/
    ProductMatcherInterface.php      WCRI\Product\ProductMatcherInterface
    SkuProductMatcher.php            WCRI\Product\SkuProductMatcher
    ProductMatchResult.php           WCRI\Product\ProductMatchResult
  Review/
    ReviewValidator.php              WCRI\Review\ReviewValidator
    ReviewCreator.php                WCRI\Review\ReviewCreator
    DuplicateDetector.php            WCRI\Review\DuplicateDetector
    RatingUpdater.php                WCRI\Review\RatingUpdater
  Logger/
    LoggerInterface.php              WCRI\Logger\LoggerInterface
    ImportLogger.php                 WCRI\Logger\ImportLogger
    LogEntry.php                     WCRI\Logger\LogEntry
    LogExporter.php                  WCRI\Logger\LogExporter
  Settings/
    SettingsRepository.php           WCRI\Settings\SettingsRepository
    SettingsPage.php                 WCRI\Settings\SettingsPage
  Security/
    CapabilityChecker.php            WCRI\Security\CapabilityChecker
    NonceVerifier.php                WCRI\Security\NonceVerifier
  Support/
    Requirements.php                 WCRI\Support\Requirements
    FileSystem.php                   WCRI\Support\FileSystem
    Sanitizer.php                    WCRI\Support\Sanitizer
    ResponseFactory.php              WCRI\Support\ResponseFactory
assets/
  css/admin.css
  js/admin-import.js
templates/
  admin/import-page.php
sample/
  reviews-template.csv
languages/
tests/
```

This structure can be adjusted during implementation, but each class should keep a clear single responsibility.

## Class Responsibilities

### `WCRI\Plugin`

Central composition root for the plugin.

Responsibilities:

- Instantiate services.
- Register WordPress hooks.
- Register admin, AJAX, settings, importer, and logger components.
- Keep wiring logic in one place.

It should not parse CSV files, render full templates, create reviews, or perform import business logic directly.

### Admin Classes

`AdminMenu` registers WooCommerce -> Import Reviews.

`ImportPage` renders the import screen and passes data to templates.

`AssetManager` enqueues admin CSS and JavaScript only on plugin admin pages.

`TemplateController` handles CSV template downloads.

Admin classes must verify capabilities and escape all output.

### AJAX Controller

`ImportAjaxController` exposes the AJAX endpoints used by the admin interface.

Responsibilities:

- Verify nonce and capabilities.
- Start import sessions.
- Process the next batch.
- Pause, resume, and cancel imports.
- Return JSON responses with status, progress, statistics, and recent log messages.

The AJAX controller delegates real work to importer services.

### CSV Classes

`CsvUploadHandler` receives and stores uploaded CSV files using WordPress upload APIs.

`CsvValidator` validates file type, headers, encoding expectations, and basic readability.

`CsvParser` streams CSV rows and normalizes headers without loading the full file into memory.

`CsvRow` represents a normalized row with row number and original values.

CSV classes do not create reviews.

### Importer Classes

`ImportController` coordinates a complete import session.

`ImportJob` represents current import state, including file path, current row, totals, counters, status, and timestamps.

`ImportJobRepository` persists and retrieves job state using WordPress-owned storage such as options, transients, or upload metadata.

`BatchProcessor` processes a limited number of rows per request.

`ImportResult` carries structured success, skipped, warning, and error information back to callers.

Importer classes coordinate services but should avoid direct UI rendering.

### Product Classes

`ProductMatcherInterface` defines a contract for resolving a row to a WooCommerce product.

`SkuProductMatcher` resolves products by SKU.

`ProductMatchResult` represents found, not found, invalid, or ambiguous matching outcomes.

Future strategies can support Product ID, GTIN, UPC, EAN, and ASIN without changing importer orchestration.

### Review Classes

`ReviewValidator` validates row data needed to create a review.

`ReviewCreator` creates reviews using WordPress comment APIs and WooCommerce-compatible metadata.

`DuplicateDetector` optionally detects duplicates by product, email, and review content.

`RatingUpdater` recalculates WooCommerce product rating data and clears related caches.

Review classes should use WooCommerce and WordPress APIs rather than direct database writes wherever possible.

### Logger Classes

`LoggerInterface` defines logging behavior.

`ImportLogger` records import log entries by session.

`LogEntry` represents one log row with type, message, row number, product reference, and context.

`LogExporter` produces downloadable logs for administrators.

Logs must be sanitized when stored and escaped when displayed.

### Settings Classes

`SettingsRepository` reads, writes, sanitizes, and provides defaults for plugin settings.

`SettingsPage` renders and registers settings fields if settings are shown separately from the import page.

Settings should include review status, verified owner, batch size, duplicate detection, logging, and execution limits.

### Security Classes

`CapabilityChecker` centralizes capability checks for admin and AJAX actions.

`NonceVerifier` centralizes nonce verification.

Security classes help keep authorization behavior consistent across controllers.

### Support Classes

`Requirements` checks PHP, WordPress, and WooCommerce compatibility.

`FileSystem` centralizes plugin-owned directory and file handling.

`Sanitizer` centralizes reusable sanitization helpers where WordPress core functions alone are not enough.

`ResponseFactory` creates consistent AJAX response arrays.

## Data Flow

High-level data flow:

```text
Administrator
↓
WooCommerce -> Import Reviews admin page
↓
CSV upload request
↓
CsvUploadHandler
↓
CsvValidator
↓
ImportJobRepository creates import job
↓
AJAX batch loop begins
↓
BatchProcessor reads rows through CsvParser
↓
ProductMatcher resolves product
↓
ReviewValidator validates row
↓
DuplicateDetector checks optional duplicate rule
↓
ReviewCreator creates WordPress comment review
↓
RatingUpdater schedules or performs product rating refresh
↓
ImportLogger records row outcome
↓
ImportJobRepository saves progress
↓
AJAX response updates admin progress UI
```

The admin UI never directly imports reviews. It starts and monitors import jobs through AJAX.

## AJAX Import Flow

The import engine must not process a full CSV in one request.

Expected AJAX flow:

```text
1. User uploads CSV and clicks Start Import
2. Browser sends `start_import` AJAX request
3. Server validates nonce, capability, file, settings, and headers
4. Server creates an ImportJob with status `pending`
5. Browser begins repeated `process_batch` AJAX requests
6. Server marks job `running`
7. Server processes up to configured batch size, default 100 rows
8. Server saves counters, current row, recent logs, and status
9. Browser updates progress bar and statistics
10. Browser continues until status is `completed`, `paused`, `cancelled`, or `failed`
```

Pause flow:

```text
User clicks Pause
↓
Browser sends `pause_import`
↓
Server verifies nonce and capability
↓
ImportJobRepository sets status `paused`
↓
Next process request returns paused status without processing rows
```

Resume flow:

```text
User clicks Resume
↓
Browser sends `resume_import`
↓
Server verifies nonce and capability
↓
ImportJobRepository sets status `running`
↓
Browser restarts process_batch loop from saved row position
```

Cancel flow:

```text
User clicks Cancel
↓
Browser sends `cancel_import`
↓
Server verifies nonce and capability
↓
ImportJobRepository sets status `cancelled`
↓
Temporary files may be cleaned up if safe
↓
No more rows are processed
```

AJAX responses should include:

- Job ID
- Status
- Total rows if known
- Processed rows
- Imported count
- Skipped count
- Warning count
- Error count
- Progress percentage
- Recent log messages
- Human-readable status message
- Estimated remaining time when available

## Error Handling Flow

The importer must never terminate an entire import because one row fails.

Row-level error flow:

```text
CsvParser reads row
↓
Row normalization or validation fails
↓
BatchProcessor catches row-level issue
↓
ImportLogger records warning, skipped, or error entry
↓
ImportJob counters are updated
↓
BatchProcessor continues with next row
```

Product matching error flow:

```text
ProductMatcher receives SKU or product reference
↓
Product not found or reference is invalid
↓
Result is returned as not found or invalid
↓
Review creation is skipped for that row
↓
Logger records skipped row with reason
↓
Batch continues
```

Review creation error flow:

```text
ReviewCreator attempts WordPress comment insertion
↓
WordPress returns error or invalid comment ID
↓
Error is converted into ImportResult failure
↓
Logger records row-level error
↓
Batch continues
```

Batch-level error flow:

```text
AJAX request starts batch
↓
Unexpected exception or fatal-risk condition occurs
↓
Server records an import-level error when possible
↓
Job status becomes failed only if processing cannot safely continue
↓
AJAX response returns failure details to admin UI
```

A job should be marked `failed` only for unrecoverable problems, such as unreadable source file, missing job state, invalid permissions, corrupted import metadata, or repeated infrastructure-level failure. Data problems in individual rows should produce skipped/error log entries and allow the import to continue.

## Security Model

Every admin and AJAX action must enforce:

- Current user capability checks.
- Nonce verification.
- Sanitization of input.
- Escaping of output.
- Upload validation.
- MIME validation.
- CSV injection protection when displaying or exporting CSV-derived values.

Suggested capability: `manage_woocommerce`, with fallback consideration for `manage_options` only if WooCommerce is unavailable during setup screens.

The plugin must not trust uploaded file names, CSV headers, CSV cell values, AJAX parameters, settings values, or log output.

## Performance Model

Large import support depends on these rules:

- Stream CSV rows instead of reading the full file into memory.
- Process a configurable batch size, default 100 rows per request.
- Persist job progress after each batch.
- Avoid expensive duplicate checks where possible.
- Avoid repeated full-product recalculation inside tight row loops when a deferred or per-product summary update is safer.
- Clear WooCommerce caches at controlled points.
- Keep AJAX responses compact.

The target is 100,000+ reviews without timeout or memory exhaustion.

## Extension Points

The architecture should allow future additions without rewriting the core importer:

- New product matchers: Product ID, GTIN, UPC, EAN, ASIN.
- New import sources: Amazon, AliExpress, Temu.
- Review images.
- CSV export.
- REST API.
- WP-CLI commands.
- Cron-based imports.
- AI-generated reviews.

Future import sources should produce normalized row objects compatible with importer services instead of duplicating review creation logic.
