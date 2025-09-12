# JobHunt AI - Database Schema

## Overview

This document defines the complete database schema for JobHunt AI using PostgreSQL. The schema follows normalization principles, implements proper indexing strategies, and includes audit trails for compliance.

## Database Configuration

```sql
-- Database settings
CREATE DATABASE jobhuntai
    WITH 
    OWNER = jobhuntai_admin
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0
    CONNECTION LIMIT = -1;

-- Enable extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
CREATE EXTENSION IF NOT EXISTS "btree_gin";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";
```

## Schema Structure

```sql
-- Create schemas
CREATE SCHEMA IF NOT EXISTS users;
CREATE SCHEMA IF NOT EXISTS jobs;
CREATE SCHEMA IF NOT EXISTS applications;
CREATE SCHEMA IF NOT EXISTS communications;
CREATE SCHEMA IF NOT EXISTS ai;
CREATE SCHEMA IF NOT EXISTS integrations;
CREATE SCHEMA IF NOT EXISTS analytics;
CREATE SCHEMA IF NOT EXISTS audit;
```

## Users Schema

### users.users

```sql
CREATE TABLE users.users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    email_verified BOOLEAN DEFAULT FALSE,
    google_id VARCHAR(255) UNIQUE,
    name VARCHAR(255) NOT NULL,
    picture_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    last_login_at TIMESTAMP WITH TIME ZONE,
    deleted_at TIMESTAMP WITH TIME ZONE,
    deletion_requested_at TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    subscription_tier VARCHAR(50) DEFAULT 'free',
    subscription_expires_at TIMESTAMP WITH TIME ZONE,
    CONSTRAINT check_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

CREATE INDEX idx_users_email ON users.users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_google_id ON users.users(google_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_created_at ON users.users(created_at DESC);
```

### users.profiles

```sql
CREATE TABLE users.profiles (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    headline VARCHAR(255),
    summary TEXT,
    location VARCHAR(255),
    phone VARCHAR(50),
    linkedin_url VARCHAR(500),
    github_url VARCHAR(500),
    portfolio_url VARCHAR(500),
    years_experience INTEGER,
    current_salary INTEGER,
    desired_salary_min INTEGER,
    desired_salary_max INTEGER,
    availability VARCHAR(50), -- immediate, 2_weeks, 1_month, passive
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_profile UNIQUE(user_id)
);

CREATE INDEX idx_profiles_user_id ON users.profiles(user_id);
```

### users.skills

```sql
CREATE TABLE users.skills (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    skill_name VARCHAR(100) NOT NULL,
    skill_level VARCHAR(20), -- beginner, intermediate, advanced, expert
    years_experience DECIMAL(3,1),
    is_primary BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_skill UNIQUE(user_id, skill_name)
);

CREATE INDEX idx_skills_user_id ON users.skills(user_id);
CREATE INDEX idx_skills_name ON users.skills(skill_name);
CREATE INDEX idx_skills_primary ON users.skills(user_id, is_primary) WHERE is_primary = TRUE;
```

### users.experience

```sql
CREATE TABLE users.experience (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    company_name VARCHAR(255) NOT NULL,
    job_title VARCHAR(255) NOT NULL,
    description TEXT,
    location VARCHAR(255),
    start_date DATE NOT NULL,
    end_date DATE,
    is_current BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_experience_user_id ON users.experience(user_id);
CREATE INDEX idx_experience_dates ON users.experience(user_id, start_date DESC);
```

### users.education

```sql
CREATE TABLE users.education (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    institution_name VARCHAR(255) NOT NULL,
    degree VARCHAR(255) NOT NULL,
    field_of_study VARCHAR(255),
    start_date DATE,
    end_date DATE,
    gpa DECIMAL(3,2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_education_user_id ON users.education(user_id);
```

### users.resumes

```sql
CREATE TABLE users.resumes (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_url TEXT NOT NULL,
    file_size INTEGER,
    file_hash VARCHAR(64),
    version INTEGER DEFAULT 1,
    is_primary BOOLEAN DEFAULT FALSE,
    parsed_data JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_resumes_user_id ON users.resumes(user_id);
CREATE INDEX idx_resumes_primary ON users.resumes(user_id, is_primary) WHERE is_primary = TRUE;
CREATE INDEX idx_resumes_created_at ON users.resumes(created_at DESC);
```

### users.preferences

