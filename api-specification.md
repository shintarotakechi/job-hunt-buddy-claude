# JobHunt AI - API Specification

## Overview

This document defines the RESTful API endpoints, request/response schemas, and WebSocket specifications for JobHunt AI. All APIs follow REST principles with JSON payloads and use JWT for authentication.

## Base Configuration

### Base URLs

```
Production: https://api.jobhuntai.com/v1
Staging: https://staging-api.jobhuntai.com/v1
Development: http://localhost:5000/api/v1
```

### Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```http
Authorization: Bearer <jwt_token>
```

### Common Headers

```http
Content-Type: application/json
Accept: application/json
X-Request-ID: <unique-request-id>
X-Client-Version: 1.0.0
```

### Rate Limiting

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
```

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

## Authentication Endpoints

### POST /auth/google

Authenticate with Google OAuth

**Request:**
```json
{
  "idToken": "google_id_token_here"
}
```

**Response:**
```json
{
  "accessToken": "jwt_access_token",
  "refreshToken": "refresh_token",
  "expiresIn": 3600,
  "user": {
    "id": "user_123",
    "email": "user@example.com",
    "name": "John Doe",
    "picture": "https://example.com/photo.jpg",
    "isNewUser": false
  }
}
```

### POST /auth/refresh

Refresh access token

**Request:**
```json
{
  "refreshToken": "refresh_token_here"
}
```

**Response:**
```json
{
  "accessToken": "new_jwt_access_token",
  "expiresIn": 3600
}
```

### POST /auth/logout

Logout and invalidate tokens

**Request:**
```json
{
  "refreshToken": "refresh_token_here"
}
```

**Response:**
```json
{
  "message": "Successfully logged out"
}
```

## User Management Endpoints

### GET /users/profile

Get current user profile

**Response:**
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "John Doe",
  "picture": "https://example.com/photo.jpg",
  "createdAt": "2024-01-01T00:00:00Z",
  "profile": {
    "headline": "Software Engineer",
    "summary": "Experienced developer...",
    "location": "San Francisco, CA",
    "skills": ["JavaScript", "React", "Node.js"],
    "experience": 5,
    "education": [{
      "degree": "BS Computer Science",
      "school": "University Name",
      "year": 2018
    }],
    "resumeUrl": "https://storage.example.com/resume.pdf",
    "linkedInUrl": "https://linkedin.com/in/johndoe"
  },
  "preferences": {
    "jobTypes": ["full-time", "remote"],
    "locations": ["San Francisco", "New York"],
    "radius": 50,
    "salaryMin": 100000,
    "salaryMax": 150000,
    "industries": ["Technology", "Finance"],
    "companySizes": ["startup", "medium"],
    "remoteOnly": false
  },
  "settings": {
    "emailNotifications": true,
    "applicationFrequency": "aggressive",
    "autoApply": true,
    "followUpDays": 7
  }
}
```

### PUT /users/profile

Update user profile

**Request:**
```json
{
  "name": "John Doe",
  "profile": {
    "headline": "Senior Software Engineer",
    "summary": "Updated summary...",
    "location": "San Francisco, CA",
    "skills": ["JavaScript", "React", "Node.js", "Python"]
  }
}
```

**Response:**
```json
{
  "message": "Profile updated successfully",
  "profile": { /* updated profile object */ }
}
```

### POST /users/resume

Upload resume

**Request:**
```
Content-Type: multipart/form-data

