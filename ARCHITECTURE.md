# CivilHuB Design Review App: Technical Architecture

## 1. Overview

This document defines the technical architecture for a Bluebeam-like design review platform for construction, engineering, and architecture teams. The application allows users to upload and view design documents, apply markups, measure drawings, track issues, collaborate in real time, and manage revisions and approvals across projects.

The product is centered on a browser-based document review experience. It is intended to support internal teams, consultants, contractors, and stakeholders reviewing plans, drawings, and specification documents.

## 2. Product Goals

### Business goals
- Provide a collaborative design review application for project teams
- Enable markup, issue tracking, and review workflow management
- Support document version tracking and approval processes
- Make review workflows easy for project managers, reviewers, and clients
- Build a scalable system for multi-project, multi-user use

### Technical goals
- Separate concerns across client, API, services, storage, and real-time layers
- Support large document sets and concurrent users
- Keep markup and issue records auditable and reliable
- Support revision-aware document management
- Allow future growth into CAD/BIM and advanced workflows

## 3. Scope

### In scope for MVP
- User authentication and authorization
- Project and document management
- PDF and image upload
- Document viewer with zoom, pan, and page navigation
- Annotation and markup tools
- Measurement tools
- Issue creation and workflow management
- Comments and activity log
- Document revision tracking
- Approval workflow
- Real-time collaboration notifications
- Export/report generation

### Out of scope for MVP
- Native CAD model editing
- IFC/BIM exploitation
- 3D visual review
- AI-based issue detection
- offline-first support
- mobile-only field app

## 4. High-Level Architecture

The application uses a layered architecture:

1. Client Layer
2. API/Application Layer
3. Domain Services
4. Data Layer
5. File Storage Layer
6. File Processing / Rendering Layer
7. Real-Time Collaboration Layer
8. External Integration Layer

This structure supports modularity, experimentation, and future scale.

## 5. Architectural Principles

- Keep document files outside the relational database and use object storage
- Treat each document revision as immutable
- Make the viewer independent from the issue database
- Use role-based permissions for every major resource
- Push heavy processing into asynchronous workers
- Keep collaboration updates event-driven
- Maintain audit trails for approvals and significant actions

## 6. Users and Roles

### Primary roles
- Admin
- Project Manager
- Reviewer / Engineer
- Commenter
- Assignee
- Viewer
- External Stakeholder

### Role model
Roles are project-scoped and can be extended with custom permissions per document or workflow state.

Examples:
- Viewer: can view documents and comments
- Commenter: can add comments and markup
- Reviewer: can add issue and markup data
- Manager: can approve and assign issues
- Admin: can manage users, permission groups, and projects

## 7. Functional Modules

### 7.1 Authentication and identity
- User registration and login
- SSO support for enterprise users
- JWT token authentication
- Refresh token rotation
- Password reset and email verification

### 7.2 Project management
- Create project
- Invite users
- Manage teams and disciplines
- Set project settings and review workflow
- Organize documents by folder or discipline

### 7.3 Document management
- Upload files
- Store document metadata
- Track revisions
- Link pages, markups, issues, and approvals
- Maintain audit trail of document actions

### 7.4 Viewer layer
- Render page thumbnails
- Display pages in a zoomable canvas or viewer
- Navigate pages and document revisions
- Layer markup overlays on top of the base document
- Trigger measurement and markup tools

### 7.5 Markup engine
- Comments
- Sticky notes
- Highlights
- Arrows
- Callouts
- Rectangles
- Clouds
- Freehand drawing
- Stamps and symbols
- Layering and color management

### 7.6 Measurement engine
- Distance
- Area
- Perimeter
- Radius
- Angle
- Scale-aware dimensioning

### 7.7 Issue management
- Create issue from markup or document location
- Assign issue to user or team
- Status transitions
- Priority and due date tracking
- Attach files and notes
- Link issue to document page and revision

### 7.8 Collaboration
- Real-time comments
- Presence indicators
- Activity feed
- Notifications
- Multi-user review sessions

### 7.9 Approval workflow
- Draft review
- Under review
- Approved
- Rejected
- Obsolete
- Finalized

### 7.10 Reporting and export
- Markup summary report
- Issue status dashboard
- Review package export
- Document revision history
- PDF export with object overlays

## 8. System Components

### 8.1 Frontend application
The frontend is a browser-based application for project viewing and review.