```sql
CREATE TABLE users.preferences (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    job_types TEXT[], -- full_time, part_time, contract, freelance
    work_modes TEXT[], -- onsite, remote, hybrid
    locations TEXT[],
    location_radius_miles INTEGER DEFAULT 25,
    salary_min INTEGER,
    salary_max INTEGER,
    salary_currency VARCHAR(3) DEFAULT 'USD',
    company_sizes TEXT[], -- startup, small, medium, large, enterprise
    industries TEXT[],
    exclude_companies TEXT[],
    min_company_rating DECIMAL(2,1),
    auto_apply BOOLEAN DEFAULT FALSE,
    application_frequency VARCHAR(20) DEFAULT 'moderate', -- conservative, moderate, aggressive
    max_applications_per_day INTEGER DEFAULT 10,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_preferences UNIQUE(user_id)
);

CREATE INDEX idx_preferences_user_id ON users.preferences(user_id);
```

### users.settings

```sql
CREATE TABLE users.settings (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    email_notifications BOOLEAN DEFAULT TRUE,
    email_frequency VARCHAR(20) DEFAULT 'daily', -- realtime, daily, weekly
    follow_up_enabled BOOLEAN DEFAULT TRUE,
    follow_up_days INTEGER DEFAULT 7,
    interview_reminders BOOLEAN DEFAULT TRUE,
    reminder_minutes INTEGER[] DEFAULT ARRAY[15, 60],
    timezone VARCHAR(50) DEFAULT 'America/Los_Angeles',
    language VARCHAR(10) DEFAULT 'en',
    theme VARCHAR(20) DEFAULT 'light',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_settings UNIQUE(user_id)
);

CREATE INDEX idx_settings_user_id ON users.settings(user_id);
```

## Jobs Schema

### jobs.sources

```sql
CREATE TABLE jobs.sources (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100) UNIQUE NOT NULL,
    type VARCHAR(50) NOT NULL, -- job_board, company_site, aggregator
    base_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    crawl_frequency_hours INTEGER DEFAULT 24,
    last_crawled_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

INSERT INTO jobs.sources (name, type, base_url) VALUES
    ('LinkedIn', 'job_board', 'https://linkedin.com/jobs'),
    ('Indeed', 'job_board', 'https://indeed.com'),
    ('Glassdoor', 'job_board', 'https://glassdoor.com'),
    ('AngelList', 'job_board', 'https://angel.co');
```

### jobs.companies

```sql
CREATE TABLE jobs.companies (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    website VARCHAR(500),
    logo_url TEXT,
    industry VARCHAR(100),
    size_range VARCHAR(50), -- 1-10, 11-50, 51-200, 201-500, 501-1000, 1000+
    headquarters_location VARCHAR(255),
    description TEXT,
    glassdoor_rating DECIMAL(2,1),
    linkedin_url VARCHAR(500),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_company_website UNIQUE(website)
);

CREATE INDEX idx_companies_name ON jobs.companies(name);
CREATE INDEX idx_companies_industry ON jobs.companies(industry);
```

### jobs.jobs

```sql
CREATE TABLE jobs.jobs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    external_id VARCHAR(255),
    source_id UUID REFERENCES jobs.sources(id),
    company_id UUID REFERENCES jobs.companies(id),
    title VARCHAR(500) NOT NULL,
    description TEXT NOT NULL,
    requirements TEXT,
    benefits TEXT,
    location VARCHAR(255),
    is_remote BOOLEAN DEFAULT FALSE,
    is_hybrid BOOLEAN DEFAULT FALSE,
    job_type VARCHAR(50), -- full_time, part_time, contract, internship
    experience_level VARCHAR(50), -- entry, junior, mid, senior, lead, executive
    salary_min INTEGER,
    salary_max INTEGER,
    salary_currency VARCHAR(3) DEFAULT 'USD',
    salary_period VARCHAR(20) DEFAULT 'year', -- hour, day, week, month, year
    application_url TEXT NOT NULL,
    posted_date TIMESTAMP WITH TIME ZONE,
    expiry_date TIMESTAMP WITH TIME ZONE,
    is_active BOOLEAN DEFAULT TRUE,
    hash VARCHAR(64), -- For duplicate detection
    raw_data JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_job_external UNIQUE(source_id, external_id)
);

CREATE INDEX idx_jobs_company ON jobs.jobs(company_id);
CREATE INDEX idx_jobs_title ON jobs.jobs(title);
CREATE INDEX idx_jobs_location ON jobs.jobs(location);
CREATE INDEX idx_jobs_posted_date ON jobs.jobs(posted_date DESC);
CREATE INDEX idx_jobs_salary ON jobs.jobs(salary_min, salary_max);
CREATE INDEX idx_jobs_active ON jobs.jobs(is_active, posted_date DESC) WHERE is_active = TRUE;
CREATE INDEX idx_jobs_hash ON jobs.jobs(hash);
-- Full text search
CREATE INDEX idx_jobs_search ON jobs.jobs USING gin(to_tsvector('english', title || ' ' || description));
```

