# L200 Modernization, Day 3: Speaker Notes, Slides 1-21

**Source deck:** `raw/Trainingprograms/Kyndryl/L200 - Modernization-US-Nov16/Day 3/L200 Modernization - Day 3.pptx` (read only; the deck itself is unchanged).
**Covers:** slide 1 (cover) to slide 21, which is the opening of Session 3.1, Platform Tools and Agent Builder Configuration.
**Format per slide:** *Say* (what to tell the room), *Show* (what to point at), *Ask or check* (optional), *Watch for* (common trap). Slides 8, 9, 10, 13 and 15 are screenshots with no text on the slide, so their notes describe what the screenshot shows.
**Timings** are suggestions: about 5 minutes of opening (slides 1-4) and 40-50 minutes for Session 3.1 up to slide 21. Adjust to the room.

---

## Opening (slides 1-4)

### Slide 1: Cover
**Say:** "Welcome to Day 3. Today is the day we configure the platform, design against it, and break it on purpose. Give people a couple of minutes to join, then we start."
**Do:** Keep this slide up while people join. Greet people as they arrive, and ask them to confirm that their Agent Builder login and local environment from Day 2 still work.
**Watch for:** Anyone who could not run a workflow on Day 2. Sort them out in the first break, not mid-session, because the day is hands-on.

### Slide 2: Where we are in the program
**Say:** "Five days, one arc. Day 1 gave you the landscape, the platform and the propositions. Day 2 gave you prebuilt workflows, agents and extensibility. Today is configuration, insights and troubleshooting. Day 4 is the immersive client role play and Day 5 is client presentations."
**Show:** The "YOU ARE HERE" marker on Day 3.
**Key point:** Today is the bridge. You learn to configure and diagnose so that on Day 4 you can hold a client conversation about a platform you actually understand.
**Watch for:** The Day 3 title on this slide says "Configuration, Insights and Troubleshooting", while the agenda (next slide) says "Configuration, Workflow and Agent Design, Troubleshooting". Use either wording, but do not stop to explain the difference.

### Slide 3: Day 3 agenda
**Say:** "Six blocks. 3.1 is platform tools and Agent Builder configuration, a lecture. 3.2 is workflow design and build. 3.3 is agent design and build. 3.4 is advanced troubleshooting and edge cases. 3.5 is the client introduction and role-play preparation, and 3.6 is the Day 3 client role play."
**Show:** The numbered list. Point out the pattern: configure, build a workflow, build an agent, break it, then use it in front of a client.
**Ask or check:** "Which of these blocks do you expect to be hardest?" A quick show of hands helps you weight your time later.

### Slide 4: Recap of Day 2 and today's objectives
**Say:** "You already have the workflow catalog, the seven core propositions, agent families and orchestration, three client scenarios, and a workflow you built and ran yourself. By the end of today you should be able to do three more things."
**Show:** The three objective cards, one at a time:
1. **Configure the platform:** the four control planes, the model gateway end to end, access and administration, and moving a workflow between environments.
2. **Design at real scale:** a multi-branch, template-driven workflow against real client-shaped data, and an agent designed to catalog standard.
3. **Diagnose and escalate:** one method, six real failure categories, two triage trees, and knowing what belongs in an escalation.
**Key point:** Each objective maps to a later block, so people can see why they are sitting through configuration first.

---

## Session 3.1: Platform Tools and Agent Builder Configuration

### Slide 5: Section divider (Lecture)
**Say:** "Session 3.1 is a lecture. It is a tour of where configuration lives. Everything after this builds on it, so please ask questions as we go."
**Do:** Pause for a few seconds. Mention that the session is mostly walkthrough, with the admin screens shown from a live instance.

