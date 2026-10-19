# INSPIRE-MORTALS-DAILY-CONTENT-ENGINE

# INSPIRE MORTALS — DAILY CONTENT ENGINE
## Complete Step-by-Step Build Manual

**Repository:** `inspire-mortals-daily-content-engine`  
**Automation platform:** n8n  
**Data storage:** Google Sheets  
**Original deployment:** n8n Cloud  
**Local deployment:** n8n Community Edition on Windows using Docker Desktop  
**Scheduled execution:** Daily at 5:00 AM  
**Time zone:** `Africa/Lagos`

---

# PART I — UNDERSTANDING THE PROJECT

## Step 1: Define the objective

The project is designed to create motivational content for Inspire Mortals, review that content, and save approved material into a Google Sheets content library.

The intended process is:

1. Start automatically at a scheduled time.
2. Prepare the brand information and content instructions.
3. Generate a draft using an AI model.
4. Parse the AI response into structured fields.
5. Submit the draft for editorial review.
6. Parse the reviewer's decision.
7. Approve the content or send it for revision.
8. Limit repeated revisions.
9. Save approved content into Google Sheets.

The key principle is that **the workflow must not send unapproved content to the approved-content library**.

## Step 2: Define the brand input

The workflow begins with an Edit Fields node containing the brand information.

| Field | Value |
|---|---|
| `brand` | `Inspire Mortals` |
| `purpose` | `STOP SCROLLING-LISTEN-REFLECT-TAKE POSITIVE ACTION` |

These fields provide the content-generation node with its basic instructions.

---

# PART II — APPROACH A: BUILDING THE WORKFLOW IN N8N CLOUD

## Stage 1: Create the workflow

### Step 1.1 — Open n8n

1. Open your browser.
2. Sign in to your n8n Cloud account.
3. Open the Workflows page.
4. Select **Create Workflow**.
5. Name the workflow:

   `Inspire Mortals - Daily Content Engine`

6. Save the workflow.

### Step 1.2 — Understand the workflow canvas

The canvas is where nodes are added and connected.

A node represents an action, such as:

- Preparing fields.
- Calling an AI model.
- Running JavaScript.
- Checking a condition.
- Writing a row to Google Sheets.

A connection between two nodes determines the direction in which data flows.

### Step 1.3 — Add the initial fields

1. Add an **Edit Fields** node.
2. Open its Parameters tab.
3. Choose Manual Mapping.
4. Add a field named `brand`.
5. Set its value to `Inspire Mortals`.
6. Add a field named `purpose`.
7. Set its value to `STOP SCROLLING-LISTEN-REFLECT-TAKE POSITIVE ACTION`.
8. Keep the input fields required by later nodes available when configuring the node.
9. Save the node.

**Expected result:** The node produces one item containing the brand and purpose fields.

---

## Stage 2: Add the first AI content-generation node

### Step 2.1 — Add the node

1. Select the add-node control.
2. Search for **Message a model**.
3. Add the node.
4. Connect the initial Edit Fields node to it.
5. Open the node's Parameters tab.

### Step 2.2 — Configure the model node

The recorded project used:

- Resource: `Text`
- Operation: `Message a Model`
- Credential: `Connect to OpenAI`
- Model displayed in the imported workflow: `GPT-5.6-LUNA`

The node is intended to generate the first content draft.

The model must receive the brand information and the content instructions through its message configuration.

**Important:** The model selection shown in the historical workflow is not proof that the selected model was available through a working API. The node later failed because of an n8n AI Gateway licensing restriction. A working provider and model must be selected before production use.

### Step 2.3 — Verify the expected output

The first AI node should produce a draft containing the information needed by the downstream parser.

Before continuing, confirm the actual response structure returned by the selected model node. Do not assume that every model integration stores its response under the same JSON property.

---

## Stage 3: Prepare the content data

### Step 3.1 — Add Edit Fields1

1. Add another Edit Fields node.
2. Connect the first AI node to it.
3. Open Parameters.
4. Select Manual Mapping if required by the existing configuration.
5. Configure the fields needed for downstream processing.
6. Preserve the AI response and any original input fields required later.
7. Save the node.

The purpose of this stage is to prepare the generated response for parsing and editorial review.

### Step 3.2 — Add the first JavaScript Code node

1. Add a **Code** node.
2. Name it `Code in JavaScript`.
3. Connect Edit Fields1 to it.
4. Open the code editor.
5. Configure the JavaScript to read the actual response from the previous node.
6. Parse the response into the structured fields required by the content library.
7. Save the node.

The original JavaScript source must be preserved from the exported workflow JSON if an exact reconstruction is required. It should not be replaced with guessed code because the parser depends on the AI node's real output format.

### Step 3.3 — Test the parser