Recommended stack:
- React
- TypeScript
- Vite or Next.js
- Zustand or Redux Toolkit
- React Query / TanStack Query
- Tailwind CSS or MUI
- WebSocket client

Responsibilities:
- Render project dashboards and document lists
- Display viewer and page navigation
- Manage review tools and markup interaction
- Display issue panels and activity feeds
- Communicate with the backend API
- Handle auth state and permissions

### 8.2 Backend API
The backend provides all business logic and persistence orchestration.

Recommended stack:
- Node.js with NestJS
- Or ASP.NET Core
- REST API with JSON payloads
- JWT auth
- OpenAPI/Swagger documentation

Responsibilities:
- validate requests
- authorize actions
- manage project/document objects
- control issue lifecycle and workflows
- handle markup records and permissions
- publish real-time events

### 8.3 Domain services
Examples of domain services:
- AuthService
- ProjectService
- DocumentService
- RevisionService
- MarkupService
- MeasurementService
- IssueService
- NotificationService
- ApprovalService
- AuditService
- PermissionService

### 8.4 Database layer
Primary relational database:
- PostgreSQL

Purpose:
- manage structured application entities
- enforce relational integrity
- support transactional updates for issues and revisions
- serve reporting and dashboard queries

### 8.5 Object storage layer
File storage solution:
- AWS S3
- Azure Blob Storage
- Google Cloud Storage

Stored objects:
- original uploaded files
- generated previews
- thumbnails
- exported review packages
- issue attachments

### 8.6 File processing worker
This layer handles heavy document processing asynchronously.

Responsibilities:
- validate uploads
- detect file type and size
- generate PDF page thumbnails
- create page preview images
- extract text metadata where needed
- build review export packages
- process revision comparisons

Suggested stack:
- Node.js worker or Python worker
- image processing tools such as Sharp
- PDF rendering tools such as Ghostscript, Poppler, or a PDF library

### 8.7 Real-time collaboration layer
Real-time communication is critical for active review sessions.

Responsibilities:
- broadcast new comments
- sync issue state updates
- push live collaborator presence
- propagate activity feed changes
- support collaborative annotation notifications

Recommended stack:
- WebSockets
- Socket.IO or custom gateway
- Redis pub/sub for scaled event distribution

## 9. Detailed Functional Architecture

### 9.1 Authentication and access control
Authentication flow:
- user signs in
- backend validates credentials or external identity
- backend issues access token and refresh token
- frontend stores token securely
- API validates permissions on every request

Authorization model:
- resource-scoped permissions
- role-based access control
- project membership enforcement
- optional attribute-based checks for document visibility

### 9.2 Project model
Project fields:
- id
- name
- organizationId
- ownerUserId
- status
- createdAt
- updatedAt
- configuration

Project relationships:
- project has many users
- project has many documents
- project has many issues
- project has many approval workflows

### 9.3 Document model
Document fields:
- id
- projectId
- title
- originalFileName
- mimeType
- storageKey
- uploadedByUserId
- uploadedAt
- discipline
- status
- tags

Document revision model:
- id
- documentId
- revisionNumber
- versionLabel
- storageKey
- checksum
- uploadedByUserId
- uploadedAt
- approvalState
- notes

Document page model:
- id
- revisionId
- pageNumber
- imageUrl
- thumbnailUrl
- width
- height
- createdAt

### 9.4 Markup model
Markup fields:
- id
- projectId
- documentId
- revisionId
- pageId
- authorUserId
- markupType
- color
- layer
- visibility
- createdAt
- updatedAt
- geometry
- status
- issueId

Markup type examples:
- note
- highlight
- rectangle
- cloud
- arrow
- line
- polygon
- freehand
- dimension
- stamp

Geometry storage options:
- JSON payload for points and bounds
- normalized coordinates in dedicated columns for queryable data
- stored as JSON for flexibility in MVP

### 9.5 Issue model
Issue fields:
- id
- projectId
- documentId
- revisionId
- pageId
- authorUserId
- assigneeUserId
- title
- description
- status
- severity
- priority
- dueDate
- createdAt
- updatedAt
- resolvedAt
- markupId

Issue lifecycle:
- Open
- In Progress
- Pending Review
- Resolved
- Closed
- Reopened

### 9.6 Comment model
Comment fields:
- id
- issueId
- markupId
- parentCommentId
- authorUserId
- content
- createdAt
- editedAt
- isSystemEvent

### 9.7 Measurement model
Measurement fields:
- id
- markupId
- measurementType
- value
- unit
- startPoint
- endPoint
- scaleFactor
- createdByUserId
- createdAt

