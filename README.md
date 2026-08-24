# DevSynt-TeamAI-Task3-ATS
# DevSynt Autonomous ATS & Intelligent Routing Engine

**Developer:** Raheela Daud  
**Project Lead:** Usama  
**Assigned Track:** Team AI — Task 3  
**Implementation State:** Production-Ready (100% Evaluation Criteria Met)  

---

## 1. System Overview
An enterprise-grade autonomous Applicant Tracking System (ATS) orchestrated via **n8n**, powered by **Google Gemini AI**, persistent on **Supabase (PostgreSQL)**, and integrated with the **Gmail API**. The pipeline continuously ingests candidate profiles across multi-channel sources, cleanses data, scores applicant resumes via LLM evaluation, and autonomously executes deterministic 3-way routing with personalized candidate communications.

---

## 2. Core Architecture & Workflows
[Google Form (Webhook)] ──┐├──> [Schema Normalizer] ──> [Instant Receipt ACK Email][Candidate Inbound Email] ┘                                    │▼[Google Gemini AI Evaluator]│▼[Supabase Database Persistence](status: pending_review)│▼[Scheduled Poller & Switch Node]│┌────────────────────────────────────┼────────────────────────────────────┐▼                                    ▼                                    ▼[Strong Track (Score > 75)]          [Average Track (Score 50-75)]         [Weak Track (Score < 50)]│                                    │                                    │▼                                    ▼                                    ▼[Gmail: Assessment Invite]               [Gmail: HR Review Alert]              [Gmail: Polite Rejection]│                                    │                                    │▼                                    ▼                                    ▼[Supabase: status = 'notified']       [Supabase: status = 'manual_review']    [Supabase: status = 'notified']
### Workflow 1: Intake & AI Evaluation Engine
* **Multi-Source Ingestion:** Webhook listener for structured form entries and Gmail trigger for raw email resumes.
* **Schema Normalization:** Maps heterogeneous fields into a single applicant standard (`name`, `email`, `role_applied`, `resume_text`).
* **Automated Candidate Acknowledgment:** Immediately sends receipt confirmation to applicants.
* **LLM ATS Evaluation:** Google Gemini Flash computes candidate fit, returning structured JSON with quantitative score (`0–100`) and qualification tiers (`Strong`, `Average`, `Weak`).
* **Relational Persistence:** Stores candidate state as `pending_review` in Supabase.

### Workflow 2: Scheduled Poller & 3-Way Deterministic Routing
* **Scheduled Poller:** Periodically queries Supabase for all records with `status = 'pending_review'`.
* **Switch Routing Logic:**
  * **Strong Track:** Automated Technical Assessment invitation; updates database status to `notified`.
  * **Average Track:** Alerts hiring manager with full resume summaries; updates status to `manual_review`.
  * **Weak Track:** Dispatches polite rejection notice; updates database status to `notified`.
* **State Integrity:** Atomic database status transitions eliminate duplicate notifications.

---

## 3. Database Schema (Supabase DDL)

```sql
CREATE TABLE public.candidates (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    email TEXT NOT NULL,
    phone TEXT,
    source TEXT NOT NULL,
    role_applied TEXT NOT NULL,
    resume_text TEXT NOT NULL,
    resume_url TEXT,
    ats_score INTEGER,
    classification TEXT CHECK (classification IN ('Strong', 'Average', 'Weak')),
    status TEXT DEFAULT 'pending_review',
    manager_notes TEXT,
    applied_at TIMESTAMPTZ DEFAULT now(),
    created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_candidates_status ON public.candidates(status);
4. Test Suite Validation ResultsCandidate NameSourceATS ScoreClassificationBranch TriggeredFinal DB StatusStatusHamza TariqGoogle Form92 / 100StrongTechnical Assessment InvitenotifiedVerifiedSara KhanInbound Email78 / 100StrongTechnical Assessment InvitenotifiedVerifiedAli AhmedGoogle Form65 / 100AverageHR Manual Review Alertmanual_reviewVerifiedBilal RazaInbound Email40 / 100WeakPolite Rejection DispatchednotifiedVerified5. Official Deliverables Checklist[x] Workflow 1 JSON Blueprint exported and attached[x] Workflow 2 JSON Blueprint exported and attached[x] Technical Specification Report PDF attached[x] Validation Execution Proof Screenshots attached[x] Production Database Schema DDL defined
