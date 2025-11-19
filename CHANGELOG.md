# Changelog

All notable changes to Curly will be documented in this file.

## [2025-11-19]

### Added
- **File Upload Support** - Added `-T/--upload-file` option for uploading files in API requests
  - Automatically detects Content-Type based on file extension (json, xml, pdf, images, etc.)
  - Content-Type can be overridden with explicit header
  - Validates file existence before attempting upload
  - Cannot be used simultaneously with `-d/--data` option
  - Full documentation and examples added to README

### Changed
- **README Documentation** - Simplified and clarified documentation
  - Removed marketing/sales-pitch language
  - Made documentation more technical and straightforward
  - Better focused on practical usage and examples

### Fixed
- **State File Bug** - Fixed issue with `.curly-state` file being incorrectly overridden
  - Changed STATE_FILE from readonly constant to regular variable to allow proper updates
  - Fixed jq value extraction to use compact output (`-rc` flag) preventing formatting issues
  - Ensures saved values are properly stored without unnecessary whitespace or formatting