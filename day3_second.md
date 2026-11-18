# L200 Modernization, Day 3: Speaker Notes, Slides 39-46 (Session 3.3, Agent Design and Build)

**Source deck:** `raw/Trainingprograms/Kyndryl/L200 - Modernization-US-Nov16/Day 3/L200 Modernization - Day 3.pptx` (read only; unchanged).
**Covers:** the opening lecture of Session 3.3, from the section divider (slide 39) to the anchor example (slide 46). Slides 39-46 are text slides, so these notes follow the slide text and the deck has no speaker notes of its own for them.
**Format per slide:** *Say*, *Show*, *Ask or check*, *Watch for*. **Timing (suggested):** about 25-30 minutes for the eight slides; slides 43 and 44 (naming and verbs) deserve the most time because people will use them in the build exercise that follows.
**Connection to the repo:** several points tie to the `consult-modernization-enablement` repository reviewed earlier. They are marked **Repo tie-in** and are optional.

---

### Slide 39: Section divider, Session 3.3 Agent Design and Build (Lecture)
**Say:** "This is the second lecture of the day. In 3.1 we looked at where configuration lives. In 3.2 you built a workflow. Now we step back and look at how agents are designed, named and promoted, so that what you build is something the catalog can keep."
**Do:** Pause on the divider. Set expectations: this is a standards session, and the standards are applied straight away in the build that follows.

### Slide 40: Five phases, agent and workflow (revisiting)
**Say:** "Every agent or workflow moves through five phases, from need to deployed asset. You saw this on Day 1. Now we use it as a design checklist."
**Show:** The five boxes and the arrows between them, then the banner at the top: phases are iterative. "Deploy feedback loops back to Requirements, and Build often loops back to Design. Nobody gets an agent right on the first pass, and the process assumes that."
1. **Requirements:** identify the use case and business outcome. Capture inputs, outputs, integrations (GitHub, Azure Boards, downstream APIs) and the human-in-the-loop checkpoints. Confirm it is genuinely needed and not a duplicate of something already in the catalog.
2. **Analysis:** decompose into single-responsibility agents. For each, define the prompt strategy, tool calls, data contract and what state passes on. Look for reuse before designing anything new.
3. **Design:** sequence the agents (sequential pipeline, fan-out, or triage-and-escalate). Define handoff schemas, error paths and observability points. Decide where SME review or human approval gates go.
4. **Build:** compose the workflow in Agent Builder, wire the prompts, integrate GitHub and Azure Boards, write unit and acceptance tests. Iterate on prompts locally before any sandbox deployment.
5. **Deploy:** promote from local to Sandbox (draft), then Sandbox (SME-approved), then Demo. Each promotion has a gate. Demo is final and only happens with Demo gatekeeper approval after team-lead validation.
**Key point:** Phases 1-3 happen before anyone opens the canvas. Most weak agents come from skipping them.
**Ask or check:** "Which phase do you think people rush most?" Expect Requirements and Analysis.
**Repo tie-in:** the repo's own promotion path is `DEV` version on `main` → released `#.#.#` version with release notes → demo deployment through the approval step. It is the same idea from the engineering side.

### Slide 41: Analysis, decompose before you design
**Say:** "The reuse check comes first, because it is the step everyone skips."
**Show:** The four bullets, then the highlighted box.
- **Check the catalog first.** There are many agents already available. Building a new one is the last option, not the first.
- **One responsibility per agent.** If the one-line description needs an "and", it is two agents.
- **Define four things per agent:** prompt strategy, the tool calls it may invoke, its data contract, and the state that passes onward.
- **Name what it must not do.** The boundary is part of the design, not an afterthought.
**Key point (test box):** "Write the one-line description before you write anything else. If you cannot describe what the agent does without listing two verbs, you are designing two agents and you will pay for it at the hand-off."
**Do (short exercise, 2 minutes):** Give a vague agent idea such as "reads a ticket and creates a summary and raises a Jira item" and ask people to split it. The answer is at least two agents.
**Watch for:** People will define inputs and outputs but not the "must not do" line. Ask for it explicitly.
**Repo tie-in:** the repo has seven near-identical summarization and task-extraction agents. Without the catalog check, that is what duplication looks like.

