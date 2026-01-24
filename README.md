# Valora AI 

**Valora AI** is an Automated Feature Prioritization Engine that predicts the business ROI of product features and helps teams objectively decide what to build next.

> Build what matters. Every sprint.

---

## Problem
Product roadmaps are often driven by opinion, hierarchy, or intuition rather than evidence. This leads to:
- Low-impact features being prioritized
- Wasted engineering effort
- Poor alignment between business goals and delivery
- Lack of measurable ROI from sprints

There is no continuous, data-driven system that learns from outcomes and improves prioritization over time.

---

## Solution
Valora AI uses machine learning on real product and engineering data to:
- Predict ROI for each feature or epic
- Rank backlog items objectively
- Recommend what should be built next
- Create a closed feedback loop between delivery and impact

It acts as an intelligent prioritization layer on top of existing tools like Jira and GitHub.

---

## Key Features
- 🔗 Jira API integration for issues and epics
- 📊 Feature ranking dashboard
- 🤖 ML-based ROI prediction engine
- 🔁 Continuous learning from delivery outcomes
- ⚙️ Modular architecture for multi-platform integrations

---

## Tech Stack
- **Frontend:** React
- **Backend:** FastAPI (Python)
- **Machine Learning:** Scikit-learn
- **Database:** PostgreSQL
- **APIs:** Jira REST API
- **Deployment:** AWS Sagemaker

---

## Architecture Overview
- Data ingestion from Jira/GitHub
- Feature engineering pipeline
- ML model predicts ROI score
- Ranked results exposed via API
- Dashboard visualizes priorities

---
## Diagram
PM ——–> View ranked features PM ——–> Approve AI suggestions System —-> Create Jira epics System —-> Retrain ML model Eng Manager -> Monitor delivery analytics 

