

##= 📜 Project Abstract

Northeastern University (NEU) researchers face a significant challenge in navigating the complex, fragmented landscape of Open Access (OA) publishing. While the university funds numerous agreements to cover Article Processing Charges (APCs), this information is spread across multiple sources, and many agreements are subject to rapidly diminishing annual quotas. This project proposes the **NEU Open Access Journal Bot**, an intelligent tool to consolidate these disparate data sources into a single, user-friendly interface. The bot will analyze an author's paper draft to suggest topically relevant journals, presenting a clear "pros and cons" list that critically highlights **APC status** and **real-time quota availability**. By integrating data from the NEU Library's static agreement page, the dynamic SciFree search portal, and the Directory of Open Access Journals (DOAJ), the bot provides actionable intelligence. It will further assist researchers by providing an AI-powered reformatting tool to adapt their manuscript to a chosen journal's specific guidelines, thereby saving time, reducing administrative burden, and maximizing the value of the university's OA investments.

---

### 🚀 Step-by-Step Project Plan

Here is a high-level, 10-step plan to build, and deploy the bot.

**Phase 1: Foundation & Data Ingestion (Weeks 1-3)**
1.  **Define Database Schema:** Architect the "Master Journal" database. Key fields will include `Journal_Name`, `Publisher`, `ISSN`, `NEU_APC_Status` (e.g., Covered, Capped, Discounted), `NEU_Quota_Remaining`, `Is_DOAJ_Free` (True/False), and `Guidelines_URL`.
2.  **Build Data Ingestors (Scrapers/API):** Develop the scripts to automatically fetch data from the three primary sources. (This is detailed in the next section).
3.  **Create Aggregation Logic:** Write the core backend script that runs on a schedule (e.g., daily). This script will execute the ingestors, clean the data, and update the Master Journal database, making sure to cross-reference publisher quotas (e.g., "Elsevier") with the specific journals covered by that publisher.

**Phase 2: Backend & Core Logic (Weeks 4-6)**
4.  **Develop the Draft Analysis API:** Create an endpoint that accepts an uploaded text file (`.pdf`, `.docx`). This API will parse the text and use a natural language processing (NLP) model (e.g., TF-IDF or an LLM) to extract key topics and keywords.
5.  **Build the Suggestion Engine API:** Create a second endpoint that takes the keywords from the analysis API and queries the Master Journal database. Its logic will rank suggestions, prioritizing `Fully Covered (No-Cost)` and `Fully Covered (Capped)` journals that are a good topical match.

**Phase 3: Frontend & User Interface (Weeks 7-9)**
6.  **Design the User Interface (UI):** Create a simple, clean web interface with three parts:
    * A page to upload a draft.
    * A results page to display the "pros and cons" list of suggested journals.
    * A chat interface for the "Reformatting Assistant."
7.  **Develop the Frontend:** Build the UI and connect it to the backend APIs. The results page must clearly and visually emphasize the quota status (e.g., using red/yellow/green indicators).

**Phase 4: AI Integration & Deployment (Weeks 10-12)**
8.  **Integrate the Reformatting Assistant:** Connect the chat UI to a large language model (LLM) API (like the Gemini API). The backend will need to fetch the full text from the journal's `Guidelines_URL` and "prime" the AI with it as context, so its answers are accurate.
9.  **Beta Testing & Feedback:** Deploy the tool to a staging server. Ask a few NEU librarians and researchers to test it with real drafts, focusing on the accuracy of the suggestions and quota data.
10. **Final Deployment:** Refine the tool based on feedback and launch it for the NEU community.

---

### 📊 Step-by-Step: How to Find Journals & Meta-Info

This is the detailed plan for **Step 2** of the project. Your bot will need to combine four distinct data-sourcing methods to build its "Master Journal Profile."

**1. Source 1: The "Quota Data" (NEU Library Page)**
This source provides the *most critical* and time-sensitive information: the APC quotas.

* **How to get it:** This will be a targeted **web scraper** (e.g., using Python with libraries like `BeautifulSoup` or `Playwright`).
* **Step 1:** Write a script that loads the NEU Library Open Access page you provided.
* **Step 2:** Program the script to look for specific publisher names (e.g., "American Chemical Society:", "Elsevier:", "SpringerNature:").
* **Step 3:** Extract the text immediately following these names to find the quota numbers (e.g., "32 remaining", "14 remaining", "13 remaining", "0 remaining").
* **Step 4:** Store this in a simple "Publisher Quota" table in your database. This scraper *must* run daily, as this data changes frequently.

