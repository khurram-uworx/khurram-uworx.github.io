---
title: "Systematic Poker"
date: 2025-07-12
tags:
- ML
- "Scrum"
comments: true
---

Lets build a high-level strategy outlining a systematic process for Jira Story Point Prediction. This approach focuses on defining and refining the workflow rather than jumping into the implementation details.

---

Below is a final review of the approach with additional considerations to ensure nothing is missed while keeping it easily implementable:

---

### Final Comprehensive Review

* **Structured Data Collection:**

  * **Ticket Details:** Ensure every ticket captures the basics (title, description, reporter).
  * **Enrichment During Refinement:**

    * **Metadata:** Ticket type, priority, labels, components, and any custom fields.
    * **Associations:** Clear links to epics/parent tickets that allow aggregation of historical trends.
    * **Spike Tickets:** Option to link separate spike/investigation tickets, or include detailed sub-tasks or elaborative comments within the main ticket.

* **Contextual Data Integration:**

  * **Historical Performance:** Incorporate data on past tickets—both within the same epic and similar tasks—to capture trends and productivity metrics (including developer performance on similar components).
  * **Comment Analysis:**

    * **Role Identification:** Capture who is commenting (e.g., developer, QA, product owner).
    * **Sentiment & Content Categorization:** Tag the types of comments (e.g., clarifications, risk flags, technical concerns) to quantify uncertainty or complexity.

* **Data Quality & Standardization:**

  * **Controlled Vocabulary:** Enforce consistent naming for labels, components, and epic/parent associations.
  * **Validation Rules:** Implement automated checks during the refinement stage to ensure completeness and standardization of inputs.

* **Feature Engineering for ML:**

  * **Comprehensive Feature Set:**

    * Textual features from descriptions and comments.
    * Categorical features from ticket metadata.
    * Relationship-based features (e.g., aggregate measures from linked epics, spike ticket insights).
    * Developer productivity metrics from historical assignments.
  * **Flexibility:** Allow optional data (like spike details or comment roles) so teams not using these aspects can still participate.

* **Pre-Estimation & Prediction:**

  * **Automated ML Input:** Use the enriched and standardized data to generate initial story point predictions via ML models (e.g., ML.NET or TorchSharp).
  * **Feedback Loop:** Present predictions alongside relevant historical context for review, allowing teams to provide corrections or further enrich the data.

* **Post-Completion Analysis & Continuous Improvement:**

  * **Outcome Comparison:** Regularly compare predicted vs. actual effort and iterate on feature engineering.
  * **Model Refinement:** Use discrepancies and team feedback to continually retrain and improve the prediction model.

* **Integration & Practicality:**

  * **Toolchain Compatibility:** Ensure the workflow can integrate smoothly with existing tools (e.g., version control, CI/CD, time tracking) for a comprehensive dataset.
  * **Iterative Implementation:** The approach is modular—start with core fields and basic refinement, then progressively add associations, comment analysis, and historical aggregations.
  * **Flexibility & Scalability:** Designed to adapt to various team practices without enforcing a one-size-fits-all model, while still capturing essential features for ML prediction.

---

This strategy confirms that the approach is comprehensive yet flexible, capturing both the structured and contextual aspects of tickets. It is designed to be implemented in stages, ensuring teams can gradually adopt the process without overwhelming changes.

---

Below is a proposed MVP iteration plan, focusing on the most critical elements to get a working prototype in place:

1. **Essential Data Capture:**

   * **Ticket Core Data:** Ensure every ticket has a clear title and description.
   * **Structured Metadata:** Enforce fields for ticket type, priority, labels, and components.
   * **Epic/Parent Association:** Capture the linkage to an epic or parent ticket to enable basic historical aggregation.

2. **Data Quality & Standardization:**

   * Implement automated validations during the refinement stage to ensure consistency (e.g., controlled vocabularies for labels and priority scales).

3. **Feature Extraction:**

   * **Textual Features:** Extract key phrases and terms from the title and description.
   * **Categorical Data:** Encode ticket type, priority, labels, and components.
   * **Contextual Data (Optional):** If available, aggregate simple metrics from linked epic/parent tickets.

4. **Basic ML Model:**

   * Use a simple regression or classification model (with ML.NET, TorchSharp, or a combination) trained on historical ticket data to predict story points.
   * Focus on a minimal set of features without delving into complex comment or spike ticket analysis at this stage.

5. **Feedback Loop:**

   * Present the model’s predictions alongside the relevant ticket context for team review.
   * Collect feedback and compare predicted versus actual story points to iteratively improve data quality and model performance.

This MVP approach is designed to be easily implementable by focusing on core ticket attributes and a straightforward ML pipeline, with room for enhancement in future iterations.

---

### **Next Action Items for MVP Implementation**

#### **1. Define Data Schema & Standardization Rules**

* Identify and finalize **mandatory fields** (ticket type, priority, labels, components, epic/parent, etc.).
* Establish **controlled vocabularies** for categorical fields to maintain consistency.
* Ensure **epic/parent ticket linkage** is properly structured.

#### **2. Collect & Clean Historical Data**

* Extract past Jira tickets along with their **story points, metadata, and epic associations**.
* Clean and preprocess data by handling missing values and ensuring **standardized labeling**.
* If feasible, prepare an initial dataset to validate feature extraction.

#### **3. Implement Basic Feature Extraction**

* Convert **ticket metadata** into a structured format for ML.
* Apply **text processing** on descriptions/titles for potential keywords.
* Aggregate **basic historical context** (e.g., average story points of past tickets in the same epic).

#### **4. Train a Baseline ML Model**

* Use **ML.NET or TorchSharp** to build an initial story point prediction model.
* Start with a **simple regression or classification model** using structured metadata.
* Validate performance using historical ticket data and compare against actual story points.

#### **5. Integrate Prediction in the Workflow**

* Create an interface to **display model-generated estimates** before manual estimation.
* Allow teams to **review, adjust, and provide feedback** on predictions.

#### **6. Plan for Iterative Improvements**

* Based on MVP results, define next steps, such as:

  * Enhancing historical context with **developer productivity insights**.
  * Exploring **comment analysis** for additional signals.
  * Linking and analyzing **spike tickets** in future iterations.
