---
title: "Smart Sprint Analyzer"
date: 2025-07-10
tags:
- "ML"
- Scrum
comments: true
---

That's a compelling and relatable idea—especially for engineers familiar with Agile practices and tools like Jira. Here's a refined version of your concept, with a clearer problem definition and TensorFlow relevance, that balances simplicity with educational value:

---

## **Project Title:**

**"Smart Sprint Analyzer: Predict & Detect Anomalies in Team Velocity and Work Distribution"**

---

## **Real-World Problem Statement:**

Development teams often use tools like Jira to track progress, assign tasks, and estimate workloads via story points. However, the actual distribution of work can become skewed due to biases, incorrect estimations, or systemic team behavior.

**Can we build an ML model that learns from historical sprint data to:**

1. **Predict expected story point distribution for upcoming sprints.**
2. **Identify anomalies in how work is distributed across engineers.**

---

## **Why TensorFlow?**

TensorFlow is suitable because:

* It introduces key ML concepts (training, prediction, anomaly detection).
* Provides easy-to-use high-level APIs for building classification or regression models.
* Enables experimentation with basic time series analysis (team velocity over time).
* Gives exposure to tabular data processing pipelines (through `tf.data`, `keras` preprocessing layers).

---

## **Dataset Features (from Jira Sprint exports):**

For each sprint:

* Ticket ID
* Assignee
* Story Points
* Status
* Sprint ID
* Completion Time
* Ticket Type (Bug, Feature, Task, etc.)
* Estimated vs. Actual time (if available)

---

## **Tasks to Build With:**

### 1. **Sprint Distribution Predictor (Regression / Classification):**

* Train a model to predict the expected number of story points per engineer per sprint based on past data.
* TensorFlow use: Simple dense model with tabular data input.

### 2. **Anomaly Detection:**

* Use unsupervised learning (e.g., Autoencoders) or statistical thresholds on predicted vs. actual assignments to flag anomalies.
* TensorFlow use: Autoencoder to detect skewed assignments (e.g., someone consistently under-assigned or over-assigned).

---

## **What They Learn:**

* Data ingestion and preprocessing in TensorFlow.
* Tabular model training using Keras.
* Simple anomaly detection techniques using ML.
* Practical application of AI to improve team performance metrics.
* How to work with real-world data that’s often messy and incomplete.

---

## **Stretch Ideas:**

* Visual dashboard with TensorFlow\.js (optional).
* Clustering similar types of tasks (unsupervised learning).
* Predict sprint success probability based on early ticket updates.

---

Great! Let’s break this down into two parts:

1. **Sample dataset schema (CSV or JSON style)**
2. **TensorFlow model scaffold** (for predicting story point distribution per engineer)

---

## 🧾 Sample Dataset Schema (CSV)

This represents Jira Sprint data you’d typically export or fetch via API:

```csv
ticket_id,sprint_id,assignee,story_points,status,completion_time_days,ticket_type,created_date
TCK-101,Sprint-12,Alice,5,Done,3,Story,2024-04-01
TCK-102,Sprint-12,Bob,3,Done,2,Bug,2024-04-01
TCK-103,Sprint-12,Alice,8,Done,5,Task,2024-04-02
TCK-104,Sprint-12,Charlie,2,Done,1,Story,2024-04-02
TCK-105,Sprint-12,Alice,1,Done,1,Bug,2024-04-03
TCK-106,Sprint-12,Bob,5,Done,3,Task,2024-04-03
```

---

### 📘 Feature Engineering Targets:

* Grouped per `sprint_id` and `assignee`
* Aggregate total `story_points`
* Add total ticket count, average ticket type distribution, etc.

---

## ⚙️ TensorFlow Model Scaffold (Tabular Regression)

Goal: Predict `story_points_assigned` for each `assignee` in a new sprint based on past sprint patterns.

```python
import tensorflow as tf
import pandas as pd

# Load preprocessed sprint-assignee summary (after grouping original data)
data = pd.read_csv('sprint_summary.csv')  # assume columns like assignee, sprint_id, avg_ticket_type_bug, total_tickets, etc.

# Encode categorical columns
assignees = data['assignee'].unique().tolist()
data['assignee_id'] = data['assignee'].apply(lambda x: assignees.index(x))

# Features & label
features = data[['assignee_id', 'total_tickets', 'avg_completion_time', 'avg_ticket_type_bug', 'avg_ticket_type_story']]
labels = data['total_story_points']

# Split
train_ds = tf.data.Dataset.from_tensor_slices((features.values, labels.values))
train_ds = train_ds.shuffle(100).batch(8)

# Model
model = tf.keras.Sequential([
    tf.keras.layers.Input(shape=(features.shape[1],)),
    tf.keras.layers.Dense(32, activation='relu'),
    tf.keras.layers.Dense(16, activation='relu'),
    tf.keras.layers.Dense(1)  # Regression output
])

model.compile(optimizer='adam', loss='mse', metrics=['mae'])
model.fit(train_ds, epochs=10)
```