**2. Source 2: The "NEU Coverage" Data (SciFree Portal)**
This source confirms if a *specific journal* is part of *any* NEU agreement.

* **How to get it:** This requires reverse-engineering the SciFree search tool.
* **Step 1 (Ideal):** Check the browser's "Network" tab while using the SciFree search to see if it calls a hidden **API**. If it does, your bot can call this API directly (the best-case scenario).
* **Step 2 (Likely):** If there's no API, your bot will have to **scrape the search results**. When the bot needs to check a journal (e.g., "Nature Communications"), it will query the SciFree URL (`https://search.scifree.se/northeastern?query=Nature+Communications`) and parse the resulting HTML to see if it's listed as "Covered."

**3. Source 3: The "Truly Free" Journals (DOAJ)**
This source provides the list of high-quality journals that charge *no fees to anyone*.

* **How to get it:** The **DOAJ API** or their downloadable **CSV file**. This is the easiest and most reliable data source.
* **Step 1:** Go to `doaj.org/docs/api/` (for the API) or `doaj.org/docs/csv/` (for the file).
* **Step 2:** Fetch the data for all journals.
* **Step 3:** Filter this massive list to find only the journals where the "APC" column is "No" or "0". This is your baseline list of "Always Free" journals.

**4. Source 4: The "Journal Meta-Info" (Publisher Websites)**
This source provides the specific author guidelines (e.g., abstract length, reference style) needed for the AI reformatting assistant.

* **How to get it:** A **generic web scraper**. This is the most complex scraper.
* **Step 1:** For each journal in your database, get its homepage URL (this is usually included in the DOAJ data).
* **Step 2:** Write a scraper that visits that homepage and searches for HTML links (`<a>` tags) containing keywords like "Author Guidelines," "Information for Authors," "Submit Your Paper," or "For Authors."
* **Step 3:** Store the *URL* of this guideline page in your database's `Guidelines_URL` column.
* **Step 4:** The bot will fetch the *full text* from this URL *only* when a user selects that journal for reformatting, passing it to the LLM as context.

### **How to Find the Journal List for Your Bot**

You cannot find a single "list" of no-cost journals. Instead, your bot will need to get its data from **three different sources** to provide an accurate answer.

1.  **The "Open Access Agreements" Page (The text you provided):** This is your most important source. It lists all publishers that have special "fully covered" or "discounted" APC (Article Processing Charge) deals with Northeastern.
    * **Why it's key:** It contains the **critical "pros and cons" data**, specifically the *limits (quotas)* on free articles. For example, it states that for 2025, Taylor & Francis and Wiley have **0 remaining** APCs, while Elsevier has **14** and Springer has **13**. Your bot *must* use this info.
2.  **The SciFree Search Portal (`https://search.scifree.se/northeastern`):** This is not a list you can download; it's a *search tool*. The library directs authors to use it to *confirm* if a specific journal is part of an agreement.
    * **How your bot must use it:** Your bot will need to query this tool (ideally via an API, if one exists) or scrape its search results to check the status of journals it suggests.
