---
trigger: always_on
description: Invariants for Vertex AI Gemini model endpoint locations vs. Agent Runtime deployment regions.
---

# Vertex AI & Agent Runtime Region Conventions

When writing agent code, client initializations, or deployment configurations for this project:

1. **Gemini Model Endpoints (`global`):**
   - Always target the `global` region for Gemini model inference.
   - Example:
     ```python
     from google import genai
     client = genai.Client(vertexai=True, location="global")
     ```
   - **Never** use regional endpoints (e.g., `us-east1`, `us-central1`) for Gemini model calls.

2. **Agent Runtime Deployments (Regional):**
   - Agent Runtime deployments are regional resources and must specify a non-global GCP region (e.g., `us-east1`, `us-central1`).
   - Agent Runtime **cannot** be deployed to the `global` region.

3. **Separation of Concerns:**
   - Keep deployment location and model endpoint location distinct.
   - Environment variables like `GOOGLE_CLOUD_LOCATION` representing deployment targets must **not** be used as the model endpoint location.
