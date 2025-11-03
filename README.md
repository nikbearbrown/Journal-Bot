

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
