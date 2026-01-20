# CGS Assist Platform - Release Notes

## Version 1.3.0 - January 8, 2026

This major release introduces comprehensive automation features, significant architectural improvements, and numerous bug fixes. Version 1.3.0 represents a substantial enhancement to the CGS Assist Platform with focus on use case automation, multi-user collaboration, and system reliability.

---

## 🚀 Major Features

### Use Case Automation System
Complete automation framework for scheduled use case executions with comprehensive logging and monitoring.

- **Automated Scheduling** (#42)
  - Schedule use cases with determined (specific date/time) or periodic (intervals) execution
  - Support for weekly, daily, hourly, and minute-based intervals
  - APScheduler integration with Redis-backed job store
  - Visual schedule management interface in admin panel
  - Cron-like job triggers for complex scheduling patterns
  - **References**: `AUTOMATION_COMPLETION_LOGGING.md`, `AUTOMATION_OPTIMIZATION_COMPLETED.md`

- **Automation Logging & Monitoring**
  - Dual log entry system: separate start and completion logs for complete audit trail
  - Full result data storage including LLM responses, token usage, and execution metadata
  - Role-based permission system for automation logs (`AutomatedUseCaseRolePermission`)
  - Visual log viewer with detailed modal for inspection of results
  - Real-time unread badge counts with role-based filtering
  - Structured result display with color-coded sections and collapsible raw JSON view
  - **References**: `AUTOMATION_COMPLETION_LOGGING.md`, `IMPLEMENTATION_SUMMARY.md`

- **Multi-User Read Tracking**
  - Hybrid read system: global read status + individual user confirmations
  - Visual indicators for unread entries (bold text, blue left border, gray background)
  - "Read by users" tracking with usernames and timestamps
  - Independent read status per user with global badge updates
  - Dynamic button text ("Mark as read" vs "I have it read too")
  - **References**: `AUTOMATION_MULTI_USER_READ_TRACKING.md`, `GLOBAL_READ_SYSTEM_IMPLEMENTATION.md`, `HYBRID_READ_SYSTEM_SUMMARY.md`

- **Scheduler Optimization**
  - Designated scheduler worker pattern to prevent duplicate job execution
  - Redis-based distributed locking with double-check pattern
  - Automatic cleanup of unauthorized jobs from non-designated workers
  - Post-scheduling verification with detailed logging
  - Extended lock timeout (60 seconds) to prevent race conditions
  - **References**: `AUTOMATION_OPTIMIZATION_COMPLETED.md`, `RACE_CONDITION_FIX.md`

### Task Execution Framework Refactoring
Complete architectural overhaul of task execution system with responsibility-driven design.

- **Execution Types & Specialized Executors**
  - Four distinct execution types: CHAT, USECASE, WORKFLOW, AUTOMATION
  - Factory pattern for executor creation (`ExecutorFactory`)
  - Specialized executors with clear separation of concerns:
    - `ChatExecutor` - Direct chat interactions
    - `UseCaseExecutor` - Standard use case executions
    - `WorkflowExecutor` - Multi-step workflow executions
    - `AutomationExecutor` - Scheduled automated executions
  - **References**: `TASK_EXECUTION_REFACTORING_COMPLETE.md`, `REFACTORING_COMPLETE_SUMMARY.md`

- **Notification System**
  - Centralized notification service with WebSocket messaging
  - Handler-based architecture for different entity types
  - `UseCaseNotificationHandler` for execution status updates
  - `AutomationLogHandler` for automation log entries
  - User permission-aware notifications
  - **References**: `AUTOMATION_WEBSOCKET_NOTIFICATIONS.md`

- **Execution Context**
  - Type-safe execution contexts for each execution type
  - Encapsulated execution parameters and dependencies
  - Clean separation between coordination and execution logic
  - TaskManager refactored to lightweight coordinator role

### Customer Onboarding System
Comprehensive project management templates for customer deployments.

- **Onboarding Template**
  - 8-phase comprehensive checklist (Phase 0-7) with 190+ tasks
  - Project initialization, requirements gathering, deployment, and handover phases
  - Authentication method selection (Local, LDAP, SAML, OAuth2, MFA)
  - LLM provider and model selection workflows
  - Use case clarification (ARC, General, Individual specifications)
  - Automated acceptance testing procedures
  - Workshop and training scheduling
  - **References**: `ONBOARDING_TEMPLATE_COMPLETE.md`, `ONBOARDING_TEMPLATE_SETUP.md`, `QUICK_START_ONBOARDING.md`

- **GitHub Issue Templates**
  - Bug report template with structured sections
  - Customer onboarding checklist integration
  - Project reference and title format guidance
  - **References**: `GITHUB_ISSUE_TEMPLATES_SETUP.md`

### Use Case Role-Based Permissions
Fine-grained access control for use cases and automation features.

- **Role Restrictions**
  - Use case access tied to user roles
  - Separate permissions for execution vs automation access
  - Role-based filtering for automation logs and unread counts
  - Migration script for existing deployments (`migrate_usecase_roles.py`)
  - **References**: `USECASE_ROLE_QUICK_REFERENCE.md`, `DEPLOYMENT_GUIDE_USECASE_ROLES.md`

### AWS Bedrock Integration (#16, #21)
Support for AWS Bedrock as LLM provider with inference profiles.

- **Provider Support**
  - AWS Bedrock inference profile integration
  - Enhanced Mistral model handling
  - Provider type and extra configuration support
  - Cross-region inference capabilities
  - **Commits**: `40a484e`, `02f54e6`, `873e8f7`

---

## 🎨 User Interface Improvements

### Navigation & Sidebar
- **Sidebar Enhancements**
  - Drag functionality for resizable sidebar
  - Active link highlighting with scroll synchronization
  - Visual execution selection helper function
  - Responsive behavior for mobile and desktop
  - Use cases section heading and organization
  - **Commits**: `f3a5002`, `8ec0517`, `3920fee`

### Admin Interface
- **Use Case Management**
  - Model selection dropdown for use case execution (#14, #15, #20)
  - Schedule type selection (determined vs periodic)
  - Automation log viewer with filtering and sorting
  - Enhanced modal layouts (modal-xl for better viewing)
  - **Commits**: `c735cec`, `02aff3d`, `068bc04`

### Chat Interface
- **Chat UI Refinements** (#39)
  - Fixed chat box overflow issues
  - Improved responsive design with footer adjustments
  - Enhanced copy button styling with better text overflow management
  - Active chat highlighting in sidebar
  - Token usage info tooltip improvements
  - **Commits**: `c3a9acb`, `f050c2c`, `621cef7`, `1f19f5d`

### Links & Security (#41)
- **Link Handling**
  - All external links open in new tab (`target="_blank"`)
  - Added `rel="noopener noreferrer"` for security
  - Enhanced markdown rendering with proper link attributes
  - HTML template updates for consistent link behavior
  - **Commits**: `3f84db4`

---

## 🐛 Bug Fixes

### Critical Fixes

- **Race Condition in Automation Scheduling** (Dec 12-17, 2025)
  - Fixed double-check lock pattern to prevent duplicate job execution
  - Resolved issue where multiple workers scheduled the same automation jobs
  - Implemented post-scheduling verification and unauthorized job cleanup
  - Extended lock timeout from 30s to 60s to prevent premature lock release
  - **References**: `RACE_CONDITION_FIX.md`, `CRITICAL_LOCK_RELEASE_BUG_FIX.md`

- **GPT-5.1 API Crash** (#19)
  - Fixed model parameter handling for GPT-5+ and o1 models
  - Use `max_completion_tokens` instead of `max_tokens` for newer models
  - Improved model detection logic with regex patterns
  - Prevented false positives in model version detection
  - **Commits**: `1f79a28`, `58994e4`, `d64ae39`, `fab89a1`, `478219d`

- **Modal Backdrop Issue**
  - Fixed duplicate modal backdrop creation when marking logs as read/unread
  - Implemented `updateModalContent()` function to refresh modal without recreating instance
  - Resolved page interaction blocking caused by leftover backdrops
  - **References**: `MODAL_BACKDROP_FIX.md`

### WebSocket & Real-Time Updates

- **WebSocket Improvements**
  - Fixed and improved WebSocket connection handling
  - Style improvements for connection status indicators
  - Enhanced authentication and redirect logic
  - Connection management per user session
  - **References**: `WEBSOCKET_AUTH_FIX_SUMMARY.md`, `WEBSOCKET_REFACTORING_COMPLETE.md`, `WEBSOCKET_UTILS_QUICK_REFERENCE.md`
  - **Commits**: `c93c3f3`

- **Use Case Execution Overlay**
  - Fixed overlay not appearing when use case execution starts
  - Added `UseCaseExecutor.prepare()` method with WebSocket notification
  - Form becomes readonly during execution with visual feedback
  - **References**: `USECASE_OVERLAY_FIX.md`

- **Chat Response Display**
  - Fixed frontend waiting for wrong status ('completed' vs 'finished')
  - Standardized status definitions across frontend and backend
  - **References**: `TASK_STATUS_DEFINITION.md`

### Workflow System

- **Workflow Progress Updates**
  - Centralized workflow utility functions (`workflow_utils.py`)
  - Fixed progress update handling in event parser
  - Consistent progress reporting across all workflow operators
  - Enhanced debugging with detailed progress messages
  - **References**: `WORKFLOW_PROGRESS_FIX.md`, `WORKFLOW_AUTOMATION_FIX.md`

### Import & Code Quality

- **Module Import Refactoring**
  - Converted to relative imports for better module encapsulation
  - Fixed circular import issues in executor framework
  - Lazy imports in all executors to prevent startup issues
  - Cleaned up unused imports and variables
  - Pinned `pytz` version for stability
  - **References**: `FIX_MODULE_IMPORTS.md`
  - **Commits**: `10e527d`, `6f9e4bc`, `1168598`

---

## 🔧 Technical Improvements

### Database Changes

- **New Tables & Columns**
  - `automated_usecase_schedules` - Schedule management table
  - `automated_usecase_logs` - Execution log entries
  - `automated_usecase_role_permissions` - Role-based access control
  - `is_read_global`, `marked_as_read_by`, `marked_as_read_at` - Global read tracking
  - `read_by_users` JSON column - Multi-user confirmation tracking
  - **Migration Scripts**: `f19667f3a4a5_add_global_read_status_to_automation_logs.py`

- **Pydantic Models**
  - `AutomatedUseCaseLogRead` with computed fields
  - `AutomatedUseCaseScheduleCreate/Read/Update`
  - `AutomatedUseCaseRolePermission`
  - Enhanced validation and type safety

### API Enhancements

- **New Endpoints**
  - `/api/automation/schedules` - Schedule CRUD operations
  - `/api/automation/logs/{usecase_id}` - Log retrieval with permissions
  - `/api/automation/logs/{log_id}/mark-read` - Read status management
  - `/api/automation/unread-count` - Role-aware unread counting
  - Enhanced `/usecase-execute` with execution type detection

- **Permission System**
  - `check_automation_permission()` helper function
  - Two-layer filtering: use case access + automation role permissions
  - Detailed permission logging for debugging
  - **References**: `AUTOMATION_UNREAD_LOGIC.md`

### Redis & Locking

- **Distributed Lock Improvements**
  - Enhanced `acquire_lock()` with PID and UUID tracking
  - Better lock holder visibility in logs
  - Atomic Redis operations with `nx=True` flag
  - Lock timeout management (60 seconds for scheduler)
  - **References**: `AUTOMATION_REDIS_LOCKS.md`

### Logging & Debugging

- **Log Level Centralization**
  - Centralized log level mapping configuration
  - Consistent DEBUG, INFO, WARNING, ERROR usage across modules
  - Reduced noise in production logs
  - Enhanced debugging capabilities for automation system
  - **References**: `LOGLEVEL_CENTRALIZATION.md`, `DEBUGGING_AUTOMATION_LOGS.md`, `CSS_REFACTORING_AUTOMATION_LOGS.md`

### BaFin Use Case Enhancements
- **RSS Feed Improvements**
  - Time period filtering (24h, 7d, 30d, 90d, all)
  - Keyword search functionality
  - Date parsing and entry formatting enhancements
  - Refined system prompts for better responses
  - **Commits**: `fa46703`, `15a6c02`, `26a768f`

---

## 📦 Dependencies & Infrastructure

### Docker & Deployment
- **Docker Improvements**
  - Updated Dockerfile with corrected file copy paths
  - BaFin image build and push integration
  - Customer deployment folder with docker-compose management script
  - Caddy reverse proxy configuration for SSL/TLS
  - Auto-restart policies and pull strategies
  - **Commits**: `bb8800b`, `940123c`, `663c745`, `978e355`, `dd15ca0`

### Version Management
- **Version Bumps Throughout Development**
  - CGS_TAG: 1.3.0
  - LLM_TAG: 1.3.0
  - RAG_TAG: 1.3.0
  - BAFIN_TAG: 1.0.0
  - Incremental version updates during feature development (1.1.1, 1.2.3, 1.2.5, 1.2.6)

---

## 📚 Documentation

### New Documentation Files
- `AUTOMATION_COMPLETION_LOGGING.md` - Complete logging system guide
- `AUTOMATION_MULTI_USER_READ_TRACKING.md` - Multi-user read tracking details
- `AUTOMATION_OPTIMIZATION_COMPLETED.md` - Scheduler optimization documentation
- `AUTOMATION_REDIS_LOCKS.md` - Redis locking mechanisms
- `AUTOMATION_UNREAD_LOGIC.md` - Unread count calculation logic
- `AUTOMATION_WEBSOCKET_NOTIFICATIONS.md` - Real-time notification system
- `TASK_EXECUTION_REFACTORING_COMPLETE.md` - Execution framework architecture
- `GLOBAL_READ_SYSTEM_IMPLEMENTATION.md` - Hybrid read system design
- `ONBOARDING_TEMPLATE_COMPLETE.md` - Customer onboarding guide
- `RACE_CONDITION_FIX.md` - Race condition resolution details
- `WORKFLOW_PROGRESS_FIX.md` - Workflow progress system
- `MODAL_BACKDROP_FIX.md` - UI bug fix documentation
- `QUICK_REFERENCE_MULTI_USER_READ.md` - Quick reference guide
- `USECASE_ROLE_QUICK_REFERENCE.md` - Role permission reference
- `WEBSOCKET_UTILS_QUICK_REFERENCE.md` - WebSocket utilities guide

### Developer Documentation
- Complete agent framework documentation
- Customer deployment guides
- Testing guides for automation permissions
- Migration scripts documentation

---

## 🔄 Migration & Upgrade Notes

### Database Migrations Required
Run the following Alembic migrations:
```bash
alembic upgrade head
```

Key migrations:
- Add `is_read_global`, `marked_as_read_by`, `marked_as_read_at` columns to `automated_usecase_logs`
- Create `automated_usecase_schedules` table
- Create `automated_usecase_role_permissions` table
- Add schedule type and periodic fields

### Configuration Updates
- **Redis**: Ensure Redis server is configured for distributed locking
- **Celery Workers**: Configure designated scheduler worker for automation jobs
- **Environment Variables**: Review `.env` for new automation-related settings
- **APScheduler**: Redis job store configuration required

### Role Permission Migration
For existing deployments, run:
```bash
python migrate_usecase_roles.py
```

This migrates existing use case access to the new role-based permission system.

---

## 🎯 Breaking Changes

### API Changes
- `start_prompt_task` now requires `execution_type` parameter (defaults to CHAT)
- Task status changed from 'completed' to 'finished' (frontend updated)
- Automation logs now use dual entry system (start + completion)

### Database Schema
- New required tables for automation features
- Modified `automated_usecase_logs` schema with additional columns
- Role-based permission system requires migration for existing use cases

### WebSocket Messages
- Updated message format for execution status updates
- New notification types for automation logs
- Enhanced user permission filtering

---

## 🙏 Contributors

- **Tobias Sobek** - Core development, automation system, refactoring
- **Dominik Molitor** - Brecht use cases, SharePoint syncing, deployment
- **copilot-swe-agent[bot]** - Code quality improvements, bug fixes

---

## 📊 Statistics

- **Total Commits**: 150+ commits in v1.3.0 development cycle
- **Development Period**: November 2025 - January 2026
- **Files Changed**: 100+ files across backend, frontend, and documentation
- **New Features**: 15+ major features
- **Bug Fixes**: 20+ resolved issues
- **Documentation**: 20+ new documentation files

---

## 🚦 Known Issues

None reported at release time. Please report issues via GitHub issue tracker using provided templates.

---

## 📅 Next Steps

Future development priorities for v1.4.0:
- Enhanced workflow visualization
- Advanced analytics dashboard
- Additional LLM provider integrations
- Performance optimizations for large-scale deployments

---

## 📖 Additional Resources

- **Full Documentation**: `docs/README.md`
- **Deployment Guide**: `DEPLOYMENT_CHECKLIST.md`
- **Testing Guide**: `TESTING_GUIDE.md`
- **Agent Framework**: `common/agent_framework/README.md`

---

For detailed technical information on any feature, please refer to the specific documentation files referenced throughout these release notes.