### Slide 6: Configuration lives in four places, not one
**Say:** "This is the orientation you did not get on Day 1 or Day 2. Configuration is spread over four distinct control planes, and they have four different owners."
**Show:** The four cards:
- **Plane 01, Agent Builder canvas:** node parameters, input mappings, connections, per-step settings. "Where you spent Day 2."
- **Plane 02, LiteLLM AI Models:** models, credentials, virtual keys, budgets, rate limits.
- **Plane 03, Access control:** internal users, teams, organizations and the four access levels.
- **Plane 04, Catalog structure:** projects, categories, sub-categories, sort order. The shape of the library itself.
**Key point:** When something is wrong, the first question is "which plane is this on?" A model that will not respond is Plane 02. A user who cannot see a workflow is Plane 03 or 04. A node that fails is Plane 01.
**Ask or check:** Give a symptom ("a colleague cannot find my workflow in the catalog") and ask which plane. Plane 04, or Plane 03 if it is a permissions issue.

### Slide 7: LiteLLM, one OpenAI-compatible endpoint for every model
**Say:** "One proxy sits in front of every provider, so the code you run on your laptop is the code that runs in the platform."
**Show:** The flow: your local code (OpenAI SDK pointed at localhost) to the LiteLLM proxy (one key, model config, routing) to the model provider (Azure AI, Bedrock, local models).
**Say (left column, what LiteLLM is):** It speaks the OpenAI API to any provider. One config file maps public model names to real deployments. It holds the provider credentials and issues its own keys. It tracks spend, budgets and rate limits per caller.
**Say (right column, local development):** Run the proxy locally and set `base_url` to localhost. One dev key on the laptop, no provider secrets to manage. Change model by name in config, not in application code. Every local call is logged, so cost is visible while you build.
**Key point (bottom line):** "Local and deployed environments use the same model names, so a call that works on your machine works in the platform."
**Watch for:** People who think they need the real provider key locally. They do not. They need a LiteLLM virtual key.

### Slide 8: Models and endpoints (screenshot of the LiteLLM Model Management page)
**Show:** The **AI Gateway** section of the left menu: Virtual Keys, Playground, Models + Endpoints, Agents, MCP Servers, Guardrails, Tools; below it Observability (Usage, Logs) and Access Control (Internal Users, Teams, Organizations, Budgets). This single menu is where Planes 02 and 03 live.
**Say:** "This is the All Models tab. Seven models are listed, all routed through Azure AI: GPT-4o, GPT-5.1, text-embedding-3-small, Claude Sonnet 4.5, Claude Opus 4.1 and GPT-5.4, plus a test entry."
**Point out:**
- **Model ID** is an internal identifier. The **Model Information** column shows the public name (for example `claude-sonnet-4-5`) on top and the real deployment under it (`azure_ai/claude-sonnet-4-5`). Developers use the public name. That is the mapping from slide 7.
- **Created By** reads "Defined in config" for models set in the config file and "Unknown" for models added another way. Ask: why does it matter who or what created a model? (Because config-defined models survive a redeploy; ones added by hand may not.)
- The tabs across the top (Add Model, LLM Credentials, Pass-Through Endpoints, Health Status, Model Analytics, Model Retry Settings) are the places you will come back to.
- Note "Current Team: Personal" and the message that to use these models you create a virtual key without selecting a team.
**Watch for:** `gpt-4o_delme` looks like a leftover test entry. Mention it as a real-world example of why models need an owner and a clean-up routine. Skip this if it feels like criticising the environment.

### Slide 9: Adding a model (screenshot of the Add Model tab)
**Show:** The **Add Model** tab, with **Provider** (required), **LiteLLM Model Name(s)**, and the **Model Mappings** table with two columns, Public Model Name and LiteLLM Model Name. The page header shows the LiteLLM version (v1.80.10).
**Say:** "Adding a model is three decisions. Which provider. What name does LiteLLM send to the provider's API. And what public name will your code use. The mapping table is where those two names are joined."
**Key point:** The helper text on the field reads "The model name LiteLLM will send to the LLM API". That is the real deployment name. The public name is free to choose, which is what lets you swap the model behind it without touching application code.
**Ask or check:** "If I want to move all our workflows from one model to a newer one, which of the two names do I change?" The LiteLLM model name, keeping the public name stable.
**Do (optional):** Remind people that the placeholder `claude-3-opus` in the form is only an example.