### Slide 42: Design constraint, who owns what
**Say:** "This is the most important design line of the session. The orchestrator owns the workflow. The agent owns the thinking."
**Show:** The two columns.
- **The orchestrator owns:** workflow state, sequencing, retry and timeout, and A2A routing between agents.
- **The agent owns:** sense, reason, decide, act; tool selection; LLM calls; confidence scoring and escalation.
**Say (why the line matters):** "If your agent is deciding what runs next, you have put orchestration inside an agent. The workflow can no longer retry it cleanly, and the failure is no longer isolatable to one step."
**Show (the two anti-patterns):**
- An agent that sequences other work is an orchestrator wearing the wrong name.
- An orchestrator that reasons about content is an agent with too much access.
- "Both are anti-patterns, and both are easy to build by accident."
**Ask or check:** "An agent finishes its work and calls the next agent directly. Which side of the line did it cross?" It took over sequencing, which belongs to the orchestrator.
**Key point:** Tell people this ties to Block 3.4. When a step fails, a clean boundary tells you where to look.

### Slide 43: Standards, the naming convention
**Say:** "Any developer, architect or stakeholder should understand an agent from its name and its two-line description alone."
**Show:** The pattern at the top: **`[stream]-[template]-[domain]-[verb]-agent`**.
- **stream:** a 2-3 character code for the stream.
- **template:** the agent template used, in kebab-case.
- **domain:** the subject area or technology it operates on.
- **verb:** the primary action, from the approved list (next slide).
- **agent:** a literal suffix, always present.
**Examples to read out:** `alm-maf-cobol-analyse-agent`, `mfm-gaa-jcl-parse-agent`, `alm-ska-api-generate-agent`. Take one apart: App-Led Modernisation, Microsoft Agent Framework, COBOL, analyse.
**Stream codes:** `alm` App-Led Modernisation; `mfm` Mainframe; `dpe` Data Platform Engineering; `sec` Security and Compliance; `inf` Infrastructure; `obs` Observability and Operations; `cmn` Common cross-stream.
**Template codes:** `lca` LangChain; `gaa` Google ADK; `asa` AWS Strands; `maf` Microsoft Agent Framework; `lga` LangGraph Workflow; `ska` Semantic Kernel, **currently not supported**.
**Four artefact types that are not interchangeable:** an **agent** reasons about which tools to invoke; a **workflow** sequences agents; a **service** is a backend capability an agent calls; a **utility** is a pure shared helper with no reasoning.
**Ask or check:** "Is a PDF text extractor with no reasoning an agent or a utility?" A utility.
**Watch for:** The `ska` code appears in one of the examples (`alm-ska-api-generate-agent`) but the slide says Semantic Kernel is currently not supported. Say that the code exists for the convention, but do not build new agents on it.
**Repo tie-in (optional, check before saying):** most agents in the repo use the `sk_base` Semantic Kernel template and carry names like `dd-17_agent_roi_planner` and `Tools-discovery agent`, so existing names do not follow this convention. Present it as the standard for new agents. Confirm with the program team whether existing ones will be renamed.

### Slide 44: The approved verb list
**Say:** "Agents must use one of these nine verbs, so that capability is consistently interpretable across streams."
**Show:** Walk the table (verb, what the agent does, example):
- **analyse:** reads input artefacts and produces insights, scores or findings. `alm-cobol-analyse-agent`.
- **parse:** converts raw or unstructured input into structured data objects. `mfm-jcl-parse-agent`.
- **generate:** produces new artefacts: code, documentation, test cases. `alm-api-generate-agent`.
- **transform:** converts artefacts from one form or language to another. `alm-cobol-transform-agent`.
- **validate:** checks artefacts or outputs against rules, schemas or standards. `cmn-output-validate-agent`.
- **extract:** pulls specific data elements from a larger input set. `mfm-business-rule-extract-agent`.
- **map:** creates correspondences between source and target artefacts. `alm-schema-map-agent`.
- **recommend:** produces ranked suggestions or action plans. `alm-tech-recommend-agent`.
- **summarise:** condenses long-form inputs into concise human-readable outputs. `cmn-artefact-summarise-agent`.
**Key point (bottom note):** "Note what is not on the list." Discovery, assessment, health check and scoring are real things agents do, but none is an approved verb. That is deliberate: each maps onto analyse, extract or validate.
**Ask or check (3 minutes):** Give three agent ideas and have people pick the verb: "score a CMDB's health" (analyse), "pull the SLA clauses out of a contract" (extract), "check generated code against our standards" (validate).
**Watch for:** People will use "assess" or "discover" out of habit. Redirect to the verb that matches what the agent actually does with its input.
**Repo tie-in (optional):** the repo has agents called infrastructure discovery, maturity scoring and health analysis. Under this list, they would become analyse or extract agents.