### jobs.job_skills

```sql
CREATE TABLE jobs.job_skills (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    job_id UUID NOT NULL REFERENCES jobs.jobs(id) ON DELETE CASCADE,
    skill_name VARCHAR(100) NOT NULL,
    is_required BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_job_skill UNIQUE(job_id, skill_name)
);

CREATE INDEX idx_job_skills_job_id ON jobs.job_skills(job_id);
CREATE INDEX idx_job_skills_name ON jobs.job_skills(skill_name);
```

### jobs.saved_jobs

```sql
CREATE TABLE jobs.saved_jobs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    job_id UUID NOT NULL REFERENCES jobs.jobs(id) ON DELETE CASCADE,
    notes TEXT,
    saved_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_saved_job UNIQUE(user_id, job_id)
);

CREATE INDEX idx_saved_jobs_user ON jobs.saved_jobs(user_id, saved_at DESC);
CREATE INDEX idx_saved_jobs_job ON jobs.saved_jobs(job_id);
```

### jobs.job_matches

```sql
CREATE TABLE jobs.job_matches (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    job_id UUID NOT NULL REFERENCES jobs.jobs(id) ON DELETE CASCADE,
    match_score INTEGER CHECK (match_score >= 0 AND match_score <= 100),
    skill_match_score INTEGER,
    location_match_score INTEGER,
    salary_match_score INTEGER,
    experience_match_score INTEGER,
    match_reasons JSONB,
    is_recommended BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_job_match UNIQUE(user_id, job_id)
);

CREATE INDEX idx_job_matches_user ON jobs.job_matches(user_id, match_score DESC);
CREATE INDEX idx_job_matches_job ON jobs.job_matches(job_id);
CREATE INDEX idx_job_matches_score ON jobs.job_matches(match_score DESC) WHERE match_score > 70;
```

## Applications Schema

### applications.applications

```sql
CREATE TABLE applications.applications (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    job_id UUID NOT NULL REFERENCES jobs.jobs(id) ON DELETE CASCADE,
    status VARCHAR(50) NOT NULL DEFAULT 'draft',
    -- draft, queued, sent, delivered, viewed, responded, interviewing, rejected, offered, withdrawn
    cover_letter_id UUID,
    resume_id UUID REFERENCES users.resumes(id),
    applied_at TIMESTAMP WITH TIME ZONE,
    scheduled_for TIMESTAMP WITH TIME ZONE,
    response_received_at TIMESTAMP WITH TIME ZONE,
    last_activity_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_job_application UNIQUE(user_id, job_id),
    CONSTRAINT check_status CHECK (status IN (
        'draft', 'queued', 'sent', 'delivered', 'viewed', 
        'responded', 'interviewing', 'rejected', 'offered', 'withdrawn'
    ))
);

CREATE INDEX idx_applications_user ON applications.applications(user_id, created_at DESC);
CREATE INDEX idx_applications_job ON applications.applications(job_id);
CREATE INDEX idx_applications_status ON applications.applications(status, created_at DESC);
CREATE INDEX idx_applications_applied_at ON applications.applications(applied_at DESC) WHERE applied_at IS NOT NULL;
```

### applications.application_activities

```sql
CREATE TABLE applications.application_activities (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    application_id UUID NOT NULL REFERENCES applications.applications(id) ON DELETE CASCADE,
    activity_type VARCHAR(50) NOT NULL,
    -- created, updated, sent, delivered, opened, clicked, responded, status_changed
    description TEXT,
    metadata JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_app_activities_application ON applications.application_activities(application_id, created_at DESC);
CREATE INDEX idx_app_activities_type ON applications.application_activities(activity_type);
```

### applications.interviews