### Slide 10: Admin Dashboard, Feature Configuration (screenshot)
**Show:** The **Feature Configuration** page, under Administration, Manage Configuration. The header reads "Control platform feature access globally and per tenant/team", with a **Sync Defaults** button and a **Global Features** table with the columns Feature, Description, Status and Tenant Overrides.
**Say:** "This is the on and off switch panel for the whole platform. Each feature has a global status, and a tenant can override it."
**Point out:**
- **Enabled:** Auto Create Users (creates a user with the Viewer role on first login), Create Agent, Create MCP Server, Design Workflows, Execution Cost Tracking, Model Cost Calculator.
- **Disabled:** Activity Logs and Model Metrics Report.
- **Tenant Overrides** (the "None" and plus button) lets you turn a feature on for one tenant only.
**Key point:** Auto Create Users is a good one to discuss. On means anyone who signs in becomes a Viewer. Off means an administrator must pre-register people. That is a security and onboarding trade-off the client may ask about.
**Ask or check:** "Why might a client switch off Create Agent for most teams?" Governance, cost, quality control.

### Slide 11: Admin Dashboard, Manage Projects (screenshot plus catalog scoping chain)
**Show:** The Admin Dashboard with four tabs (Tenants, Teams, Projects, Users) and counts beside each. The Projects tab lists a single **Default Project**, a sandbox for getting started, owned by the Administrators team. On the slide, the chain **Project, Category, Sub-category, Sort order** shows how catalog scoping works.
**Say:** "A project is the unit of ownership. It sits under a team, and every workflow is saved into a project."
**Key point:** Projects are the first level of the scoping chain on the right of the slide. Category, sub-category and sort order decide where a workflow appears in the catalog. Tie this forward to slide 18, the Save dialog, where Project is a required field.
**Watch for:** People will confuse project (ownership) with category (where it is displayed). One owns, the other organises.

### Slide 12: Admin Dashboard, Manage Category (screenshot plus catalog scoping chain)
**Show:** **Category Management** ("Manage workflow categories and sub-categories"). One top-level category, **Cross Industry Catalog**, with numbered sub-categories beneath it: Mainframe App Mod Workflows, Mainframe Modernization, Mainframe Modernization (Optimization), Modern Edge and Connectivity, App-Led Mainframe Modernization Workflows, Rates Engine Modernization Workflows, Distributed App Mod Workflows, SAP Transformation Workflows and more.
**Say:** "Categories are the shelves, sub-categories are the sections of each shelf. The up and down arrows beside each entry change the sort order, and the numbers show the current order."
**Point out:** The **Add Category** button, the add-sub-category, edit and delete icons on each row, and the small grey tag on each sub-category (its identifier).
**Key point:** Sort order is a catalog design decision. People see the first sub-categories first, so put the ones you want clients to see at the top.
**Watch for:** Deleting a category affects the workflows filed under it. Say this out loud before anyone clicks the red bin.

### Slide 13: Admin Dashboard, Create User (screenshot of the Create user dialog)
**Show:** The **Create user** modal: Username (required), Email (required), Full Name, Team (required), Role (greyed out, "inherited from team"), and an Active checkbox. Behind it, the user list with each person's team, team role and status.
**Say:** "Notice what is missing. There is no role field you can edit here. The role is inherited from the team you choose."
**Key point:** The hint under the role field reads: user permissions are inherited from their team's role, and roles are managed in the Teams section. So you control access by choosing the team, not by editing the person.
**Ask or check:** "A new contractor needs read-only access. Which team do you put them in?" Viewers.
**Watch for:** People will look for a role dropdown and think it is broken. It is by design.

