# Product Requirements Document (PRD)

# Project

WooCommerce Review Importer

Version: 1.0

Author: Product Owner

Status: Ready for Development

---

# 1. Project Overview

## Objective

Develop a professional, production-ready WordPress plugin that allows WooCommerce administrators to import product reviews in bulk from CSV files.

The plugin must be scalable, modular, maintainable, and compatible with future feature expansion.

This plugin will become a reusable internal tool across multiple WooCommerce stores.

It must be designed as enterprise-quality software rather than a one-off import script.

---

# 2. Target Environment

* WordPress 6.8+
* WooCommerce 10+
* PHP 8.1+
* MySQL 8+
* Compatible with Blocksy, Astra, Kadence and default WooCommerce themes.
* No dependency on any specific theme.

---

# 3. Development Principles

The plugin must follow the following principles:

* WordPress Coding Standards
* PSR-12 formatting where compatible
* Object-Oriented Programming
* SOLID Principles
* DRY (Don't Repeat Yourself)
* KISS (Keep It Simple)
* Security First
* Performance First
* Extensibility First

The plugin should be designed for long-term maintenance.

---

# 4. Technical Constraints

The plugin MUST NOT:

* modify WooCommerce core
* modify WordPress core
* modify database tables
* modify the active theme
* require editing functions.php
* require custom SQL tables

Use WordPress APIs whenever possible.

---

# 5. Plugin Architecture

The plugin must use a modular architecture.

Suggested structure:

wc-review-importer/

assets/

includes/

admin/

templates/

languages/

sample/

tests/

vendor/

The architecture must support adding new modules without modifying existing importer logic.

---

# 6. Admin Interface

Create a submenu:

WooCommerce

└── Import Reviews

The page should contain:

* Upload CSV
* Download CSV Template
* Import Options
* Start Import
* Import Progress
* Import Statistics
* Import Logs

Use WordPress admin UI components.

---

# 7. CSV Specification

UTF-8 encoded.

Header:

sku,name,email,rating,title,review,date

Only SKU is required.

Other fields may be optional.

Ignore unknown columns.

Support files containing more than 100,000 rows.

---

# 8. Import Workflow

Workflow:

Upload CSV

↓

Validate file

↓

Read rows

↓

Queue import tasks

↓

Import review

↓

Update rating

↓

Write log

↓

Display statistics

The importer must never terminate because of one failed row.

---

# 9. Product Matching

Support:

SKU (default)

Product ID

Future:

GTIN

UPC

EAN

ASIN

Architecture should allow new match strategies.

---

# 10. Review Creation

Create reviews using WordPress APIs.

Populate:

Author

Email

Content

Rating

Date

Approval Status

Verified Owner

Title (if supported)

Do not bypass WordPress comment APIs.

---

# 11. Rating Update

After import:

Recalculate

Average Rating

Review Count

Rating Histogram

WooCommerce cache

Transient cache

Everything should immediately appear correctly on the frontend.

---

# 12. Duplicate Detection

If enabled:

Duplicate means:

Same Product

Same Email

Same Review Content

Duplicates should be skipped.

---

# 13. Import Engine

Do NOT import everything in one request.

Use AJAX Batch Import.

Each request:

100 reviews

Continue automatically until finished.

Should support:

Pause

Resume

Cancel

Progress percentage

Remaining time estimation

---

# 14. Logging

Store logs during import.

Log types:

Information

Warning

Error

Skipped

Summary

Allow administrator to download logs.

---

# 15. Error Handling

Examples:

SKU not found

Invalid Rating

Invalid Email

Empty Review

Malformed CSV

Encoding Errors

Errors should never stop the import process.

---

# 16. Security

Must implement:

Nonce verification

Capability checks

Escaping

Sanitization

Upload validation

MIME validation

CSV injection protection

Never trust user input.

---

# 17. Performance

Target:

Import 100,000 reviews

Without timeout

Without exhausting memory

Use batch processing.

Avoid loading the entire CSV into memory.

---

# 18. Settings

Settings page:

Default Review Status

Default Verified Owner

Default Batch Size

Duplicate Detection

Logging

Maximum Execution Time

Future options should be easy to add.

---

# 19. Extensibility

Design for future modules:

Amazon Review Import

AliExpress Review Import

Temu Review Import

CSV Export

Review Images

AI Review Generator

REST API

WP-CLI

Cron Jobs

No importer logic should depend on the admin UI.

---

# 20. Coding Requirements

Every class should have one responsibility.

Avoid static classes unless justified.

Avoid global variables.

Avoid duplicated code.

Document all public methods.

Use dependency injection where appropriate.

---

# 21. Testing

Test cases:

Small CSV

Large CSV

100k reviews

Missing SKU

Duplicate Reviews

Special Characters

UTF-8

Emoji

Multiline Reviews

Very Long Reviews

Invalid CSV

Interrupted Import

Resume Import

---

# 22. Acceptance Criteria

The plugin is complete only when:

✓ Installs successfully

✓ Activates without warnings

✓ Uninstalls cleanly

✓ Imports CSV correctly

✓ Updates WooCommerce ratings

✓ No PHP warnings

✓ No deprecated notices

✓ Supports AJAX import

✓ Supports 100,000+ reviews

✓ Displays progress

✓ Displays statistics

✓ Displays logs

✓ Passes WordPress Coding Standards

✓ Compatible with PHP 8.1+

✓ Compatible with WooCommerce 10+

✓ Compatible with latest WordPress

---

# 23. Future Roadmap

Version 1.0

CSV Import

Version 1.1

AJAX Import

Progress Bar

Logs

Version 1.2

Review Images

Pros & Cons

Review Replies

Version 2.0

Amazon Import

AliExpress Import

REST API

WP-CLI

AI Review Generator

---

# 24. Development Workflow

Before implementing any feature:

1. Analyze the existing plugin architecture.
2. Reuse existing classes whenever possible.
3. Do not introduce duplicate functionality.
4. Keep modules loosely coupled.
5. Implement one feature at a time.
6. Verify that existing functionality is not broken.
7. Commit changes in logical, reviewable increments.

Always favor maintainability over short-term convenience.

---

# 25. Deliverables

The final result must be a production-ready WordPress plugin.

The plugin should be installable as a ZIP package from the WordPress admin dashboard without requiring any manual code modifications.

All code should be clean, documented, extensible, and suitable for long-term maintenance.

