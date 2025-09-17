# Construction MVP - Feature Specifications

## Overview
Field operations platform connecting PMs to shop/field crews with real-time control and traceability from schedule → work orders → piece marks → installation → sign-off.

---

# Feature 1: Authentication & Organization Management
**Purpose**: Secure multi-tenant organization setup with role-based access control and audit trails
**User Flow**: Admin logs in → manages organization → creates users/roles → configures projects → monitors audit logs

## Frontend Requirements
**UI Components**:
- Login/signup forms with field PIN option
- Organization setup wizard
- User management table with role assignment
- Audit log viewer with filtering
- Role permission matrix

**Pages/Layout**:
- `/login` - Authentication with PIN option
- `/admin/organization` - Org settings
- `/admin/users` - User management
- `/admin/audit` - Audit log viewer
- Protected route wrapper for role-based access

**State Management**:
- User session and permissions
- Organization settings
- Active audit filters

**User Experience**:
- Quick PIN re-auth for field workers
- Clear role hierarchy display
- Real-time audit log updates

## Backend Requirements
**API Endpoints**:
- `POST /api/auth/login` - Standard and PIN login
- `GET /api/admin/users` - User listing
- `POST /api/admin/users` - Create user
- `PUT /api/admin/users/:id` - Update user/roles
- `GET /api/admin/audit` - Audit log with filters
- `GET /api/admin/organization` - Org settings

**Database Models**:
- User (id, email, name, pin_hash, role, organization_id)
- Organization (id, name, settings, created_at)
- AuditLog (id, user_id, action, resource, changes, timestamp)
- Role (id, name, permissions, organization_id)

**Authentication**:
- NextAuth with PIN support
- Role-based middleware
- Audit logging on all mutations

**Data Validation**:
- Strong PIN requirements (6-8 digits)
- Email validation
- Role permission validation

---

# Feature 2: Project Management Console
**Purpose**: PMs plan, control, and monitor projects with work orders, scheduling, and real-time crew communication
**User Flow**: PM logs in → creates project → assigns work orders → schedules crews → monitors progress → handles approvals

## Frontend Requirements
**UI Components**:
- Project dashboard with status cards
- Gantt chart scheduler with drag-drop
- Work order creation wizard
- Piece mark tracking table with QR generation
- Real-time chat interface
- Budget vs actual progress charts

**Pages/Layout**:
- `/projects` - Project list and creation
- `/projects/:id` - Project detail with tabs
- `/projects/:id/schedule` - Interactive scheduler
- `/projects/:id/work-orders` - WO management
- `/projects/:id/piece-marks` - Tracking dashboard
- `/projects/:id/communications` - Project chat

**State Management**:
- Project list and active project
- Work order filters and selections
- Schedule view state (week/month)
- Unread message counts

**User Experience**:
- Drag-drop scheduling
- Real-time status updates
- Quick work order creation
- Mobile-responsive for site visits

## Backend Requirements
**API Endpoints**:
- `GET /api/projects` - Project listing
- `POST /api/projects` - Create project
- `GET /api/projects/:id/work-orders` - WO listing
- `POST /api/projects/:id/work-orders` - Create WO
- `PUT /api/work-orders/:id` - Update WO status
- `GET /api/projects/:id/piece-marks` - Piece mark tracking
- `POST /api/projects/:id/chat` - Send message
- `GET /api/projects/:id/schedule` - Schedule data

**Database Models**:
- Project (id, name, status, pm_id, client_id, budget, timeline, organization_id)
- WorkOrder (id, project_id, type, priority, status, assigned_to, checklist, attachments)
- PieceMark (id, project_id, mark, spec, weight, location, status, qr_code)
- Schedule (id, project_id, work_order_id, assigned_to, start_date, end_date)
- Message (id, project_id, user_id, content, attachments, timestamp)
- Client (id, name, email, organization_id)

**Authentication**:
- PM role required for project creation
- Project-based access control
- Client read-only access

**Data Validation**:
- Budget/timeline constraints
- Work order assignment validation
- Schedule conflict detection

---

# Feature 3: Field Operations App
**Purpose**: Shop/Field workers execute work orders offline-first with piece mark scanning and progress tracking
**User Flow**: Worker logs in → sees daily assignments → scans piece marks → logs progress/photos → submits completion offline

## Frontend Requirements
**UI Components**:
- "My Day" task dashboard
- QR/barcode scanner interface
- Photo capture with annotations
- Offline sync status indicator
- Time tracking widget
- Checklist completion forms

**Pages/Layout**:
- `/field/dashboard` - Daily assignments
- `/field/work-order/:id` - WO detail and checklist
- `/field/scanner` - QR/barcode scanning
- `/field/photos` - Photo management
- `/field/sync` - Offline sync status
- PWA support for mobile installation

**State Management**:
- Offline queue for all operations
- Sync status and conflicts
- Active work order progress
- Photo upload queue

**User Experience**:
- Offline-first design
- Quick barcode scanning
- Visual sync health indicators
- Touch-friendly mobile interface

## Backend Requirements
**API Endpoints**:
- `GET /api/field/assignments` - Worker's assignments
- `POST /api/field/work-orders/:id/progress` - Log progress
- `POST /api/field/piece-marks/:id/scan` - Record scan
- `POST /api/field/photos` - Upload photos
- `GET /api/field/sync` - Sync status
- `POST /api/field/sync/batch` - Batch sync operations

**Database Models**:
- WorkProgress (id, work_order_id, user_id, step, completed, timestamp, photos)
- PieceMarkScan (id, piece_mark_id, user_id, timestamp, location)
- Photo (id, work_order_id, piece_mark_id, user_id, url, notes, timestamp)
- SyncQueue (id, user_id, operation, data, status, timestamp)

**Authentication**:
- Worker role access
- Assignment-based data filtering
- Offline token validation

**Data Validation**:
- Photo size/format validation
- GPS coordinate validation
- Scan data integrity checks

**External APIs**:
- Cloud storage for photos (AWS S3/Cloudinary)
- Push notifications for urgent updates

## Integration & Flow
**Data Flow**:
PM creates work orders → assigned to workers → workers sync → scan/progress → real-time updates to PM dashboard

**Real-time Updates**:
WebSocket connections for PM dashboard updates, push notifications for priority changes

**Error Handling**:
Offline queue with conflict resolution, graceful degradation when offline, clear error messages

**Success Indicators**:
Work orders completed on time, real-time visibility of crew progress, zero data loss in offline scenarios