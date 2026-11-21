# Changelog

This document records the modifications and enhancements in this custom edition compared to upstream [XayahSuSuSu/Android-DataBackup](https://github.com/XayahSuSuSu/Android-DataBackup).

## v3.16.0 (2026-11-14)

### Added

- **Display tags in app list**: Added "Display tags in list" toggle in the filter panel (off by default). When enabled, all tags associated with each application are displayed directly in the app list items without needing to open the details page.
  - Tags match accurately by version identity (package name + user ID + preserveId); multi-user profiles and protected versions remain strictly isolated;
  - Backed by the complete tag relationship table, completely unaffected by active tag filters (all tags remain visible under any filter state);
  - Automatically filters orphaned relations left by deleted tags; app list refreshes immediately upon batch tagging, removing, or deleting tags;
  - Displayed as compact capsule chips in a single line (up to 3 chips, with remainder collapsed into a +N badge); overlong tags ellipsize within single chips; +N badge is protected from squishing even on narrow screens; enabling adds one row to list items.
- **One-click restore flow**: Added a dedicated entry in "Backup & Migration" page: dialog to select local or cloud backup source -> one-line overwrite confirmation -> guided restore flow; perfectly symmetrical with backup flow, allowing fine-tuning for Apps, Files, and System Data. Automatically chains into file restore upon app restore completion.
  - System data restores exclusively from cloud backups (consistent with backup scope); executes selected items upon clicking "Continue"; schema discrepancies (missing columns or tables in backup database) prompt a confirmation dialog before applying changes, never overwriting silently;
  - Resets and clears stale residual selections before selecting all to avoid mixing selections from previous restore flows.
- **One-click backup includes system data**: SMS, Call Logs, and Wi-Fi networks upload alongside the "Continue" step to the  directory of the active cloud account, sharing the directory structure and filenames with manual cloud backups; System Data row in guided flow matches Apps/Files format (dedicated icon, subtitle, right arrow), allowing item-by-item selection and truthful result reporting.
- **Distinguishable file historical versions (File Strategy Phase 0)**: Previously, multiple historical file backups shared an identical shield icon with no way to tell version order. Now, shield icons display sequence numbers in their center (Shield 1 for newest), calculated by  rather than backup timestamp (file "Protect" only changes version field; entire backup group shares same timestamp); details page displays version timestamps, supporting both legacy 13-digit millisecond timestamps and new 14-digit date strings (). "Protect" action standardized to 14-digit format and synchronized with cloud account and config files, preventing "ghost" versions that have database records but missing directories.
- **Conflict resolution strategy parameter mapping (File Strategy Phase 0)**: Added reusable strategy enums and pure functions verified with GNU tar 1.35 on physical devices. Corrected a hard incompatibility:  and  are mutually exclusive; passing both causes  and total extraction failure; confirmed that enabling "Clean Restore" silently overrides all other conflict strategies.
- **Configurable file restore conflict strategies (File Strategy Phase 1)**: Added strategy selection under "Settings -> Restore Settings -> Clean Restore" (Replace, Skip, Keep Both, Ask per directory).
  - Replace: Overwrites local files (default, behavior consistent with previous releases);
  - Skip: Keeps local files untouched, only extracting newly added files from backup;
  - Keep Both: Packages conflicting local files into a restorable snapshot before extraction;
  - Effective across both local and cloud restore pipelines. Independent from backup version strategies. Greys out with explanation when "Clean Restore" is active.
- **Pre-restore preview and completion page statistics (File Strategy Phase 2)**: Reads archive contents and compares with local directory before extraction, popping up a breakdown: count of items to be overwritten, skipped, preserved, added, or deleted by clean restore. User confirms before extraction begins; canceling skips all directories in the current task, prompting only once per task. Completion page displays itemized cumulative counts and skipped directory counts.
- **Keep Both upgraded to local snapshot version (File Strategy Solution 2)**: Selecting "Keep Both" no longer renames old files into scattered  files. Instead, it packages only conflicting local files into an archive, writes config JSON, and registers them as .
  - File restore list displays "Main Version + Shield 1/2" with checkbox, delete support, sequence number, and timestamp, perfectly consistent with app backups;
  - Snapshots write strictly to local backup directory and are never uploaded to cloud storage (local privacy data stays off cloud);
  - Automatically degrades to legacy numbered renaming if packaging fails or archive entries exceed 20,000 files to guarantee zero data loss.
- **Interactive per-directory conflict prompt "Ask Every Time" (File Restore Phase 2 Batch 2)**: Prompts a decision dialog for each conflicting directory, showing conflict and addition counts. Allows choosing "Replace / Skip / Keep Both" on the spot, with an option to apply to all remaining directories without prompting repeatedly.
  - Granular per-directory prompting: tar unpacks entire archives and cannot pause per file; this fits varying requirements across directories;
  - Safe fallback: If decision channel is interrupted, defaults to  rather than overwriting.
- **Automatic historical backup retention for files (File Strategy Phase 1)**: Added independent "Preserve historical file backups" toggle and "File historical version limit" slider under "Settings -> Backup Settings".
  - Archives old main backup to  while creating a new main backup;
  - Default OFF with default limit of 2, completely independent from app backup settings to protect storage space;
  - Archives only when content actually changes; skips archiving when unchanged under incremental skip;
  - Properly removes oldest versions when exceeding limit without deleting all historical versions.
- **Three-way file backup strategy selector (File Strategy Phase 2)**: Upgraded Phase 1 toggle to a three-way strategy: Replace (default, overwrite old backup), Skip (skip backup if directory already backed up), Copy (archive previous backup as historical version).
  - Limit slider only shown when "Copy" is active. Smooth migration preserves legacy preferences;
  - Skip exits early before directory creation or archive compression, preventing empty remote directories on cloud accounts.
- **System data itemized results on completion page**: Dedicated "System Data" card on final completion page listing itemized success/failure status and counts for SMS, call logs, and Wi-Fi.

### Fixed & Improved

- **Fixed restore reading outdated version from JSON (Custom Fix)**: Package restore now updates existing package metadata directly from backup JSON files rather than relying on stale database entries, completely eliminating the bug where apps displayed and restored obsolete versions (such as 18.2 instead of 20.0).
- **Eliminated app list lag and counter jitter on resume (Custom Fix)**: Disabled automatic heavy re-scanning of all installed packages whenever the app resumes (). Manual refresh button remains instant and responsive, eliminating screen stutter and counter jumpiness.
- **Cleaned obsolete legacy backup folders (Custom Fix)**: Automatically delete leftover legacy backup directories when generating new main backups, preventing accumulation of orphaned folders.
- **Accurate backup pruning by actual creation date (Custom Fix)**: Preserved historical backups are now sorted strictly by actual creation timestamp () rather than version numbers, ensuring the true oldest backups are pruned according to user retention limits.
- **Fixed text layout and vertical squishing glitch (Custom Fix)**: Restructured  rows to position action buttons on a dedicated line beneath titles and subtitles, and placed the  badge on its own line below the title, eliminating vertical single-letter text squishing across all screen sizes and languages.
- **Ultra-fast Arm64 CI/CD builds (Custom Fix)**: Added Arm64-only build target, reducing CI build time from 12+ minutes to ~2 minutes.
- **Cloud retention limit pruning only deleted DB records while leaving remote folders**: Fixed cleanup disconnecting before remote deletion; now creates a short-lived dedicated connection to delete remote directories reliably, ensuring remote directory, config, and database remain synchronized.
- **System data selection rollback in one-click dialog**: Canceling the system data dialog now properly reverts selections to previous snapshot instead of keeping changes.
- **Prevented silent skipping of system data**: Resetting default selections when entering one-click backup/restore so system data is not silently skipped with .
- **"Select None" failing to clear filtered items**: "Select None" now clears selections across the entire table by operation type, unaffected by active filters or search terms.
- **One-click cloud restore switching to local when tapping Apps/Files**: Passed backup source parameters to list navigation routes so cloud restore stays on cloud source.
- **Restore list counter mismatch**: Fixed header counter  to filter by active backup source rather than counting the entire database.
- **Counter disappearing when selecting none**: Fixed header counter hiding when 0 items were selected; now displays  properly.
- **One-click restore showing 0.00 Bytes initially**: Synchronously scans cloud/local backup manifest when entering one-click restore if table is empty.
- **Refreshing restore list clearing selections**: Preserved existing checked states when refreshing package entities instead of resetting to unchecked.
- **One-click cloud restore failing for files**: Fixed independent scanning triggers for apps and files, and ensured cloud entity is properly bound before processing.
- **Connecting to wrong account in one-click cloud restore**: Synchronized active cloud account immediately upon selecting backup source in dialog.
- **Cloud upload failure deleting local archives**: Preserved local archives and temporary parts when upload fails to allow retry and debugging.
- **In-app changelog swallowing blockquotes**: Filtered out blockquotes, Obsidian comments, HTML comments, and code blocks from item bodies in .
- **Untrimmed label names**: Trimmed label names in  to prevent invisible blank labels.
- **Completion page saying "files" instead of "directories"**: Corrected terminology across Chinese and English strings.
- **Crash when chaining from app restore to file restore**: Normalized URL parameters with proper encoding/decoding to prevent route resolution crashes.
- **Completion page counting skipped items as success**: Excluded skipped items from success count and derived accurate skipped directory counts.
- **One-click cloud backup placing files locally when apps unselected**: Propagated cloud account and remote directory parameters to file backup graph.
- **Snapshot fallback for >20,000 files**: Degraded to tar numbered backup when directory exceeds 20,000 files to guarantee zero loss.
- **Archive rollback on failure under "Copy" strategy**: Cleanly restored previous main version and database records if compression or upload fails.
- **Cloud upload failure archives deleted by cleanup**: Moved failed upload archives to persistent private  directory.
- **Consistency fixes across status and results**: Fixed failure deletion logic, added 5-minute timeout for interactive decision prompt, and verified config JSON return values.
- **Encoding and timestamp consistency**: Fixed URL percent-encoding and enforced  on calendar timestamps.
- **Phantom versions remaining after rollback**: Resolved true database IDs by composite keys when deleting rolled-back records.
- **Zero-byte archive false success reporting**: Added strict archive non-empty checks before declaring backup success.

### Optimizations

- Completed missing Traditional Chinese strings.
- Added  to Room flows on label relations table.
- Skipped label queries when "Display tags in list" is disabled to save main thread overhead.
- Added concurrency checks and exception handling for one-click backup/restore.

## v3.15.2 (2026-10-19)

### Fixed

- **Three-line app list items**: List items now show App Name, Version, and Package Name on separate lines to prevent clipping on high-density screens and large font settings.
  - Line 1: App Name; Line 2: Version (version name + version code in smaller font); Line 3: Package Name;
  - Retains behavior of both toggles: turning off package name leaves two lines; turning off version places app name and package name on single rows;
  - Increased item height ensures all three items remain readable.

## v3.15.0 (2026-10-17)

### Added

- **Batch tagging**: Added batch tag operations (Add existing tag, Create and apply new tag, Remove tags) for selected applications from the overflow menu.
  - Tagging and untagging operate on full database selection set rather than visible filtered list;
  - Menu actions permanently visible with guidance toast when no tags exist;
  - Writes combined into a single atomic database transaction.
- **Persistent sort preferences**: Sorting mode and direction are now independently remembered for Apps and Files across app restarts.
- **List display toggles**: Filter panel now includes independent toggles to show/hide package names and version numbers in the list.
- **Details page version split**: Version string split into "Version Name" (vendor version string) and "Version Code" (numeric build comparison).
- **FAT32 partition pre-check**: Pre-checks target filesystem type before backup, skipping FAT32 partitions if uncompressed source exceeds 6 GiB to avoid write errors.

### Fixed

- **Batch tagging filtered items**: Resolved an issue where batch tagging only applied to visible filtered items rather than all selected items.
- **Tag list refresh**: App list immediately refreshes upon applying or deleting tags.
- **Tag deletion cleanup**: Cascading cleanup when tags are deleted to prevent orphaned relations.
- **Filter panel scrolling**: Resolved panel clipping and gesture conflicts when displaying extensive tag lists.
- **Residual screen-off countdown on completion page**: Fixed indefinite countdown prompt remaining visible when tasks finish under 3 seconds.
- **Bare parentheses in list subtitles**: Fixed solitary  appearing when  is empty.

## v3.14.0 (2026-10-05)

### Added

- **Incremental backup**: Calculates source directory manifest hash before backup and skips unchanged data types without redundant re-compression.
  - Skipping unchanged types saves heavy CPU overhead and heat on large apps;
  - Repackages only truly modified data types while inheriting unchanged archives from historical versions;
  - Full backups remain standard on cloud storage.
- **Backup version notes**: Added custom notes for individual backup archives accessible directly from application details, displayed in app list below app names.
  - Notes saved per version identity (package name + user ID + preserveId + storage location);
  - Search bar matches custom version notes.
- **Manifest metadata**: Generates  metadata alongside archives to track source hashes and packaging parameters for incremental checks.

### Fixed

- **Cloud backup reload list empty**: Fixed reload logic so valid package entities are re-inserted into the database immediately upon directory scan.
- **Data types restore checkable state**: Fixed an issue where USER_DE, DATA, OBB, and MEDIA types were not marked as restorable during cloud reload.
- **Preserved archive completeness**: Ensured unselected data types retain their previous archives when creating historical backups.
- **Data type cleared disappearing from main backup**: Preserved previous archives for types no longer present locally to ensure main backup completeness.
- **Leftover partial files on inheritance failure**: Cleaned up copied files if inheritance from protected versions fails.
- **Redundant pre-backup directory scan**: Combined duplicate scans into a single calculation shared across archive decisions.
- **Documentation and assumptions correction**: Clarified that  on Android is FUSE and does not support hard links, relying on  for full scans.

## v3.13.4 (2026-10-04)

### Fixed

- **MediaProvider trashed and pending files exclusion**: Unified exclusion pattern using AOSP  across backup and restore paths to prevent restoring inaccessible deleted images.
- **Exclusion logging**: Detailed logging of skipped trashed files with sample filenames and total counts.
- **File media exclusion logging**: Added exclusion logs to file media backups matching existing app data behavior.

## v3.13.3 (2026-09-22)

### Fixed

- **Skipped deleted photos in media backups**: Excluded  temporary recycle bin files during photo gallery backups and restores.

## v3.13.2 (2026-09-20)

### Fixed

- **Cloud file restore showing 0.00 Bytes**: Corrected initialization order so cloud entities are bound before file counts are calculated.
- **Empty cloud file backup search**: Fixed parameter passing so  correctly scans remote directories.
- **System data cloud restore error handling**: Greys out cloud accounts that contain no system data backups rather than failing upon restore attempt.
- **Permission denied false alarms on unreadable files**: Evaluated archive validity using  exit status rather than treating non-zero reading exit codes as immediate failures.

## v3.13.1 (2026-09-20)

### Fixed

- **Reload parsing md5 files as archives**: Filtered out  checksum files before traversal to prevent bogus parse errors.
- **Infinite loading state on reload exception**: Wrapped reload execution in  to guarantee state reset.
- **Hardcoded current version string**: Replaced hardcoded string with localized resource .

### Optimizations

- **Compression options clarification**: Clarified descriptions for compression levels and thread counts across 19 languages.

## v3.13.0 (2026-09-18)

### Added

- **Cloud backup multipart splitting**: Added option to split cloud archives into chunks (1 GB / 2 GB / 4 GB) to bypass cloud storage single-file limits.
- **Large archive warning**: Notifies in task logs when unsegmented cloud archives exceed 10 GB.
- **In-app changelog**: Added dedicated Changelog page in Settings -> About and via version badge in Dashboard, reading offline from packaged markdown.

### Optimizations

- **Default disabled restore permissions on HyperOS**: Defaults permission restoration to off on HyperOS to prevent forced permission overrides.
- **Backup settings layout**: Grouped cloud split size under app killing options and made compression settings collapsible.

### Fixed

- **Skipped user data when restoring multipart backups**: Fixed existence checks to recognize  multipart archives on cloud backends.
- **Corrupted merged files**: Replaced fragile shell concatenation with buffered streams and length self-verification.

## v3.12.1 (2026-09-15)

### Fixed

- **Notification throttling during app scan**: Throttled foreground notification updates to 250 ms intervals, preventing main-thread ANR on devices with hundreds of apps.
- **Background start exceptions**: Wrapped  in safe handlers to prevent background startup crashes on Android 12+.

### Optimizations

- **Cached permission definitions**: Cached  results at the process level to eliminate redundant cross-process IPC.
- **Reused SsaidUtil per userId**: Reused handler threads and parsed SSAID instances per user.
- **Room Converters Gson reuse**: Reused singleton Gson instances in Room type converters to avoid per-row allocations.

## v3.12.0 (2026-09-14)

### Added

- **System data backup and restore**: Added dedicated system data backup module for SMS/MMS, Call Logs, and Wi-Fi networks with local and cloud storage support.
- **Pre-restore schema compatibility check**: Validates database table and column schemas before overwriting system databases.

### Security

- **Clear text credential protection**: Ensured Wi-Fi credentials and sensitive command-line parameters are scrubbed from log exports.
- **Dynamic UID assignment**: Read UID, permissions, and SELinux contexts dynamically instead of hardcoding.

### Fixed

- **Schema check missing entire table**: Fixed schema validation to flag missing tables as discrepancies before overwrite.

## v3.11.0 (2026-09-13)

### Fixed

- **False failures on cloud export cleanup**: Upload success is now preserved even if local temporary cleanup encounters minor errors.
- **WebDAV integrity verification**: Added remote file size checks following WebDAV uploads.
- **Cleaned remote partial archives**: Removed partial uploads on network interruption across FTP, SFTP, SMB, and WebDAV.
- **Export page ANR**: Moved large directory scans from main thread to background IO dispatcher.
- **Screen rotation interrupting export**: Bound migration exports to  to persist through orientation changes.
- **Legacy backup directory recognition**: Added detection for both  and legacy pure pkg 0.1.0 -- tlx package manager

  pkg update              fetch repository indexes
  pkg upgrade             upgrade installed packages
  pkg install <pkg...>    resolve, download, extract
  pkg remove <pkg...>     remove installed packages
  pkg clean               remove cached .deb archives
  pkg list [regex]        available packages
  pkg installed           installed packages
  pkg info <pkg>          package metadata
  pkg search <regex>      search by name
  pkg why <file>          which package owns a file
  pkg selftest            offline self-test
  pkg help | version

env: PREFIX=/data/user/0/com.agy/files/usr  PKG_ROOT=/data/user/0/com.agy/files/usr/var/lib/pkg directory structures.

### Added

- **Dedicated cloud migration success message**: Distinct success confirmation for cloud uploads across 18 languages.

### Optimizations

- **Four-stage export progress**: Segmented progress tracking (verify / pack / hash / upload) with real-time transfer rates.
- **Throttled upload callbacks**: Throttled WebDAV upload progress updates to 300 ms.

## v3.10.5 (2026-09-12)

### Fixed

- **Cloud backup crash on cross-parent rename**: Ensured archive operations take place strictly within the directory parent path.
- **Cloud delete path resolution**: Correctly resolved target paths according to the package-only directory setting.
- **Reversible protection toggle**: Allowed toggling between protected and unprotected states with conflict prevention.
- **Empty parent directories cleanup**: Cleaned up empty parent application folders upon backup deletion.

## v3.10.1 (2026-09-11)

### Fixed

- **Accurate app size calculation**: Calculated display sizes based strictly on selected components (APK, data, etc.) rather than total size.

## v3.10.0 (2026-09-11)

### Fixed

- **Navigation filter state loss**: Maintained filter button and user space selection state when returning to list views.
- **Batch selection system apps exclusion**: Excluded system applications from batch selection by default unless explicitly enabled.
- **Deleted multi-user spaces handling**: Marked removed user spaces properly in restore lists and cleaned stale references.

## v3.9.0 (2026-09-06)

### Notice: Signing Key Migration

- Transitioned to permanent release signing key. Users updating from older builds should backup settings before reinstalling.

### Added

- **Package-only cloud directory toggle**: Enabled using pure package names for cloud backup folders to prevent encoding errors on non-UTF8 FTP servers.

### Fixed

- **Cold start scheduled backup environment**: Initialized binaries and environment paths before executing scheduled backup tasks.
- **Log system hardening**: Captured comprehensive crash diagnostics and system dropbox entries in exported logs.

## v3.6.7 (2026-08-27)

### Added

- **Cloud migration packages**: Added direct export and import of full migration bundles to cloud storage ().
- **Companion CLI diagnose mode**: Added  command to companion script for automated environment diagnostics.

### Fixed

- **Export directory stat errors**: Restricted migration export list strictly to local backups.
- **Human-readable migration filenames**: Formatted migration filenames with localized timestamps for quick identification.

## v3.6.6 (2026-08-25)

### Fixed

- **Handled live file modifications during backup**: Treated GNU tar exit code 1 (changed: cannot open: No such file or directory
as:      cannot open: No such file or directory
we:      cannot open: No such file or directory
read:    cannot open: No such file or directory
it:      cannot open: No such file or directory) as benign when archive structure passes integrity checks.
- **Dual-app user space deletion crash**: Fixed index boundary exceptions in  when deleting multi-user backups.
- **External storage SELinux contexts**: Enforced  on restored  directories.

## v3.6.5 (2026-08-24)

### Added

- **Repair data ownership**: Added automated UID ownership verification and correction for application data directories to fix permission errors after reinstalling.
- **Desktop application name localization**: Dynamically localized application label to match system language.

### Fixed

- **APK backup command argument quoting**: Resolved wildcard expansion failure when backing up APK files.

## v3.6.0 (2026-08-23)

### Security

- **Shell command injection prevention**: Standardized all root shell command arguments on strict POSIX single-quote escaping ().
- **Migration package import path traversal protection**: Rejected migration bundles containing relative paths () or absolute paths.
- **SHA-256 integrity verification**: Added cryptographic checksum verification for exported and imported migration archives.
- **WebDAV HTTPS enforcement**: Enforced secure HTTPS connections unless insecure connections are explicitly permitted.

## v3.5.0 (2026-08-19)

### Fixed

- Resolved device data migration crashes and character encoding issues on HyperOS devices.
- Fixed main-thread ANR during high-volume migration exports and imports.

## v3.4.0 (2026-08-19)

### Added

- **Pre-restore integrity check**: Tested archive integrity with  before initiating extraction.

### Fixed

- Resolved application restoration failures following major operating system version upgrades.

## v3.2.3 (2026-08-18)

### Fixed

- Queried active package version directly from  during backup to avoid recording stale cached version numbers.

## v3.2.2 (2026-08-18)

### Added

- **One-click device migration**: Introduced full migration bundle creation and extraction for rapid device transitions.

## v3.2.1 (2026-08-17)

### Fixed

- Resolved WebDAV URL encoding issues, protected version timestamp formatting, and size display glitches.

## v3.2.0 (2026-08-17)

### Added

- **Device fingerprint clearance**: Added GAID and GSF ID randomization during restore.

### Fixed

- Maintained denied permission states when restoring application permissions.

## v3.1.8 (2026-08-15)

### Optimizations

- Accelerated app details calculation with parallel processing and timeouts.
- Prevented null pointer exceptions when traversing large filesystem directories.
- Removed list translation animations to reduce UI stutter.

## v3.1.7 (2026-08-15)

### Added

- Implemented global memory caching for application icons with background decoding.
- Added comprehensive log export to zip format via system document provider.

## v3.1.5 (2026-08-15)

### Added

- **Preserve historical backups**: Added retention toggle and limit slider for app backups with numbered shield indicators.
- **Multiple version identification**: Displayed backup creation dates and shield badges in restore views.

## v3.1.0 (2026-08-14)

### Added

- **Android ID randomization**: Supported generating randomized Android IDs for applications upon restore.