---

## 📈 Optional Next Steps

* Use predictions to **compare against actuals** → find deltas/skewed assignments.
* Use **Autoencoders** to detect outliers in story point patterns.

Let’s proceed in two steps:

---

## ✅ Step 1: Simulate Sprint-Assignee Summary from Raw Ticket Data

Assume you start with raw Jira-like ticket-level CSV. Here’s how to group it into sprint-level features per assignee.

### 🧪 Sample Code: From Raw to Summary (Using Pandas)

```python
import pandas as pd

# Load raw Jira data
df = pd.read_csv('raw_sprint_data.csv')

# One-hot encode ticket_type
ticket_types = pd.get_dummies(df['ticket_type'], prefix='type')

# Merge encoded ticket types
df = pd.concat([df, ticket_types], axis=1)

# Group by sprint and assignee
summary = df.groupby(['sprint_id', 'assignee']).agg({
    'story_points': 'sum',
    'ticket_id': 'count',
    'completion_time_days': 'mean',
    'type_Bug': 'mean',
    'type_Story': 'mean',
    'type_Task': 'mean'
}).reset_index()

# Rename for clarity
summary.columns = [
    'sprint_id', 'assignee',
    'total_story_points', 'total_tickets', 'avg_completion_time',
    'avg_ticket_type_bug', 'avg_ticket_type_story', 'avg_ticket_type_task'
]

summary.to_csv('sprint_summary.csv', index=False)
```

This gives us the input needed for model training.

---

## ✅ Step 2: Anomaly Detection with Autoencoder

Use this model to learn normal story point distribution patterns per assignee and flag anomalies.

### 🧪 TensorFlow Autoencoder for Anomaly Detection

```python
import tensorflow as tf
import pandas as pd
import numpy as np

# Load preprocessed sprint-assignee summary
data = pd.read_csv('sprint_summary.csv')

# Convert assignee to ID
assignees = data['assignee'].unique().tolist()
data['assignee_id'] = data['assignee'].apply(lambda x: assignees.index(x))

# Features only (no labels needed for autoencoder)
features = data[['assignee_id', 'total_tickets', 'avg_completion_time',
                 'avg_ticket_type_bug', 'avg_ticket_type_story', 'avg_ticket_type_task',
                 'total_story_points']]

# Normalize
from sklearn.preprocessing import StandardScaler
scaler = StandardScaler()
features_scaled = scaler.fit_transform(features)

# Build Autoencoder
input_dim = features_scaled.shape[1]

model = tf.keras.Sequential([
    tf.keras.layers.Input(shape=(input_dim,)),
    tf.keras.layers.Dense(16, activation='relu'),
    tf.keras.layers.Dense(8, activation='relu'),
    tf.keras.layers.Dense(16, activation='relu'),
    tf.keras.layers.Dense(input_dim)
])

model.compile(optimizer='adam', loss='mse')
model.fit(features_scaled, features_scaled, epochs=30, batch_size=8, verbose=0)

# Compute reconstruction error
reconstructed = model.predict(features_scaled)
mse = np.mean(np.square(features_scaled - reconstructed), axis=1)

# Set threshold
threshold = np.percentile(mse, 95)  # top 5% as anomalies
data['anomaly'] = mse > threshold

# Save with anomaly flag
data.to_csv('sprint_summary_with_anomalies.csv', index=False)
```

---

## 🧠 Outcome:

You now have:

* A **prediction model** for estimating workload
* An **unsupervised model** for identifying anomalous sprint distributions

Next steps you can explore:

* Visualizing anomalies with Dash or Streamlit
* Using time series (velocity trends over sprints)
* Adding role or capacity as a feature (e.g. part-time engineer)

Great — let’s generate **synthetic Jira sprint data** and prepare it for use in the ML pipeline we’ve built.

---

## ✅ Step 1: Generate Synthetic Jira Ticket Data

This will simulate realistic sprints with random story points, assignees, ticket types, and durations.

### 🧪 Synthetic Data Generator (Pandas)