file: resume.pdf
```

**Response:**
```json
{
  "message": "Resume uploaded successfully",
  "resumeUrl": "https://storage.example.com/resumes/user_123/resume.pdf",
  "parsedData": {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "skills": ["JavaScript", "React", "Node.js"],
    "experience": [/* parsed experience */],
    "education": [/* parsed education */]
  }
}
```

### PUT /users/preferences

Update user preferences

**Request:**
```json
{
  "jobTypes": ["full-time", "contract"],
  "locations": ["San Francisco", "Austin"],
  "radius": 25,
  "salaryMin": 120000,
  "salaryMax": 180000,
  "industries": ["Technology"],
  "remoteOnly": true
}
```

**Response:**
```json
{
  "message": "Preferences updated successfully",
  "preferences": { /* updated preferences */ }
}
```

### DELETE /users/account

Delete user account

**Request:**
```json
{
  "confirmation": "DELETE_MY_ACCOUNT",
  "reason": "No longer needed"
}
```

**Response:**
```json
{
  "message": "Account scheduled for deletion",
  "deletionDate": "2024-01-30T00:00:00Z"
}
```

## Job Discovery Endpoints

### GET /jobs/discover

Discover jobs matching user preferences

**Query Parameters:**
- `page` (int): Page number (default: 1)
- `limit` (int): Items per page (default: 20, max: 100)
- `sortBy` (string): Sort field (relevance, date, salary)
- `order` (string): Sort order (asc, desc)

**Response:**
```json
{
  "jobs": [
    {
      "id": "job_456",
      "title": "Senior Software Engineer",
      "company": "Tech Corp",
      "location": "San Francisco, CA",
      "remote": true,
      "salary": {
        "min": 130000,
        "max": 180000,
        "currency": "USD"
      },
      "description": "Job description...",
      "requirements": ["5+ years experience", "React expertise"],
      "benefits": ["Health insurance", "401k"],
      "postedDate": "2024-01-10T00:00:00Z",
      "source": "linkedin",
      "url": "https://linkedin.com/jobs/123",
      "matchScore": 85,
      "matchReasons": [
        "Skills match: React, Node.js",
        "Location preference match",
        "Salary range match"
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "pages": 8
  }
}
```

### GET /jobs/{id}

Get job details

**Response:**
```json
{
  "id": "job_456",
  "title": "Senior Software Engineer",
  "company": {
    "name": "Tech Corp",
    "logo": "https://example.com/logo.png",
    "size": "1000-5000",
    "industry": "Technology",
    "website": "https://techcorp.com"
  },
  "location": "San Francisco, CA",
  "remote": true,
  "salary": {
    "min": 130000,
    "max": 180000,
    "currency": "USD"
  },
  "description": "Full job description...",
  "requirements": [/* detailed requirements */],
  "benefits": [/* detailed benefits */],
  "postedDate": "2024-01-10T00:00:00Z",
  "applicationDeadline": "2024-02-10T00:00:00Z",
  "source": "linkedin",
  "url": "https://linkedin.com/jobs/123",
  "stakeholders": [
    {
      "name": "Jane Smith",
      "title": "Hiring Manager",
      "linkedIn": "https://linkedin.com/in/janesmith",
      "email": "jane.smith@techcorp.com"
    }
  ],
  "matchScore": 85,
  "applicationStatus": "not_applied"
}
```

### POST /jobs/search

Search jobs with filters

**Request:**
```json
{
  "query": "React developer",
  "filters": {
    "locations": ["San Francisco", "Remote"],
    "salaryMin": 100000,
    "companies": ["Google", "Meta"],
    "experienceLevel": ["senior", "lead"],
    "postedWithin": 7,
    "remote": true
  },
  "page": 1,
  "limit": 20
}
```

**Response:**
```json
{
  "jobs": [/* array of job objects */],
  "facets": {
    "companies": [
      {"name": "Google", "count": 15},
      {"name": "Meta", "count": 12}
    ],
    "locations": [
      {"name": "San Francisco", "count": 45},
      {"name": "Remote", "count": 78}
    ]
  },
  "pagination": { /* pagination object */ }
}
```

### POST /jobs/{id}/save

Save job for later

**Response:**
```json
{
  "message": "Job saved successfully",
  "savedAt": "2024-01-15T10:30:00Z"
}
```

### DELETE /jobs/{id}/save

Remove saved job

**Response:**
```json
{
  "message": "Job removed from saved list"
}
```

## Application Management Endpoints

### POST /applications

Create new application

**Request:**
```json
{
  "jobId": "job_456",
  "coverLetter": "Generated or custom cover letter...",
  "resume": "resume_version_id",
  "answers": [
    {
      "question": "Years of React experience?",
      "answer": "5 years"
    }
  ],
  "sendImmediately": false
}
```

**Response:**
```json
{
  "id": "app_789",
  "jobId": "job_456",
  "status": "draft",
  "createdAt": "2024-01-15T10:30:00Z",
  "scheduledFor": null
}
```

### GET /applications

Get user's applications

**Query Parameters:**
- `status` (string): Filter by status
- `page` (int): Page number
- `limit` (int): Items per page

**Response:**
```json
{
  "applications": [
    {
      "id": "app_789",
      "job": {
        "id": "job_456",
        "title": "Senior Software Engineer",
        "company": "Tech Corp"
      },
      "status": "applied",
      "appliedAt": "2024-01-15T10:30:00Z",
      "lastActivity": "2024-01-16T09:00:00Z",
      "activities": [
        {
          "type": "applied",
          "timestamp": "2024-01-15T10:30:00Z"
        },
        {
          "type": "viewed",
          "timestamp": "2024-01-16T09:00:00Z"
        }
      ],
      "responseReceived": false,
      "interviewScheduled": false
    }
  ],
  "statistics": {
    "total": 45,
    "applied": 40,
    "interviewing": 3,
    "rejected": 2,
    "offers": 0
  },
  "pagination": { /* pagination object */ }
}
```

### GET /applications/{id}

Get application details

**Response:**
```json
{
  "id": "app_789",
  "job": { /* full job object */ },
  "status": "applied",
  "coverLetter": "Full cover letter text...",
  "resumeUsed": {
    "id": "resume_123",
    "version": 3,
    "url": "https://storage.example.com/resume_v3.pdf"
  },
  "answers": [/* application answers */],
  "timeline": [
    {
      "event": "application_created",
      "timestamp": "2024-01-15T10:00:00Z"
    },
    {
      "event": "application_sent",
      "timestamp": "2024-01-15T10:30:00Z"
    },
    {
      "event": "email_opened",
      "timestamp": "2024-01-16T09:00:00Z"
    }
  ],
  "emails": [
    {
      "id": "email_111",
      "type": "application",
      "subject": "Application for Senior Software Engineer",
      "sentAt": "2024-01-15T10:30:00Z",
      "status": "delivered",
      "opened": true
    }
  ]
}
```

### PUT /applications/{id}/status

Update application status

**Request:**
```json
{
  "status": "interviewing",
  "note": "Phone screen scheduled for next week"
}
```

**Response:**
```json
{
  "message": "Status updated successfully",
  "application": { /* updated application */ }
}
```

### POST /applications/{id}/follow-up

Send follow-up email

**Request:**
```json
{
  "message": "Custom follow-up message...",
  "sendAt": "2024-01-22T10:00:00Z"
}
```

**Response:**
```json
{
  "message": "Follow-up scheduled",
  "scheduledFor": "2024-01-22T10:00:00Z"
}
```

## AI Generation Endpoints

### POST /ai/cover-letter

Generate cover letter

**Request:**
```json
{
  "jobId": "job_456",
  "tone": "professional",
  "length": "medium",
  "emphasize": ["leadership", "technical skills"],
  "additionalContext": "Mention my open source contributions"
}
```

**Response:**
```json
{
  "coverLetter": "Generated cover letter text...",
  "wordCount": 350,
  "readabilityScore": 72,
  "atsScore": 85,
  "suggestions": [
    "Consider adding specific metrics",
    "Mention the company's recent product launch"
  ],
  "alternativeVersions": [
    {
      "id": "version_1",
      "preview": "First 100 characters..."
    }
  ]
}
```

### POST /ai/resume-tailor

Tailor resume for job

**Request:**
```json
{
  "jobId": "job_456",
  "resumeId": "resume_123",
  "optimizeFor": "ats",
  "includeKeywords": true
}
```

**Response:**
```json
{
  "tailoredResume": {
    "id": "resume_456",
    "changes": [
      {
        "section": "skills",
        "action": "reordered",
        "reason": "Match job requirements priority"
      },
      {
        "section": "experience",
        "action": "emphasized",
        "details": "React projects highlighted"
      }
    ],
    "keywordsAdded": ["React", "Node.js", "AWS"],
    "atsScore": 92,
    "url": "https://storage.example.com/tailored_resume.pdf"
  }
}
```

### POST /ai/analyze-job

Analyze job fit

**Request:**
```json
{
  "jobId": "job_456"
}
```

**Response:**
```json
{
  "matchScore": 85,
  "strengths": [
    "Strong skill alignment",
    "Location preference match",
    "Industry experience"
  ],
  "gaps": [
    "Missing AWS certification",
    "Less experience with GraphQL"
  ],
  "recommendations": [
    "Emphasize React expertise",
    "Mention similar cloud experience",
    "Include team leadership examples"
  ],
  "competitionEstimate": "high",
  "salaryAnalysis": {
    "yourRange": {"min": 120000, "max": 150000},
    "jobRange": {"min": 130000, "max": 180000},
    "recommendation": "Ask for $145,000"
  }
}
```

## Email Management Endpoints

### GET /emails

Get email history

**Query Parameters:**
- `applicationId` (string): Filter by application
- `type` (string): Email type (application, follow-up, response)
- `status` (string): Email status

**Response:**
```json
{
  "emails": [
    {
      "id": "email_111",
      "applicationId": "app_789",
      "type": "application",
      "recipient": "hr@techcorp.com",
      "subject": "Application for Senior Software Engineer",
      "status": "delivered",
      "sentAt": "2024-01-15T10:30:00Z",
      "openedAt": "2024-01-16T09:00:00Z",
      "clickedLinks": ["resume_link"],
      "response": {
        "received": true,
        "timestamp": "2024-01-17T14:00:00Z",
        "classification": "interview_request"
      }
    }
  ],
  "pagination": { /* pagination object */ }
}
```

### POST /emails/send

Send email manually

**Request:**
```json
{
  "to": ["hr@techcorp.com"],
  "cc": ["recruiter@techcorp.com"],
  "subject": "Application for Senior Software Engineer",
  "body": "Email body HTML...",
  "attachments": ["resume_id"],
  "trackOpens": true,
  "trackClicks": true
}
```

**Response:**
```json
{
  "id": "email_222",
  "status": "queued",
  "scheduledFor": "2024-01-15T10:35:00Z"
}
```

### POST /emails/classify

Classify email response

**Request:**
```json
{
  "emailContent": "Thank you for your application. We would like to schedule an interview...",
  "subject": "Re: Application for Senior Software Engineer"
}
```

**Response:**
```json
{
  "classification": "interview_request",
  "confidence": 0.95,
  "extractedData": {
    "interviewType": "phone_screen",
    "suggestedDates": ["2024-01-22", "2024-01-23"],
    "duration": "30 minutes",
    "interviewer": "Jane Smith"
  },
  "suggestedAction": "schedule_interview",
  "responseTemplate": "Thank you for the opportunity..."
}
```

## Google Integration Endpoints

### POST /integrations/google/sheets

Setup Google Sheets integration

**Request:**
```json
{
  "spreadsheetId": "existing_sheet_id_or_null",
  "createNew": true,
  "sheetName": "Job Applications 2024"
}
```

**Response:**
```json
{
  "spreadsheetId": "1abc...xyz",
  "spreadsheetUrl": "https://docs.google.com/spreadsheets/d/1abc...xyz",
  "connected": true,
  "lastSync": null
}
```

### POST /integrations/google/sheets/sync

Sync data to Google Sheets

**Response:**
```json
{
  "message": "Sync completed successfully",
  "recordsSynced": 45,
  "lastSync": "2024-01-15T10:30:00Z",
  "nextSync": "2024-01-15T10:35:00Z"
}
```

### POST /integrations/google/calendar

Setup Google Calendar integration

**Request:**
```json
{
  "calendarId": "primary",
  "createInterviewCalendar": true,
  "enableReminders": true,
  "reminderMinutes": [15, 60]
}
```

**Response:**
```json
{
  "connected": true,
  "calendarId": "primary",
  "interviewCalendarId": "interviews_abc123",
  "settings": {
    "autoSchedule": true,
    "reminders": [15, 60],
    "timezone": "America/Los_Angeles"
  }
}
```

### POST /integrations/google/calendar/events

Create calendar event

**Request:**
```json
{
  "title": "Interview with Tech Corp",
  "description": "Phone screen for Senior Software Engineer position",
  "startTime": "2024-01-22T14:00:00Z",
  "endTime": "2024-01-22T14:30:00Z",
  "location": "Phone: 555-0123",
  "attendees": ["interviewer@techcorp.com"],
  "reminders": [15, 60]
}
```

**Response:**
```json
{
  "eventId": "event_abc123",
  "htmlLink": "https://calendar.google.com/event?eid=abc123",
  "created": true
}
```

## Analytics Endpoints

### GET /analytics/dashboard

Get dashboard statistics

**Response:**
```json
{
  "overview": {
    "totalApplications": 156,
    "activeApplications": 45,
    "interviews": 5,
    "offers": 1,
    "responseRate": 28.5,
    "averageResponseTime": "3.2 days"
  },
  "timeline": [
    {
      "date": "2024-01-15",
      "applications": 5,
      "responses": 2,
      "interviews": 1
    }
  ],
  "topCompanies": [
    {"name": "Google", "applications": 8, "responseRate": 37.5},
    {"name": "Meta", "applications": 6, "responseRate": 33.3}
  ],
  "skills": [
    {"name": "React", "demand": 78, "match": 95},
    {"name": "Python", "demand": 65, "match": 60}
  ]
}
```

### GET /analytics/reports

Generate detailed report

**Query Parameters:**
- `startDate` (date): Report start date
- `endDate` (date): Report end date
- `format` (string): Output format (json, csv, pdf)

**Response:**
```json
{
  "reportId": "report_123",
  "generatedAt": "2024-01-15T10:30:00Z",
  "downloadUrl": "https://storage.example.com/reports/report_123.pdf",
  "expiresAt": "2024-01-22T10:30:00Z",
  "summary": {
    "period": "2024-01-01 to 2024-01-15",
    "applications": 45,
    "interviews": 5,
    "successRate": 11.1
  }
}
```

## WebSocket Events

### Connection

```javascript
const ws = new WebSocket('wss://api.jobhuntai.com/ws');

ws.onopen = () => {
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'jwt_token_here'
  }));
};
```

### Event Types

#### job.discovered
```json
{
  "type": "job.discovered",
  "data": {
    "job": { /* job object */ },
    "matchScore": 85
  }
}
```

#### application.statusChanged
```json
{
  "type": "application.statusChanged",
  "data": {
    "applicationId": "app_789",
    "oldStatus": "applied",
    "newStatus": "interviewing",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

#### email.received
```json
{
  "type": "email.received",
  "data": {
    "emailId": "email_333",
    "applicationId": "app_789",
    "classification": "interview_request",
    "requiresAction": true
  }
}
```

#### interview.reminder
```json
{
  "type": "interview.reminder",
  "data": {
    "interviewId": "int_123",
    "company": "Tech Corp",
    "position": "Senior Software Engineer",
    "startsIn": "15 minutes",
    "joinUrl": "https://zoom.us/j/123456789"
  }
}
```

#### sync.completed
```json
{
  "type": "sync.completed",
  "data": {
    "service": "google_sheets",
    "recordsSynced": 45,
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

## Rate Limiting

### Limits by Tier

| Tier | Requests/Hour | Concurrent Jobs | AI Generations/Day |
|------|--------------|-----------------|-------------------|
| Free | 100 | 10 | 5 |
| Basic | 1,000 | 50 | 50 |
| Pro | 10,000 | 200 | 200 |
| Enterprise | Unlimited | Unlimited | Unlimited |

### Rate Limit Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1640995200
X-RateLimit-Retry-After: 3600
```

## Error Codes

| Code | Description | HTTP Status |
|------|-------------|------------|
| AUTH_REQUIRED | Authentication required | 401 |
| AUTH_INVALID | Invalid authentication token | 401 |
| AUTH_EXPIRED | Token expired | 401 |
| PERMISSION_DENIED | Insufficient permissions | 403 |
| NOT_FOUND | Resource not found | 404 |
| VALIDATION_ERROR | Request validation failed | 400 |
| RATE_LIMITED | Rate limit exceeded | 429 |
| INTERNAL_ERROR | Internal server error | 500 |
| SERVICE_UNAVAILABLE | Service temporarily unavailable | 503 |
| QUOTA_EXCEEDED | Usage quota exceeded | 402 |

## API Versioning

The API uses URL versioning. The current version is v1. When breaking changes are introduced, a new version will be created.

### Version Headers

```http
X-API-Version: 1.0
X-API-Deprecated: false
X-API-Sunset: 2025-01-01
```

## SDK Examples

### JavaScript/TypeScript

```typescript
import { JobHuntClient } from '@jobhuntai/sdk';

const client = new JobHuntClient({
  apiKey: 'your_api_key',
  version: 'v1'
});

// Discover jobs
const jobs = await client.jobs.discover({
  page: 1,
  limit: 20,
  sortBy: 'relevance'
});

// Create application
const application = await client.applications.create({
  jobId: 'job_456',
  coverLetter: 'Generated letter...',
  sendImmediately: true
});

// Subscribe to events
client.on('job.discovered', (job) => {
  console.log('New job found:', job);
});
```

### Python

```python
from jobhuntai import JobHuntClient

client = JobHuntClient(
    api_key='your_api_key',
    version='v1'
)

# Discover jobs
jobs = client.jobs.discover(
    page=1,
    limit=20,
    sort_by='relevance'
)

# Create application
application = client.applications.create(
    job_id='job_456',
    cover_letter='Generated letter...',
    send_immediately=True
)

# Subscribe to events
@client.on('job.discovered')
def on_job_discovered(job):
    print(f'New job found: {job}')
```

### C# / .NET

```csharp
using JobHuntAI.SDK;

var client = new JobHuntClient(new JobHuntOptions
{
    ApiKey = "your_api_key",
    Version = "v1"
});

// Discover jobs
var jobs = await client.Jobs.DiscoverAsync(new DiscoverRequest
{
    Page = 1,
    Limit = 20,
    SortBy = SortField.Relevance
});

// Create application
var application = await client.Applications.CreateAsync(new CreateApplicationRequest
{
    JobId = "job_456",
    CoverLetter = "Generated letter...",
    SendImmediately = true
});

// Subscribe to events
client.OnJobDiscovered += (sender, job) =>
{
    Console.WriteLine($"New job found: {job.Title}");
};
```