# Changelog

All notable changes to the CGS Assist Platform will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.3.0] - 2026-01-08

### Added

#### Automation System
- Complete use case automation framework with scheduled executions (#42)
- Determined (specific date/time) and periodic (interval-based) scheduling options
- APScheduler integration with Redis-backed job store
- Dual log entry system: separate start and completion logs for audit trail
- Full result data storage including LLM responses, token usage, and metadata
- Visual automation log viewer with detailed modal for result inspection
- Role-based permission system for automation access (`AutomatedUseCaseRolePermission`)
- Real-time unread badge counts with role-aware filtering
- Designated scheduler worker pattern to prevent duplicate job executions
- Redis-based distributed locking with double-check pattern
- Post-scheduling verification and unauthorized job cleanup

#### Multi-User Collaboration
- Hybrid read system: global read status + individual user confirmations
- "Read by users" tracking with usernames and timestamps
- Visual indicators for unread entries (bold text, blue border, gray background)
- Independent read status per user with global badge updates
- Dynamic button text based on read status

#### Task Execution Framework
- Four execution types: CHAT, USECASE, WORKFLOW, AUTOMATION
- Factory pattern for specialized executor creation
- Separate executors: `ChatExecutor`, `UseCaseExecutor`, `WorkflowExecutor`, `AutomationExecutor`
- Centralized notification service with WebSocket messaging
- Handler-based notification architecture
- Type-safe execution contexts for each execution type

#### Customer Onboarding
- Comprehensive 8-phase onboarding template with 190+ tasks
- Authentication method selection workflow
- LLM provider and model selection guides
- Use case clarification templates (ARC, General, Individual)
- Automated acceptance testing procedures
- Workshop and training scheduling templates
- GitHub issue templates for bug reports and onboarding checklists

#### AWS Integration
- AWS Bedrock provider support (#16, #21)
- Inference profile integration
- Enhanced Mistral model handling
- Cross-region inference capabilities

#### UI Enhancements
- Model selection dropdown for use case execution (#14, #15, #20)
- Resizable sidebar with drag functionality
- Active link highlighting with scroll synchronization
- Visual execution selection helper
- Enhanced modal layouts (modal-xl for better viewing)

#### BaFin Features
- Time period filtering (24h, 7d, 30d, 90d, all)
- Keyword search functionality
- Enhanced RSS feed date parsing and formatting

### Changed

#### Architecture
- Refactored TaskManager to lightweight coordinator role
- Moved execution logic to specialized executors
- Implemented responsibility-driven design pattern
- Separated notification logic into dedicated handlers
- Centralized workflow utility functions

#### API
- Updated `/start_prompt_task` to accept `execution_type` parameter
- Enhanced `/usecase-execute` with automatic execution type detection
- Improved permission checking with detailed logging
- Standardized task status to 'finished' (previously 'completed')

#### Database
- Added `is_read_global`, `marked_as_read_by`, `marked_as_read_at` columns to automation logs
- Created `automated_usecase_schedules` table
- Created `automated_usecase_role_permissions` table
- Added `read_by_users` JSON column for multi-user tracking

#### Logging
- Centralized log level configuration
- Reduced production log noise
- Enhanced debugging capabilities for automation
- Consistent DEBUG, INFO, WARNING, ERROR usage

#### Dependencies
- Converted to relative imports for better module encapsulation
- Pinned `pytz` version for stability
- Updated Docker configurations

### Fixed

#### Critical
- **Race condition in automation scheduling**: Fixed duplicate job execution by multiple workers
- **GPT-5.1 API crash**: Fixed parameter handling for GPT-5+ and o1 models (#19)
- **Modal backdrop bug**: Fixed duplicate backdrop creation blocking page interaction
- **Circular imports**: Resolved with lazy imports in executor framework

#### WebSocket
- Improved connection handling and authentication
- Enhanced redirect logic for authentication failures
- Fixed connection management per user session
- Improved style and visual indicators

#### Use Case Execution
- Fixed overlay not appearing on execution start
- Added proper WebSocket notification in `UseCaseExecutor.prepare()`
- Form correctly becomes readonly during execution

#### Workflow
- Fixed progress update handling in event parser
- Centralized progress reporting functions
- Enhanced debugging with detailed progress messages

#### Chat UI
- Fixed chat box overflow issues (#39)
- Improved responsive design
- Enhanced copy button text overflow management
- Fixed token usage tooltip display

#### Links & Security
- All external links now open in new tab (#41)
- Added `rel="noopener noreferrer"` security attributes
- Enhanced markdown link rendering

#### Code Quality
- Fixed unused imports and variables
- Resolved variable redefinition issues
- Cleaned up deprecated code
- Improved type annotations

### Security

- Added `rel="noopener noreferrer"` to external links
- Enhanced permission checking for automation logs
- Role-based access control for use case automation
- Improved WebSocket authentication handling

### Deprecated

- Old task status 'completed' replaced with 'finished'
- Direct automation log creation in scheduler (now handled by executor)

### Removed

- Monolithic task execution logic from TaskManager
- Manual WebSocket notification calls in routes
- Deprecated authentication flow components

### Documentation

- Added 20+ new implementation documentation files
- Created comprehensive automation system guides
- Documented task execution architecture
- Added customer onboarding guides
- Created quick reference guides for developers
- Enhanced deployment documentation

### Migration Notes

**Required Actions:**
1. Run database migrations: `alembic upgrade head`
2. Configure Redis server for distributed locking
3. Set up designated scheduler worker for Celery
4. Run role permission migration: `python migrate_usecase_roles.py`
5. Review and update `.env` configuration

**Breaking Changes:**
- API endpoints now require `execution_type` parameter
- Task status changed from 'completed' to 'finished'
- New database tables required for automation features
- WebSocket message format updated

---

## [1.2.6] - 2025-12-07

### Changed
- Refined BaFin news fetching user messages for better clarity
- Version bump for stable release

---

## [1.2.5] - 2025-12-06

### Added
- Enhanced BaFin news fetching with time period filters
- Keyword search functionality for BaFin RSS feeds
- Improved system prompts

---

## [1.2.3] - 2025-10-28

### Added
- Active link highlighting in sidebar
- Icon hover effects improvements

### Fixed
- Translation key for feedback button
- Token usage info tooltip display

---

## [1.2.2] - 2025-10-27

### Changed
- Refactored navbar and button styles
- Enhanced copy button styling

### Added
- Filtering by text content in metadata search

---

## [1.1.1] - 2026-01-05

### Added
- Automation flag support
- Updated Docker build context

---

## [1.0.2] - 2025-11-03

### Added
- File upload for Document B in document comparison use case

---

## Earlier Versions

For release notes prior to v1.0.2, please refer to git commit history.

---

[1.3.0]: https://github.com/CGSmbH/cgs-assist/compare/v1.2.6...v1.3.0
[1.2.6]: https://github.com/CGSmbH/cgs-assist/compare/v1.2.5...v1.2.6
[1.2.5]: https://github.com/CGSmbH/cgs-assist/compare/v1.2.3...v1.2.5
[1.2.3]: https://github.com/CGSmbH/cgs-assist/compare/v1.2.2...v1.2.3
[1.2.2]: https://github.com/CGSmbH/cgs-assist/compare/v1.1.1...v1.2.2
[1.1.1]: https://github.com/CGSmbH/cgs-assist/compare/v1.0.2...v1.1.1
[1.0.2]: https://github.com/CGSmbH/cgs-assist/releases/tag/v1.0.2