```python
import pandas as pd
import numpy as np
from datetime import timedelta, datetime
import random

# Constants
engineers = ['Alice', 'Bob', 'Charlie', 'Dana']
ticket_types = ['Story', 'Bug', 'Task']
sprints = [f'Sprint-{i}' for i in range(1, 11)]
start_date = datetime(2024, 1, 1)

records = []

ticket_id = 100

for sprint in sprints:
    sprint_start = start_date + timedelta(days=14 * (int(sprint.split('-')[1]) - 1))
    for _ in range(random.randint(8, 15)):  # tickets per sprint
        assignee = random.choice(engineers)
        ticket_type = random.choice(ticket_types)
        story_points = random.choice([1, 2, 3, 5, 8])
        completion_time = np.clip(np.random.normal(loc=story_points * 0.8, scale=1.0), 1, 10)
        created_date = sprint_start + timedelta(days=random.randint(0, 3))
        records.append([
            f"TCK-{ticket_id}", sprint, assignee, story_points, "Done",
            round(completion_time, 1), ticket_type, created_date.strftime('%Y-%m-%d')
        ])
        ticket_id += 1

df = pd.DataFrame(records, columns=[
    'ticket_id', 'sprint_id', 'assignee', 'story_points',
    'status', 'completion_time_days', 'ticket_type', 'created_date'
])

df.to_csv('raw_sprint_data.csv', index=False)
```

This creates a CSV you can feed into the sprint summary generator from before.

---

## ✅ Step 2: Build Summary and Run Models

You can now:

* Use the earlier `groupby` script to create `sprint_summary.csv`
* Train the regression model and autoencoder
* Inspect `sprint_summary_with_anomalies.csv`

---

## ✅ Step 3 (Optional): Streamlit Dashboard (to Visualize Anomalies)

If you're interested, I can help you build a **simple Streamlit dashboard** where:

* You upload the `sprint_summary_with_anomalies.csv`
* View sprint and assignee performance
* Highlight anomalies

Perfect! Here’s a **simple but functional Streamlit dashboard** to visualize sprint summaries and highlight anomalies.

---

## ✅ Step 4: Streamlit Dashboard (`app.py`)

```python
import streamlit as st
import pandas as pd
import plotly.express as px

# Load data
uploaded_file = st.file_uploader("Upload Sprint Summary CSV", type=["csv"])

if uploaded_file:
    df = pd.read_csv(uploaded_file)

    st.title("📊 Sprint Analyzer Dashboard")
    
    st.subheader("1. Sprint Overview")
    sprint = st.selectbox("Select Sprint", sorted(df["sprint_id"].unique()))
    sprint_data = df[df["sprint_id"] == sprint]

    fig = px.bar(
        sprint_data, 
        x="assignee", 
        y="total_story_points", 
        color="anomaly",
        color_discrete_map={False: "blue", True: "red"},
        title=f"Story Points per Assignee - {sprint}",
        labels={"total_story_points": "Story Points", "assignee": "Engineer"},
    )
    st.plotly_chart(fig)

    st.subheader("2. Raw Sprint Data")
    st.dataframe(sprint_data)

    st.subheader("3. Anomaly Summary")
    anomalies = df[df["anomaly"] == True]
    st.metric("Anomalies Detected", anomalies.shape[0])
    st.dataframe(anomalies)

    st.download_button("📥 Download Anomaly Report", anomalies.to_csv(index=False), file_name="anomalies.csv")
```

---

## 🛠 Setup Instructions

1. Install required packages:

```bash
pip install streamlit plotly pandas
```

2. Run the app:

```bash
streamlit run app.py
```

3. Upload the `sprint_summary_with_anomalies.csv` file.

---

## 📌 What It Shows:

* Story points per engineer for a selected sprint
* Anomalies highlighted in red
* Filtered view of raw data
* Quick download button for anomaly report

---

Would you like enhancements like:

* Time series chart of total story points per engineer across sprints?
* Correlation heatmap?
* Predict next sprint's distribution from user input?

Awesome — you've just designed a **hands-on, real-world ML project** that's:

* ✅ Relevant to young engineers
* ✅ Teaches core ML/AI principles
* ✅ Uses real team data
* ✅ Has a visual dashboard for impact
* ✅ Encourages ethical & intelligent workload balancing

---

### 🔄 Summary of What You Have:

| Component                           | Purpose                                         |
| ----------------------------------- | ----------------------------------------------- |
| `raw_sprint_data.csv`               | Simulated Jira-style ticket data                |
| `sprint_summary.csv`                | Aggregated assignee-sprint features             |
| TensorFlow Regression Model         | Predict expected story point distribution       |
| Autoencoder Model                   | Detect skewed workload patterns                 |
| `sprint_summary_with_anomalies.csv` | Flagged anomalies in team performance           |
| Streamlit App (`app.py`)            | Visual dashboard with charts, filtering, export |

---

