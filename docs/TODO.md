# Development Milestones

This TODO tracks implementation order for WooCommerce Review Importer v1.0. Each milestone should be completed and reviewed before moving to the next milestone. Do not implement unrelated future roadmap features during v1.0 work.

## Milestone 0: Repository Preparation

- [ ] Confirm final plugin slug: `wc-review-importer`.
- [ ] Confirm PHP namespace root: `WCRI`.
- [ ] Confirm WordPress, WooCommerce, and PHP version requirements.
- [ ] Add `.github/copilot-instructions.md` if project tooling requires that exact path.
- [ ] Add sample CSV template under `sample/`.
- [ ] Add test fixture CSV files for valid, invalid, duplicate, UTF-8, multiline, and large-import cases.

## Milestone 1: Plugin Bootstrap

- [ ] Create the main plugin file with WordPress plugin headers.
- [ ] Define plugin constants for version, path, URL, basename, and minimum requirements.
- [ ] Add activation and deactivation hooks.
- [ ] Add runtime checks for PHP, WordPress, and WooCommerce compatibility.
- [ ] Add an autoloader or Composer PSR-4 setup for the `WCRI` namespace.
- [ ] Create the central `Plugin` class responsible for wiring services.
- [ ] Load text domain for translations.
- [ ] Ensure the plugin activates without PHP warnings or deprecated notices.

## Milestone 2: Core Architecture Skeleton

- [ ] Create namespaced directories for Admin, CSV, Importer, Product, Review, Logger, Settings, Security, and Support classes.
- [ ] Define service classes with single responsibilities.
- [ ] Add dependency injection through constructors where practical.
- [ ] Add interfaces for replaceable strategies, especially product matching and logging.
- [ ] Keep importer logic independent from the admin UI.
- [ ] Document all public methods with PHPDoc.

## Milestone 3: Settings

- [ ] Register plugin settings using WordPress Settings API.
- [ ] Add default review status setting.
- [ ] Add default verified owner setting.
- [ ] Add default batch size setting.
- [ ] Add duplicate detection setting.
- [ ] Add logging enabled setting.
- [ ] Add maximum execution time or per-request processing limit setting.
- [ ] Sanitize all settings before saving.
- [ ] Escape all settings output in admin screens.

## Milestone 4: Admin Interface

- [ ] Add WooCommerce submenu: WooCommerce -> Import Reviews.
- [ ] Render upload panel, import options, progress area, statistics area, and logs area.
- [ ] Add CSV template download action.
- [ ] Add nonce and capability checks for every admin action.
- [ ] Enqueue admin CSS and JavaScript only on the plugin page.
- [ ] Keep admin UI code separate from importer services.

## Milestone 5: CSV Upload And Validation

- [ ] Accept UTF-8 CSV files only.
- [ ] Validate file extension and MIME type.
- [ ] Reject oversized or unreadable uploads with clear admin errors.
- [ ] Store uploaded files in a controlled WordPress uploads subdirectory.
- [ ] Protect stored CSV files from direct unintended access where possible.
- [ ] Detect missing required headers.
- [ ] Ignore unknown columns.
- [ ] Apply CSV injection protection for logged or exported cell-like values.

## Milestone 6: CSV Parser

- [ ] Implement streaming CSV reading without loading the full file into memory.
- [ ] Support header mapping for `sku,name,email,rating,title,review,date`.
- [ ] Normalize row data before validation.
- [ ] Preserve row numbers for logs and error reporting.
- [ ] Support multiline reviews.
- [ ] Support special characters, UTF-8, and emoji.
- [ ] Skip malformed rows without stopping the import.
- [ ] Provide parser result objects or arrays with consistent keys.

## Milestone 7: Import Job State

- [ ] Create import session/job state storage using WordPress options, transients, or upload metadata.
- [ ] Track file path, current offset or row index, totals, processed count, success count, skipped count, warning count, and error count.
- [ ] Track status: pending, running, paused, cancelled, completed, failed.
- [ ] Track timestamps for start, update, completion, and cancellation.
- [ ] Support resume without reprocessing completed rows.
- [ ] Clean up stale import state and uploaded files safely.