### 9.8 Approval model
Approval model fields:
- id
- projectId
- documentId
- revisionId
- requestedByUserId
- approvedByUserId
- status
- requestedAt
- respondedAt
- decisionNotes

## 10. Viewer Architecture

The viewer is central to the product. It must support a high-quality review experience with overlay controls and fast rendering.

### Recommended viewer structure
- base document rendering layer
- overlay annotation layer
- interaction layer
- issue layer
- measurement layer
- navigation layer

### Viewer responsibilities
- page zoom and pan
- page selection and thumbnail display
- markup selection and editing
- event handling for tool clicks and mouse movement
- measurement capture
- issue highlighting and linking

### Rendering options
For PDFs and image-based drawings:
- render page previews into images and overlay annotations on top
- keep annotation coordinates tied to document page coordinates

For a more advanced future implementation:
- use PDF library rendering with a custom overlay layer
- optionally support CAD-like viewport transformations

## 11. API Design

### Core resource groups
- /auth
- /users
- /projects
- /projects/:id/members
- /projects/:id/documents
- /documents/:id/revisions
- /revisions/:id/pages
- /markups
- /issues
- /comments
- /notifications
- /approvals
- /reports

### API conventions
- RESTful resource naming
- JSON response schema
- pagination for large list endpoints
- consistent error codes and payloads
- idempotent operations when possible
- versioned API if needed later

### Example endpoints
- POST /api/auth/login
- GET /api/projects
- POST /api/projects
- GET /api/projects/:id/documents
- POST /api/documents/:id/revisions
- POST /api/revisions/:id/pages/:pageId/markups
- POST /api/issues
- GET /api/issues?status=open
- POST /api/comments
- POST /api/approvals/request
- GET /api/reports/project-summary

## 12. Real-Time Features and Event Architecture

Real-time activity is useful for collaborative design reviews.

### Real-time features
- presence notifications
- live issue updates
- comment synchronization
- document status updates
- activity feed refresh

### Event-driven pattern
- backend emits domain events
- Redis pub/sub distributes events
- WebSocket gateway sends updates to connected clients

### Example events
- DocumentUploaded
- RevisionCreated
- MarkupCreated
- MarkupUpdated
- IssueAssigned
- IssueStatusChanged
- CommentAdded
- ApprovalRequested
- ApprovalCompleted

## 13. Security Architecture

### Authentication security
- secure password hashing
- MFA support for enterprise users
- OAuth2/OIDC support for SSO integration
- short-lived access tokens
- secure refresh token handling

### Authorization security
- role-based access checks
- project membership verification
- document visibility checks
- API-level permission validation
- audit logging for privileged actions

### Data security
- TLS for all network requests
- encryption at rest for sensitive data
- secure object storage access policies
- sanitization for user-supplied markup and issue text
- file scanning for malicious uploads

## 14. Scalability Strategy

### Horizontal scaling
- deploy stateless API instances behind a load balancer
- share DB and Redis services across app instances
- keep workers independent from the API
- store large document artifacts externally

### Performance strategies
- lazy load page thumbnails
- render page images once and cache them
- paginate large issue and comment queries
- restrict viewer operations to active pages
- use async job workers for heavy processing
- index commonly queried fields such as projectId, documentId, revisionId, pageId, issueId

### Enterprise scaling plan
- multi-tenant architecture support
- organization-level access boundaries
- per-project permission enforcement
- isolated storage policies and retention
- larger analytics and reporting infrastructure as needed

## 15. Deployment Architecture

### Recommended deployment model
- frontend served from static hosting or CDN
- API deployed in a containerized environment
- PostgreSQL as managed database
- Redis for cache and pub/sub
- AWS S3 / Azure Blob / GCS for file storage
- worker services for document processing and exports

### Containerization
- Docker containers for API, worker, and frontend build/runtime
- Kubernetes or ECS for orchestration in production

### CI/CD
- GitHub Actions or equivalent
- lint/test/build pipeline on every PR
- staging environment for QA
- deployment tagging and rollback support

## 16. Database Design

### Relational tables
Core tables include:
- users
- organizations
- projects
- project_members
- roles
- permissions
- documents
- document_revisions
- document_pages
- markups
- comments
- issues
- issue_status_history
- notifications
- approvals
- activity_log

### Design principles
- use foreign keys to enforce correctness
- normalize frequently queried metadata
- keep files outside the database
- preserve old document revisions rather than overwriting them
- include timestamps on all mutable records