### Slide 45: Documentation, gates and CI/CD, what "catalog-grade" means
**Say:** "Four things beyond a good name, and none of them is optional."
**Show:** The four standards:
1. **Documentation:** a one-line description, the input artifacts, the output artifacts, the GitHub branch and the Azure Board work item ID. Checked in alongside the workflow, "not in someone's notes".
2. **Sandbox to Demo flow:** draft in Sandbox, SME review, SME-approved publish, Demo gatekeeper approval, Demo deployment. "No skipping. No exceptions for urgency."
3. **CI/CD:** workflow code lives in GitHub, with a feature branch per workflow. Pipelines build automatically. GitHub issues track failures and remediation.
4. **Quality gates:** sandbox dry-run verified, output artifacts validated, Azure Board updates confirmed. Only then promotable. Each gate has a named owner: Engineering, SME or Gatekeeper.
**Key point:** Name the owner at each gate. A gate with no owner is a suggestion.
**Ask or check:** "A client wants a demo tomorrow and the SME has not approved. What do you do?" You do not skip the gate. You show it from the sandbox and say it is not yet approved.
**Repo tie-in:** this is what the repo's pipeline enforces in practice: build on merge to `main`, version rules, release notes, and a demo deployment that waits for approval.

### Slide 46: Anchor example, a well-designed single-purpose agent
**Say:** "One slide of source material carrying a complete agent spec. Every principle from this session is visible in it."
**Show:** The four blocks:
- **Three declared inputs:** structured JSON from any upstream agent; a markdown report template from blob storage; raw context documents (PDF, XLSX, TXT).
- **Six behaviours:** fetch the report template; fetch and extract raw context; map JSON fields to template placeholders; synthesize narrative from data plus context; generate markdown to the template structure; strip internal IDs, scores and metadata.
- **Human in the loop:** an SME verifies before the customer sees it. "Verification is a declared step, not a review someone remembers to do."
- **Seven stated outcomes:** client-ready with no agent metadata; consistent structure across report types; works with any upstream agent's JSON; a new report type needs only a new template; less manual report writing; evidence-backed, citing source documents.
**Key point (closing line):** "Three declared inputs, one responsibility, a template-driven contract, an explicit human gate, and outcomes that follow from the design rather than being claimed alongside it. That is what catalog-grade looks like on one slide."
**Ask or check:** Have people point to where each of slides 41-45 shows up: one responsibility (41), orchestrator not involved in content (42), a verb it would use (43-44), SME gate and documentation (45).
**Repo tie-in (optional):** this matches the repo's `ams-json-to-markdown-agent`, which converts DD agent JSON into client-ready Markdown using blob templates. If you want to show a real one, show it. I have not checked that its behaviour matches every point on the slide.
**Transition:** "Now you apply this. You will design and build an agent to this standard." Move to the next slide, which is not covered in these notes.

---

## Before you present

- **Slide 43 has two small tensions** worth knowing: `ska` is listed as unsupported but is used in an example, and existing repo agents do not follow the convention.
- **Spelling:** the deck uses British spellings (analyse, summarise, artefact) on these slides. Keep them consistent in anything people name.
- **Timings** are suggestions; the deck has no timing for these slides.
- **Repo tie-ins** reflect a read-only review of the repo. Remove them if you do not want to refer to the repo in class.