```sql
CREATE TABLE applications.interviews (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    application_id UUID NOT NULL REFERENCES applications.applications(id) ON DELETE CASCADE,
    interview_type VARCHAR(50), -- phone_screen, technical, behavioral, onsite, final
    scheduled_at TIMESTAMP WITH TIME ZONE NOT NULL,
    duration_minutes INTEGER,
    location TEXT,
    meeting_url TEXT,
    interviewer_name VARCHAR(255),
    interviewer_email VARCHAR(255),
    notes TEXT,
    calendar_event_id VARCHAR(255),
    status VARCHAR(50) DEFAULT 'scheduled', -- scheduled, completed, cancelled, rescheduled
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_interviews_application ON applications.interviews(application_id);
CREATE INDEX idx_interviews_scheduled ON applications.interviews(scheduled_at);
CREATE INDEX idx_interviews_status ON applications.interviews(status);
```

### applications.offers

```sql
CREATE TABLE applications.offers (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    application_id UUID NOT NULL REFERENCES applications.applications(id) ON DELETE CASCADE,
    salary_offered INTEGER,
    salary_currency VARCHAR(3) DEFAULT 'USD',
    bonus_offered INTEGER,
    equity_offered VARCHAR(100),
    benefits JSONB,
    start_date DATE,
    expiry_date DATE,
    status VARCHAR(50) DEFAULT 'pending', -- pending, accepted, declined, negotiating
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_application_offer UNIQUE(application_id)
);

CREATE INDEX idx_offers_application ON applications.offers(application_id);
CREATE INDEX idx_offers_status ON applications.offers(status);
```

## Communications Schema

### communications.email_templates

```sql
CREATE TABLE communications.email_templates (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users.users(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- application, follow_up, thank_you, withdrawal
    subject VARCHAR(500),
    body TEXT NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    is_system BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_email_templates_user ON communications.email_templates(user_id);
CREATE INDEX idx_email_templates_type ON communications.email_templates(type);
```

### communications.emails

```sql
CREATE TABLE communications.emails (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    application_id UUID REFERENCES applications.applications(id) ON DELETE CASCADE,
    template_id UUID REFERENCES communications.email_templates(id),
    type VARCHAR(50) NOT NULL, -- application, follow_up, response, thank_you
    from_email VARCHAR(255) NOT NULL,
    to_emails TEXT[] NOT NULL,
    cc_emails TEXT[],
    bcc_emails TEXT[],
    subject VARCHAR(500) NOT NULL,
    body TEXT NOT NULL,
    attachments JSONB,
    status VARCHAR(50) DEFAULT 'draft',
    -- draft, queued, sending, sent, delivered, bounced, failed
    sent_at TIMESTAMP WITH TIME ZONE,
    delivered_at TIMESTAMP WITH TIME ZONE,
    opened_at TIMESTAMP WITH TIME ZONE,
    clicked_at TIMESTAMP WITH TIME ZONE,
    bounce_reason TEXT,
    provider_message_id VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_emails_user ON communications.emails(user_id, created_at DESC);
CREATE INDEX idx_emails_application ON communications.emails(application_id);
CREATE INDEX idx_emails_status ON communications.emails(status);
CREATE INDEX idx_emails_sent_at ON communications.emails(sent_at DESC) WHERE sent_at IS NOT NULL;
```

### communications.email_events

```sql
CREATE TABLE communications.email_events (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email_id UUID NOT NULL REFERENCES communications.emails(id) ON DELETE CASCADE,
    event_type VARCHAR(50) NOT NULL, -- sent, delivered, opened, clicked, bounced, complained
    event_data JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_email_events_email ON communications.email_events(email_id, created_at DESC);
CREATE INDEX idx_email_events_type ON communications.email_events(event_type);
```

### communications.email_responses

```sql
CREATE TABLE communications.email_responses (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email_id UUID REFERENCES communications.emails(id) ON DELETE CASCADE,
    from_email VARCHAR(255) NOT NULL,
    subject VARCHAR(500),
    body TEXT,
    classification VARCHAR(50),
    -- interview_request, rejection, request_info, automated, spam
    confidence_score DECIMAL(3,2),
    extracted_data JSONB,
    received_at TIMESTAMP WITH TIME ZONE NOT NULL,
    processed_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_email_responses_email ON communications.email_responses(email_id);
CREATE INDEX idx_email_responses_classification ON communications.email_responses(classification);
CREATE INDEX idx_email_responses_received ON communications.email_responses(received_at DESC);
```

