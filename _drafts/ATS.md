---
published: false
---

Here are some hiring systems (ATS – Applicant Tracking Systems) that can help you sort and manage candidates applying through LinkedIn and your website:

---

### ✅ **Top Hiring Systems with LinkedIn & Website Integration**

#### 1. **Workable**

* **Features:** Job posting, AI-powered sourcing, evaluations, customizable hiring pipeline.
* **Integrations:** LinkedIn, website embed, Google Workspace, Slack.
* **Pros:** Clean UI, good automation, flexible workflows.

#### 2. **Lever**

* **Features:** Candidate CRM, automation, analytics, LinkedIn sourcing plugin.
* **Integrations:** LinkedIn Recruiter, website forms, Zoom, Outlook.
* **Pros:** Strong pipeline management, collaboration tools.

#### 3. **Greenhouse**

* **Features:** Structured hiring, scorecards, interview kits, job board integration.
* **Integrations:** LinkedIn, company website (via embed code or API), many HR tools.
* **Pros:** Very scalable, great analytics, customizable.

#### 4. **Breezy HR**

* **Features:** Visual pipelines, automated messaging, video interviews, resume parsing.
* **Integrations:** LinkedIn, careers page widget, job boards.
* **Pros:** Easy to use, ideal for small to mid-size teams.

#### 5. **Teamtailor**

* **Features:** Career site builder, automation, analytics, nurturing features.
* **Integrations:** LinkedIn, job boards, custom career pages.
* **Pros:** Great branding tools, intuitive UI.

---

### 🧩 **Things to Look For**

* **LinkedIn Integration:** Sync applications and post directly from ATS.
* **Career Page Embeds:** Add job listings to your website.
* **Sorting Tools:** Tagging, filtering, stages, candidate scoring.
* **Automation:** Auto-response, follow-ups, resume screening.
* **Collaboration:** Notes, scorecards, and team feedback.

---

Given your setup — **Microsoft 365, Teams, SharePoint, Power Apps, and Power Automate**, and internal dev resources — building a **custom hiring system in-house** is not only feasible but **strategically smart** for a software house.

---

### ✅ **Why It Makes Sense for You**

#### 1. **Full Control & Customization**

* Design workflows tailored to your hiring pipeline.
* Build custom scoring, sorting, filtering — exactly as you need.

#### 2. **Deep Integration with Your Stack**

* Use **Power Automate** to integrate with:

  * **LinkedIn (via webhook or email parsing)**.
  * **Forms on your website** (store to SharePoint or Dataverse).
* Leverage **Teams** for alerts/interviews/collaboration.
* Store resumes and candidate data in **SharePoint Lists** or **Dataverse**.

#### 3. **No Extra Licensing**

* Reuse existing M365 licenses = low/no extra cost.

#### 4. **Scalable & Extendable**

* Add features like:

  * Role-based access.
  * Resume parsing via AI Builder.
  * Interview scheduling with Outlook Calendar.
  * Power BI dashboards for hiring analytics.

---

### 🔧 Example Architecture

* **Frontend (Candidate Entry):**

  * Website: HTML form → Power Automate → SharePoint/Dataverse.
  * LinkedIn: Auto-forwarded emails → Power Automate → SharePoint.

* **Backend (Internal Use):**

  * Power Apps: List/sort candidates, view status, feedback, etc.
  * SharePoint/Dataverse: Store data, resumes, metadata.

* **Automation:**

  * Power Automate: Notifications, reminders, scoring logic, scheduling interviews.

* **Collaboration:**

  * Teams channel per role or candidate stage.
  * Adaptive Cards to review/respond directly from Teams.

---

### 🚀 Suggestion: Build Phase-Wise

1. **MVP:**

   * Inbound application capture (web + LinkedIn via email).
   * SharePoint list + Power Apps frontend.
   * Teams notifications.

2. **Phase 2:**

   * Scoring system + filters.
   * Interview scheduling.

3. **Phase 3:**

   * Resume parsing (AI Builder or Azure).
   * Power BI dashboard.
   * Candidate feedback collection.

---

Absolutely. Here's a **step-by-step guide** to walk you through how a candidate can apply via your **static HTML website**, upload a resume, and how you can pull that into **Power Automate** for processing.

---

## 🧩 Objective

From your **HTML form**, capture:

* Candidate data (e.g., name, email, message)
* Resume upload (PDF/doc)
  And bring this into your internal Microsoft environment using **Power Automate**.

---

## ✅ Step-by-Step Guide

### **Step 1: Handle Form Submission with File Upload**

Since your site is static, use a **backend form handler**. Two options:

#### Option A: Use Microsoft Forms (Easiest)

* Embed a **Microsoft Form** in your website (`<iframe>` or hyperlink).
* It supports file uploads (stores to OneDrive).
* Power Automate has native triggers for MS Forms.