### 🔮 Optional Enhancements

Here are 3 juicy additions you can layer in:

1. **📈 Time Series Velocity Chart (Per Engineer)**
   Show a line chart of each engineer’s total story points over time (sprint\_id on x-axis).

2. **🧠 Interactive Prediction Tool**
   Let the user input future sprint features and get predicted story point expectations using the trained model.

3. **📊 Add Team Insights Dashboard**

   * Average ticket type distribution
   * Sprint success prediction
   * Workload fairness scores

---

Exactly — and that's a much sharper, more practical framing. Instead of passively predicting the future, you're enabling the **Scrum Master** to **actively identify and fix skewed work distribution**.

---

## 🎯 Reframed Objective:

> **“Build a system that helps Scrum Masters detect and correct imbalances in sprint planning — using ML to flag unhealthy patterns and suggest better workload normalization.”**

---

## ✅ What Changes With This Shift?

### 1. **No need to predict future assignments**

We’re not forecasting who *will* get what — instead, we’re measuring what *should not* happen again.

---

### 2. **Anomaly detection becomes the core**

Autoencoder or statistical ML highlights “unhealthy” workload splits (e.g. one person always getting 2x the points).

---

### 3. **Suggest normalization**

You can derive a **"suggested redistribution"** based on past average workloads per person.

---

## 🧠 NEW FEATURE: Suggested Balanced Distribution

### Step-by-step idea:

1. Compute team average story points per engineer per sprint.
2. When an anomaly is found:

   * Suggest how points could be re-assigned more evenly.
   * Recommend limiting per-sprint max per engineer to 120% of their rolling average.

---

### 📊 Streamlit UI Upgrade

* 🔴 Highlight overworked engineers
* 🟢 Highlight underloaded engineers
* 🔄 Show a **“Suggested Balanced View”** (editable table)

---

### 🧪 Code Concept (Balancing Function)

```python
def suggest_balanced_distribution(sprint_df):
    team_total = sprint_df["total_story_points"].sum()
    avg_per_person = team_total / len(sprint_df)
    
    sprint_df["suggested_story_points"] = avg_per_person.round(1)
    sprint_df["delta"] = (sprint_df["total_story_points"] - sprint_df["suggested_story_points"]).round(1)
    
    return sprint_df
```

This can be visualized alongside actual assignment and anomalies.

---

## ✅ Deliverables Scrum Masters Would Love

| Tool                    | Purpose                                           |
| ----------------------- | ------------------------------------------------- |
| 📈 Anomaly report       | Flags skewed assignments                          |
| 📊 Balanced suggestions | Provides concrete fix proposals                   |
| 💬 Sprint insights      | Shows which engineers are consistently overloaded |
| ✅ Download plan         | New proposed sprint assignment CSV                |

---

Awesome — let’s add the **suggested story point rebalancing** to your Streamlit dashboard. Below is a drop-in replacement for your `app.py` that:

* Loads the anomaly-tagged sprint summary
* Visualizes story point distribution
* Flags anomalies
* Calculates and shows **suggested balanced assignments**
* Highlights over/under allocations and deltas
* Offers a CSV download of the suggested plan

---

## ✅ Updated `app.py` with Balanced Distribution Suggestion

```python
import streamlit as st
import pandas as pd
import plotly.express as px

st.set_page_config(page_title="Sprint Analyzer", layout="wide")

# Load data
uploaded_file = st.file_uploader("Upload Sprint Summary CSV", type=["csv"])

if uploaded_file:
    df = pd.read_csv(uploaded_file)

    st.title("📊 Sprint Analyzer & Balancer")

    st.subheader("1. Select a Sprint")
    sprint = st.selectbox("Choose Sprint", sorted(df["sprint_id"].unique()))
    sprint_data = df[df["sprint_id"] == sprint].copy()

    # Suggest balanced distribution
    def suggest_balanced_distribution(sprint_df):
        team_total = sprint_df["total_story_points"].sum()
        avg_per_person = team_total / len(sprint_df)
        sprint_df["suggested_story_points"] = avg_per_person.round(1)
        sprint_df["delta"] = (sprint_df["total_story_points"] - sprint_df["suggested_story_points"]).round(1)
        return sprint_df

    balanced_df = suggest_balanced_distribution(sprint_data)

    st.subheader("2. Actual vs Balanced Story Points")

    fig = px.bar(
        balanced_df.melt(id_vars=["assignee"], value_vars=["total_story_points", "suggested_story_points"]),
        x="assignee", y="value", color="variable",
        barmode="group", title="Actual vs Suggested Story Points"
    )
    st.plotly_chart(fig, use_container_width=True)

    st.subheader("3. Detailed Sprint Table")
    st.dataframe(balanced_df.style
        .applymap(lambda v: "color: red;" if abs(v) >= 5 else "", subset=["delta"])
        .format({"total_story_points": "{:.1f}", "suggested_story_points": "{:.1f}", "delta": "{:+.1f}"})
    )

    st.subheader("4. Anomalies in Sprint")
    anomalies = balanced_df[balanced_df["anomaly"] == True]
    st.write(f"🔍 **{len(anomalies)} anomaly{'ies' if len(anomalies)!=1 else ''} detected**")
    st.dataframe(anomalies)

    # Download option
    download_cols = ["sprint_id", "assignee", "total_story_points", "suggested_story_points", "delta", "anomaly"]
    st.download_button("📥 Download Balanced Plan", balanced_df[download_cols].to_csv(index=False), file_name="balanced_sprint_plan.csv")
```