## AI Schema

### ai.cover_letters

```sql
CREATE TABLE ai.cover_letters (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    job_id UUID REFERENCES jobs.jobs(id) ON DELETE CASCADE,
    application_id UUID REFERENCES applications.applications(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    version INTEGER DEFAULT 1,
    tone VARCHAR(50), -- professional, casual, enthusiastic
    length VARCHAR(20), -- short, medium, long
    quality_score INTEGER,
    ats_score INTEGER,
    model_used VARCHAR(50),
    tokens_used INTEGER,
    generation_time_ms INTEGER,
    is_edited BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_cover_letters_user ON ai.cover_letters(user_id, created_at DESC);
CREATE INDEX idx_cover_letters_job ON ai.cover_letters(job_id);
CREATE INDEX idx_cover_letters_application ON ai.cover_letters(application_id);
```

### ai.prompts

```sql
CREATE TABLE ai.prompts (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- cover_letter, resume_tailor, email_response
    template TEXT NOT NULL,
    variables TEXT[],
    version INTEGER DEFAULT 1,
    is_active BOOLEAN DEFAULT TRUE,
    success_rate DECIMAL(5,2),
    usage_count INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_prompts_type ON ai.prompts(type, is_active);
```

### ai.generations

```sql
CREATE TABLE ai.generations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    prompt_id UUID REFERENCES ai.prompts(id),
    type VARCHAR(50) NOT NULL,
    input_data JSONB,
    output TEXT,
    model VARCHAR(50),
    tokens_input INTEGER,
    tokens_output INTEGER,
    cost_cents INTEGER,
    duration_ms INTEGER,
    quality_score INTEGER,
    user_rating INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_generations_user ON ai.generations(user_id, created_at DESC);
CREATE INDEX idx_generations_type ON ai.generations(type);
```

## Integrations Schema

### integrations.google_sheets

```sql
CREATE TABLE integrations.google_sheets (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    spreadsheet_id VARCHAR(255) NOT NULL,
    spreadsheet_url TEXT,
    sheet_name VARCHAR(255),
    is_active BOOLEAN DEFAULT TRUE,
    last_sync_at TIMESTAMP WITH TIME ZONE,
    sync_frequency_minutes INTEGER DEFAULT 5,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_sheets UNIQUE(user_id)
);

CREATE INDEX idx_google_sheets_user ON integrations.google_sheets(user_id);
```

### integrations.google_calendar

```sql
CREATE TABLE integrations.google_calendar (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    calendar_id VARCHAR(255) NOT NULL,
    is_active BOOLEAN DEFAULT TRUE,
    auto_schedule BOOLEAN DEFAULT TRUE,
    reminder_minutes INTEGER[] DEFAULT ARRAY[15, 60],
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_calendar UNIQUE(user_id)
);

CREATE INDEX idx_google_calendar_user ON integrations.google_calendar(user_id);
```

### integrations.oauth_tokens

```sql
CREATE TABLE integrations.oauth_tokens (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    provider VARCHAR(50) NOT NULL, -- google, linkedin, github
    access_token TEXT NOT NULL,
    refresh_token TEXT,
    expires_at TIMESTAMP WITH TIME ZONE,
    scope TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_provider UNIQUE(user_id, provider)
);

CREATE INDEX idx_oauth_tokens_user ON integrations.oauth_tokens(user_id);
CREATE INDEX idx_oauth_tokens_expires ON integrations.oauth_tokens(expires_at) WHERE expires_at IS NOT NULL;
```

## Analytics Schema

### analytics.user_events

```sql
CREATE TABLE analytics.user_events (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users.users(id) ON DELETE CASCADE,
    session_id VARCHAR(255),
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB,
    ip_address INET,
    user_agent TEXT,
    referrer TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Partitioned by month for better performance
CREATE INDEX idx_user_events_user ON analytics.user_events(user_id, created_at DESC);
CREATE INDEX idx_user_events_type ON analytics.user_events(event_type, created_at DESC);
CREATE INDEX idx_user_events_session ON analytics.user_events(session_id);
```

### analytics.application_metrics

