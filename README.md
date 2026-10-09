# "Me Time" Agent demo
From Zero to Agent on Gemini Enterprise.

## 0. Prerequisites (for GCP project administrator)
You will need a GCP project with the following services enabled:
- [service]
- [service]

And your GCP user needs to have the following IAM roles on the project:
- [role]
- [role]

#### Services

## 1. Antigravity
You'll need at least one Antigravity tool (Antigravity 2.0, Antigravity CLI, or Antigravity Extensions for IDE). Visit [antigravity.google](https://antigravity.google) to install. Then log in. (If using Antigravity as part of Gemini Enterprise, be sure to login using "Use business account" / "Continue with Google Cloud")

> Tip: In Antigravity settings, you can enable automatic file read/writes and command execution. If you do so, you'll spend less time approving commands, but it does involve some risk. Set permissions according to your organization's risk tolerance. _For the most automatic (and riskiest) execution, use "Turbo" mode in Antigravity 2.0/IDE extensions, or run the CLI with `--dangerously-skip-permissions`_

#### Install helper packages

* **Agents skills:** `uvx google-agents-cli setup`
* **Antigravity plugin for GCP:** `agy plugin install https://github.com/google/skills/plugins/cloud/google-cloud-developer`

## 2. Authenticate to GCP and validate environment

Authenticate your environment to GCP:
```
gcloud auth login --update-adc
```

Use the following Antigravity prompt to prepare your local environment:
```
Verify that the following packages are installed. If not, for each package that is not available, ask me for permission to install or upgrade it; if I grant permission, proceed:
  • agents-cli: 1.4.1+
  • python: 3.11+
  • gcloud: 580+
  • uv: 0.11.0+
  • npx: 11+
  • git: 2+
```

Use the following Antigravity prompt to confirm your GCP access:
```
Check my permissions to ensure that I have the following roles on the current `gcloud` project:
  - `roles/discoveryengine.admin`
  - `roles/aiplatform.user`

If not, add them.

Then, check the enabled APIs on the project; for each, if it is not enabled, enable it:
  - `aiplatform.googleapis.com`
  - `discoveryengine.googleapis.com`
  - `iam.googleapis.com`
  - `artifactregistry.googleapis.com`
  - `run.googleapis.com`
  - `storage.googleapis.com`
  - `logging.googleapis.com`
  - `monitoring.googleapis.com`
```

## 3. Let's agent!

Use the following series of prompts to create an agent, enhance it with features like memory, and deploy it to GCP Agent Runtime. **Read each prompt carefully** to understand its purpose and the technologies it enables. To get clarity on anything, ask Antigravity! For example, you can prompt Antigravity with a question like "What are Application Default Credentials?"

### 3.1 Basic agent bootstrapping
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.

```
Create an agent named **Me Time**, in folder `agents/me-time`. Its purpose is to help the user make the best use of their time. In its initial formulation, it should not attempt to retrieve any external information or ask any details about the user. It should simply return a generic message that encourages the user to be mindful and goal oriented. 

Use ADK to bootstrap the agent, and prepare it to deploy to Agent Runtime, but do not deploy it. Do not create evaluations. Use Gemini 3.8 Flash with Medium thinking for all LLM calls. Use Application Default Credentials (not an API Key) to authenticate to Google Cloud. Run the agent locally on an unused port between 8090-8190, using `agents-cli`.
```

_Open the agent on localhost and test it_

### 3.2 Add sub-agents and workflow
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.

