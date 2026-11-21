# Changelog

This document records the modifications and enhancements in this custom edition compared to upstream [XayahSuSuSu/Android-DataBackup](https://github.com/XayahSuSuSu/Android-DataBackup).

## v3.16.0 (2026-11-14)

### Added

- **Display tags in app list**: Added "Display tags in list" switch in the filter panel (off by default). When enabled, all tags associated with each application are displayed directly in the app list items as compact capsule chips without needing to open the details page.
  - Precise matching by package identity (package name + user ID + preserveId); multi-user profiles and protected versions remain strictly isolated;
  - Backed by the complete tag relationship table, unaffected by active tag filters;
  - Orphaned relationships from deleted tags are automatically filtered; list updates immediately upon batch tagging or untagging;
  - Compact chips displayed in a single line (up to 3 chips, remainder collapsed into +N badge); long labels gracefully ellipsize.
- **One-click restore flow**: Added a dedicated entry in "Backup & Migration" page. Select local or cloud backup source with single confirmation dialog to enter guided restore flow covering Apps, Files, and System Data.
  - System data restores exclusively from cloud backups; column discrepancies prompt confirmation before applying changes;
  - Clears stale residual selections before selecting all to avoid mixing previous tasks.
- **System data cloud backup integration**: SMS, call logs, and Wi-Fi configurations can be uploaded to the active cloud storage under the system_data directory.
- **Distinguishable file backup versions**: Shield icons in file backup list now display numbered sequence badges (Shield 1 for newest) based on preserveId, making version ordering clear and unambiguous.
- **Conflict resolution strategies for file restore**: Choose how to handle filename conflicts during restore: Replace (default, overwrites local files), Skip (preserves local files and only extracts newly added files), Keep Both (creates a restorable local snapshot), or Ask per directory.
- **Pre-restore preview & summary**: Preview files and directories before extraction with detailed counts of overwritten, skipped, preserved, and deleted items.
- **File backup retention & version limit**: Added independent options to preserve multiple historical versions for files with customizable retention limits.
- **File backup strategy selector**: Choose between Replace (default), Skip (if existing backup exists), or Copy (archive previous backup as historical version).
- **Comprehensive system data restore summary**: The final completion page now displays itemized results for SMS, call logs, and Wi-Fi.

### Fixed & Improved

- **Fixed restore reading outdated version from JSON**: Resolved an issue where package restore would display and restore old versions (e.g. 18.2 instead of 20.0) from stale database records by always updating package metadata directly from the backup JSON file.
- **Eliminated app list lag & counter jitter**: Disabled automatic heavy re-scanning on screen resume (onResume) while preserving responsive manual refresh, eliminating UI freezes and flickering counters.
- **Cleaned obsolete legacy backup folders**: Automatically remove leftover obsolete backup directories when creating a new main backup to keep storage clean.
- **Accurate backup pruning by creation date**: Preserved historical backups are now sorted strictly by actual backup creation timestamp (lastBackupTime) rather than version numbers, ensuring the true oldest backups are pruned according to user retention limits.
- **Fixed text layout & vertical squishing glitch**: Restructured selectable settings rows (Selectable) to place action buttons on their own line below titles and descriptions, and moved the (Experimental) badge to its own dedicated horizontal line, eliminating vertical single-letter text squishing across all languages.
- **Ultra-fast Arm64 CI/CD builds**: Added Arm64-only build optimization, reducing compilation times from 12+ minutes to ~2 minutes.
- **Cloud historical backup cleanup**: Fixed an issue where cloud cleanup only deleted database entries while leaving remote directories behind.
- **One-click restore source synchronization**: Fixed navigation arguments so clicking Apps or Files during one-click cloud restore correctly retains the cloud source instead of reverting to local.
- **Accurate item counting**: Fixed list header (N/M) counters in restore views to accurately match filtered results by backup source.
- **Crash fix on file restore chaining**: Resolved crash when chaining from app restore to file restore caused by duplicate URL decoding.
- **Correct completion page counts**: Fixed completion page counting skipped items as successes.
- **Proper UI text pluralization**: Fixed completion page referring to backup directories as "files" instead of "directories" across translations.
- **Snapshot fallback for large directories**: If local snapshot archives exceed 20,000 files during "Keep Both" restore, safely fall back to numbered backups without losing data.
- **Archive rollback on failure**: If compression or upload fails under "Copy" strategy, previous backup archives and database records are cleanly restored.

## v3.15.2 (2026-10-19)

### Fixed

- **Three-line app list items**: List items now show App Name, Version, and Package Name on separate lines to prevent clipping on high-density screens and large font settings.

## v3.15.0 (2026-10-17)

### Added

- **Batch tagging**: Added batch tag operations (Add existing tag, Create and apply new tag, Remove tags) for selected applications from the overflow menu.
- **Persistent sort preferences**: Sorting mode and direction are now independently remembered for Apps and Files across app restarts.
- **List display toggles**: Filter panel now includes independent toggles to show/hide package names and version numbers in the list.
- **FAT32 partition pre-check**: Pre-checks target filesystem type before backup, skipping FAT32 partitions if uncompressed source exceeds 6 GiB to avoid write errors.

### Fixed

- **Batch tagging filtered items**: Resolved an issue where batch tagging only applied to visible filtered items rather than all selected items.
- **Tag list refresh**: App list immediately refreshes upon applying or deleting tags.
- **Tag deletion cleanup**: Cascading cleanup when tags are deleted to prevent orphaned relations.

## v3.14.0 (2026-10-05)

### Added

- **Incremental backup**: Calculates source directory manifest hash before backup and skips unchanged data types without redundant re-compression.
- **Backup version notes**: Added custom notes for individual backup archives accessible directly from application details.
- **Manifest metadata**: Generates `.manifest` metadata alongside archives to track source hashes and packaging parameters for incremental checks.

### Fixed

- **Cloud backup reload list empty**: Fixed reload logic so valid package entities are re-inserted into the database immediately upon directory scan.
- **Data types restore checkable state**: Fixed an issue where USER_DE, DATA, OBB, and MEDIA types were not marked as restorable during cloud reload.
- **Preserved archive completeness**: Ensured unselected data types retain their previous archives when creating historical backups.

## v3.13.4 (2026-10-04)

### Fixed

- **MediaProvider trashed and pending files exclusion**: Unified exclusion pattern using AOSP MediaProvider FileUtils pattern across backup and restore paths to prevent restoring inaccessible deleted images.
- **Exclusion logging**: Detailed logging of skipped trashed files with sample filenames and total counts.

## v3.13.0 (2026-09-18)

### Added

- **Cloud backup multipart splitting**: Added option to split cloud archives into chunks (1 GB / 2 GB / 4 GB) to bypass cloud storage single-file limits.
- **In-app changelog**: Added dedicated Changelog page in Settings -> About and via version badge in Dashboard.