## Milestone 8: Product Matching

- [ ] Define product matching interface.
- [ ] Implement SKU matcher as the default strategy.
- [ ] Add Product ID matcher only if required for v1.0 import options.
- [ ] Return structured match results for found, not found, invalid input, and ambiguous matches.
- [ ] Log SKU not found and invalid product references without stopping the batch.
- [ ] Keep design open for GTIN, UPC, EAN, and ASIN strategies.

## Milestone 9: Review Importer

- [ ] Validate rating, email, author name, review content, and date per row.
- [ ] Apply defaults for missing optional fields.
- [ ] Create reviews through WordPress comment APIs.
- [ ] Store WooCommerce rating comment meta.
- [ ] Store verified owner metadata according to settings.
- [ ] Set approval status according to settings.
- [ ] Preserve review title only if supported by the selected storage approach.
- [ ] Ensure one failed row does not terminate the import.

## Milestone 10: Duplicate Detection

- [ ] Implement optional duplicate detection by product, email, and review content.
- [ ] Check duplicates before creating the review.
- [ ] Log skipped duplicates with row number and product reference.
- [ ] Keep duplicate detection efficient for large imports.
- [ ] Allow duplicate detection to be disabled through settings.

## Milestone 11: Rating And Cache Updates

- [ ] Recalculate WooCommerce average rating.
- [ ] Recalculate WooCommerce review count.
- [ ] Recalculate rating histogram/count metadata where applicable.
- [ ] Clear product cache and transients after batch or job completion.
- [ ] Verify frontend product rating display updates immediately after import.
- [ ] Avoid direct SQL unless no WordPress or WooCommerce API exists.

## Milestone 12: Logging

- [ ] Implement log types: info, warning, error, skipped, summary.
- [ ] Include row number, product reference, message, and context where useful.
- [ ] Store logs per import session.
- [ ] Display logs in the admin page.
- [ ] Add log download action.
- [ ] Sanitize log content and escape log output.
- [ ] Protect logs from unauthorized access.

## Milestone 13: AJAX Batch Import

- [ ] Add AJAX action to start an import session.
- [ ] Add AJAX action to process the next batch.
- [ ] Add AJAX action to pause import.
- [ ] Add AJAX action to resume import.
- [ ] Add AJAX action to cancel import.
- [ ] Default each batch to 100 rows, configurable through settings.
- [ ] Return progress percentage, counters, current status, and recent messages.
- [ ] Estimate remaining time when enough progress data exists.
- [ ] Verify nonce and capabilities for every AJAX request.
- [ ] Ensure batch processing avoids timeouts and memory exhaustion.

## Milestone 14: Admin Progress And Statistics

- [ ] Build progress bar behavior in admin JavaScript.
- [ ] Display total, processed, imported, skipped, warnings, and errors.
- [ ] Display current status and completion summary.
- [ ] Support pause, resume, and cancel controls.
- [ ] Handle AJAX failures gracefully with retry-safe messaging.
- [ ] Prevent duplicate start requests while an import is running.

## Milestone 15: Testing And Quality Gates

- [ ] Test plugin activation and deactivation.
- [ ] Test small valid CSV import.
- [ ] Test invalid CSV and missing required headers.
- [ ] Test missing SKU rows.
- [ ] Test SKU not found rows.
- [ ] Test invalid rating and invalid email rows.
- [ ] Test duplicate review detection.
- [ ] Test UTF-8, emoji, multiline, and very long reviews.
- [ ] Test interrupted import and resume behavior.
- [ ] Test cancel behavior and cleanup.
- [ ] Test large import target of 100,000 rows.
- [ ] Run PHP syntax checks.
- [ ] Run WordPress Coding Standards checks when tooling is available.
- [ ] Verify no PHP warnings, notices, or deprecated messages.

## Milestone 16: Packaging And Release Readiness

- [ ] Confirm installable ZIP structure.
- [ ] Exclude development-only files from release package if needed.
- [ ] Confirm plugin uninstall behavior is clean and documented.
- [ ] Confirm README accurately reflects implemented features.
- [ ] Update changelog for the completed release.
- [ ] Perform final manual test on a WooCommerce store.
