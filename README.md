# E-Admit Portal

## Overview
E-Admit Portal is a multi-tenant web application that helps educational institutions and examination agencies create, manage, and distribute digital admit cards for exams and events. The platform provides an authenticated agency dashboard for exam and student management, and a public student portal for admit card access, verification, and printing.

## Architecture
### Technology Stack
- **Frontend:** React 18 with functional components and hooks
- **Styling:** Tailwind CSS with shadcn/ui (Radix UI primitives)
- **State Management:** TanStack React Query for server state
- **Routing:** React Router DOM
- **Animations:** Framer Motion
- **Icons:** Lucide React
- **Date Handling:** date-fns
- **Backend (platform-agnostic):** REST/GraphQL API with authentication, file storage, and email OTP support

### High-Level Modules
- **Agency Portal:** Create and manage exams, configure templates, manage students, and track analytics.
- **Student Portal:** Search for exams by code, verify identity (optional), and download/print admit cards.
- **Branding & Templates:** Agency-specific branding applied across all admit card templates.
- **Analytics:** Track preview/download/print actions for auditing and reporting.

## Data Model
### Exam
Core exam/event information and configuration.

**Fields:**
- `id` (UUID)
- `title`
- `exam_code` (unique, shareable)
- `description`
- `exam_mode` (`online` | `offline`)
- `exam_date`
- `reporting_time`
- `venue` (offline) / `exam_url` (online)
- `instructions`
- `status` (`draft` | `active` | `archived`)
- `template` (`classic` | `modern` | `minimal` | `minimalist`)
- `custom_fields` (array of `{ label, field_key, required }`)
- `otp_verification_enabled` (boolean)
- `agency_id`
- `created_at`, `updated_at`

### Student
Student/candidate records linked to an exam.

**Fields:**
- `id` (UUID)
- `full_name`
- `roll_number`
- `date_of_birth`
- `email`
- `exam_code`
- `exam_id`
- `seat_number`
- `additional_notes`
- `custom_data` (object keyed by `custom_fields.field_key`)
- `agency_id`
- `created_at`, `updated_at`

### Agency Settings
Agency branding and admit card customization.

**Fields:**
- `id` (UUID)
- `organization_name`
- `logo_url`
- `e_signature_url`
- `primary_color`
- `secondary_color`
- `font_family` (`inter` | `roboto` | `serif` | `monospace`)
- `contact_email`
- `contact_phone`
- `address`
- `signature_title`
- `created_by`
- `created_at`, `updated_at`

### Admit Card Download
Tracks every admit card access event.

**Fields:**
- `id` (UUID)
- `student_id`
- `exam_id`
- `exam_code`
- `roll_number`
- `download_type` (`preview` | `download` | `print`)
- `created_at`

### User
Authenticated agency users.

**Fields:**
- `id` (UUID)
- `email`
- `full_name`
- `role` (`admin` | `user`)
- `created_at`, `updated_at`

## Application Structure
### Pages
1. **Home** (`pages/Home.jsx`)
   - Public landing page with exam code search.
   - Highlights platform features and agency login CTA.

2. **Agency Dashboard** (`pages/AgencyDashboard.jsx`)
   - Protected page listing all agency exams.
   - Shows totals (students, active/draft exams) and download stats.
   - Actions: edit, archive, delete, copy exam codes.

3. **Create Exam** (`pages/CreateExam.jsx`)
   - Exam creation and editing.
   - Sections: basic info, exam mode, schedule, custom fields, template selection, settings.
   - Live template preview modal.

4. **Manage Students** (`pages/ManageStudents.jsx`)
   - Student management per exam.
   - Manual add/edit/delete, CSV import/export, search/filter.
   - Dynamic custom fields based on exam configuration.

5. **Student List** (`pages/StudentList.jsx`)
   - Public exam page accessed via exam code.
   - Searchable student list with admit card preview/download.
   - OTP verification if enabled.

6. **Agency Settings** (`pages/AgencySettings.jsx`)
   - Branding: logo, signature, colors, fonts, contact info.
   - Applied to all templates.