```sql
CREATE TABLE analytics.application_metrics (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users.users(id) ON DELETE CASCADE,
    date DATE NOT NULL,
    applications_sent INTEGER DEFAULT 0,
    applications_viewed INTEGER DEFAULT 0,
    responses_received INTEGER DEFAULT 0,
    interviews_scheduled INTEGER DEFAULT 0,
    offers_received INTEGER DEFAULT 0,
    average_response_days DECIMAL(5,2),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT unique_user_date_metrics UNIQUE(user_id, date)
);

CREATE INDEX idx_application_metrics_user ON analytics.application_metrics(user_id, date DESC);
CREATE INDEX idx_application_metrics_date ON analytics.application_metrics(date DESC);
```

### analytics.system_metrics

```sql
CREATE TABLE analytics.system_metrics (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    metric_type VARCHAR(100) NOT NULL,
    metric_value DECIMAL(20,4),
    metric_data JSONB,
    recorded_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_system_metrics_type ON analytics.system_metrics(metric_type, recorded_at DESC);
```

## Audit Schema

### audit.audit_logs

```sql
CREATE TABLE audit.audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users.users(id) ON DELETE SET NULL,
    entity_type VARCHAR(100) NOT NULL,
    entity_id UUID,
    action VARCHAR(50) NOT NULL, -- create, update, delete, view
    changes JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Partitioned by month
CREATE INDEX idx_audit_logs_user ON audit.audit_logs(user_id, created_at DESC);
CREATE INDEX idx_audit_logs_entity ON audit.audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created ON audit.audit_logs(created_at DESC);
```

## Functions and Triggers

### Updated At Trigger

```sql
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at column
CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users.users
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_profiles_updated_at BEFORE UPDATE ON users.profiles
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

-- ... (apply to all other tables with updated_at)
```

### Audit Log Function

```sql
CREATE OR REPLACE FUNCTION create_audit_log()
RETURNS TRIGGER AS $$
DECLARE
    changes JSONB;
BEGIN
    IF TG_OP = 'UPDATE' THEN
        changes := to_jsonb(NEW) - to_jsonb(OLD);
    ELSIF TG_OP = 'INSERT' THEN
        changes := to_jsonb(NEW);
    ELSIF TG_OP = 'DELETE' THEN
        changes := to_jsonb(OLD);
    END IF;
    
    INSERT INTO audit.audit_logs (
        user_id,
        entity_type,
        entity_id,
        action,
        changes
    ) VALUES (
        current_setting('app.current_user_id', true)::UUID,
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        LOWER(TG_OP),
        changes
    );
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

### Job Match Scoring Function

```sql
CREATE OR REPLACE FUNCTION calculate_job_match_score(
    p_user_id UUID,
    p_job_id UUID
) RETURNS INTEGER AS $$
DECLARE
    skill_score INTEGER := 0;
    location_score INTEGER := 0;
    salary_score INTEGER := 0;
    experience_score INTEGER := 0;
    total_score INTEGER := 0;
BEGIN
    -- Calculate skill match (40% weight)
    SELECT COUNT(*) * 10 INTO skill_score
    FROM users.skills us
    JOIN jobs.job_skills js ON us.skill_name = js.skill_name
    WHERE us.user_id = p_user_id AND js.job_id = p_job_id;
    
    skill_score := LEAST(skill_score, 40);
    
    -- Calculate location match (20% weight)
    -- ... location matching logic
    
    -- Calculate salary match (20% weight)
    -- ... salary matching logic
    
    -- Calculate experience match (20% weight)
    -- ... experience matching logic
    
    total_score := skill_score + location_score + salary_score + experience_score;
    
    RETURN LEAST(total_score, 100);
END;
$$ LANGUAGE plpgsql;
```

## Indexes for Performance

### Composite Indexes

```sql
-- User search
CREATE INDEX idx_users_search ON users.users 
    USING gin(to_tsvector('english', name || ' ' || email));

-- Job search
CREATE INDEX idx_jobs_filter ON jobs.jobs(location, is_remote, job_type, experience_level) 
    WHERE is_active = TRUE;

-- Application dashboard
CREATE INDEX idx_applications_dashboard ON applications.applications(user_id, status, created_at DESC);

-- Email tracking
CREATE INDEX idx_emails_tracking ON communications.emails(application_id, status, sent_at DESC);
```

### Partial Indexes

```sql
-- Active users
CREATE INDEX idx_users_active ON users.users(created_at DESC) 
    WHERE is_active = TRUE AND deleted_at IS NULL;

