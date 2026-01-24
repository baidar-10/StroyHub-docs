# Worker Application System

## Overview
The booking system has been updated from first-come-first-served to an application-based model where workers submit applications and customers choose the best candidate.

## How It Works

### For Workers

1. **Browse Available Jobs**: Go to "Available Jobs" page (`/available-jobs`)
2. **Apply for Jobs**: Click "Apply for Job" button on any job listing
3. **Redirected to Application Page**: Navigate to `/apply/:bookingId` with full page layout
4. **Submit Application**: Fill in:
   - Message to customer (required) - explain why you're a good fit
   - Proposed price (optional) - suggest your price in KZT
5. **Return to Job List**: After submission, redirected back to `/available-jobs`
6. **Track Applications**: Jobs you've applied to show "Applied ✓" badge
7. **Wait for Response**: Customer will review and either accept or reject your application

### For Customers

1. **Create Open Job**: Post a job without selecting a specific worker
2. **Receive Applications**: Workers can submit applications with messages and proposed prices
3. **Review Applications**: 
   - Navigate to "Job Applications" from profile menu
   - Or go to `/applications` page
   - See all your open jobs with application counts
4. **Accept/Reject**: 
   - Review worker profiles, ratings, messages, and proposed prices
   - Accept one application (others will be automatically rejected)
   - Or reject individual applications
5. **Job Assignment**: When you accept an application, the worker is assigned to the job

## Technical Implementation

### Backend

**Database**: `booking_applications` table
- Columns: id, booking_id, worker_id, message, proposed_price, status, timestamps
- Unique constraint: (booking_id, worker_id) - one application per worker per job
- Indexes on: booking_id, worker_id, status

**API Endpoints**:
```
POST   /api/applications              - Create application
GET    /api/applications/my           - Get worker's applications  
GET    /api/applications/booking/:id  - Get applications for booking
PUT    /api/applications/:id/accept   - Accept application (customer)
PUT    /api/applications/:id/reject   - Reject application (customer)
```

**Business Logic** (application_service.go):
- `CreateApplication`: Validates open booking, checks for duplicate applications
- `AcceptApplication`: Assigns worker to booking, sets booking.isOpen=false, rejects other applications
- `RejectApplication`: Changes status to 'rejected'

### Frontend

**New Components**:

1. **ApplyJobPage.jsx**: Full-page worker application form (replaces modal)
   - Displays complete job details with customer info
   - Message textarea (required)
   - Proposed price input (optional)
   - Submit to `/api/applications`
   - Includes tips section for better applications
   - Full page layout with navbar and footer

2. **BookingApplications.jsx**: Customer's application list for a booking
   - Shows all applications with worker info, avatars, ratings
   - Displays message and proposed price
   - Accept/Reject buttons for pending applications
   - Status badges (Pending/Accepted/Rejected)

3. **BookingApplicationsPage.jsx**: Main page for customers
   - Left sidebar: List of open jobs with application counts
   - Right panel: Applications for selected job

**Updated Components**:

1. **AvailableJobsPage.jsx**: 
   - Changed "Accept Job" button to "Apply for Job"
   - Shows "Applied ✓" badge for jobs already applied to
   - Navigates to `/apply/:bookingId` instead of opening modal
   - Removed modal state management

2. **ProfileMenu.jsx**: 
   - Added "Job Applications" link for customers

**Routes**:
- `/applications` - Customer's application management page
- `/available-jobs` - Worker's job browsing page (updated)
- `/apply/:bookingId` - Worker's application submission page (new full-page form)

**Translations**: Added `application.*` keys to en.json, ru.json, kk.json

## Usage Examples

### Worker Flow
```
1. Login as worker
2. Click "Available Jobs" in navbar
3. Browse open jobs
4. Click "Apply for Job"
5. Redirected to full-page application form (/apply/:bookingId)
6. View complete job details (customer info, description, budget, location)
7. Write message: "I have 5 years experience in plumbing..."
8. Enter price: 15000 (optional)
9. Submit application
10. Redirected back to /available-jobs
11. See "Applied ✓" badge on that job
```

### Customer Flow
```
1. Login as customer
2. Create open job (without selecting worker)
3. Wait for applications
4. Click profile menu → "Job Applications"
5. See list of open jobs with application counts
6. Click on a job to view applications
7. Review worker profiles, messages, prices
8. Click "Accept" on preferred application
9. Other applications automatically rejected
10. Worker assigned to booking
```

## Key Features

- **Duplicate Prevention**: Workers can't apply twice to same job
- **Automatic Rejection**: Accepting one application rejects all others
- **Rich Profiles**: See worker avatars, ratings, specialties
- **Price Negotiation**: Workers can propose their own prices
- **Status Tracking**: Pending/Accepted/Rejected badges
- **Responsive Design**: Works on mobile and desktop
- **Multilingual**: Supports English, Russian, Kazakh

## Migration

Database migration file: `005_add_booking_applications.sql`

Run migration:
```bash
docker-compose exec postgres psql -U constructionuser -d construction_db < migrations/005_add_booking_applications.sql
```

Or via heredoc:
```bash
docker-compose exec -T postgres psql -U constructionuser -d construction_db <<EOF
-- SQL content here
EOF
```

## Testing Checklist

- [ ] Worker can view available jobs
- [ ] Worker can submit application with message
- [ ] Worker can submit application with price
- [ ] "Applied ✓" badge shows for applied jobs
- [ ] Can't apply twice to same job
- [ ] Customer sees open jobs with application counts
- [ ] Customer can view all applications for a job
- [ ] Worker avatar/rating displays correctly
- [ ] Customer can accept application
- [ ] Other applications auto-reject on accept
- [ ] Customer can reject individual application
- [ ] Accepted worker is assigned to booking
- [ ] Translations work in all 3 languages
