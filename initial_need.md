Prompt (English):

I want to build a SaaS product for the U.S. job market: an AI-powered job search automation bot that removes the main pain point of job hunting—spending hours writing resumes, applying with enthusiasm, only to be rejected in one line or never receive a reply, leading to burnout. Job hunting is like a marathon, and the biggest issue is losing motivation. I want AI to handle everything in the background.

Key features required:

Web crawling to automatically find jobs that match the user’s resume.

AI-generated personalized cover letters tailored to each job.

Stakeholder search and display (e.g., hiring managers, recruiters).

Automated email sending (resume + cover letter), receiving, and classification.

Google Sheets integration to track job status automatically: applied, awaiting response, rejected, interview scheduled, follow-ups after one week, etc.

Google Calendar integration for interview scheduling and reminders.

Fully autonomous workflow using Playwright.NET, Semantic Kernel, and Hangfire, running continuously in the background.

User flow:

Users log in with their Google account.

Select job categories and upload their resume.

Set preferences (e.g., location radius, remote allowed).

The system continuously updates in the background.

Users occasionally check back to review the automatically updated Google Sheet, where all applications and statuses are neatly organized.

Technical requirements:

Frontend: Next.js 15

Backend: ASP.NET

Cloudflare Workers for deployment and edge handling

Wrangler for configuration and deployment

Single GitHub repository managing both frontend and backend in a monorepo structure.

Complete setup of Wrangler, backend, frontend, and integration with Google APIs.

Deliverable:
A full requirements definition for this SaaS, including detailed architecture and implementation plan, with backend and frontend fully completed, deployed, and ready to run from a single GitHub repo.