### Slide 14: Admin Dashboard, Manage User Roles (screenshot plus access levels)
**Show:** The Teams tab and, on the slide, the four access levels: **Viewer, Executor, Editor, Administrator**.
**Say:** "There are four access levels, from least to most. Viewer is read-only. Executor can view and run workflows. Editor can create, edit and manage workflows and agents. Administrator has full system access."
**Key point:** Roles are assigned to teams, and users take the role of their team. The five teams on screen are Administrators, Editors, Executors, an external facilitating team (an Editor-role team) and Viewers.
**Ask or check:** "Who needs Executor but not Editor?" Business users who run approved workflows but should not change them.

### Slide 15: Manage User Roles (screenshot of the Teams tab)
**Show:** The same Teams table, now in more detail: Name, Tenant, Role, Members, Status and Actions. Each team has a one-line description, for example "Can create, edit, and manage workflows and agents" for Editors and "Read-only access to workflows and executions" for Viewers.
**Say:** "Here is the same screen in full. Look at the Members column. The Executors team has zero members, which tells you the role exists but nobody uses it yet."
**Do:** Point at the folder, pencil and bin icons in the Actions column and say what each does (view members, edit, delete).
**Key point:** Role design is a conversation to have with a client early. Most clients start with too many Editors.
**Note:** Slides 14 and 15 use the same screenshot. Slide 14 frames it as the four levels; slide 15 is the detail. You can move quickly through one of them.

---

## Moving a workflow between environments (slides 16-18)

### Slide 16: Exporting a workflow
**Say:** "For backup, version control, or moving between environments. And read the dialog before you click download."
**Show:** The three steps:
1. **Open in canvas** from drafts or from the catalog. Export works on both a draft and a published workflow.
2. **Read the dialog.** It shows the workflow summary and step count, dependencies, warnings, and the export format version, currently v2.0.0.
3. **Download the JSON.** It holds node configurations, step connections and data-flow mappings, metadata and the dependency manifest.
**Key points:**
- **The manifest** lists the activities and agents that must already exist in the target environment. Without them, the import fails on first run.
- **Warnings are not decorative.** The dialog flags import problems before you download. An export with warnings still downloads, then fails in the target.
**Ask or check:** "What do you do if the dialog shows a warning?" Fix or note it before exporting.

### Slide 17: The Import Wizard, three steps and one that bites
**Say:** "Workflow Builder, then Import. Three steps: file upload and validation, agent mapping, and confirmation. The middle one is the one that goes wrong in practice."
**Show:** The three boxes: File upload and validation, Agent mapping (match exported to available), Confirmation and final import.
**Key point (from the notes on the slide):** The wizard asks you to match every agent in the export to an agent that exists in the target environment. If the target does not have it, the workflow will not run there, whatever the JSON says. "A clean import is not a working workflow. It is a workflow whose dependencies were all satisfiable."
**What survives an export and import:** Node configurations and per-step settings. Connections and data-flow mappings. Category, project, creator and cost estimates.
**What does not travel with it:** The agents themselves (only references to them). Credentials and virtual keys (these are per environment). Anything sitting in blob storage that the workflow reads.
**Ask or check:** "You import a workflow into test and it runs, but it returns empty data. What did you forget?" The blob storage content or the credentials.

### Slide 18: The Save dialog is the catalog's configuration model
**Say:** "You filled this in on Day 2 without being told what it was doing. It is the catalog's configuration model."
**Show:** Walk the table, field by field:
- **Workflow Name** (required): letters, numbers, hyphens and underscores only, no spaces.
- **Description** (optional): "the one line a stranger uses to decide whether this is the workflow they want." Write it for a stranger.
- **Category** and **Sub-Category** (both required): classification from the organisation's catalog (slide 12).
- **Sort Order** (optional): lower numbers first.
- **Project** (required): the owning project. The team is auto-assigned from it.
- **Created By** (required): auto-populated from your account.
- **Cost Estimation** (optional): minimum and maximum cost per execution.
**Key point (callout):** "Project is the field people skip past." Because the team comes from the project, the project field is what makes a workflow findable, ownable and costable. A workflow saved into the wrong project belongs to the wrong team, and nobody notices until someone asks who owns it.
**Do:** Connect it back to slides 11 and 12. The admin screens create the options; this dialog is where they are chosen.