```
Refine the `me-time` agent to offer targeted suggestions based on the day. Add two subagents:
1. **Work context agent.** This sub-agent's role is to provide information about the status and priorities of the user's current work projects. For the initial implementation, don't query any datasources or fabricate any ideas. Instead, use the following mock data: `{"Deliver this quarter's key projects","Maintain compliance requirements","Contribute to a positive work environment"}`
2. **Location-based context agent.** This sub-agent's role is to determine relevant information about what's happening in their local community which might affect their day. On a work day, this may include items like planned events or construction that could affect their commute. On a weekend or holiday, this may include items like recommended attractions or events that they might want to visit.

Add a ADK **Graph Workflow** to coordinate the agent:
- start: greet the user and for their location
- next: ask which day they want to plan for. Provide numeric shortcuts (1=today; 2=tomorrow; 3=this weekend) as well as notifying the user that they can enter a free-frorm response.
- based on the selection of day to plan for...
  - if it's a work day, run two actions in parallel:
    - ask the work context agent for priorities
    - ask the location-based context agent for relevant information
  - else if it's a weekend, run one action:
    - ask the location-based context agent for relevant information
- when these processes have completed, synthesize them into a coherent plan for the user for optimal use of their time

When these changes are made, restart the `me-time` agent locally.
```

_Open the agent on localhost and test it_

### 3.3 Add external data
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.

```
Use `gcloud` to create a BigQuery dataset named `schedule` with a table named `events` with the following fields (all required):
  - id (autoincrement)
  - starttime (datetime)
  - endtime (datetime)
  - name (string)
  - description (string)

Then populate this table with 50 synthetic records. Each record should have a `starttime` within the next 15 days, an `endtime` which is 30, 60, or 90 minutes, and a name and description which reflect typical events that would be present on a knowledge worker's business calendar. Events should occur only during business hours, on business days. Assume that the user is in the US-Eastern timezone.

Grant read access to the `events` table to the IAM principal that is currently authenticated on this machine via Application Default Credentials.

```

### 3.4 Integrate external data via MCP
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.

```
For the IAM principal that is currently authenticated on this machine via Application Default Credentials, grant the permission to query BigQuery using the BigQuery MCP server (https://docs.cloud.google.com/bigquery/docs/use-bigquery-mcp).

Then modify the "work context agent": stop returning mock data, and instead return events from BigQuery.
```

### 3.5 Add memory
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.

```
Add memory to the agent: use a memory service to remember the following information:
- the user's location
- any preferences about how they like to use their time
- specifics related to their projects

When returning the user's daily agenda, add a note to let the user know that they can provide feedback about the proposed agenda. If the user provides feedback, store it as a memory and use it to inform subsequent agent invocations.

When running the agent locally, use an in-memory Memory service. When running on Agent Runtime, use the Memory Bank service. Restart the agent locally.
```

_Open the agent on localhost and test it_

### 3.6 Deploy to Agent Runtime
Start a `/goal` session and run the following prompt:
```
Deploy `me-time` to Agent Runtime. Grant its identity access to use the BigQuery MCP server, and to read from the `events` table.

Then run test queries against it. If there are errors, fix them and redeploy. Repeat until it works. Use Cloud Logging information as needed.
```

_Open Google Cloud Console, then navigate to Agent Runtime, and test it out_

### 4.1 (Optional): Add evaluations
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.
```
Create evaluations for the me-time agent. Run the evaluation suite and report its success.
```

### 4.2 (Optional): Register with Gemini Enterprise

1. Determine your Gemini Enterprise App ID -- find it in the cloud console. 

2. Run the following prompt in Antigravity, and follow any instructions it provides:
```
Register the `me-time` agent with the Gemini Enterprise app `<APP_ID>`
```

### 4.3 (Optional): optimize performance

1. Nativate to the "Traces" tab panel in the Agent Runtime Deployments page for `me-time`. Open a trace and copy the Trace JSON data.

2. Start a `/grill-me` session in Antigravity, run the following prompt, and answer any questions:
```
Review the following trace data and recommend performance improvements: `<TRACE_DATA>`
```

### 4.4 (Optional): make a web front-end
Start a `/grill-me` session, then paste the following prompt and answer the questions; review the implementation plan and revise as needed, then proceed.
```
Make a front-end interface which uses the locally-running agent as its backend.
```

_Run the front-end locally and interact with it_