### Components
#### Admit Card Templates (`components/admit-card/`)
- **ClassicTemplate**: Formal layout with table sections and double border.
- **ModernTemplate**: Gradient header, iconography, card sections.
- **MinimalTemplate**: Clean two-column layout with simple typography.
- **MinimalistTemplate**: Maximum whitespace and minimalist typography.

All templates:
- Use A4 layout (210mm × 297mm).
- Include photo placeholder, exam instructions, candidate and authority signatures.
- Render custom fields dynamically.
- Apply agency branding (logo, colors, fonts, signature).

#### AdmitCardPreview (`components/admit-card/AdmitCardPreview.jsx`)
- Modal for previewing admit cards.
- Supports print and PDF download.
- Logs preview/download/print actions.

#### OTPVerification (`components/admit-card/OTPVerification.jsx`)
- Email OTP verification before admit card access.
- Used when `otp_verification_enabled` is true for an exam.

## Key Features
### Multi-Tenant Agency System
- Each agency operates independently with data isolation by `agency_id`.
- Agency-specific branding across templates and public pages.

### Flexible Exam Configuration
- Supports online and offline exams.
- Custom fields system for extended student data.
- Exam lifecycle: draft → active → archived.

### Custom Fields System
- Agencies define custom fields per exam.
- Student records store corresponding values in `custom_data`.
- Templates render these fields dynamically.

### Bulk Student Management
- CSV import with preview and validation.
- CSV export including custom fields.
- Batch operations with progress feedback.

### Download Tracking & Analytics
- Every preview/download/print action creates an analytics record.
- Dashboard aggregates unique downloads and per-exam statistics.

### Template System
- Four professional templates with live preview.
- Brand colors and fonts are applied globally.

### Security & Access Control
- Auth-protected agency pages.
- Public student access via exam code.
- Optional OTP verification for download access.

## Data Flow
### Student Admit Card Download
1. Student enters exam code on Home page.
2. StudentList fetches exam and student records.
3. Student searches roll number.
4. "View Admit Card" triggers OTP verification (if enabled).
5. AdmitCardPreview renders selected template.
6. Preview/Download/Print actions are logged.

### Agency Exam Creation
1. Agency logs in → dashboard.
2. Create exam with schedule, mode, instructions.
3. Add custom fields and select a template.
4. Save exam and manage students.
5. Activate exam and distribute exam code.

## Custom Fields Implementation
- **Configuration:**
  ```json
  [{ "label": "Father's Name", "field_key": "father_name", "required": true }]
  ```
- **Student storage:**
  ```json
  { "father_name": "John Doe" }
  ```
- **Rendering:**
  Templates iterate `exam.custom_fields` and display matching values from `student.custom_data`.

## API Integration (Platform-Agnostic)
### Typical REST Endpoints
**Auth**
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/logout`
- `GET /auth/me`

**Exams**
- `GET /exams`
- `POST /exams`
- `GET /exams/:id`
- `PUT /exams/:id`
- `DELETE /exams/:id`
- `GET /exams/code/:code`

**Students**
- `GET /students?exam_id=:id`
- `POST /students`
- `POST /students/bulk`
- `PUT /students/:id`
- `DELETE /students/:id`
- `GET /students/export?exam_id=:id`

**Agency Settings**
- `GET /agency-settings`
- `PUT /agency-settings`

**Analytics**
- `POST /analytics/download`
- `GET /analytics/stats?exam_id=:id`

**OTP**
- `POST /otp/send`
- `POST /otp/verify`

## Styling & UX
- Responsive, mobile-first layouts.
- Accessible, keyboard-friendly components.
- Loading states and clear success/error feedback.
- Smooth transitions via Framer Motion.

## Scalability Considerations
- Index frequently queried columns (e.g., `exam_code`, `agency_id`).
- Paginate large student lists.
- Batch operations for CSV imports.
- Cache frequently accessed exam details.

## Future Enhancements
- Batch printing and multi-student PDF generation.
- QR code generation and verification.
- SMS notifications and multi-language support.
- Advanced analytics and template builder.