-- Pending applications
CREATE INDEX idx_applications_pending ON applications.applications(scheduled_for) 
    WHERE status = 'queued' AND scheduled_for IS NOT NULL;

-- Unprocessed responses
CREATE INDEX idx_email_responses_unprocessed ON communications.email_responses(received_at) 
    WHERE processed_at IS NULL;
```

## Data Retention Policies

```sql
-- Archive old audit logs
CREATE TABLE audit.audit_logs_archive (LIKE audit.audit_logs INCLUDING ALL);

-- Archive function
CREATE OR REPLACE FUNCTION archive_old_audit_logs()
RETURNS void AS $$
BEGIN
    INSERT INTO audit.audit_logs_archive
    SELECT * FROM audit.audit_logs
    WHERE created_at < CURRENT_DATE - INTERVAL '90 days';
    
    DELETE FROM audit.audit_logs
    WHERE created_at < CURRENT_DATE - INTERVAL '90 days';
END;
$$ LANGUAGE plpgsql;

-- Schedule archival
SELECT cron.schedule('archive-audit-logs', '0 2 * * *', 'SELECT archive_old_audit_logs()');
```

## Security Configurations

### Row Level Security

```sql
-- Enable RLS on user tables
ALTER TABLE users.users ENABLE ROW LEVEL SECURITY;
ALTER TABLE users.profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE applications.applications ENABLE ROW LEVEL SECURITY;

-- User can only see their own data
CREATE POLICY users_own_data ON users.users
    FOR ALL USING (id = current_setting('app.current_user_id')::UUID);

CREATE POLICY profiles_own_data ON users.profiles
    FOR ALL USING (user_id = current_setting('app.current_user_id')::UUID);

CREATE POLICY applications_own_data ON applications.applications
    FOR ALL USING (user_id = current_setting('app.current_user_id')::UUID);
```

### Encryption

```sql
-- Encrypt sensitive columns
CREATE EXTENSION IF NOT EXISTS pgcrypto;

-- Example: Encrypt OAuth tokens
ALTER TABLE integrations.oauth_tokens
    ALTER COLUMN access_token TYPE TEXT USING pgp_sym_encrypt(access_token, current_setting('app.encryption_key')),
    ALTER COLUMN refresh_token TYPE TEXT USING pgp_sym_encrypt(refresh_token, current_setting('app.encryption_key'));
```

## Maintenance Scripts

### Vacuum and Analyze

```sql
-- Regular maintenance
VACUUM ANALYZE users.users;
VACUUM ANALYZE jobs.jobs;
VACUUM ANALYZE applications.applications;

-- Reindex heavily used tables
REINDEX TABLE jobs.jobs;
REINDEX TABLE applications.applications;
```

### Statistics Update

```sql
-- Update table statistics
ANALYZE users.users;
ANALYZE jobs.jobs;
ANALYZE applications.applications;
```

## Backup Strategy

```sql
-- Backup script
pg_dump -h localhost -U jobhuntai_admin -d jobhuntai -f backup_$(date +%Y%m%d_%H%M%S).sql

-- Point-in-time recovery setup
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'
wal_level = replica
max_wal_senders = 3
```

## Monitoring Queries

### Application Statistics

```sql
-- User activity summary
SELECT 
    u.id,
    u.email,
    COUNT(DISTINCT a.id) as total_applications,
    COUNT(DISTINCT CASE WHEN a.status = 'interviewing' THEN a.id END) as interviews,
    COUNT(DISTINCT CASE WHEN a.status = 'offered' THEN a.id END) as offers,
    AVG(EXTRACT(DAY FROM (a.response_received_at - a.applied_at))) as avg_response_days
FROM users.users u
LEFT JOIN applications.applications a ON u.id = a.user_id
WHERE u.created_at > CURRENT_DATE - INTERVAL '30 days'
GROUP BY u.id, u.email;

-- Job match quality
SELECT 
    jm.match_score,
    COUNT(*) as job_count,
    COUNT(DISTINCT a.id) as applications,
    AVG(CASE WHEN a.status IN ('interviewing', 'offered') THEN 1 ELSE 0 END) as success_rate
FROM jobs.job_matches jm
LEFT JOIN applications.applications a ON jm.job_id = a.job_id AND jm.user_id = a.user_id
GROUP BY jm.match_score
ORDER BY jm.match_score DESC;
```