### Query optimization notes
- index by projectId, documentId, revisionId, pageId, issueId, assigneeUserId
- use pagination where datasets may grow large
- separate reporting queries from transactional workloads

## 17. File Processing Pipeline

### Document upload pipeline
1. User uploads a file
2. API validates size and file type
3. File is stored in object storage
4. Metadata is inserted into database
5. Worker service creates page previews and thumbnails
6. Viewer is ready to render the uploaded document

### Processing tasks
- file hash generation
- checksum verification
- page extraction
- preview image generation
- OCR/text extraction if required
- revision comparison generation

## 18. Reporting and Analytics

### Useful reports
- project issue summary
- unresolved issues by discipline
- document approval progress
- design review activity by date
- number of markups per revision
- average review turnaround time

### Data sources
- issue records
- markup records
- document revisions
- comments and approvals
- user activity logs

## 19. Observability and Operations

### Logging
- API request logs
- worker job logs
- failed upload and processing errors
- permission denial events
- audit log activity

### Monitoring
- API latency
- worker runtime
- queue depth
- storage utilization
- failed authentication attempts
- system error rates

### Metrics to track
- number of active projects
- number of uploaded documents
- number of comments per issue
- average document review time
- approval rate by workflow

## 20. Security and Compliance Considerations

- role isolation by project and document
- least-privilege access design
- encryption in transit and at rest
- access logs for privileged events
- retention policies for old revisions
- explicit permission management for external stakeholders

## 21. Testing Strategy

### Unit testing
- service logic
- permission checks
- issue transitions
- document revision logic
- notification creation

### Integration tests
- API routes and database interactions
- issue creation and assignment flows
- revision upload and processing
- approval workflow

### End-to-end tests
- login and project creation
- document upload and viewer access
- markup creation and issue assignment
- comment updates and notification delivery
- final approval workflow

### Performance testing
- large document loading
- concurrent comment activity
- high issue volume
- export service behavior

## 22. MVP Implementation Priorities

### Phase 1: Foundation
- authentication
- project creation
- document upload
- basic PDF rendering
- page navigation

### Phase 2: Review features
- markup tools
- comments
- issue creation
- issue status updates

### Phase 3: Collaboration
- real-time notifications
- activity feed
- user presence
- comments synchronization

### Phase 4: Revision & approvals
- revision tracking
- approval workflow
- reporting exports

### Phase 5: Hardening
- performance tuning
- permission tests
- larger-scale file processing
- production operations

## 23. Risks and Mitigations

### Risk: large document rendering performance
Mitigation:
- pre-render page previews
- cache generated images
- lazy load pages

### Risk: markup synchronization conflicts
Mitigation:
- optimistic locking or version stamps
- event-based update pipeline
- merge logic for conflicting changes

### Risk: permission bugs
Mitigation:
- centralize permission logic
- test every role-resource combination
- audit permission checks

### Risk: file processing bottlenecks
Mitigation:
- asynchronous worker queue
- scale workers horizontally
- isolate heavy processing from API requests

## 24. Recommended Technology Stack Summary

### Frontend
- React
- TypeScript
- Vite / Next.js
- Tailwind CSS or MUI
- Zustand / Redux Toolkit
- React Query

### Backend
- Node.js + NestJS
- PostgreSQL
- Redis
- WebSockets
- JWT authentication

### Storage and processing
- S3 or Azure Blob Storage
- PDF/image processing library
- Async worker service

### DevOps
- Docker
- GitHub Actions
- Kubernetes or ECS for production

## 25. Recommended Architecture Summary

The best architecture for this app is a modular, event-driven document review platform built around:
- a browser-based viewer
- relational data for metadata and workflow
- object storage for large files
- asynchronous processing for previews and exports
- a real-time collaboration layer for live review activity
- strict permissions and issuer event tracking

This architecture supports a strong MVP and provides a realistic path toward a larger enterprise document review product.

## 26. Final Recommendation

Start with a focused MVP centered on:
- project and user management
- document upload and preview
- markup tools
- issue tracking
- comments
- revision tracking
- real-time notifications

Once the review workflow is stable, expand into more advanced features such as CAD integration, BIM support, and enterprise workflow automation.

This architecture is intentionally designed to keep the product scalable while remaining manageable for an initial build.

---

This document represents the technical architecture foundation for the Bluebeam-like design review app and should be revisited as the product evolves.