Execute the upstream nodes with sample data.

Confirm that the Code node produces the expected fields and does not return malformed JSON.

Do not proceed until the parser output matches the data expected by the reviewer and Google Sheets.

---

## Stage 4: Add the editorial reviewer

### Step 4.1 — Add the reviewer model

1. Add another **Message a model** node.
2. Connect the first JavaScript parser to it.
3. Configure the reviewer instructions.
4. Require the reviewer to assess the content against the project's editorial standards.
5. Require a clearly defined decision, such as `APPROVE` or `REVISE`.
6. Save the node.

The reviewer's role is different from the creator's role: the creator produces content, while the reviewer evaluates it.

### Step 4.2 — Add the review parser

1. Add another Code node.
2. Name it `Code in JavaScript1`.
3. Connect the reviewer node to it.
4. Parse the reviewer's response.
5. Extract the decision and any revision feedback.
6. Preserve the content fields needed by the following branches.
7. Save the node.

### Step 4.3 — Add the approval condition

1. Add an **IF** node.
2. Connect `Code in JavaScript1` to the IF node.
3. Configure the left-hand value as an expression:

   `{{ $json.decision }}`

4. Set the operator to **is equal to**.
5. Enter the comparison value:

   `APPROVE`

6. Save the node.

The true branch represents approval. The false branch represents content that requires further handling.

---

## Stage 5: Configure the approval branch

### Step 5.1 — Add Approved Content Output

1. Add an Edit Fields node.
2. Name it `Approved Content Output`.
3. Connect the true output of the approval IF node to it.
4. Open its Parameters tab.
5. Use Manual Mapping.
6. Keep **Include Other Input Fields** enabled.
7. Ensure the required content fields remain available.
8. Save the node.

### Step 5.2 — Add Edit Fields4

1. Add another Edit Fields node.
2. Name it `Edit Fields4`.
3. Connect Approved Content Output to it.
4. Configure the field:

   - Name: `editorial_status`
   - Value: `APPROVED`

5. Enable **Include Other Input Fields**.
6. Save the node.

This node explicitly marks the content as approved before it reaches the spreadsheet.

---

## Stage 6: Configure Google Sheets

### Step 6.1 — Create the spreadsheet

1. Open Google Sheets.
2. Create a new spreadsheet.
3. Name it:

   `Inspire Mortals - Approved Content Library`

4. Rename the worksheet:

   `Approved Content`

5. Add the following column headers in the first row, preserving the exact spelling:

   `speaker`, `quote`, `quote_source`, `verification_status`, `theme`, `hook`, `voiceover`, `lesson`, `action_challenge`, `closing`, `facebook_caption`, `hashtags`, `onscreen_text`, `thumbnail_concept`, `visual_direction`, `music_direction`, `revision_count`, `editorial_status`.

6. Save the spreadsheet.

### Step 6.2 — Add the Google Sheets node

1. Return to n8n.
2. Add a Google Sheets node.
3. Connect Edit Fields4 to it.
4. Select the Google Sheets credential.
5. Configure:

   - Resource: `Sheet Within Document`
   - Operation: `Append Row`
   - Document: `Inspire Mortals - Approved Content Library`
   - Sheet: `Approved Content`
   - Mapping Column Mode: `Map Each Column Manually`

6. Map each spreadsheet column to the corresponding JSON field.

For example:

| Spreadsheet column | n8n expression |
|---|---|
| `speaker` | `{{ $json.speaker }}` |
| `quote` | `{{ $json.quote }}` |
| `quote_source` | `{{ $json.quote_source }}` |
| `verification_status` | `{{ $json.verification_status }}` |
| `theme` | `{{ $json.theme }}` |
| `hook` | `{{ $json.hook }}` |
| `voiceover` | `{{ $json.voiceover }}` |
| `lesson` | `{{ $json.lesson }}` |
| `action_challenge` | `{{ $json.action_challenge }}` |
| `closing` | `{{ $json.closing }}` |
| `facebook_caption` | `{{ $json.facebook_caption }}` |
| `hashtags` | `{{ $json.hashtags }}` |
| `onscreen_text` | `{{ $json.onscreen_text }}` |
| `thumbnail_concept` | `{{ $json.thumbnail_concept }}` |
| `visual_direction` | `{{ $json.visual_direction }}` |
| `music_direction` | `{{ $json.music_direction }}` |
| `revision_count` | `{{ $json.revision_count }}` |
| `editorial_status` | `{{ $json.editorial_status }}` |

7. Save the node.

The spreadsheet headers and expressions must match. If a field is absent from the incoming data, the expression cannot manufacture its value.

---

## Stage 7: Configure the revision branch

### Step 7.1 — Add Code in JavaScript2