> 💡 Fastest way to integrate without backend code.

#### Option B: Use Formspree / Web3Forms / Your Own API (More Control)

* Use a service like [Formspree](https://formspree.io/) or create a **small serverless function** (Azure Function or Firebase).
* Make form submit to this endpoint.
* Upload file to **OneDrive or Azure Blob** from there.

---

### **Step 2: Trigger Power Automate**

Depending on the option above:

#### 🔹 If using Microsoft Forms

1. Create a Form:

   * Add fields (Name, Email, etc.)
   * Enable **file upload** (OneDrive)
2. In **Power Automate**:

   * Trigger: “**When a new response is submitted**” (Microsoft Forms)
   * Actions:

     * Get response details
     * Get file from OneDrive
     * Save candidate info to SharePoint / Dataverse
     * Send notification to Teams or email

#### 🔹 If using Custom Endpoint (e.g., with Azure)

1. In form handler (e.g., Azure Function), after file upload:

   * Send an HTTP request to a Power Automate **HTTP-triggered flow**.
2. In **Power Automate**:

   * Trigger: “**When an HTTP request is received**”
   * Parse JSON (name, email, resume URL)
   * Save info to SharePoint
   * Download file to your internal storage (e.g., SharePoint or OneDrive)
   * Notify via Teams/email

---

## 🗂 Suggested SharePoint Schema (for candidate data)

Create a SharePoint List:

| Column Name     | Type           |
| --------------- | -------------- |
| FullName        | Text           |
| Email           | Text           |
| Phone           | Text           |
| PositionApplied | Choice/Text    |
| ResumeLink      | Hyperlink      |
| SubmissionDate  | DateTime       |
| Notes/Tags      | Text/Multiline |

---

## ⚙️ Bonus: Embed in Website

You can embed a Microsoft Form like this:

```html
<iframe src="https://forms.office.com/Pages/ResponsePage.aspx?id=..." width="100%" height="600px" frameborder="0"></iframe>
```

Or link directly to it from an **“Apply Now”** button.

---

## 🚀 What's Next?

Once this flow is working, your devs can build:

* A **Power Apps frontend** to view, filter, and rate applicants.
* Auto-tagging resumes with Power Automate + AI Builder.

---

Would you like:

* A sample Microsoft Form?
* A ready-to-import Power Automate flow?
* Or help building a lightweight backend to receive form submissions and upload to OneDrive?

You're absolutely right — **Microsoft Forms does not support public file uploads** (only internal users), so it's not viable for external candidates.
✅ Going with an **Azure Function** is the correct direction — and yes, it **can fully handle file uploads and form data**, making it a flexible and powerful backend for your static website's “Apply Now” form.

---

## ✅ What Azure Function Can Do for You

1. **Receive multipart/form-data** POST requests (file + fields).
2. **Extract and save the resume** (e.g., to Azure Blob Storage or OneDrive via Graph API).
3. **Store form fields + file reference** as structured JSON (e.g., write to Azure Table, Cosmos DB, or send to Power Automate).
4. **Trigger downstream workflows** in Power Automate, Dataverse, or SharePoint.

---

## 🧩 How It Will Work

### 1. **HTML Form (Static Site)**

```html
<form action="https://your-function.azurewebsites.net/api/ApplyNow" method="post" enctype="multipart/form-data">
  <input type="text" name="fullName" required />
  <input type="email" name="email" required />
  <input type="file" name="resume" accept=".pdf,.doc,.docx" required />
  <textarea name="message"></textarea>
  <button type="submit">Apply Now</button>
</form>
```

### 2. **Azure Function (C# or Node.js)**

Handles:

* Reading the form fields (text inputs)
* Parsing and saving the file (resume)
* Returning a success response or triggering Power Automate

You can save the resume file:

* To **Azure Blob Storage** (best for scale)
* Or send directly to Power Automate via webhook or Microsoft Graph

---

## ✅ Example Use Cases After Upload

* Trigger a **Power Automate** flow using HTTP/Webhook.
* Save candidate info in **SharePoint** list or **Dataverse**.
* Send a **notification in Teams**.
* Auto-email confirmation to the candidate.

---

## 💡 Architecture Summary

```plaintext
[Website HTML Form] → [Azure Function] → 
    [Blob Storage for file] +
    [Structured JSON for metadata] →
        [Power Automate / Dataverse / SharePoint / Teams]
```

---

## ⚙️ Next Step

Would you like:

* A **ready-to-use C# Azure Function** to handle this upload?
* Integration into **Blob Storage + JSON metadata output**?
* A follow-up that sends that metadata to **Power Automate via webhook**?

Great idea — using an LLM to score resumes against job descriptions is **a powerful and scalable way** to enhance your hiring process. Now, let’s break down how to approach this and whether to use **Power Automate** or **extend your Azure Function**.

---

## ✅ Short Answer

👉 **Extend your Azure Function** to include the LLM scoring logic.
**Why?**

* You already have the resume and job description together at the point of upload.
* Azure Function can directly call **Azure OpenAI** (or OpenAI API) with full control.
* Power Automate is better for no-code orchestration but **less suitable for LLM processing at scale or with file content**.

---

## 🧩 Suggested Flow

```plaintext
[Candidate Form Submission]
      ↓
[Azure Function]
  - Parse resume file (PDF/Docx)
  - Extract job ID (or job description)
  - Call LLM (OpenAI/Azure OpenAI)
  - Score match % based on skills, experience
  - Return/store result
      ↓
[Save to SharePoint / Dataverse / trigger Teams alert]
```

---

## ✅ Why Azure Function is the Right Place for LLM

| Feature                     | Power Automate         | Azure Function                 |
| --------------------------- | ---------------------- | ------------------------------ |
| Resume file parsing         | ❌ Limited              | ✅ Full control                 |
| LLM integration             | ⚠️ Basic via connector | ✅ Direct OpenAI API            |
| Token control               | ❌ No                   | ✅ Yes                          |
| Custom scoring prompt logic | ⚠️ Limited             | ✅ Full power                   |
| Cost & performance control  | ❌ Not efficient        | ✅ Better for batch and compute |

---

## ⚙️ What You'll Need in Azure Function

### 1. **Parse Resume File**

* Use `PdfSharp`, `iTextSharp`, or `DocumentFormat.OpenXml` (for `.docx`) to extract raw text.

### 2. **Job Description Input**

* Passed in the form (or fetched from your system by job ID).

### 3. **Call OpenAI API**

* Craft a prompt like:

```plaintext
Job Description:
<...>

Resume:
<...>

Evaluate how well the resume matches the job. Return a score from 0–100 and mention missing or matched skills.
```

* Use `HttpClient` to send to OpenAI.

### 4. **Process Response**

* Store candidate info + score in SharePoint, Dataverse, or trigger Power Automate for human review.

---

## 🧠 Bonus Ideas

* Auto-tag candidates based on score.
* Auto-advance top 80% scorers for review.
* Run resume + LinkedIn profile comparison (if link provided).

---

## 🚀 Next Steps

Would you like:

* A **sample Azure Function** in C# that:

  * Parses resume PDF
  * Accepts job description
  * Calls OpenAI
  * Returns score + summary
* OR a **clean interface for your recruiters to review results** in Power Apps?

Great — since you're already using **Together AI** and interested in **minimizing cost**, using **LLaMA 3 via Together AI** for resume scoring is definitely feasible, but with a few **key implementation differences** compared to OpenAI.

---

## ✅ Short Answer

Yes, you can use **Together AI + LLaMA 3** to:

* Ingest parsed resume text (you'll need to extract it yourself).
* Compare it with a job description.
* Get a **match score and analysis** using a custom prompt.

But:
**Together AI does not support file upload or auto-document parsing like OpenAI** — you'll need to **preprocess the file** (PDF/Docx) yourself in the Azure Function before sending the text to the model.

---

## 🧩 How It Works with Together AI

### 🔹 Your Role (in Azure Function)

1. **Parse uploaded resume** into plain text.

   * Use `PdfSharp`, `iText7`, `Aspose.PDF`, or `OpenXml` (for `.docx`).
2. **Fetch/receive job description** (from SharePoint, DB, or part of submission).
3. **Craft a prompt** that feeds both:

   * `Resume text`
   * `Job description`
   * Ask the model to return a structured JSON with:

     * Match percentage
     * Matching skills
     * Missing skills

### 🔹 Together AI API

Call their `chat/completions` endpoint using LLaMA 3 (`meta-llama/Llama-3-70b-chat` or `Llama-3-8b-chat`):

**POST [https://api.together.xyz/v1/chat/completions](https://api.together.xyz/v1/chat/completions)**

```json
{
  "model": "meta-llama/Llama-3-70b-chat",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert recruiter helping assess candidate resumes."
    },
    {
      "role": "user",
      "content": "Given the following resume and job description, provide a match score (0-100), list matching and missing skills.\n\nResume:\n<resume text>\n\nJob Description:\n<job description>"
    }
  ],
  "temperature": 0.2,
  "top_p": 0.9,
  "max_tokens": 1024
}
```

> 🔐 You’ll need to use your Together AI API key in the `Authorization` header.

---

## ✅ Benefits of This Setup

| Feature            | Together AI + LLaMA 3                             |
| ------------------ | ------------------------------------------------- |
| Lower Cost         | ✅ Very cost-effective vs GPT-4                    |
| File Uploads       | ❌ You handle parsing                              |
| Prompt Control     | ✅ Full                                            |
| Fine-tuned scoring | ✅ Prompt-based                                    |
| Accuracy           | ✅ LLaMA 3 performs well on structured prompts     |
| Privacy            | ✅ Keeps data in your infra if parsing is in Azure |

---

## 🚀 Suggested Flow

```plaintext
[HTML Form + Resume Upload]
      ↓
[Azure Function]
  - Parse resume text
  - Get job description
  - Call Together AI with prompt
  - Receive match score + missing skills
  - Store in SharePoint or Dataverse
  - Notify via Teams (optional)
```

---

There currently aren't any well‑maintained, modern open‑source Applicant Tracking Systems (ATS) built in C#. Most popular open‑source ATS platforms use PHP (like OpenCATS) or JavaScript/TypeScript stacks.

---

### 🔍 Known Open-Source ATS Options

* **OpenCATS** (PHP/MySQL)
  A mature, self-hosted ATS. While not in C#, it’s widely used and actively maintained ([OpenCATS][1]).

* **FreeATS**, **iKrut**, **HireList**, etc.
  These are also PHP or other stacks—not C#. ([LinuxWays][2]).

* **Staff Optima**, **OpenATS**
  Both modern, but built in TypeScript/Node or serverless JS ([GitHub][3]).

---

### 🛠 C#‑centric Alternatives?

**SplendidCRM** is an open‑source CRM built in ASP .NET (C#) . It’s not an ATS per se, but you could:

* Use its C#/.NET foundation as a base.
* Extend it with modules for candidate pipelines, resume parsing, interview stages, etc.

---

### ✅ Possible Paths

1. **Adapt an existing C# CRM**
   Customize SplendidCRM or another .NET open‑source CRM to add ATS features (job posting, application pipelines, resume handling).

2. **Build from scratch in C#/.NET**
   You could implement basic ATS functionality: job listings, candidate tracking, interviews, offer status. You can use .NET Core Web API + EF Core + a component UI library.

---

### TL;DR

No ready-made, open-source ATS in C# right now. The closest is to adapt a C# CRM like SplendidCRM or build a lightweight ATS in .NET yourself.

---

[1]: https://www.opencats.org/ "Home - OpenCATS Applicant Tracking System"
[2]: https://linuxways.net/centos/top-8-free-and-open-source-applicant-tracking-software/ "Top 8 Free and Open-Source Applicant Tracking Software – LinuxWays"
[3]: https://github.com/AshrafElshaer/staff-Optima "GitHub - AshrafElshaer/staff-Optima: Open source applicant tracking system"

In 2025, you should prioritize modern, maintainable, and scalable technologies. Here's a recommended **technology stack** for a **basic application tracking system (ATS)** and the reasons for each:

---

### ✅ **Frontend**

**Framework:** React (with TypeScript)
**Why:**

* Widely adopted, well-supported
* Component-based, fast UI updates
* Strong TypeScript support for maintainability

**UI Library:** Tailwind CSS + ShadCN/UI or Radix UI
**Why:**

* Utility-first styling (Tailwind)
* Modern, accessible components (ShadCN or Radix)

**Bundler:** Vite
**Why:**

* Fast dev builds
* Works great with modern JS/TS frameworks

---

### ✅ **Backend**

**Language:** C# (.NET 8 or later)
**Why:**

* Mature and performant
* Strong support for APIs and background jobs
* Great tooling (e.g., Visual Studio, Rider)

**Framework:** ASP.NET Core Web API
**Why:**

* RESTful API support
* Built-in DI, logging, etc.
* Easy integration with databases and auth

---

### ✅ **Database**

**Relational DB:** PostgreSQL
**Why:**

* Open-source, robust
* Great support for relational data
* Scales well, strong community

**ORM:** Entity Framework Core
**Why:**

* Integrated with .NET
* Migrations, LINQ support, async-first

---

### ✅ **Authentication**

**Auth Provider:** Auth0 / Azure AD B2C / Firebase Auth
**Why:**

* Secure, externalized auth
* SSO, 2FA, social login ready
* Reduces internal security burden

---

### ✅ **Deployment & Hosting**

**Platform:**

* **Frontend:** Vercel / Netlify / Azure Static Web Apps
* **Backend:** Azure App Service / AWS App Runner / Fly.io
* **Database:** Supabase / Railway / PlanetScale (PostgreSQL)

**Why:**

* Fully managed, auto-scaling
* Easy CI/CD integration
* Less infrastructure overhead

---

### ✅ **Optional Enhancements**

| Feature          | Tool/Tech Suggestion      |
| ---------------- | ------------------------- |
| Realtime updates | SignalR (C#) or Socket.io |
| Job queues       | Hangfire (for .NET)       |
| Search indexing  | Meilisearch / Typesense   |
| Error monitoring | Sentry                    |
| Analytics        | PostHog / Plausible       |

---

