---
published: false
---

### Systematic Refinement Process for Jira Story Point Prediction  
To enable effective ML-driven story point prediction, the refinement process must systematically capture contextual, historical, and team-specific metadata. Below is a structured workflow to standardize inputs while retaining flexibility for real-world engineering practices:

---

#### **1. Ticket Categorization**  
**Objective:** Ensure consistent classification for feature engineering.  
- **Mandatory Fields:**  
  - **Type:** Predefined categories (e.g., `customer bug`, `internal bug`, `improvement`, `new feature`).  
  - **Priority:** Team-specific scale (e.g., `P0-P3`, `Critical-Low`) standardized across projects.  
  - **Labels:** Enforce a shared taxonomy (e.g., `UI`, `API`, `DB`, `security`) with multi-label support.  
  - **Components:** Map to codebase modules (e.g., `checkout-service`, `user-dashboard`).  

**Flexibility:** Allow teams to extend labels/components but enforce backward-compatible naming conventions.  

---

#### **2. Contextual Enrichment**  
**Objective:** Capture task complexity and scope.  
- **Acceptance Criteria:** Require *explicit* success conditions (e.g., "User can save profile changes").  
- **Test Cases:** Attach automated/in-scope test coverage (e.g., "Add Cypress test for checkout flow").  
- **Dependencies:** Link blockers (e.g., "Requires Auth API v2 rollout").  

**Validation:** Automated checks flag incomplete tickets (e.g., missing acceptance criteria).  

---

#### **3. Assignment & Historical Context**  
**Objective:** Link tickets to team/developer expertise.  
- **Assignment Logic:**  
  - **Manual Assignment:** Developers/teams are selected during refinement.  
  - **Auto-Suggested Context:** System surfaces historical data for the assignee(s):  
    - Avg. velocity on similar `labels/components` (e.g., "Dev A: 3 SP avg. on `UI` tasks").  
    - Recent performance trends (e.g., "Dev B: 20% slower on `DB` tickets in Q2").  
- **Fallback Rules:** If unassigned, use team averages for similar tasks.  

---

#### **4. Validation & Completeness Gates**  
**Objective:** Ensure tickets are ML-ready before estimation.  
- **Pre-Estimation Checklist:**  
  - All mandatory fields populated (type, priority, labels, components).  
  - Acceptance criteria/test cases reviewed by the team.  
  - Dependencies resolved or acknowledged.  
- **Exception Handling:** Allow provisional estimation for urgent tickets but flag for model retraining.  

---

#### **5. Post-Completion Feedback Loop**  
**Objective:** Continuously improve prediction accuracy.  
- **Actuals Capture:** Record:  
  - **Story Points Delivered** (if differing from estimate).  
  - **Time Spent** (e.g., calendar days, effort hours).  
  - **Reasons for Variance** (free-text or tags like `scope-creep`, `unforeseen-complexity`).  
- **Historical Updates:** Automatically refresh developer/team performance metrics.  

---

### Key Process Considerations  
1. **Taxonomy Governance:**  
   - How are new labels/components proposed and standardized?  
   - Who arbitrates conflicting definitions (e.g., `frontend` vs. `UI`)?  

2. **Assignment Dynamics:**  
   - How to handle reassignments mid-sprint? (Track reassignment history as a feature.)  
   - How to weight recent vs. older historical performance?  

3. **Team Autonomy vs. Consistency:**  
   - Should priority scales be unified across teams or remain team-specific?  
   - How to normalize story points if teams use different scales (e.g., Fibonacci vs. T-shirt sizes)?  

4. **Cold-Start Scenarios:**  
   - How to estimate tasks with new components/labels without historical data?  
   - Should fallback logic use team averages, cross-team data, or heuristic rules?  

---

### Next Steps for ML Integration  
Once this process is institutionalized, the following data becomes available for modeling:  
- **Features:** Ticket type, labels, components, assignee historical performance, test case count, dependencies.  
- **Target Variable:** Story points (with post-completion actuals for supervised learning).  

This structured workflow ensures the ML model receives consistent, high-quality inputs while respecting team workflows. Would you like to explore how specific process components (e.g., dependency tracking) translate into model features, or discuss governance challenges?