---

## Features and secrets (slides 19-21)

### Slide 19: Advanced features worth knowing
**Say:** "Four features that will not change your design, but will change your day."
**Show:**
1. **AI Workflow Assistant:** describe what you want in natural language and it suggests agents and configurations. "A starting point, not a finished workflow."
2. **Conversational launcher:** for supported workflows, execute through a chat interface rather than the canvas. "The demo-friendly path, and the one a client is most likely to see."
3. **Keyboard shortcuts:** Delete or Backspace to delete a node, Ctrl/Cmd+A to select all, Ctrl/Cmd+Z undo, Ctrl/Cmd+Shift+Z redo, Ctrl/Cmd and plus or minus to zoom, Ctrl/Cmd+0 to fit the view.
4. **Timeouts and retries:** each agent step carries its own timeout and retry count. Set them per step so a slow agent fails fast instead of hanging the run.
**Key point:** Timeouts and retries come back in Block 3.4 on troubleshooting. A hanging run is one of the most common failures.
**Ask or check:** "When would you use the conversational launcher with a client?" When you want to show a workflow without exposing the canvas.

### Slide 20: Injecting environment variables and secrets
**Say:** "The `.env` file holds sensitive values, so it is never committed and never baked into the image. We encrypt it, and keep the passphrase in GitHub."
**Show:**
- **Step 01, encrypt the .env file:** `gpg --symmetric --cipher-algo AES256 .env`. This produces `.env.gpg`, and that encrypted file is the only version that goes into the repository.
- **Step 02, store the passphrase in GitHub:** Settings, Secrets and variables, Actions, New repository secret. Name it `MY_PASSPHRASE` and paste the passphrase as the value.
**Key point:** Two things are stored in two places. The encrypted file is in the repo. The key to open it is in GitHub secrets. Neither is useful without the other.
**Watch for:** Committing the plain `.env`. Make sure it is in `.gitignore` before anyone runs `git add .`. Also remind people not to reuse a real production passphrase in a training environment.

### Slide 21: Pointing the agent at the secret
**Say:** "The agent folder names the secret. The workflow reads that name, decrypts `.env.gpg`, and injects the variables at run time."
**Show:**
- **Step 03:** in the agent's folder, create a file called `.env.gpg.github` containing one line, the name of the GitHub secret that holds your passphrase (`MY_PASSPHRASE`). Run `cat .env.gpg.github` to confirm.
- **Step 04:** check the two files are in place with `ls -1 .env*`. You should see `.env.gpg` and `.env.gpg.github`.
**Key point:** Both files are safe to commit, because one is encrypted and the other is only a name. The plain `.env` stays in `.gitignore`.
**Ask or check:** "What does `.env.gpg.github` contain, the passphrase or the name of the secret?" The name of the secret.
**Transition:** "That is how an agent gets its secrets without them ever being committed in plain text." Then move to slide 22 (not covered in these notes; check it before you present).

---

## Gaps and things to confirm before delivery

- **Screenshot slides (8, 9, 10, 11, 12, 13, 14, 15):** the notes describe what the images show. If your instance differs on the day (different models, teams or user counts), say what is on your screen rather than reading the notes.
- **Personal details in screenshots:** the screenshots on slides 10-15 show a real administrator's name and email, and a list of named users. If the deck is shared outside the cohort, consider cropping or masking them. The notes do not repeat any names.
- **Timings** are suggested, not taken from the deck. The source deck has no timing information.
- **Existing notes:** the source deck already has notes on slides 1, 17, 20 and 21; these have been reworked into the notes above, with slide 17's "what survives" and "what does not travel" lists kept as written.