3.  **The "Guide to Choosing a Publication Venue" (and the DOAJ):** The library pages also point to a "Guide to Choosing a Publication Venue." This guide contains the third type of no-cost journal: journals that are free for *everyone*, not just NEU authors (they don't charge any APCs).
    * **Why it's key:** This list is often managed by the **Directory of Open Access Journals (DOAJ)**. Your bot should pull this list of journals as the "truly free" (non-quota) options.

---

Here is a formal requirements document for your project.

### **Requirements Document: NEU Open Access Journal Bot**

**1.0 Introduction**

This document outlines the requirements for the **NEU Open Access Journal Bot**, an automated tool designed to assist Northeastern University (NEU) researchers in identifying no-cost publishing venues and preparing their manuscripts for submission.

The bot's primary functions are:
1.  To identify journals where NEU authors can publish at no cost, based on library agreements and other open-access models.
2.  To analyze a researcher's paper draft and suggest suitable journals.
3.  To present the "pros and cons" of each suggestion (e.g., APC quotas, discounts).
4.  To provide AI-powered assistance for reformatting the manuscript to a chosen journal's specifications.

**2.0 Core Features & User Stories**

* **As a researcher,** I want to know which journals I can publish in for free *before* I start writing, so I can target the right publication.
* **As a researcher,** I want to upload my completed draft so the bot can analyze its topic and suggest relevant, no-cost journals.
* **As a researcher,** I want to see the pros and cons of each suggested journal, *especially* if the "free" publishing is a limited-time offer or has a quota, so I don't waste my time.
* **As a researcher,** once I pick a journal, I want the bot to help me reformat my paper (abstract length, reference style, etc.) to match its guidelines, so I can submit it faster.

**3.0 Functional Requirements (FR)**

**FR-01: Journal Database**
The system shall maintain a central database of journals populated and continuously updated from three primary data sources (see Section 4.0).

**FR-02: APC Status Logic**
The system must categorize each journal into one of the following statuses for NEU authors:
* **Fully Covered (No-Cost):** Journals from publishers with a full "read-and-publish" agreement (e.g., ACM, ASME, Cambridge University Press).
* **Fully Covered (Capped):** Journals from publishers with a limited quota. The system *must* track and display the current number of remaining APCs for the year (e.g., ACS, Elsevier, Springer).
* **Discounted:** Journals where NEU authors receive a discount but must pay a partial APC (e.g., BiomedCentral, MDPI).
* **No-Cost (DOAJ):** Journals that are free for all authors and charge no APCs.
* **Full Price:** All other journals not in an agreement.

**FR-03: Draft Analysis Engine**
* The system shall allow users to upload a paper draft (e.g., `.docx`, `.pdf`, `.txt`).
* The system shall parse the draft's text to extract:
    * Keywords
    * Abstract
    * Key topics and themes
    * (Optional) Cited references, to infer the paper's field.

**FR-04: Journal Suggestion Engine**
* The system shall match the analyzed draft (FR-03) against the Journal Database (FR-01).
* The system shall present a ranked list of suggested journals, prioritizing **Fully Covered (No-Cost)** and **Fully Covered (Capped)** options that are a good topical fit.

**FR-05: Pros and Cons Display**
For each suggested journal, the system *must* display:
* **Journal Name & Publisher**
* **APC Status:** (e.g., "Fully Covered by NEU agreement")
* **Pros:** Scope match, potential for high impact, etc.
* **Cons:**
    * **Quota Status:** (e.g., "CRITICAL: Only **13** free APCs remain for 2025" or "EXHAUSTED: **0** free APCs remain for 2025").
    * Topical mismatch, niche audience, etc.

**FR-06: Reformatting Assistant**
* After a user selects a journal, the system shall fetch that journal's "Author Guidelines" (e.g., by scraping the URL or from a pre-fed database).
* The system shall provide an AI-powered chat interface where the user can:
    * Ask questions ("What is the word limit for the abstract?").
    * Paste their text (e.g., abstract, references) and ask the bot to reformat it to the journal's style.

**4.0 Data Sources**

This section details the data required to power the bot's database (FR-01).

| Data Source | Type | Content Provided | Update Frequency |
| :--- | :--- | :--- | :--- |
| **1. NEU Library Open Access Page** | Static Text (from user) | The primary list of publisher agreements (e.g., ACS, Wiley, Elsevier) and, most importantly, the **APC quotas and remaining numbers** for the current year. | Monthly (or as needed) to check for quota updates. |
| **2. SciFree Search Portal** | Dynamic Search Tool | A "live" source of truth for confirming if a *specific journal* is covered by an NEU agreement. | Real-time (via API query or scrape). |
| **3. Directory of Open Access Journals (DOAJ)** | External Database | A comprehensive list of vetted, high-quality "No-Cost (DOAJ)" journals that do not charge authors any fees. | Quarterly (to pull the latest DOAJ list). |


**5.0 Non-Functional Requirements**

* **NFR-01: Data Currency:** The APC quota information (from Data Source 1) is time-sensitive and *must* be the most accurate information available. The system must have a process for a human administrator to update these quota numbers as they change.
* **NFR-02: Security & Privacy:** User-uploaded paper drafts are confidential intellectual property. The system must process them in a secure environment and delete them immediately after the user's session ends.
* **NFR-03: Usability:** The interface must be simple, clear, and fast for researchers who are not technical experts.