1. Add a Code node named `Code in JavaScript2`.
2. Connect it to the false output of the first approval IF node.
3. Configure the code to track or increment `revision_count`.
4. Preserve the content and reviewer feedback.
5. Save the node.

The exact original JavaScript should be retrieved from the exported workflow JSON if the revision logic must be reproduced exactly.

### Step 7.2 — Add the revision-limit IF node

1. Add another IF node.
2. Name it `If1`.
3. Connect Code in JavaScript2 to it.
4. Configure its condition to check whether `revision_count` has reached the configured maximum.
5. Route content that has exhausted the allowed attempts to manual review.
6. Route content that still has revision attempts available to the revision process.

The recorded workflow included a revision-limit check. Its precise original threshold and code should be verified from the exported workflow rather than guessed.

### Step 7.3 — Add the revision model

1. Add a model node for revision.
2. Connect the branch for content that is still eligible for revision.
3. Pass the original content and reviewer feedback to the model.
4. Ask the model to revise the content in response to the feedback.
5. Connect the revised content back into the editorial-review path.

### Step 7.4 — Add manual review

Connect the branch that has reached the revision limit to a manual-review path.

This prevents the workflow from repeatedly generating revisions without a limit.

---

## Stage 8: Configure the daily schedule

### Step 8.1 — Add Schedule Trigger

1. Add a **Schedule Trigger** node.
2. Open Parameters.
3. Under Trigger Rules, select the daily interval.
4. Set Days Between Triggers to `1`.
5. Set Trigger at Hour to `5am`.
6. Set Trigger at Minute to `0`.
7. Set the workflow time zone to `Africa/Lagos`.
8. Connect the trigger to the first Edit Fields node.
9. Save the workflow.

The schedule is intended to run every day at 5:00 AM Nigerian time.

### Step 8.2 — Test the trigger separately

Use the trigger's test function if appropriate, or execute the workflow manually from the canvas.

A trigger test confirms the trigger output. It does not prove that the complete AI and Google Sheets workflow succeeds.

---

## Stage 9: Publish and test

### Step 9.1 — Publish

1. Save the workflow.
2. Select **Publish**.
3. Enter a version name if prompted.
4. Optionally describe the changes.
5. Confirm publication.

The local workflow was subsequently shown as **Published**.

### Step 9.2 — Test the full workflow

Run a full execution and inspect the result of each node.

Verify the following:

- The trigger produces an item.
- The initial fields contain the expected values.
- The AI content node returns a response.
- The JavaScript parser produces structured data.
- The reviewer returns a decision.
- The approval IF node selects the correct branch.
- Approved content reaches Edit Fields4.
- The Google Sheets node receives the expected fields.
- The approved row appears in the correct worksheet.

Do not treat a successful trigger-only execution as proof that the complete workflow is working.

---

## Stage 10: Record the actual failure

During testing, the `Message a model` node returned:

`Your license does not allow for feat:aiGateway.`

This meant that the active n8n installation did not permit the feature used by that node.

A subsequent attempt to configure a direct OpenAI credential returned an unauthorized error.

The OpenAI API Keys page showed no existing keys in the selected project and displayed a request to add credits.

Therefore, the complete AI workflow was not confirmed to be working.

The Google Sheets connection test, however, had succeeded separately.

These are two different test results and must be documented separately.

---

# PART III — APPROACH B: RUNNING N8N ON THE LAPTOP

## Stage 11: Prepare the Windows laptop

### Step 11.1 — Check Windows

1. Press `Windows + R`.
2. Type `winver`.
3. Press Enter.
4. Record the Windows edition and version.
5. Confirm that the computer meets Docker Desktop's current system requirements.

Do not assume that every Windows version supports the same Docker configuration.

### Step 11.2 — Install Docker Desktop

1. Visit https://www.docker.com/products/docker-desktop/
2. Download the Windows installer.
3. Run the installer.
4. Follow the setup prompts.
5. Restart if requested.
6. Open Docker Desktop.
7. Wait for the engine to start.

### Step 11.3 — Verify Docker

Open PowerShell and run:

```powershell
docker --version
```

Then run:

```powershell
docker info
```

If the second command fails, resolve the Docker engine problem before installing n8n.

---

## Stage 12: Run n8n locally

### Step 12.1 — Create a folder

Create:

```text
C:\n8n
```

Inside it, create a file named:

```text
compose.yaml
```

### Step 12.2 — Configure Docker Compose

Use the following as a starting configuration:

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    restart: unless-stopped
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      - GENERIC_TIMEZONE=Africa/Lagos
      - TZ=Africa/Lagos
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

This configuration preserves n8n data in a Docker volume and binds the interface to the local machine.

### Step 12.3 — Start n8n

Open PowerShell in `C:\n8n` and run:

```powershell
docker compose up -d
```

Check the container:

```powershell
docker compose ps
```

Open:

http://localhost:5678

Complete the initial local n8n setup if prompted.

### Step 12.4 — Inspect errors

If n8n fails to start:

```powershell
docker compose logs -f n8n
```

Read the logs before changing the configuration.

---

## Stage 13: Import the workflow

1. Open the local n8n interface.
2. Open the workflow list.
3. Check whether `Inspire Mortals - Daily Content Engine` already exists.
4. If it exists, open it.
5. If it does not exist, import the exported workflow JSON.
6. Inspect the canvas and connections.
7. Save the workflow.

The project was imported more than once during the original setup. The correct practice is to check for an existing workflow before importing.

---

## Stage 14: Configure local Google Sheets access

The local OAuth configuration must match the local n8n instance.

The callback URL recorded during setup was:

```text
http://localhost:5678/rest/oauth2-credential/callback
```

To configure the connection:

1. Open Google Cloud Console.
2. Open the project used for Inspire Mortals automation.
3. Check the Google Sheets API and Google Drive API configuration.
4. Open Google Auth Platform.
5. Check the OAuth consent configuration.
6. Confirm the OAuth test-user configuration if the app is in testing mode.
7. Open the OAuth client used by local n8n.
8. Confirm the authorized redirect URI matches the URI displayed in n8n.
9. Return to n8n.
10. Open the Google Sheets credential.
11. Reconnect the Google account if needed.
12. Save and test the connection.

The recorded OAuth client was named:

`n8n Local Google Sheets`

The recorded Google Cloud project was:

`Inspire Mortals Automation`

The earlier local Google Sheets test succeeded, but imported credentials should still be checked in the active instance.

---

## Stage 15: Test Google Sheets independently

Before testing AI, run the simple workflow:

```text
Manual Trigger
      |
      v
Edit Fields
      |
      v
Google Sheets — Append Row
```

Use sample values and verify that a row is added to the correct worksheet.

This confirms the data-storage path without requiring an AI model.

---

## Stage 16: Choose a local AI approach

The local n8n installation does not automatically provide a free AI model.

Possible approaches include:

1. Run a compatible local model service.
2. Use manually generated content for the time being.
3. Connect a hosted AI provider if its current billing and access requirements are acceptable.

For a local model, investigate https://ollama.com/.

If n8n runs inside Docker, remember that `localhost` inside the container refers to the container itself. The local model service must be reachable from the n8n container.

The model's performance depends on the laptop's available RAM, CPU, and storage.

---

## Stage 17: Activate the local schedule

Once the workflow has passed its tests:

1. Open Schedule Trigger.
2. Set the interval to daily.
3. Set the hour to `5am`.
4. Set the minute to `0`.
5. Confirm the `Africa/Lagos` time zone.
6. Save the workflow.
7. Activate or publish it as required by the local n8n version.
8. Keep the laptop awake and Docker running at the scheduled time.
9. Inspect the execution history after the scheduled run.

The local workflow cannot run at 5:00 AM if the laptop is powered off or the n8n service is stopped.

---

# PART IV — FINAL VERIFICATION CHECKLIST

## Google Sheets

- [ ] Correct spreadsheet selected.
- [ ] Correct worksheet selected.
- [ ] Column headers match the workflow fields.
- [ ] Google credential works.
- [ ] Test row appears.

## AI workflow

- [ ] Content-generation node works.
- [ ] Parser produces structured data.
- [ ] Reviewer produces a valid decision.
- [ ] Approval condition works.
- [ ] Revision counter works.
- [ ] Revision limit leads to manual review.
- [ ] Approved content reaches the append node.

## Scheduling

- [ ] Daily interval is configured.
- [ ] Trigger time is 5:00 AM.
- [ ] Time zone is `Africa/Lagos`.
- [ ] Workflow is active.
- [ ] A scheduled execution has been verified.

## Documentation

- [ ] Exported workflow JSON preserved securely.
- [ ] Exact Code node scripts preserved.
- [ ] Credentials excluded from GitHub.
- [ ] Successful tests recorded separately from failed tests.
- [ ] Remaining work documented.

---

# IMPORTANT NOTE FOR FUTURE RECONSTRUCTION

This manual documents the known setup and the sequence used to build the workflow. Some original values—especially the full AI prompts, exact JavaScript parser code, exact revision counter logic, and all original node parameters—must be copied from the exported workflow JSON to guarantee a byte-for-byte or configuration-exact reconstruction.

Do not invent these values when reproducing the workflow. Preserve the original export, inspect it, and update this manual with the exact values from the file.

The workflow is not considered fully operational until a complete execution successfully generates content, reviews it, approves it, and writes it to Google Sheets.
