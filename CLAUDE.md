# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

File Away is a WordPress plugin (version 3.9.9.0.2) for uploading, managing, and displaying files from server directories or page attachments in stylized lists or sortable data tables.

**Requirements:** WordPress 3.7+, PHP 5.4+

## Architecture

### Entry Point and Autoloading
- `file-away.php` - Main plugin file, defines constants and uses `spl_autoload_register` to autoload classes from `lib/cls/class.{classname}.php`
- Classes are instantiated conditionally based on `is_admin()` context

### Core Classes (lib/cls/)

**Shortcode Classes** - Each registers its own WordPress shortcode:
- `fileaway` - `[fileaway]` - Display files from server directories
- `attachaway` - `[attachaway]` - Display WordPress post attachments
- `fileup` - `[fileup]` - File upload form
- `formaway` - `[formaway_open/row/cell/close]` - Custom table builder
- `stataway` - `[stataway]` and `[stataway_user]` - Download statistics display
- `playaway` - Audio playback functionality
- `feedaway` - RSS feed generation

**Support Classes:**
- `fileaway_attributes` - Base class extended by shortcode classes, handles attribute parsing
- `fileaway_definitions` - Path options, file type definitions, mobile detection
- `fileaway_admin` - Admin settings page, TinyMCE button integration
- `fileaway_utility` - Static helper methods (paths, formatting, security validation)
- `fileaway_management` - AJAX handlers for file operations (rename, delete, upload)
- `fileaway_stats` - Download tracking database operations
- `fileaway_encrypted` - URL encryption for file downloads

### Include Files (lib/inc/)
Modular PHP includes used by shortcode classes for rendering output. Files are included contextually based on shortcode attributes (e.g., `inc.flightbox.php` only loaded when lightbox enabled).

### Key Patterns

**Shortcode Pattern:**
1. Class extends `fileaway_attributes`
2. Constructor calls `parent::__construct()` and registers shortcode via `add_shortcode()`
3. `sc()` method extracts attributes, includes relevant inc files, builds HTML output

**Security:**
- Path validation via `fileaway_utility::realpath()` to prevent directory traversal
- Nonce verification on AJAX operations
- Capability checks for admin functions
- XSS prevention via `sanitize_atts()` in `fileaway_attributes` class
- File upload restricted to authenticated users with `upload_files` capability
- Dangerous file extensions blocked (php, exe, etc.)

### Security Patches Applied
- **CVE-2025-2512**: Fixed unauthenticated file upload in `upload()` method - now requires authentication and `upload_files` capability
- **CVE-2023-0431**: Fixed stored XSS via shortcode attributes - text attributes now sanitized via `esc_attr()`
- Added whitelist for unauthenticated AJAX actions (only `flightbox` and `bulkdownload` allowed)

**Dynamic Paths:**
Shortcodes support dynamic user-specific paths using placeholders: `fa-username`, `fa-userid`, `fa-userrole`, `fa-firstlast`

## Development

This is a WordPress plugin - no build process required. Install by placing in `wp-content/plugins/` directory.

### Testing Changes
Activate plugin in WordPress admin and test shortcodes on posts/pages. Debug mode available via `debug="on"` attribute on shortcodes.

### Key Files to Modify
- Shortcode behavior: `lib/cls/class.{shortcodename}.php`
- Admin settings: `lib/cls/class.fileaway_admin.php`
- File operations: `lib/cls/class.fileaway_management.php`
- Display rendering: `lib/inc/inc.*.php`