---

## ✅ Key Features Added:

* 📊 **Side-by-side bar chart**: Shows actual vs. suggested story points per engineer
* 🧮 **Auto-balancing**: Normalizes work assignment within a sprint
* ⚠️ **Delta column**: Highlights who’s over/under-assigned
* ⬇️ **Downloadable plan**: For further action outside the dashboard

---

## 🔄 Optional Next Enhancements

Would you like to:

1. Adjust for **historical average per engineer** instead of sprint-only average?
2. Set **manual limits** on max story points per person?
3. Use a **fairness score** or Gini index to quantify inequality?

That’s an *excellent* question — you’ve just hit on one of the core challenges of real-world workload planning:

> **Fair ≠ Equal** — because people have different capacities and contexts.

So let’s tackle both cases practically and upgrade our system to support **"personal capacity awareness."**

---

## 🎯 Core Insight:

We need to shift from “equal distribution” to **“fair distribution relative to each person’s capacity.”**

Let’s go case by case:

---

### 💪 Case 1: Person A Feels Underutilized with 10 SP

**Possible causes**:

* They’re high-capacity or experienced
* They’ve historically handled 13+ SP with no drop in delivery or quality

**What to do**:

* Track **historical average SP per person**
* Compare current assignment to *their own historical average*
* Flag if they're being underutilized (not just underloaded)

👉 **Update our tool** to include a column:
`personal_avg_story_points` — rolling mean over past N sprints.

Then show a `utilization_ratio = total_story_points / personal_avg_story_points`

---

### 🧠 Case 2: Person B Consistently Struggles with >8 SP

**Possible causes**:

* New team member
* Technical ramp-up, context-switching overhead
* Maybe involved in non-ticketed work (meetings, support, onboarding, etc.)

**What to do**:

* Learn their *delivery ceiling* from past sprints
* Encourage assigning \~90% of their known max, not an arbitrary average
* Flag when their sprint SP is > their typical delivery comfort zone

👉 Add a flag:
`over_capacity = total_story_points > (personal_avg_story_points + tolerance)`

---

## ✅ System Upgrade Plan

| Field                       | Description                               |
| --------------------------- | ----------------------------------------- |
| `personal_avg_story_points` | Rolling mean of story points per engineer |
| `utilization_ratio`         | This sprint vs. personal average          |
| `over_capacity`             | Boolean if current > historical + buffer  |
| `underutilized`             | Boolean if current < historical - buffer  |

---

## 🧪 Example Calculation (Per Person)

```python
def enrich_with_capacity_flags(df, history_df, buffer=2.0):
    # Get rolling personal averages from history
    personal_avg = history_df.groupby("assignee")["total_story_points"].mean().to_dict()
    
    df["personal_avg_story_points"] = df["assignee"].map(personal_avg)
    df["utilization_ratio"] = (df["total_story_points"] / df["personal_avg_story_points"]).round(2)
    df["over_capacity"] = df["total_story_points"] > (df["personal_avg_story_points"] + buffer)
    df["underutilized"] = df["total_story_points"] < (df["personal_avg_story_points"] - buffer)
    
    return df
```

This lets you:

* Adjust suggested workload based on **personal norms**
* Avoid blanket equal assignment
* Empower the Scrum Master to make informed, fair decisions

---

## ✅ UI Suggestions

Add to your Streamlit:

* 🔶 Badge for `underutilized`
* 🔴 Badge for `over_capacity`
* Hover tooltips to explain these
* Sorting/filtering by these columns

---

## 🧠 Final Thought

This enhancement pushes your tool from just “anomaly detection” to a **capacity-aware, intelligent sprint planner** — something very few orgs do well.

Would you like help adding this into your current codebase next (e.g. computing historical norms from past sprints and flagging these cases)?
