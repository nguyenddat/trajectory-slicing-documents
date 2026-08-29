---
source: "AgentRx Diagnosing AI Agent Failures from Execution Trajectories.pdf"
title: "AgentRx: Diagnosing AI Agent Failures from Execution Trajectories"
author: "Shraddha Barke; Arnav Goyal; Alind Khare; Avaljot Singh; Suman Nath; Chetan Bansal"
pages: 27
---
**AGENTRX: Diagnosing AI Agent Failures from Execution Trajectories**

**Shraddha Barke**<sup>1</sup> **Arnav Goyal**<sup>2</sup> **Alind Khare**<sup>2</sup> **Avaljot Singh**<sup>3</sup> **Suman Nath**<sup>2</sup> **Chetan Bansal**<sup>2</sup>

# **Abstract**

AI agents often fail in ways that are difficult to localize because executions are probabilistic, longhorizon, multi-agent, and mediated by noisy tool outputs. We address this gap by manually annotating failed agent runs and release a novel benchmark of 115 failed trajectories spanning structured API workflows, incident management, and open-ended web/file tasks. Each trajectory is annotated with a critical failure step and a category from a grounded-theory derived, cross-domain failure taxonomy. To mitigate the human cost of failure attribution, we present AGENTRX, an _automated domain-agnostic_ diagnostic framework that pinpoints the critical failure step in a failed agent trajectory. It synthesizes constraints, evaluates them step-by-step, and produces an auditable validation log of constraint violations with associated evidence; an LLM-based judge uses this log to localize the critical step and category. Our framework improves step localization and failure attribution over existing baselines across three domains.

# **1. Introduction**

Large Language Models (LLMs) based agents are increasingly _acting autonomously_ without a human in the loop. Modern AI agentic systems plan, coordinate, and execute complex and critical real-world tasks. These agents are no longer confined to sandboxes; they are being deployed in high-stakes environments such as recruiting LinkedIn (2024), web browsing Fourney et al. (2024), and even operating production Cloud Services Zhang et al. (2024). This ability to operate with minimal guidance dramatically amplifies human productivity Brynjolfsson et al. (2023). However, the same autonomy that enables speed also makes these agentic systems difficult to debug: trajectories are longhorizon, tool outputs are noisy, and failures can propagate

> 1Microsoft Research 2Microsoft 3UIUC. Correspondence to: Shraddha Barke _<_ sbarke@microsoft.com _>_ , Chetan Bansal _<_ chetanb@microsoft.com _>_ .

_Preprint. February 3, 2026._

_Figure 1._ Given domain policy, tool schema, and a failed trajectory, AGENTRX outputs the critical failure step and a failure category.

through side effects before they are observed. Consequently, ensuring the robustness of AI agents is not an option, but a prerequisite for their safe adoption in the real-world.

In this paper, our focus is on: _root-causing and localizing the critical failure which prevented the Agent from successful task completion_ . We define _critical failure_ as the first unrecoverable failure by any agent in an execution trajectory. To understand and characterize agentic failures, we manually analyze failed agent executions and introduce a benchmark of 115 agent execution trajectories across structured API workflows, incident management, and open-ended web/file tasks. Each trajectory includes the full execution trace (messages, tool calls, tool outputs, and observable environment state) and is annotated with the _critical step_ and a _failure category_ capturing why the run ultimately fails. The grounded-theory–based annotation process also yields a novel cross-domain failure taxonomy for root-cause attribution.

However, manual failure attribution is expensive and hard to scale. There is a significant need for automated validation of agents. In Software Engineering, reliability is enforced through contracts that make interface misuse and constraint violation explicit. Agentic systems require an analogous notion of correctness, but stated over executions: ambiguous

intents, long trajectories of messages, tool calls, observations, and state updates. To operationalize this view, we designed AGENTRX, a _domain-agnostic_ automated framework for AI agents that (1) normalizes heterogeneous multi-agent logs into a common trajectory representation, (2) synthesizes _constraints_ from tool schema, domain policies, and the observed trajectory (3) produces a step-indexed validation log of violated obligations with supporting evidence, which a downstream judge uses to localize the critical failure step and assign a category. As shown in Figure 1, after the assistant agent calls GET PRODUCT DETAILS and reports “11 available T-shirt options,” AGENTRX synthesizes a relational constraint TSHIRT COUNT MATCHES that recomputes the count from the tool response and checks agreement with the assistant’s interpretation. The resulting violation includes evidence (trajectory window), helping the judge localize the decisive failure.

We evaluate this framework on our proposed benchmark and prior work Zhang et al. (2025a). Our best method beats state of the art and baselines on both localizing the first unresolved critical failure step and categorizing the root-cause. Overall, the results show that trajectory-level constraints are a useful signal for detecting failures. Our contributions are: (i) An open-source dataset of **115 failed trajectories across 3 agentic domains** -including step-level annotations and failure categorization. (ii) a **novel domain-agnostic agentic failure diagnosis framework** that combines constraint synthesis and LLM-based adjudication using auditable violation logs. (iii) A **cross-domain failure taxonomy derived through grounded theory coding** , capturing 9 root-cause categories that generalize across diverse multi-agent settings, and (iv) Extensive experiments showing 23.6% absolute improvement in failure localization and 22.9% improvement in root-causing the failure.

# **2. The AGENTRX Benchmark**

We are the first to introduce a benchmark for _attribution of the first unrecoverable failure_ in AI agents. Our benchmark contains three diverse domains: API workflows, incident management, and real-world web/file tasks across both single-agent and multi-agent settings.For each domain, we sample the trajectories where an agent failed to solve a task and use them to construct our failure attribution benchmark. _τ_ **-bench.** _τ_ -bench Yao et al. (2024) is a tool-calling benchmark for retail tasks, with an LLM-simulated user and a single agent equipped with domain-specific APIs and policy guidelines. In a typical _τ_ -bench task, the agent can cancel or modify pending orders, return or exchange delivered orders, update user addresses, and answer product queries. We sample 115 _τ_ -bench trajectories using GPT-4O with one trial per task and analyze 29 trajectories that fail.

**Flash.** Flash Zhang et al. (2024) is a workflow automa-

tion agent designed for diagnosing recurring incidents in real-world production settings. It is a multi-agent system in which a team of specialized agents execute a troubleshooting guide for the incident being investigated. The guide involves steps like querying for clusters in a region that might be affected and also mitigation steps based on whether the incident is a false alarm or not. We sample 42 Flash trajectories containing at least one agent failure for our analysis.

**Magentic-One.** Magentic-One Fourney et al. (2024) is a generalist multi-agent system for open-ended web and file-based tasks. It comprises five specialized agents with distinct capabilities, including web browsing, file navigation, and code writing. We randomly sample 44 failed multiagent trajectories from the Who&When dataset (Zhang et al., 2025a). Together, these sources give us a total of 115 failed trajectories containing at least one agent failure that form our AGENTRX benchmark.

**Comparison to Prior Work.** We qualitatively compare our annotation strategy with Who&When Zhang et al. (2025a) (W&W)’s labeling. Their annotated failure is not necessarily the critical one as per our definition. In one trajectory (Appendix B), the Orchestrator recovers from the failure marked by W&W: by switching from WebSurfer to FileSurfer, and only becomes unrecoverable after a later hallucinated file/path error (which our annotation catches). Our annotation strategy catches this critical error. This indicates that **localization of the first unrecoverable critical failure is harder and more fine-grained** .

## **2.1. Grounded Theory Annotation of Agent Failures**

To systematically analyze the critical agent failures, we manually annotate trajectories using a grounded theory coding process (Glaser & Strauss, 2017). Grounded Theory (GT) takes qualitative data and produces a theory in an iterative process. Unlike hypothesis-driven studies, GT develops hypotheses and a higher-level theory that emerge from the data. Instead of imposing a predefined label set for agent failures, we began with _open coding_ , allowing categories to emerge from repeated failure patterns. Three annotators worked on each of the three domains and coded one trajectory at a time. During open coding, annotators read the trajectories step-by-step and wrote a structured _coding memo_ whenever a failure was observed. We define a _failure_ as any point at which the agent did not make progress toward a correct outcome _e.g.,_ violating a policy guideline or executing an invalid tool call. We follow a two-stage coding procedure:

**Phase 1: Exhaustive failure marking.** Annotators first marked _all_ failure steps, even if later steps dominate the final outcome. For each failure event, we recorded: (i) the step index; (ii) an _open code_ : a short, concrete description tightly grounded in the trace; and (iii) a reason for that

<!-- Start of picture text -->
Tau-Bench<br>30<br>25<br>20<br>15<br>10<br>5<br>0<br>0 10 20 30 40 50 60<br>Flash<br>40<br>30<br>20<br>10<br>0<br>1 2 3 4 5 6<br>45 Magentic-One<br>40<br>35<br>30<br>25<br>20<br>15<br>10<br>5<br>0<br>0 10 20 Step Number 30 40 50<br>Failure category<br>Instruction/Plan Adherence Failure Guardrails Triggered Invention of New Information<br>Misinterpretation of Tool Output Intent Not Supported System Failure<br>Intent Plan Misalignment Underspecified User Intent Invalid Invocation<br>Number of Trajectories<br>Number of Trajectories<br>Number of Trajectories<br><!-- End of picture text -->

_Figure 2._ **Failure timelines per trajectory across domains.** Each horizontal line is one trajectory. Colored ticks mark detected failures at the corresponding step number (labeled with the category); the star indicates the root-cause step. Magentic-One has 295 total failures, with 68% of trajectories containing at least two failures; Flash has 52 total failures, and _τ_ -bench has 39.

failure. This stage captures early deviations and cascading failures, including ones the agent later recovers from.

**Phase 2: Critical failure identification.** We then identified the _critical failure step_ that leads to the incorrect terminal outcome. Concretely, we work backward from the terminal failure to the earliest failure from which the agent does not recover.

Throughout the coding process, new codes were constantly compared with previously assigned ones. When a new failure matched an existing category, the same code was reused; otherwise, we created a new code and wrote a concise definition. As coding progressed, we periodically aggregated related open codes into higher-level categories, discussed them extensively among annotators, and refined category definitions. We iterated on the taxonomy until we reached _theoretical saturation_ where new trajectories did not introduce new failure phenomena. At that point, we _froze_ the category definitions and used closed coding to annotate the remaining trajectories _and to recode the previously annotated ones_ , verifying that the taxonomy covers failures beyond the subset that initially shaped it. Further, our labels are drawn from three structurally different domains spanning both single-agent and multi-agent settings, and each domain was annotated by independent annotators.

**Benchmark Format.** Each entry corresponds to a trajectory and contains (i) an identifier, (ii) a record of failure events

encountered during execution, and (iii) a critical failure. For each failure event, we store its step in the trajectory, a brief natural-language description, and a failure label from the taxonomy with supporting rationale. The list of recorded failures gives a causal chain from the first unrecoverable failure to the terminal one.

## **2.2. A Cross-Domain Failure Taxonomy**

Prior work takes a system-level view of multi-agent failures, organizing failure modes by design, coordination, and verification stages (Pan et al., 2025). We develop a taxonomy that labels the critical failure:

**1. Plan Adherence Failure** The agent fails to follow required directions or the agreed plan by ignoring directives. This covers both under-execution (missed steps) and over-execution (unplanned actions, e.g., extra tool calls) that violate the plan, domain policy, or orchestrator constraints.

**2. Invention of new information** The agent introduces, removes, or alters information that is not grounded in any available input, context, or tool output. This includes fabricating unsupported facts, hallucinated details, or omitting relevant information without justification.

**3. Invalid Invocation** The agent issues an invalid tool call, e.g., malformed inputs, missing required arguments, or values that fail schema validation.

_Table 1._ Distribution of _critical failure_ labels across domains. We normalize per domain (percent of failed trajectories)

|**Root-cause category**|_τ_**-bench **|**Flash **|**Magentic**|
|---|---|---|---|
|Instruction Adherence|10.3|23.8|18.2|
|Invention of Information|0|9.5|9.1|
|Invalid Invocation|3.4|0|0|
|Misinterpretation of Tool Output|24.1|33.3|34.1|
|Intent-Plan Misalignment|24.1|0|9.1|
|Under-specified Intent|27.6|19|0|
|Intent Not Supported|6.9|0|6.8|
|Guardrails Triggered|0|0|20.5|
|System Failure|3.4|14.3|2.3|
|**# failed trajectories**|**29**|**42**|**44**|

**4. Misinterpretation of Tool Output** The agent incorrectly reasons about its own or another agent’s tool output, leading to incorrect assumptions or actions.

**5. Intent–Plan Misalignment** The agent misinterprets the user’s goal or constraints and produces an incorrect plan.

**6. Under-specified User Intent** The agent was unable to complete the task due to lack of complete information at that point in the trajectory.

**7. Intent Not Supported** The agent is asked to perform an action for which a tool is not available.

**8. Guardrails Triggered** The agent’s execution is blocked by safety policies or external access restrictions.

**9. System Failure** The agent faces a system connectivity issue while calling a particular tool like an API endpoint not being reachable.

**Annotation Cost.** The manual effort required for these annotations is substantial. Annotators spent, on average, 20 minutes per _τ_ -bench trajectory, 22 minutes per Flash trajectory, and 24 minutes per Magentic trajectory. Across 115 trajectories, this amounts to 42.7 total human hours and is challenging to scale. This motivates the need for automated failure attribution frameworks such as AGENTRX.

# **3. AgentRx Framework Formulation**

**Setup.** AGENTRX takes as input (i) a toolset _T_ (available tools/agents with input–output schema), (ii) an optional domain policy Π (domain-specific natural-language rules), and (iii) a failed trajectory _T_ = _⟨s_ 1 _, . . . , sn⟩_ . Each step _sk_ contains multiple logged events with fields such as agent name, tool name, step index, tool output or natural language conversational content. We normalize each trajectory into a common intermediate representation (IR) because formats differ across domains. Let _T≤k_ denote the trajectory prefix up to and including step _k_ . At a high level, AGENTRX generates _executable constraints_ from the tool schema, domain policy, and _T≤k_ , and evaluates them step-by-step to produce an auditable validation log V of violations and supporting

evidence. This log is then passed to an LLM-based judge along with a taxonomy checklist K to predict a root-cause step ˆ _s_ and failure category _y_ ˆ.

**Taxonomy checklist.** We compile our failure taxonomy into a semantic checklist K: for each category _y_ in the taxonomy, K( _y_ ) specifies a small set of targeted yes/no questions (and brief decision criteria) that operationalize the category definition.

## **3.1. Global and Dynamic Constraint Synthesis**

AGENTRX synthesizes constraints in two stages: _Global constraints C_<sup>_G_</sup> are synthesized once from the tool schema and domain policy, and stored in a global store G:

_Dynamic constraints Ck_<sup>_D_are synthesized at each step from</sup> the task instruction _I_ and the observed prefix _T≤k_ , using G as context, and stored in a local store L _k_ :

Intuitively, global constraints encode domain rules, while dynamic constraints encode trajectory-specific rules and prior observations (e.g., constraints that must hold given earlier tool outputs). We define the constraints available at step _k_ as the union _Ck_ := _C_<sup>_G_</sup> _∪Ck_<sup>_D_.Inpractice,con-</sup> straint synthesis emits constraints in a lightweight schema to standardize checking across domains (subsection E.1).

## **3.2. Guarded Constraints and Evaluation**

A constraint _C_ generated by AGENTRX comprises (i) a guard _GC_ ( _T≤k, sk_ ) _∈{_ 0 _,_ 1 _}_ that determines when the constraint applies and (ii) an assertion Φ _C_ ( _T≤k, sk_ ) _∈ {_ SAT _,_ VIOL _}_ that returns a binary verdict _v_ when evaluated. Guards typically capture structural conditions (e.g., “this step contains a call to tool _t_ ”, or “a specific agent is invoked”). Evaluation at step _k_ is:

Constraints are enforced either with programmatic checks or semantic checks. Programmatic checks are predicates over structured fields (e.g., schema validity, equality, membership), while semantic checks are natural-language predicates evaluated using an LLM-based checker. Both types of checks return a verdict–evidence pair ( _v, e_ ). Examples in Appendix E.

## **3.3. Validation Log**

At step _k_ , AGENTRX evaluates each constraint _C ∈Ck_ whose guard applies, i.e., _GC_ ( _T≤k, sk_ ) = 1. We record each violated constraint and its supporting evidence in a step-indexed validation log:

V is step-indexed, auditable, and directly links each violation to supporting evidence.

## **3.4. Judge for Root Cause Attribution**

Given the task instruction _I_ , trajectory _T_ , checklist K and violation log V, an LLM-based judge outputs: (i) a critical failure step ˆ _s_ , and (ii) a critical failure category _y_ ˆ from our taxonomy.

**Critical step selection.** The judge selects ˆ _s_ as the first step _k_ whose violation(s) are sufficient to explain why the run fails with respect to _I_ (i.e., correcting step _k_ would plausibly change the outcome). Violations serve as diagnostic evidence rather than hard requirements; the judge may override them when the trajectory context indicates otherwise.

**Failure category assignment.** Each violation is mapped to multiple possible taxonomy labels (e.g., invalid tool invocation or misinterpretation of tool output). The judge sets _y_ ˆ based on the taxonomy labels associated with the violation(s), and emits a short rationale citing the specific reason for selecting the failure category.

# **4. Experimental Evaluation**

AGENTRX outputs (i) a _critical step index_ ˆ _s_ and (ii) a _failure category y_ ˆ from a fixed cross-domain taxonomy. We evaluate AGENTRX along two axes: **(i) accuracy** of step localization and category attribution, and **(ii) robustness** to cross-domain shift, different constraint generation strategies, judge stochasticity, and long traces.

**Benchmark.** We evaluate on our AGENTRX benchmark across three domains: (i) API workflows ( _τ_ -bench), (ii) incident response workflows (Flash), and (iii) open-ended web/file tasks (Magentic-One). Each instance is a step-indexed trajectory containing user/agent messages, tool invocations, tool outputs, and any logged environment state available to the agent. We report per-domain statistics: _τ_ -bench has 29 trajectories with median length 36 (range 20–62), Flash has 42 with median length 3 (2–6), and Magentic-One has 44 with median 33 (5–130 range). Flash also averages 8 substeps per step.<sup>1</sup>

1Flash steps are defined using the Troubleshooting Guide for consistency; each step expands into multiple substeps.

## **4.1. Evaluation Metrics and Experimental Settings**

**Step Localization Accuracy.** We treat step localization as a single-step identification problem over the executed trajectory. We report:

(i) **Critical Step-index Accuracy** , the fraction of trajectories where the predicted step exactly matches the annotated step;

(ii) **Critical Step-index Accuracy@** _±r_ , the fraction of trajectories where the predicted step falls within _±r_ steps of the annotated step, for _r ∈{_ 1 _,_ 3 _,_ 5 _}_ ; and

(iii) **Average Step Distance** (lower is better), which measures how far predictions are from the annotated step. Together, these metrics capture both strict correctness and utility when diagnoses are off by a small number of steps.

**Failure Category Accuracy.** This involves assigning the failure to a category in our taxonomy, or outputting _inconclusive_ (or a custom label). We report:

(i) **Critical Failure Category Accuracy** , where the predicted failure category matches the annotated category;

(ii) **Any-failure Category Accuracy** , where the prediction matches _any_ category among the trajectory’s failure steps;

(iii) **Earliest Category Accuracy** , where it matches the category at the first failure step; and

(iv) **Terminal Category Accuracy** , where it matches the category at the last failure step.

**Experimental settings.** We run each configuration _n_ =3 times and report mean _±_ standard deviation for step-index accuracy, failure category accuracy, and average step distance (Table 5). This quantifies robustness to sampling noise and judge variability. We enable both (i) _programmatic constraints_ and (ii) _semantic constraints_ (natural-language rules). Table 5 reports results with both enabled to reflect the default AGENTRX setting. We use GPT-5 as our default model for all experiments. (Results using o3 and without semantic constraints for _τ_ -bench in Appendix A).

## **4.2. Critical Failure Localization in Agentic Systems**

We compare our baseline against Who&When (W&W) on _τ_ -bench and Magentic-One. For _τ_ -bench, we evaluate on all 29 trajectories; for Magentic-One, we evaluate on the subset of 16 (out of the 44) trajectories where our dataset has the same annotated step number as W&W. We run W&W’s LLM-as-a-judge, with one prompt modification: instead of returning the first failure step, we ask it to identify the _first unrecoverable_ critical step. Even with this modification, our simplest baseline achieves higher step and agent accuracy than the best performing variant of W&W on _τ_ -bench and Magentic, as shown in Table 2. This improvement comes

_Table 2._ **Failure attribution accuracy.** Agent accuracy identifies _Table 4._ We ablate AGENTRX by enabling only global constraints the failed agent; step accuracy identifies the failure step. We run or only dynamic constraints for _τ_ -bench. Mean _±_ std over _n_ =3. both methods with GPT-5; W&W<sup>_∗_</sup> denotes our prompt-modified version. We report W&W’s best-performing variant. **Method Step-index acc. (%)** _↑_ **Category acc. (%)** _↑_

|**Method**|**Step-index acc. (%)**_↑_|**Category acc. (%)**_↑_|
|---|---|---|
|Baseline|32.2_±_3.2|25.3_±_1.6|
|Global-Only|41.4_±_2.8|28.7_±_1.6|
|Dynamic-Only|43.7_±_1.6|36.8_±_1.6|
|AGENTRX|48.3_±_0|39.1_±_1.6|

||_τ_**-bench**(#|of traj = 29)|**Magentic**(#|of traj = 16)|
|---|---|---|---|---|
|**Method**|**Agent (%)**|**Step (%)**|**Agent (%)**|**Step (%)**|
|Who&When<sup>_∗_</sup>|62|17.2|6.2|56.3|
|Our Baseline|75.9|32.2|81.2|56.3|

_Table 3._ Mean token and characters per trajectory and per step.

|**Metric**|**_τ_**-**Bench **|**Flash **|**Magentic**|
|---|---|---|---|
|**Avg tokens / step**|133|169|330|
|**Avg chars / step**|480|930|1280|
|**Avg tokens / trajectory**|4889|6415|16484|

from the _strength of our baseline_ : **it is grounded in our comprehensive failure taxonomy and prompts the LLM to use the same failure attribution procedure as our human annotators** (subsection G.1).

In addition, W&W’s reported magentic step accuracy uses their step-by-step variant, which requires 16 _×_ more LLM calls per trajectory than all-at-once and is therefore more expensive. Their all-at-once variant, which is closest to our baseline, achieves only 12.5% step accuracy on Magentic.

## **4.3. Overall Failure Attribution Performance**

We compare our baseline with the best-performing version of AGENTRX. **AGENTRX beats baseline in all three domains in both critical step-index and failure category accuracy.** Notably, in _τ_ -bench the step accuracy jumps from 32.2% in baseline to 54% with the best performing variant as shown in Table 5. Even category accuracy improves from 25.3% to 40.2%. For Flash, where our baseline is already strong, the category accuracy improves by 6.4 points (53.9% _→_ 60.3%), while the step accuracy improves by 2.4 points. Magentic is a difficult subset of our benchmark with an average of 6.7 failures per trajectory. Our best configuration of AGENTRX beats baseline by a small margin. We also report results on Magentic<sup>_∗_</sup> , a filtered subset of Magentic with 27 trajectories of length at most 50 steps to keep trajectories within a manageable horizon. On Magentic<sup>_∗_</sup> , AGENTRX improves baseline step accuracy from 42% to 46.9% (and category accuracy from 39.5% to 44.4%). Notably, the average step distance also reduces in AGENTRX compared to baseline for all three domains.

## **4.4. Impact of Violations and Taxonomy Checklist**

We consider three judge inputs: (i) violation evidence alone (Baseline+Vio.), (ii) checklist alone (Taxonomy Checklist), and (iii) checklist+violations (Checklist+Vio.). On _τ_ -bench,

violations alone yield a large gain over the baseline in step accuracy (32.2 _→_ 47.1) and category accuracy (25.3 _→_ 37.9), while the checklist alone shows no gains. On Flash, both signals help: the checklist alone improves category accuracy (53.9 _→_ 57.9), while checklist plus violations further increases it to 60.3; step accuracy also improves, from 80.9 to 83.3 using checklist alone (Table 5). On Magentic, the strongest configuration uses the checklist alone, suggesting that **semantic structure can outweigh sparse or noisy violations evidence.** The checklist plus violations setting outperforms baseline on all three metrics in Magentic<sup>_∗_</sup> . In most settings, checklist plus violations improves over either signal alone. **Overall, the validation log of violations and taxonomy checklist provide useful signal for critical failure attribution.**

## **4.5. Impact of Constraint Generation Strategies**

We evaluate two ways of generating constraints from the tool schema and optional domain policy when diagnosing a trajectory: _step-by-step_ generation, which conditions on the trajectory prefix, and _one-shot_ generation, which conditions on the full trajectory. In the step-by-step setting, at each step _k_ we generate constraints conditioned only on the prefix _T≤k_ (i.e., all events observed up to and including step _k_ ). This yields _step-conditioned_ constraints that enable incremental checking and closer to a real-world setting. In the one-shot setting, we provide the entire trajectory to the generator and produce a single set of constraints in one pass. **One-shot constraints are generated at once over the entire trajectory, serving as a cost-efficient alternative.**

Table 5 compares one-shot vs. step-by-step constraint generation across domains.

On _τ_ -bench, one-shot performs better: the best one-shot setting reaches 54.0% step accuracy and 40.2% category accuracy, whereas the best step-by-step variant reaches 41.4% and 35.6%, respectively. Flash shows a mixed pattern: stepby-step improves category accuracy, while step-index accuracy is similar across the two settings. In Magentic, longer trajectories make one-shot constraint generation brittle, we do not see a huge improvement over the baseline. We evaluated the step-by-step setting only on Magentic<sup>_∗_</sup> because of long-horizon trajectories. Step-by-step yields a sizeable gain in both step (46.9% vs. 42%) and category accuracy

_Table 5._ **Effect of judge inputs and constraint generation.** We compare one-shot vs. step-by-step constraint generation, judge inputs (violations, taxonomy checklist, both) and judging protocols (Step-then-Category or All-at-Once).

||**Judge**|**Baselines**||**AGENTRX Abl**|**ations**||
|---|---|---|---|---|---|---|
|**Metric (**_n_=3**)**|**Baseline**|**Step-then-Cat.**|**Baseline+Vio.**<br>**S**|**tep-then-Cat.+Vio.**<br>**Tax**|**onomy Checklist**|**Checklist+Vio.**|
|**Tau-Bench**|||**One-shot**|**Constraint Generation**|||
|Step-index acc. (%)_↑_|32.2_±_3.2|32.2_±_1.6|47.1_±_1.6|**54**_±_**1.6**|32.2_±_1.6|48.3|
|Category acc. (%)_↑_|25.3_±_1.6|27.6_±_2.8|37.9_±_2.8|**40.2**_±_**1.6**|25.3_±_1.6|39.1_±_1.6|
|Avg. step distance_↓_|5.7_±_0.8|6_±_1|2.8_±_0.3|**2.4**_±_**0.5**|5.8_±_0.7|3_±_0.3|
|**Tau-Bench**|||**Step-by-Ste**|**p Constraint Generation**|||
|Step-index acc. (%)_↑_|32.2_±_3.2|32.2_±_1.6|41.4_±_2.8|36.8_±_1.6|32.2_±_1.6|37.9_±_2.8|
|Category acc. (%)_↑_|25.3_±_1.6|27.6_±_2.8|35.6_±_1.6|34.5|25.3_±_1.6|35.6_±_1.6|
|Avg. step distance_↓_|5.7_±_0.8|6.0_±_1|3.3_±_0.6|4.1_±_0.1|5.8_±_0.7|3.6_±_0.1|
|**Flash**|||**One-shot**|**Constraint Generation**|||
|Step-index acc. (%)_↑_|80.9_±_2.3|73_±_1.3|81.8_±_1.3|70.6_±_2.3|**83.3**_±_**2.3**|80.1_±_1.3|
|<br>Category acc. (%)_↑_|53.9_±_3.6|55.5_±_3.6|53.9_±_7.6|52.3_±_2.3|57.9_±_2.7|58_±_3.6|
|Avg. step distance_↓_|0.25_±_0.2|0.34_±_1.3|**0.2**|0.36|0.2_±_0.3|0.2|
|**Flash**|||**Step-by-Ste**|**p Constraint Generation**|||
|Step-index acc. (%)_↑_|80.9_±_2.3|73_±_1.3|76.2|68.2_±_5.4|83.3_±_2.3|76.1_±_2.3|
|Category acc. (%)_↑_|53.9_±_3.6|55.5_±_3.6|57.1_±_6.2|59.5_±_2.3|57.9_±_2.7|**60.3**_±_**1.3**|
|Avg. step distance_↓_|0.25_±_0.2|0.34_±_1.3|0.3|0.4|0.2_±_0.3|0.3|
|**Magentic**|||**One-Shot**|**Constraint Generation**|||
|Step-index acc. (%)_↑_|31.8|29.5_±_2.3|25_±_2.3|27.3_±_1.3|31.8|24.2_±_1.3|
|Category acc. (%)_↑_|36.4_±_3.6|37.8_±_5.7|31.1_±_3.5|34.1_±_4.5|**37.1**_±_**5.7**|25_±_1.3|
|Avg. step distance_↓_|22_±_1|13.4_±_2|28_±_1.2|**13.7**_±_**2.3**|25.5_±_1.2|28.3_±_1|
|**Magentic**<sup>_∗_</sup>|||**Step-by-Ste**|**p Constraint Generation**|||
|Step-index acc. (%)_↑_|42_±_1.8|40.7|45.7_±_1.8|40.7_±_5.2|42_±_1.8|**46.9**_±_**3.5**|
|Category acc. (%)_↑_|39.5_±_3.5|40.7_±_5.2|43.2_±_1.8|42_±_1.8|35.8_±_1.8|**44.4**_±_**3**|
|Avg. step distance_↓_|5_±_0.7|4.9_±_0.8|4.9_±_0.9|5.2_±_0.4|6.6_±_0.3|**4.8**_±_**0.8**|

(39.5% vs. 44.4%) for Magentic<sup>_∗_</sup> . These results align with the trajectory length across domains (Table 3). _τ_ -bench averages 4,889 tokens per trajectory, so one-shot generation can effectively condition on relevant context in a single pass. Magentic averages 16,484 tokens, where one-shot is more susceptible to context dilution and localized step-by-step constraint generation is more reliable.

We also distinguish between (i) **global constraints** derived solely from the tool schema and domain policy (trajectoryindependent), and (ii) **dynamic constraints** generated conditionally from the observed prefix _T≤k_ (trajectorydependent). We evaluate on these two ablations: **GlobalOnly** (schema/policy only) and **Dynamic-Only** (prefixconditioned only). We run this experiment only on _τ_ -bench because Flash and Magentic do not have a domain policy. In Table 4, both ablations beat the baseline, showing that **either global constraints or dynamic constraints alone provide useful signal** on _τ_ -bench. Dynamic-Only is the stronger single component, especially for category accuracy but **combining both yields the best performance overall.**

## **4.6. LLM-as-a-Judge Evaluation**

We evaluate failure attribution with an LLM-as-judge given the trajectory, violation evidence and taxonomy checklist. We consider two judging protocols. In _All-at-Once_ , the judge sees the full validation log (and optionally the taxonomy checklist) in a single call and predicts both the critical step and failure category. In _Step-then-Category_ , the judge first selects the failure step and then, conditioned on that step, predicts the category in a second call. The default AGENTRX setting is All-at-Once. Step-then-Category can be brittle: it _commits_ to a single step before considering the failure taxonomy semantics, so a noisy step prediction can cascade into an incorrect label. This is visible on Flash, where Step-then-Category reduces step accuracy (80.9 _→_ 73.0) and is further reduced with noisy violations. Magentic and Magentic<sup>_∗_</sup> follow a similar trend. In contrast, _τ_ -bench shows the opposite trend: Step-then-Cat.+Vio. is the best setting (54.0 step-index, 40.2 category) possibly due to more compact trajectories. _Step-then-Category can show substantial improvements unless the trajectories are too long._

_Table 6._ **Step and category accuracy under tolerance** Mean _±_ std over _n_ =3 runs.

|**Domain / Method**|**Step**<br>**Acc.** _↑_|**Acc@**<br>_±_1_↑_|**Acc@**<br>_±_3_↑_|**Acc@**<br>_±_5_↑_|**Avg.**<br>**Dist.** _↓_|**Critical**<br>**Cat.** _↑_|**Any**<br>**Cat.** _↑_|**Earliest**<br>**Cat.** _↑_|**Terminal**<br>**Cat.** _↑_|
|---|---|---|---|---|---|---|---|---|---|
|_τ_-Bench (Baseline)|32.2_±_3.2|36.8_±_1.6|50.6_±_1.6|66.7_±_1.6|5.7_±_0.8|25.3_±_1.6|35.6_±_1.6|31_±_0|23_±_3.2|
|_τ_-Bench (AGENTRX)|54_±_1.6|59.8_±_1.6|72.4_±_2.8|83.9_±_1.6|2.4_±_0.5|40.2_±_1.6|41.4_±_2.8|35.6_±_0|34.5_±_2.8|
|Flash (Baseline)|80.9_±_1.9|94.4_±_1.1|100|100|0.2|53.9_±_3|62.7_±_4.4|53.2_±_2.9|62.7_±_4.4|
|Flash (AGENTRX)|83.3_±_1.9|98.4_±_1.1|100|100|0.2|60.3_±_1.1|65.8_±_2.2|58_±_1.1|65.8_±_2.2|
|Magentic (Baseline)|31.8_±_1.8|40.9|50_±_3.7|53.3_±_2.1|22.1_±_8.4|36.3_±_1.8|58.3_±_1|34.1_±_1.8|40.1_±_1|
|Magentic (AGENTRX)|31.8_±_1.8|40.9_±_3.7|47.7_±_3.7|50.8_±_1|25.5_±_1|37.1_±_4.6|57.6_±_5.9|32.6_±_2.1|41.7_±_2.8|
|Magentic<sup>_∗_</sup>(Baseline)|42_±_1.8|60.5_±_3.5|69.1_±_4.6|69.1_±_4.6|5_±_0.7|39.5_±_3.5|60.5_±_4.6|39.5_±_4.6|46.9_±_4.6|
|Magentic<sup>_∗_</sup>(AGENTRX)|46.9_±_3.5|61.7_±_6.3|72.8_±_4.6|79_±_3.5|4.8_±_0.8|44.4_±_3|67.9_±_3.5|42_±_1.8|44.4_±_3|

## **4.7. Step & Category Accuracy Under Different Settings**

**Step accuracy increases as we allow more tolerance in the predicted step** (Acc@ _±_ 1 _<_ Acc@ _±_ 3 _<_ Acc@ _±_ 5) as shown in Table 6. We also report multiple category accuracy metrics because the right notion of correctness depends on use: critical failure accuracy measures whether the key failure is identified, while Any-failure is a weaker but often practical signal to flag some real failure even if the precise cause is missed. These results indicate that **our framework can be effectively used for agent failure attribution across a range of scenarios.**

# **5. Related Work**

**Benchmarks for Agent Evaluation.** Recent work has introduced benchmarks that cover tool execution, web interaction, and assistant behavior (Barres et al., 2025; Yao et al., 2022; Drouin et al., 2024). AgentBench Liu et al. (2023) evaluates LLM agents across eight interactive environments that span code-centric, game-based, and web-based settings, including domains like operating systems and databases. WebArena Zhou et al. (2023) evaluates long-horizon web agents on real websites, testing whether they can convert language instructions into executable browser interactions. GAIA Mialon et al. (2023) targets general assistants with problems that are easy for humans but often require multistep tool use, and has been used to evaluate generalist agent systems. However, these benchmarks primarily report terminal success and do not include gold failure-attribution annotations. Our dataset annotates the first unrecoverable critical step and assigns a failure category from a crossdomain taxonomy.

**Improving Reliability of Agents.** Koohestani (2025) provides runtime assurance for AI agents by monitoring executions against learned behavioral models. Recent work also translates plans into formal representations and applies model checking to validate alignment between an agent’s behavior and its plan (Ramani et al., 2025; Zhang et al., 2025b). These methods primarily target runtime enforce-

ment or plan-conformance checks; our focus is to diagnose trajectories by extracting constraints from tools, policies, and the observed prefix, and use the resulting violations as evidence for failure attribution. A separate line of work improves agent performance through feedback and selfcorrection. Self-Refine (Madaan et al., 2023) formalizes an iterative loop in which an LLM generates critiques of its own outputs and applies revisions, showing gains across diverse tasks without additional training. Critic (Gou et al., 2024) uses external tools like search and code execution as feedback to verify and repair model outputs. Feedback is also used at training time to improve agents. Reflection and reinforced self-training methods, such as ReReST (Dou et al., 2025), iteratively curate better trajectories using reflection and reward-based filtering.

**LLM Evaluation of Agent Trajectories.** LLM-as-a-Judge uses state-of-the-art LLMs to score outputs against rubrics, reducing the cost of human evaluation. Gu et al. (2025) systematizes judge design and mitigation strategies for inconsistency and bias. Agent-as-a-Judge Zhuge et al. (2024) uses tool-using agents to evaluate other agents, arguing benefits over single-shot judges. AGENTRX is compatible with these paradigms, but shifts the judge from scoring to diagnosis by providing a step-indexed violation log as evidence.

# **6. Conclusion and Limitations**

Our failure taxonomy covers three diverse domains and annotations from three independent annotators. However, it may not cover all failure modes in other agentic domains and may require extension. Sometimes AGENTRX can be misled when validation signals are weak or contain false positives. This motivates future work on identifying the smallest set of high-quality signals needed to separate true failures from noisy flags. Although AGENTRX relies on LLM calls, it reduces manual effort for failure attribution and is easier to scale. AGENTRX also offers cost-efficient one-shot constraint generation as an alternative. We view both the framework and the dataset as a step toward systematic, evidence-based failure attribution in AI agents.

# **Impact Statement**

This paper is focused on making AI agents easier to debug. As agents increasingly run multi-step workflows: calling tools, changing state, and coordinating sub-agents failures are difficult. They can be wrong tool calls, skipped confirmations, misread outputs, or decisions that push the run onto a path it never recovers from. Our goal is to help developers pinpoint where the run first became unrecoverable and why, using evidence already present in execution traces.

The main upside is practical: faster debugging, clearer accountability, and safer iteration. A system that can surface the root cause and connect it to trace evidence can make it easier to measure reliability. The benchmark and taxonomy also encourage better instrumentation and more careful thinking about what should be logged and what should be checked.

There are also risks. Better diagnosis does not automatically mean agents are safe to deploy, and explanations can be over-trusted if people treat them as guarantees. Trajectory data can also contain sensitive information depending on the application, so anyone using these methods should handle logs carefully and follow appropriate privacy practices. Finally, a taxonomy learned from existing benchmarks can inherit their blind spots, so it should be treated as a starting point rather than a complete map of real-world failures. Overall, we expect this work to be most useful as an Agentic debugging framework: it helps find problems earlier and fix them proactively, but it does not replace careful testing.

# **References**

- Barres, V., Dong, H., Ray, S., Si, X., and Narasimhan, K. Evaluating conversational agents in a dual-control environment. _arXiv preprint arXiv:2506.07982_ , 2025.

- Brynjolfsson, E., Li, D., and Raymond, L. R. Generative ai at work. Working Paper 31161, National Bureau of Economic Research, April 2023. URL http://www. nber.org/papers/w31161.

- Dou, Z.-Y., Yang, C.-F., Wu, X., Chang, K.-W., and Peng, N. Re-rest: Reflection-reinforced self-training for language agents, 2025. URL https://arxiv.org/abs/24 06.01495.

- Drouin, A., Gasse, M., Caccia, M., Laradji, I. H., Del Verme, M., Marty, T., Boisvert, L., Thakkar, M., Cappart, Q., Vazquez, D., et al. Workarena: How capable are web agents at solving common knowledge work tasks? _arXiv preprint arXiv:2403.07718_ , 2024.

- Fourney, A., Bansal, G., Mozannar, H., Tan, C., Salinas, E., Zhu, E. E., Niedtner, F., Proebsting, G., Bassman, G., Gerrits, J., Alber, J., Chang, P., Loynd, R., West,

R., Dibia, V., Awadallah, A., Kamar, E., Hosn, R., and Amershi, S. Magentic-one: A generalist multi-agent system for solving complex tasks. Technical Report MSRTR-2024-47, Microsoft, November 2024. URL https: //www.microsoft.com/en-us/research/p ublication/magentic-one-a-generalis t-multi-agent-system-for-solving-com plex-tasks/.

- Glaser, B. and Strauss, A. _Discovery of grounded theory: Strategies for qualitative research_ . Routledge, 2017.

- Gou, Z., Shao, Z., Gong, Y., Shen, Y., Yang, Y., Duan, N., and Chen, W. Critic: Large language models can self-correct with tool-interactive critiquing, 2024. URL https://arxiv.org/abs/2305.11738.

- Gu, J., Jiang, X., Shi, Z., Tan, H., Zhai, X., Xu, C., Li, W., Shen, Y., Ma, S., Liu, H., Wang, S., Zhang, K., Wang, Y., Gao, W., Ni, L., and Guo, J. A survey on llm-as-a-judge, 2025. URL https://arxiv.org/abs/2411.1 5594.

- Koohestani, R. Agentguard: Runtime verification of ai agents, 2025. URL https://arxiv.org/abs/25 09.23864.

- LinkedIn. Hiring assistant for LinkedIn recruiter & jobs. ht tps://business.linkedin.com/talent-s olutions/hiring-assistant, 2024. Accessed: 2025-01-28.

- Liu, X., Yu, H., Zhang, H., Xu, Y., Lei, X., Lai, H., Gu, Y., Ding, H., Men, K., Yang, K., et al. Agentbench: Evaluating llms as agents. _arXiv preprint arXiv:2308.03688_ , 2023.

- Madaan, A., Tandon, N., Gupta, P., Hallinan, S., Gao, L., Wiegreffe, S., Alon, U., Dziri, N., Prabhumoye, S., Yang, Y., Gupta, S., Majumder, B. P., Hermann, K., Welleck, S., Yazdanbakhsh, A., and Clark, P. Self-refine: Iterative refinement with self-feedback, 2023. URL https:// arxiv.org/abs/2303.17651.

- Mialon, G., Fourrier, C., Wolf, T., LeCun, Y., and Scialom, T. Gaia: a benchmark for general ai assistants. In _The Twelfth International Conference on Learning Representations_ , 2023.

- Pan, M. Z., Cemri, M., Agrawal, L. A., Yang, S., Chopra, B., Tiwari, R., Keutzer, K., Parameswaran, A., Ramchandran, K., Klein, D., et al. Why do multiagent systems fail? In _ICLR 2025 Workshop on Building Trust in Language Models and Applications_ , 2025.

- Ramani, K., Tawosi, V., Alamir, S., and Borrajo, D. Bridging llm planning agents and formal methods: A case study in plan verification, 2025. URL https://arxiv.or g/abs/2510.03469.

- Yao, S., Chen, H., Yang, J., and Narasimhan, K. Webshop: Towards scalable real-world web interaction with grounded language agents. _Advances in Neural Information Processing Systems_ , 35:20744–20757, 2022.

- Yao, S., Shinn, N., Razavi, P., and Narasimhan, K. _τ_ -bench: A benchmark for tool-agent-user interaction in real-world domains, 2024. URL https://arxiv.org/abs/ 2406.12045.

- Zhang, S., Yin, M., Zhang, J., Liu, J., Han, Z., Zhang, J., Li, B., Wang, C., Wang, H., Chen, Y., et al. Which agent causes task failures and when? on automated failure attribution of llm multi-agent systems. _arXiv preprint arXiv:2505.00212_ , 2025a.

- Zhang, X., Mittal, T., Bansal, C., Wang, R., Ma, M., Ren, Z., Huang, H., and Rajmohan, S. Flash: A workflow automation agent for diagnosing recurring incidents. October 2024. URL https://www.microsoft.com/ en-us/research/publication/flash-a-w orkflow-automation-agent-for-diagnos ing-recurring-incidents/.

- Zhang, Y., Cai, Y., Zuo, X., Luan, X., Wang, K., Hou, Z., Zhang, Y., Wei, Z., Sun, M., Sun, J., Sun, J., and Dong, J. S. Position: Trustworthy AI agents require the integration of large language models and formal methods. In Singh, A., Fazel, M., Hsu, D., Lacoste-Julien, S., Berkenkamp, F., Maharaj, T., Wagstaff, K., and Zhu, J. (eds.), _Proceedings of the 42nd International Conference on Machine Learning_ , volume 267 of _Proceedings of Machine Learning Research_ , pp. 82441–82459. PMLR, 13–19 Jul 2025b. URL https://proceedings.ml r.press/v267/zhang25ds.html.

- Zhou, S., Xu, F. F., Zhu, H., Zhou, X., Lo, R., Sridhar, A., Cheng, X., Ou, T., Bisk, Y., Fried, D., et al. Webarena: A realistic web environment for building autonomous agents. _arXiv preprint arXiv:2307.13854_ , 2023.

- Zhuge, M., Zhao, C., Ashley, D., Wang, W., Khizbullin, D., Xiong, Y., Liu, Z., Chang, E., Krishnamoorthi, R., Tian, Y., et al. Agent-as-a-judge: Evaluate agents with agents. _arXiv preprint arXiv:2410.10934_ , 2024.

# **A. Additional Experiments**

_Table 7._ We evaluate AGENTRX by adding checklist and synthetic few-shot examples for _τ_ -bench (one-shot). Mean _±_ std over _n_ =3.

|**Method**|**Step-index acc. (%)**|_↑_**Category acc. (%)**_↑_|
|---|---|---|
|Baseline|32.2_±_3.2|25.3_±_1.6|
|Baseline + Viol.|47.1_±_1.6|37.9_±_2.8|
|Examples|33.3_±_1.6|31.0_±_2.8|
|Examples + Viol.|49.4_±_1.6|46.0_±_1.6|

_Table 8._ We ablate AGENTRX by enabling and disabling natural language (NL) checks during constraint generation for _τ_ -bench (step-by-step). Mean _±_ std over _n_ =3.

|**Method**|**Step-index acc. (%)**|_↑_**Category acc. (%)**_↑_|
|---|---|---|
|Baseline|32.2_±_3.2|25.3_±_1.6|
|Without NL Check Viol.|<br>36.8_±_1.6|32.2_±_3.2|
|With NL Check Viol.|41.4_±_2.8|35.6_±_1.6|

_Table 9._ **o3 judge ablation for** _τ_ **-bench.** We compare evidence sources for root-cause attribution: violations-only, violations+taxonomy-checklist, checklist-only, and the baseline judge. Metrics: step-index accuracy and root-cause category accuracy (mean _±_ std over _n_ =3 runs).

|**Metric**|**Baseline**|**Viol.**|**Chk.**|**Chk.+Viol.**|
|---|---|---|---|---|
|Step Acc.|_↑_32.2_±_3|37.9_±_2|33.3_±_1.6|41.4_±_3|
|Cat. Acc.|_↑_25.3_±_2|34.5_±_3|29.9_±_4.3|35.6_±_2|

# **B. Root-Cause Attribution VS First Failure**

The following trajectory demonstrates why root-cause failure attribution is the better objective for evaluating agent reliability. The Who&When dataset labels step 3 as failure citing that “WebSurfer’s inability to reliably access the requested documents resulted in the overall task failure, as the necessary time span data could not be extracted or compared. This underscores the need for enhanced fallback mechanisms and more robust search strategies.” This diagnosis misses a key fact: the agent recovers from this early access failure, so it is not the cause of the eventual outcome. We annotate the key failure as shown below:

- { "trajectory_id": "5f982798-16b9-4051-ab57 _⌋ �→_ -cfc7ebdb2a91",

- "failures": [ {

   - "failure_id": 1,

   - "step_number": 13,

   - "step_reason": "Websurfer could not _�→_ download a PDF file and search _�→_ throught it which was an _�→_ instruction given by

   - _�→_ Orchestrator.",

   - "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "Instruction not _�→_ followed, the agent did not _�→_ download and search through the _�→_ PDF file as instructed",

- "failed_agent": "Websurfer"

- }, { "failure_id": 2,

   - "step_number": 17,

- "step_reason": "Websurfer could not _�→_ download a PDF file and search

   - throught it which was an

- _�→_

   - instruction given by

- _�→_

- _�→_ Orchestrator",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "Websurfer could _�→_ not download a PDF file and

   - search throught it which was an instruction given by

- _�→_

- _�→_

      - Orchestrator",

   - _�→_

- "failed_agent": "Websurfer"

- },

{

"failure_id": 3,

- "step_number": 33,

- "step_reason": "FileSurfer

- _�→_ hallucinated. It downloaded the

   - file at '/workspace/workspace/htt _⌋_ p:/export.arxiv.org/pdf/2007.xx'

- _�→_

- _�→_

   - but later attempted to read a

- _�→_

   - non-existent file

- _�→_

   - 'file:///workspace/path_to_july_2 _⌋_

- _�→_

   - 020_paper.pdf'.",

- _�→_

- "failure_category": "Invention of new _�→_ information",

- "category_reason": "FileSurfer

- _�→_ hallucinated. It downloaded the

- _�→_ file at '/workspace/workspace/htt _⌋ �→_ p:/export.arxiv.org/pdf/2007.xx'

   - but later attempted to read a

- _�→_

   - non-existent file

- _�→_

   - 'file:///workspace/path_to_july_2 _⌋_

- _�→_

   - 020_paper.pdf'.",

- _�→_

- "failed_agent": "FileSurfer"

}

],

"root_cause": {

- "failure_id": 3,

- "reason_for_root_cause": "The Orchestrator _�→_ was able to recover from earlier

   - errors but the FileSurfer

- _�→_

   - hallucination was a critical failure

- _�→_

   - that prevented further progress."

- _�→_

- },

- "failure_summary": "The agent could not _�→_ download and read the specified PDF _�→_ file due to a hallucination by the _�→_ FileSurfer agent."

}

# **C. Failure Annontation for Magentic One**

An example annotation for a magentic trajectory in our benchmark. We annotate a total of 22 failures in this trajectory as shown:

- { "trajectory_id": "16d825ff-1623-4176-a5b5 _⌋ �→_ -42e0f5c2b0ac",

- "failures": [

{

- "failure_id": 1,

- "step_number": 5,

"step_reason": "WebSurfer did not get

- _�→_ arrival time information which is _�→_ important",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not _�→_ get arrival time information which _�→_ is important",

"failed_agent": "WebSurfer"

},

{

"failure_id": 2,

"step_number": 13,

- "step_reason": "WebSurfer did scroll _�→_ but did not get information for _�→_ specified date",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did _�→_ scroll but did not get information _�→_ for specified date",

"failed_agent": "WebSurfer"

- },

{

"failure_id": 3,

"step_number": 17,

"step_reason": "WebSurfer did not get _�→_ information for specified date",

   - "failure_category": "Instruction/Plan _�→_ Adherence Failure",

   - "category_reason": "WebSurfer did not _�→_ get information for specified _�→_ date",

- "failed_agent": "WebSurfer"

- },

- {

- "failure_id": 4,

- "step_number": 25,

- "step_reason": "WebSurfer did not get

Failures #1–#2 were recoverable because the orchestrator successfully re-routed execution when WebSurfer stalled. It explicitly acknowledged missing information and switched to FileSurfer, whose file-centric capabilities are better suited for handling PDFs. In other words, the system demonstrated a working fallback: early retrieval failures triggered tool substitution and continued progress rather than terminal collapse. Failure #3 was the key failure because FileSurfer saved file to the wrong path and subsequent operations were based on non-existent data, making recovery impossible.

- _�→_ information for specified date",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not get

- _�→_ information for specified date",

- "failed_agent": "WebSurfer" },

{

"failure_id": 5,

- "step_number": 29,

- "step_reason": "WebSurfer did not get

- _�→_ information for specified date",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not _�→_ get information for specified _�→_ date",

"failed_agent": "WebSurfer"

},

{

"failure_id": 6,

"step_number": 33,

"step_reason": "WebSurfer did not get _�→_ information for specified date", "failure_category": "Instruction/Plan

- _�→_ Adherence Failure",

"category_reason": "WebSurfer did not get _�→_ information for specified date", "failed_agent": "WebSurfer" }, {

"failure_id": 7,

"step_number": 37,

"step_reason": "WebSurfer did not get _�→_ information for specified date",

"failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not _�→_ get information for specified _�→_ date",

"failed_agent": "WebSurfer"

},

{ "failure_id": 8,

"step_number": 41,

"step_reason": "WebSurfer did not get _�→_ information for specified date",

"failure_category": "Instruction/Plan _�→_ Adherence Failure",

"category_reason": "WebSurfer did not get _�→_ information for specified date", "failed_agent": "WebSurfer" }, {

"failure_id": 9,

"step_number": 56,

"step_reason": "WebSurfer did not get _�→_ information for specified date",

"failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not _�→_ get information for specified

- _�→_ date",

- "failed_agent": "WebSurfer"

},

{

"failure_id": 10, "step_number": 60,

"step_reason": "WebSurfer did not get _�→_ information for specified date",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "WebSurfer did not _�→_ get information for specified

   - date",

- _�→_

},

{

},

{

},

{

},

{

"failure_id": 10,

"step_number": 60,

"step_reason": "WebSurfer did not get _�→_ information for specified date", "failure_category": "Instruction/Plan _�→_ Adherence Failure",

"category_reason": "WebSurfer did not _�→_ get information for specified

- _�→_ date",

"failed_agent": "WebSurfer"

"failure_id": 11,

"step_number": 66,

- "step_reason": "Orchestrator try to _�→_ contact through email which might _�→_ not be good strategy",

"failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator try _�→_ to contact through email which _�→_ might not be good strategy",

"failed_agent": "Orchestrator"

"failure_id": 12,

"step_number": 70,

"step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

"failure_id": 13,

"step_number": 74,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

"failure_id": 14,

- "step_number": 78,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

"failed_agent": "WebSurfer"

},

{

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

- },

{

"failure_id": 15,

"step_number": 82,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

},

{

"failure_id": 16, "step_number": 86,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

}, {

"failure_id": 17, "step_number": 90,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not

- _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent

   - again trying to email which is not

- _�→_

   - good strategy",

- _�→_

- "failed_agent": "Orchestrator"

},

{

"failure_id": 18,

"step_number": 101,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not

   - good strategy",

- _�→_

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not

   - good strategy",

- _�→_

},

{

"failure_id": 19,

"step_number": 105,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

},

{

"failure_id": 20,

"step_number": 109,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

},

{

"failure_id": 21,

"step_number": 116,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email which is not _�→_ good strategy",

- "failed_agent": "Orchestrator"

},

{

"failure_id": 22,

"step_number": 120,

- "step_reason": "Orchestrator did not _�→_ properly interpret user intent _�→_ again trying to email/phone which _�→_ is not good strategy",

- "failure_category": "Intent Plan _�→_ Misalignment",

- "category_reason": "Orchestrator did _�→_ not properly interpret user intent _�→_ again trying to email/phone which _�→_ is not good strategy",

- "failed_agent": "Orchestrator"

}

"root_cause": {

   - "failure_id": 1,

- "failed_agent": "Orchestrator"

- "reason_for_root_cause": "The primary root _�→_ cause of the failures in this

   - trajectory is the WebSurfer's

- _�→_

   - inability to retrieve specific arrival

- _�→_

   - time information for the specified

- _�→_

- _�→_ date. This lack of crucial information

- _�→_ led to a series of misinterpretations _�→_ by the Orchestrator, which repeatedly _�→_ attempted to contact through email a

   - strategy misaligned with the user's

- _�→_

   - intent. The cascading effect of these initial shortcomings resulted in multiple failures throughout the trajectory."

- _�→_

- _�→_

- _�→_

- _�→_

}

- }

- "step_reason": "At step 3, the assistant _�→_ agent did not authenticate user _�→_ information before proceeding to _�→_ provide information about available

   - t-shirts",

- _�→_

- "failure_category": "Instruction Adherence _�→_ Failure",

- "category_reason": "The assistant agent _�→_ did not follow the expected policy of _�→_ authenticating user information before _�→_ providing product details.",

- "failed_agent": "Assistant"

- }, {

- "failure_id": 2,

- "step_number": 7,

- "step_reason": "At step 7, the agent did _�→_ not correctly count the number of _�→_ available t-shirts from the tool call

# **D. Agent Trajectories for each category of our failure taxonomy across domains**

## **D.1. System Failure (Flash)**

{

"trajectory_id":

_�→_ "9_withouths_tip_session_2_417931231", "failures": [ { "failure_id": 1,

"step_number": 3,

"step_reason": "KustoApiError: Error

   - getting schema for

- _�→_

   - Cluster='https://azcore1.southeastasi _⌋_

- _�→_

   - a.kusto.windows.net/': Failed to

- _�→_

- _�→_ connect to the remote cluster:

   - InternalServiceError

- _�→_

- _�→_ (520-UnknownError) followed by a

- _�→_ SyntaxError of the Kusto query",

- "failure_category": "System Failure",

- "category_reason": "Connection failure

- _�→_ error, system error + syntax error",

- "failed_agent": "KustoAgent"

   - result.",

- _�→_

- "failure_category": "Misinterpretation of _�→_ Tool Output",

- "category_reason": "The assistant _�→_ misinterpreted the output from the _�→_ tool call, leading to an incorrect _�→_ count of available t-shirts.",

- "failed_agent": "Assistant"

}

],

- "root_cause": {

   - "failure_id": 2,

   - "reason_for_root_cause": "The assistant _�→_ finally did authenticate before _�→_ providing user specific information. _�→_ The incorrect count does not

   - _�→_ correspond with ground truth

   - _�→_ output."

},

   - "failure_summary": "The agent did not _�→_ correctly count the number of _�→_ available t-shirts from the tool call _�→→_ result."

   - _�→→_ }

- } ], "root_cause": { "failure_id": 1, "reason_for_root_cause": "Connection _�→_ failure error, system error + _�→_ syntax error"

- }, "failure_summary": "System failure + _�→_ Syntax errors"

- }

## **D.3. Instruction Adherence Failure (Flash)**

## **D.2. Misinterpretation of Tool Output (** _τ_ **-bench)**

"trajectory_id": 2, "failures": [ { "failure_id": 1, "step_number": 3,

{ "trajectory_id": _�→_ "7_withhs_tip_session_2_424614956", "failures": [ { "failure_id": 1, "step_number": 4,

- "step_reason": "The actual solution is to

- _�→_ \"Delete the VM through the provided _�→_ link, or contact the resource owner to

   - delete it.\" The model's answer does

- _�→_

   - not explicitly suggest deleting the VM

- _�→_

   - or contacting the resource owner to do

- _�→_

- _�→_ so. However, it does guide the user to _�→_ manually inspect and clean up any

   - lingering VMs or resources, which

- _�→_

   - partially aligns with the intent of

- _�→_

   - deleting the resource. The model fails

- _�→_

   - to mention using a provided link or

- _�→_

   - directly contacting the resource

- _�→_

   - owner, which are key steps in the

- _�→_

   - actual solution. Thus, while there is

- _�→_

   - some overlap specifically the

- _�→_

- _�→_ suggestion to clean up resources the _�→_ solution is incomplete and misses key _�→→_ instructions.",

- _�→→_ instructions.",

- "failure_category": "Instruction/Plan _�→_ Adherence Failure",

- "category_reason": "incomplete/absent _�→_ conclusion/mitigation step and also

_�→_ did not provide the Azure link.", "failed_agent": "Orchestrator" } ], "root_cause": { "failure_id": 1,

"reason_for_root_cause": _�→→_ "incomplete/absent

- _�→→_ "incomplete/absent _�→_ conclusion/mitigation step and also _�→_ did not provide the Azure home link"

- },

- "failure_summary": "The model's answer _�→_ does not follow the plan perfectly"

- }

- "failure_category": "Invention of New

- _�→_ Information",

- "category_reason": "hallucination of

- _�→_ link",

"failed_agent": "GeneralAssistant" } ], "root_cause": {

- "failure_id": 1,

"reason_for_root_cause": "hallucination

   - of python script + link"

- _�→_

},

- "failure_summary": "hallucination, extra _�→_ steps executed"

- }

## **D.5. Intent Not Supported (Magentic)**

{ "trajectory_id": "a1e91b78-d3d8-4675-bb8d _⌋_

   - -62741b4b68a6",

- _�→_

- "failures": [

{

- "failure_id": 1,

- "step_number": 5,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not _�→_ supported",

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failed_agent": "Websurfer"

- },

- {

- "failure_id": 2,

## **D.4. Invention of New Information (Flash)**

{ "trajectory_id": _�→_ "9_withouths_tip_session_1_445308210", "failures": [ { "failure_id": 1, "step_number": 3,

- "step_reason": "Step 3 incorrect query, _�→_ even though the Kusto query returned _�→_ None; the agent tried with a new _�→_ invented python script.",

- "failure_category": "Invention of New _�→_ Information",

- "category_reason": "hallucination of _�→→_ python script",

- _�→→_ python script",

- "failed_agent": "Coder" }, {

"failure_id": 2,

"step_number": 5,

- "step_reason": "The GeneralAssistant came _�→_ up with a link https://portal.azure.c _⌋ �→_ om/#search/152076538 instead of

   - providing the home page as per the

- _�→_

   - plan.",

- _�→_

- "step_number": 9,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not

   - supported",

- _�→_

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failed_agent": "Websurfer"

- },

{

- "failure_id": 3,

- "step_number": 13,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not

   - supported",

- _�→_

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failed_agent": "Websurfer"

- },

- {

- "failure_id": 4,

- "step_number": 17,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not _�→_ supported",

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it

   - could not do such actions",

- _�→_

- "failed_agent": "Websurfer" }, { "failure_id": 5,

- "step_number": 21,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not _�→_ supported",

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→→_ could not do such actions",

- _�→→_ could not do such

- "failed_agent": "Websurfer" }, { "failure_id": 6, "step_number": 25,

- "step_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→_ could not do such actions",

- "failure_category": "Intent not

   - supported",

- _�→_

- "category_reason": "Websurfer was asked to _�→_ take snapshot in youtube video but it _�→→_ could not do such actions",

_�→→_ could not do such "failed_agent": "Websurfer" } ], "root_cause": {

- "failure_id": 1,

- "reason_for_root_cause": "The first error _�→_ occurred when the Websurfer agent was _�→_ asked to take a snapshot in a YouTube

   - video, which is beyond its

- _�→_

- "task_instruction": "You are a business _�→_ analyst evaluating cloud storage _�→_ solutions. You need to compare the

- _�→_ current pricing plans for DataVault's _�→_ Pro and Enterprise tiers to make an _�→_ informed purchasing decision for your _�→_ company. Request the agent to find _�→_ and summarize the pricing information _�→_ including features and costs for both _�→_ tiers.",

- "trajectory_snippet": [

{

- "content": "Can you find the current _�→_ pricing plans for the Pro and _�→_ Enterprise tiers on the DataVault _�→_ cloud storage service? I need to _�→_ compare them for my business

- _�→_ decision.",

- "role": "human",

"index": 1

},

- { "content": "I'll help you find that _�→_ pricing information. Let me _�→_ navigate to DataVault's website _�→_ and locate their pricing

   - _�→_ details.",

   - "role": "Orchestrator",

   - "index": 2

},

   - { "content": "Please navigate to _�→_ DataVault's official website, find _�→_ the pricing page, and extract the _�→_ pricing and features for both Pro _�→_ and Enterprise tiers.",

   - "role": "Orchestrator (-> WebSurfer)", "index": 3

   - }, {

- _�→_ capabilities. Subsequent failures were

- _�→_

- _�→_

- },

   - repetitions of this unsupported

   - intent."

- "failure_summary": "The Websurfer agent _�→_ was repeatedly asked to perform an

- _�→_ action (taking snapshots in a YouTube

- _�→_ video) that it is not capable of

- _�→_ executing, leading to failure."

## **D.6. Guardrails Triggered (Magentic)**

{

}, {

},

{

},

{

- "content": "I navigated to

- _�→_ 'https://www.datavault.io/pricing _⌋_

- _�→_ '.\n\n**Detected blocking

mechanism:**\n\nThe page

- _�→_

   - displays:\n\n\"Access Denied\n\nYou do not have

- _�→_

- _�→_

- _�→_ permission to access this

- _�→_ page.\n\nReason: Our system has _�→_ detected that your request appears _�→_ to be coming from an automated bot

   - or script.\n\nTo protect our users

- _�→_

   - and prevent abuse, we require

- _�→_

- _�→_ human verification for pricing

   - information.\n\nError Code: 403 -

- _�→_

   - Forbidden (Bot

- _�→_

   - Detection)\nReference ID:

- _�→_

   - CDN-BLK-98472\n\nIf you are a

- _�→_

- _�→_ human user, please:\n1. Enable

- _�→_ JavaScript in your browser\n2. _�→_ Complete the CAPTCHA challenge _�→_ below\n3. Contact

   - support@datavault.io if this

- _�→_

   - problem persists\n\n[CAPTCHA

- _�→_

- _�→_ Challenge - Please verify you are _�→_ human]\n\nThis page is protected _�→_ by Cloudflare Bot _�→_ Management.\"\n\n**Navigation _�→_ blocked:** The website has _�→_ detected automated access and

- _�→_ requires human verification _�→_ (CAPTCHA) to proceed. I cannot _�→_ bypass this security measure as it _�→_ violates the site's terms of

- _�→_ service and my operational

   - guidelines.\n\nI am unable to

- _�→_

- _�→_ access the pricing information due _�→_ to the bot detection and CAPTCHA _�→_ requirement.",

- "role": "WebSurfer",

- "index": 4

- "content": "I see the direct access is _�→_ blocked. Let me try searching for _�→_ the pricing information from

- _�→_ alternative sources or cached _�→_ content.",

- "role": "Orchestrator",

- "index": 5

- "content": "Please search for

- _�→_ 'DataVault Pro Enterprise pricing _�→_ comparison' or try to find cached _�→_ or third-party sources that list _�→→_ this pricing information.",

- _�→→_ this pricing information.",

- "role": "Orchestrator (-> WebSurfer)",

- "index": 6

},

{

"content": "I performed a search for

- _�→_ 'DataVault Pro Enterprise pricing _�→_ comparison'.\n\nThe search results _�→_ show several third-party review _�→_ sites and comparison pages, but _�→_ when attempting to access detailed _�→_ pricing information, I encountered _�→_ similar restrictions:\n\n1.

- _�→_ **<sup>SoftwareReviews.com</sup> **<sup>-Pricing</sup> _�→_ details require login\n2.

- _�→_ **<sup>TechCompare.io</sup> **<sup>-Shows</sup> _�→_ \"Contact for pricing\" with no _�→_ specific numbers\n3. **G2.com** - _�→_ Lists features but pricing marked _�→_ as \"Available upon request\"\n4.

- _�→_ **<sup>Cachedpages</sup> **<sup>-Pricing</sup> _�→_ information is outdated (from _�→_ 2022) and may not be

- _�→_ current\n\nUnfortunately, _�→_ DataVault appears to keep their _�→_ Enterprise tier pricing behind _�→_ access controls across multiple _�→_ platforms. Most sources either _�→_ require account _�→_ registration/login, display

- _�→_ \"Contact sales\" instead of _�→_ actual prices, or have outdated _�→_ cached information.\n\nI cannot _�→_ retrieve the current, accurate _�→_ pricing information the user needs _�→_ due to these access

   - restrictions.",

- _�→_

- "role": "WebSurfer",

"index": 7

- "content": "I apologize, but I was _�→_ unable to retrieve the current _�→_ pricing information for _�→_ DataVault's Pro and Enterprise _�→_ tiers. The DataVault website has _�→_ bot detection and CAPTCHA

- _�→_ protection that prevents automated _�→_ access to their pricing page _�→_ (Error Code: 403 - Forbidden). _�→_ Additionally, alternative sources _�→_ like review sites and comparison _�→_ platforms either require login _�→_ credentials, show \"Contact sales _�→_ for pricing,\" or contain outdated _�→_ information.\n\nTo get this _�→_ information, you would need _�→_ to:\n1. Visit _�→_ https://www.datavault.io/pricing _�→_ directly in your browser\n2. _�→_ Complete the CAPTCHA

verification\n3. Or contact

- _�→_

- _�→_ DataVault's sales team at _�→_ support@datavault.io for _�→_ Enterprise pricing details\n\nThis _�→_ limitation is due to website _�→_ access restrictions, not a _�→_ capability issue with our search _�→→_ approach.",

- _�→→_ approach.",

- "role": "Assistant",

}

],

"index": 8

- "reason_for_category": "The category _�→_ is guardrails triggered because at _�→_ step 4, the WebSurfer agent

- _�→_ encountered an explicit access

- _�→_ denial when attempting to navigate _�→_ to DataVault's pricing page. The _�→_ website returned a 403 Forbidden _�→_ error with bot detection, _�→_ requiring CAPTCHA verification to _�→_ proceed. This is an external site _�→_ access restriction (Cloudflare Bot _�→_ Management) that blocks automated _�→_ agents from accessing the content. _�→_ The agent's plan was valid and _�→_ would have worked if this access _�→_ restriction were removed - it _�→_ correctly identified the target _�→_ URL and attempted proper _�→_ navigation. At step 7, alternative _�→_ approaches (searching third-party _�→_ sources, cached pages) also failed _�→_ due to similar access controls _�→_ (login requirements, paywalls). _�→_ This is not a malformed tool _�→_ invocation (the navigation _�→_ commands were correct), not an _�→_ infrastructure failure (the _�→_ systems are functioning as _�→_ designed), and not a planning _�→_ error (the approach was _�→_ appropriate). The failure is _�→_ purely due to external guardrails _�→_ designed to prevent automated _�→_ access to the pricing _�→_ information."

- "step_number": 27,

- "step_reason": "Assistant prematurely _�→_ called modify items tool on a pending _�→_ order which locked the order and the _�→_ user later on wasn't able to change _�→_ the backpack that he wanted.",

- "failure_category": "Intent-Plan _�→_ Misalignment",

- "category_reason": "Here, the plan _�→_ generated by the assistant is _�→_ incorrect, it should not have _�→_ prematurely called modify items tool _�→_ on a pending order before finalizing _�→_ all the items to be modified in the _�→_ user's order.",

- "failed_agent": "Assistant"

- }

],

- "root_cause": {

- "failure_id": 26,

- "reason_for_root_cause": "The task _�→_ instruction did not specify the type _�→_ of black lamp and since there are _�→_ multiple black lamps available, it led _�→_ to incorrect matching of the ground _�→_ truth actions. Did not recover from _�→_ the error."

},

- "failure_summary": "The task instruction _�→_ did not specify the type of black lamp _�→_ and since there are multiple black _�→_ lamps available, it wont be possible _�→_ to match the ground truth actions." }

}

## **D.8. Intent Plan Misalignment (** _τ_ **-bench)**

## **D.7. Underspecified User Intent (** _τ_ **-bench)**

{ "trajectory_id": 71, "failures": [ { "failure_id": 26,

"step_number": 24,

- "step_reason": "At step 24, the task _�→_ instruction did not specify the type _�→_ of black lamp and since there are

- _�→_ multiple black lamps available, it _�→_ wont be possible to match the ground

   - truth actions.",

- _�→_

- "failure_category": "Underspecified User _�→_ Intent",

- "category_reason": "The task instruction _�→_ is underspecified regarding the type _�→_ of black lamp to be added to the _�→_ cart.",

- "failed_agent": "User"

- },

{ "failure_id": 27,

{ "trajectory_id": 28, "failures": [ {

- "failure_id": 13,

- "step_number": 33,

- "step_reason": "At step 33, the assistant _�→_ mistakenly believes that it can cancel _�→_ a subset of a pending order which is _�→_ not allowed as per domain policy, as a _�→_ result the entire order got cancelled _�→_ instead of just the garden hose.",

- "failure_category": "Intent-Plan _�→_ Misalignment",

- "category_reason": "The assistant came up _�→_ with an incorrect plan based on a

- _�→_ wrong assumption that a subset of an _�→_ order can be cancelled which violates _�→_ the domain policy.",

- "failed_agent": "Assistant"

- }

],

- "root_cause": { "failure_id": 13,

- "reason_for_root_cause": "The assistant _�→_ called cancel order on the entire

   - order which led to an incorrect final

- _�→_

   - outcome as compared to ground truth actions."

- _�→_

- _�→_

- },

- "failure_summary": "Assistant mistakenly _�→_ believes that it can cancel a subset _�→_ of the pending order which is not

   - allowed as per domain policy, as a

- _�→_

   - result the entire order got cancelled instead of just the hose."

- _�→_

- _�→_

- }

],

"root_cause": {

- "failure_id": 16,

- "reason_for_root_cause": "The assistant _�→_ called the modify order tool with _�→_ invalid arguments, leading to an

- _�→_ illegal tool call, and the agent did _�→_ not recover from this error."

- },

- "failure_summary": "Assistant used modify _�→_ order to cancel a subset of orders,

   - but modify order requires a

- _�→_

- _�→_ replacement which was not provided - _�→_ illegal tool call." }

## **D.9. Invalid Invocation (** _τ_ **-bench)**

{

"trajectory_id": 34, "failures": [ { "failure_id": 16, "step_number": 17,

- "step_reason": "At step 17, the assistant _�→_ uses modify order to cancel a subset

- _�→_ of orders, however modify orders also _�→_ need to have a replacement, which it _�→_ did not provide resulting in an

- _�→_ illegal tool call",

- "failure_category": "Invalid Invocation",

- "category_reason": "The assistant calls _�→_ the modify order tool with invalid _�→_ arguments.",

- "failed_agent": "Assistant" }, { "failure_id": 17,

"step_number": 21,

- "step_reason": "At step 21, the assistant _�→_ tries to bypass the modify tool

   - argument restriction by trying to

- _�→_

   - modify the item_id with the same

- _�→_

- _�→_ item_id in order to try to cancel it",

- "failure_category": "Invalid Invocation",

- "category_reason": "The assistant again _�→_ calls modify order tool with invalid _�→_ arguments trying to bypass the _�→→_ previous error.",

- _�→→_ previous error.",

- "failed_agent": "Assistant"

},

{

- "failure_id": 18,

- "step_number": 30,

- "step_reason": "At step 30, the user does _�→_ not ask the assistant to modify the _�→_ address of the current pending order _�→_ but is clearly a part of the overall _�→_ task instruction.",

- "failure_category": "Underspecified User _�→_ Intent",

- "category_reason": "The user does not ask _�→_ the assistant to do all the tasks as _�→_ mentioned in the task instruction, _�→_ hence underspecifying its intent.",

- "failed_agent": "User"

- }

# **E. Constraint Generation**

## **E.1. AGENTRX Constraint Schema Output Format**

{

- "assertion_name":

- _�→_ "string_unique_snake_case",

- "taxonomy_targets": [

- "Instruction/PlanAdherenceFailure",

- "InventionOfNewInformation",

- "InvalidInvocation",

- "MisinterpretationOfToolOutput", "IntentPlanMisalignment",

- "UnderspecifiedUserIntent",

- "IntentNotSupported",

- "GuardrailsTriggered",

- "SystemFailure"

### ],

- "constraint_type": "SCHEMA | PROTOCOL | _�→_ RELATIONAL_POST | PROVENANCE | _�→_ TEMPORAL | CAPABILITY | ANY",

- "event_trigger": {

   - "step_index": "*|int|range",

   - "substep_index": "*|int|range",

   - "role_name": "AgentName_or_*",

   - "content_regex": "regex_or_*",

   - "tool_name": "ToolName_or_*"

- },

- "check_hint": "deterministic procedure _�→_ description in 2-8 sentences",

- "examples": {

   - "pass_scenario": "short",

   - "fail_scenario": "short"

- },

- "check_type": "python_check|nl_check",

- "python_check": {

   - "function_name":

   - _�→_ "same_as_assertion_name",

   - "args": ["trajectory",

   - _�→_ "current_step_index"],

   - "code_lines": [

   - "def same_as_assertion_name(traject _⌋ �→_ ory, current_step_index):",

   - " \"\"\"Return True iff invariant _�→_ holds.\"\"\"",

   - " # parse

   - _�→_ trajectory[current_step_index],

   - _�→_ look back if needed",

}

      - " # MUST include at least one _�→_ explicit failure path: return

      - _�→_ False",

   - " return True"

   - ]

- },

"nl_check": {

- "judge_system_prompt_template":

- _�→_ "{NL_CHECK_JUDGE_SYSTEM_PROMPT}",

- "judge_user_prompt_template":

- _�→_ "template using {POLICY_TEXT}

   - {CURRENT_EVENT_JSON}

- _�→_

      - {WINDOW_EVENTS_JSON}",

   - _�→_

   - "judge_scope_notes": "what events are _�→_ in scope and what counts as _�→_ evidence",

   - "focus_steps_instruction": "REQUIRED: _�→_ Identify 2-4 specific events by _�→_ relative position and what to _�→_ check in each.",

   - "judge_rubric": ["objective criterion _�→_ 1", "objective criterion 2",

   - _�→_ "..."],

   - "rubric_evaluation_algorithm_template _⌋ �→_ ":

   - _�→_ "{RUBRIC_EVALUATION_ALGORITHM}",

   - "output_format_template":

- _�→_ "{OUTPUT_FORMAT}"

- }

- "judge_system_prompt_template": "You are a _�→_ strict compliance judge. Evaluate only _�→_ with evidence in the provided events _�→_ and the policy text.\nDo not infer

   - intent beyond explicit

- _�→_ intent beyond explicit _�→_ statements.\nCRITICAL: If required

- _�→_ evidence is missing or ambiguous, mark _�→_ the criterion as UNCLEAR.\nWhen

- _�→_ criteria are marked as UNCLEAR, they _�→_ are discarded and do NOT contribute to _�→_ a fail verdict.\nIf all criteria are _�→_ UNCLEAR or CLEAR_PASS after discarding _�→_ UNCLEAR ones, the final verdict is _�→_ PASS.\nOnly mark as CLEAR_FAIL when _�→_ there is definitive, unambiguous _�→_ evidence of violation.\nAmbiguity or _�→_ missing evidence leads to PASS, not _�→_ FAIL.\n",

- "judge_user_prompt_template": "POLICY _�→_ TEXT:\n{POLICY_TEXT}\n\nTASK INSTRUCT _⌋ �→_ ION:\n{TASK_INSTRUCTION}\n\nCURRENT

- _�→_ EVENT:\n{CURRENT_EVENT_JSON}\n\nCONTE _⌋ �→_ XT WINDOW

- _�→_ EVENTS:\n{WINDOW_EVENTS_JSON}\n\nEval _⌋ �→_ uate whether, before the current _�→_ write-action tool call (cancel,

- _�→_ modify, exchange, return, address _�→_ update), the assistant clearly _�→_ described the action and entity ID, _�→_ the user explicitly confirmed _�→_ proceeding with that action, and the _�→_ IDs match between description and the _�→_ tool call.",

- "judge_scope_notes": "Judge only within _�→_ the window of events provided.

## **E.2. Semantic Check generated by AGENTRX**

{

- "assertion_name": "explicit_user_confirma _⌋ �→_ tion_before_write_actions",

- "taxonomy_targets": [

- "Instruction/PlanAdherenceFailure",

- "IntentPlanMisalignment"

- ],

- "invariant_type": "TEMPORAL",

- "event_trigger": {

- "role_name": "assistant",

- "content_regex": "*",

- "tool_name": "cancel_pending_order|exch _⌋ �→_ ange_delivered_order_items|return_d _⌋ �→_ elivered_order_items|modify_pending _⌋ �→_ _order_.*|modify_user_address"

- },

- "check_hint": "Before any write-action _�→_ tool invocation (cancel, exchange,

   - return, modify order/address), verify

- _�→_

   - that the assistant previously

- _�→_

   - described the intended action and

- _�→_

   - target ID, and that the user

- _�→_

   - explicitly confirmed proceeding (e.g.,

- _�→_

- _�→_ 'yes', 'confirm', 'please proceed').",

      - Determine whether this specific

   - _�→_

   - _�→_ write-action tool call is properly _�→_ preceded by a clear assistant _�→_ explanation and explicit user

      - confirmation for the same action and same order_id/user_id.",

   - _�→_

   - _�→_

   - "focus_steps_instruction": "Focus on: (1) _�→_ The current assistant tool-call step _�→_ to identify which write-action tool is _�→_ being invoked and what identifier _�→_ (order_id or user_id) is present in _�→_ its arguments. (2) The immediately _�→_ prior user message to check for _�→_ explicit affirmation language

   - _�→_ indicating consent to proceed with the _�→_ described action. (3) Assistant _�→_ messages in the 2-3 steps before the _�→_ current tool call to see whether the _�→_ assistant described the intended _�→_ action

   - _�→_ (cancel/modify/exchange/return/address _�→_ update), reminded the user to confirm _�→_ all relevant items (for

   - _�→_ exchange/modify-items), and mentioned _�→_ the same identifier as in the tool _�→_ call.",

   - "judge_rubric": [

- "check_type": "nl_check",

- "python_check": {},

- "nl_check": {

- "There exists an assistant message _�→_ earlier in the context that

   - explicitly describes the specific

- _�→_

   - write action type (cancel, modify,

- _�→_

- _�→_ exchange, return, or address update) _�→_ and includes the same identifier _�→_ (order_id or user_id) that appears _�→_ in the current tool call

   - arguments.",

- _�→_

- "There exists a user message after the _�→_ assistant's action description and _�→_ before the current tool call that _�→_ contains explicit confirmation _�→_ language agreeing to proceed with _�→_ that specific action (e.g., 'yes', _�→_ 'confirm', 'please proceed', 'go _�→_ ahead').",

- "When the action is exchange or _�→_ modify-items, the assistant reminded _�→_ the customer to confirm they have _�→_ provided all items to be changed; _�→_ and the current tool call scope is _�→_ consistent with the items described _�→_ and confirmed."

- "output_format_template": "Output _�→_ Format:\nYou MUST return a JSON

   - response with this EXACT

- _�→_

- _�→_ structure:\n{\n \"verdict\": \"pass\" _�→_ or \"fail\",\n \"rubric_results\": _�→_ [\n {\n \"criterion_index\": _�→_ 0,\n \"criterion\": \"full text

- _�→_ of the criterion from the rubric\",\n

   - \"evaluation\": \"CLEAR_PASS\" or

- _�→_

   - \"CLEAR_FAIL\" or \"UNCLEAR\",\n

- _�→_

- _�→_ \"reasoning\": \"detailed explanation _�→_ of why this evaluation was chosen, _�→_ citing specific evidence from

   - events\"\n }\n // ... one entry

- _�→_

   - per rubric criterion\n ],\n

- _�→_

- _�→_ \"final_reasoning\": \"explanation of _�→_ how the verdict was determined from

- _�→_ rubric_results, stating which criteria _�→_ were discarded as UNCLEAR and which

   - criteria drove the final

- _�→_

   - decision\"\n}\n"

- _�→_

- }

- }

],

- "rubric_evaluation_algorithm_template": _�→_ "Rubric Evaluation Algorithm:\nStep 1: _�→_ For each criterion in the rubric, _�→_ evaluate whether it can be CLEARLY

- _�→_ judged as PASS or FAIL based solely on _�→_ the provided events.\n- Mark as _�→_ CLEAR_PASS if the criterion is

   - demonstrably satisfied by the

- _�→_

- _�→_ evidence\n- Mark as CLEAR_FAIL if the _�→_ criterion is demonstrably violated by _�→_ the evidence \n- Mark as UNCLEAR if: _�→_ insufficient events to judge, _�→_ criterion is ambiguous in this

- _�→_ context, or pass/fail cannot be _�→_ decisively determined\n\nStep 2: _�→_ Discard ALL criteria marked as UNCLEAR _�→_ from consideration.\n\nStep 3: _�→_ Determine final verdict:\n- If ANY _�→_ remaining criterion is CLEAR_FAIL ==

- _�→_ return verdict 'fail'\n- If ALL _�→_ remaining criteria are CLEAR_PASS (or _�→_ no criteria remain after discarding _�→_ UNCLEAR) == return verdict

   - 'pass'\n\nImportant: Only fail when

- _�→_

   - you have CLEAR evidence of failure. When in doubt, mark as UNCLEAR and discard.\n",

- _�→_

- _�→_

- _�→_

# **F. AGENTRX Violation Examples**

AGENTRX generates the following programmatic violation because the LLM-extracted “number of t-shirts” from the tool output does not match the true t-shirt count, so the check flags a mismatch.

- { "assertion_name": "tshirt_available_o _⌋ �→_ ptions_match_variants_count",

- "taxonomy_targets": [

      - "MisinterpretationOfToolOutput",

   - ],

"invariant_type": "RELATIONAL_POST",

"event_trigger": {

"step_index": 7,

- "role_name": "assistant",

"content_regex":

   - _�→_ "T-?shirt|t-?shirt|T-Shirt",

   - "tool_name": "*"

- },

- "check_hint": "When the assistant now _�→_ reports how many T-shirt options _�→_ are available, it should compute _�→_ the count from the variants field _�→_ of the latest get_product_details _�→_ result for the T-Shirt product. _�→_ Specifically, count how many

- _�→_ variant entries have available == _�→_ true and verify that this equals _�→_ the numeric count stated by the _�→_ assistant for 'available' T-shirt _�→_ options. If the assistant gives an _�→_ 'available options' count that _�→_ does not match this filtered _�→_ count, or gives any numeric _�→_ options count without a _�→_ corresponding get_product_details _�→_ result, the invariant fails.",

"check_type": "python_check",

"python_check": { "function_name": "tshirt_available_op _⌋ �→_ tions_match_variants_count", "args": [ "trajectory", "current_step_index" ], "code_lines": [

- "import re",

"import json",

"",

- "def tshirt_available_options_match_varia _⌋ �→_ nts_count(trajectory,

   - current_step_index):",

- _�→_

- " '''Verify that the assistant's stated _�→_ count of available T-shirt options _�→_ matches the tool response.'''",

- " print('Function: tshirt_available_op _⌋ �→_ tions_match_variants_count')",

- " ",

- " # Access the current step from the _�→_ trajectory IR format",

- " steps = trajectory.get('steps', [])", " if current_step_index >= _�→_ len(steps):",

- " raise

- _�→_ IndexError(f'current_step_index

- _�→_ {current_step_index} out of bounds for _�→_ {len(steps)} steps')",

- " ",

- " current_step =

- _�→_ steps[current_step_index]",

- " substeps =

- _�→_ current_step.get('substeps', [])",

- " ", " # Find assistant substep content", " assistant_content = None", " for ss in substeps:", " if ss.get('role') ==

- _�→_ 'assistant':",

- " assistant_content = _�→_ ss.get('content')",

- " break",

- " ",

- " if assistant_content is None:", " raise KeyError('Assistant content _�→_ at current step is missing.')",

- " print(f'Assistant content at step

   - {current_step_index}:

- _�→_

   - {assistant_content}')",

- _�→_

- " ",

- " # Try to extract a number near the _�→_ word 'available'",

- " match =

- _�→_ re.search(r'(\\d+)\\s+(?:available)', _�→_ assistant_content, _�→_ flags=re.IGNORECASE)",

- " if match:", " stated_count =

- _�→_ int(match.group(1))",

- " else:",

- " # Fallback: extract the first _�→_ integer in the content",

- " any_num = re.search(r'(\\d+)', _�→_ assistant_content)",

- " if not any_num:", " raise ValueError('No numeric _�→_ count found in assistant content to _�→_ verify.')",

- " stated_count = _�→_ int(any_num.group(1))",

- " ", " print(f'Extracted stated_count: _�→_ {stated_count}')",

- " ", " # Find the most recent _�→_ get_product_details tool response _�→_ prior to this step",

- " tool_response_content = None", " for idx in range(current_step_index - _�→_ 1, -1, -1):",

- " step = steps[idx]", " for ss in step.get('substeps', _�→_ []):",

- " if ss.get('role') == _�→_ 'tool':",

- " # Check if previous _�→_ assistant step called _�→_ get_product_details",

- " # Look at the assistant _�→_ step before this tool response",

- " if idx > 0:", " prev_step = steps[idx _�→_ - 1]",

- " for prev_ss in _�→_ prev_step.get('substeps', []):",

- " if _�→_ prev_ss.get('role') == 'assistant':",

- " try:", " calls = _�→_ json.loads(prev_ss.get('content', _�→_ ''))",

- " if _�→_ isinstance(calls, list):",

- " for _�→_ call in calls:",

- " _�→_ if call.get('function', _�→_ {}).get('name') == _�→_ 'get_product_details':",

- " _�→_ tool_response_content = _�→_ ss.get('content')",

- " _�→_ print(f'Found get_product_details tool _�→_ response at step {idx}')",

- " _�→_ break",

- " except _�→_ json.JSONDecodeError:",

- " pass", " if _�→_ tool_response_content:",

- " break", " if tool_response_content:", " break", " ", " if tool_response_content is None:", " raise KeyError('No prior _�→_ get_product_details tool response _�→_ found for verification.')",

- " ",

- " print(f'Raw tool response content:

- _�→_ {tool_response_content}')",

- " ",

   - # Parse JSON content",

- "

- " try:",

- " tool_response =

- _�→_ json.loads(tool_response_content)",

except Exception as e:",

- "

   1. **Instruction/Plan Adherence Failure** The agent fails to follow the directions or the agreed plan by ignoring directives and skipping policy steps. This covers both under-execution (missed steps) and over-execution (unplanned or unnecessary actions, e.g., extra tool calls) that deviate from the static plan, domain policy or orchestrator plan.

- " raise ValueError(f'Tool response

- _�→_ content is not valid JSON: {e}')",

- " ",

- " variants =

- _�→_ tool_response.get('variants')",

- " if variants is None:",

- " raise KeyError('Missing variants

- _�→_ in get_product_details tool

- _�→_ response.')",

- " if not isinstance(variants, dict):", " raise TypeError('variants should

- _�→_ be a dict.')",

- " ",

- " # Compute available count", " available_count = 0",

   2. **Invention of New Information:** The agent introduces, removes, or alters information that is not grounded in any available input, context, or tool output. This includes fabricating unsupported facts, hallucinating details, or omitting relevant information without justification.

   3. **Invalid Invocation:** The agent encounters errors triggered by inputs that cannot be parsed or validated e.g., Kusto syntax errors or tool calls with bad/missing arguments. Not involving wrong logic; just invalid inputs.

- " for k, v in variants.items():",

- " if not isinstance(v, dict):", " print(f'Skipping variant {k}:

- _�→_ not a dict')",

- " continue",

- " avail_flag = v.get('available',

- _�→_ None)",

- " print(f'Variant {k} available _�→_ flag: {avail_flag}')",

- " if avail_flag is True:",

" available_count += 1",

" ",

- " print(f'Computed available_count from _�→_ tool response: {available_count}')",

- " ", " result = (stated_count ==

- _�→_ available_count)",

- " print(f'Assertion result (stated == _�→_ computed): {result}')",

- " return result"

   - ],

- "nl_check": {}

- }

# **G. Judge Prompts**

## **G.1. Baseline Judge Prompt**

You are an Expert Failure-Categorization Judge. You will be provided with a trajectory of an agent’s interaction with a user. **Given:** a full trajectory of an agent’s conversation with the user (step-indexed)

**YOUR TASK:** is determine why the agent failed, which failure category applies from the taxonomy below and exactly which step index the failure occurred at. The failure taxonomy has the following categories:

4. **Misinterpretation of Tool Output / Handoff Failure:** The agent incorrectly reasons about its own or another agent’s tool output (like computation errors), leading to incorrect assumptions or actions. This also includes cases where the agent considered only partial tool output.

5. **Intent-Plan Misalignment:** The agent misreads the user’s goal or constraints and produces the wrong step sequence or structure. Covers both bad ordering/structure and plans aimed at the wrong objective.

6. **Underspecified User Intent:** The agent was unable to complete the task due to lack of complete information at any point in the trajectory/plan execution.

7. **Intent Not Supported:** The agent/user is asking to perform an action for which a tool is not available, like listening to an audio file.

8. **Guardrails Triggered:** The agent is blocked by safety policies or by external site access restrictions, preventing execution despite a valid plan. Examples include policy refusals (e.g., unsafe content, privacy-protected data), CAPTCHA/robot blocks, login/paywall/403/robots.txt denials, or site forbids automation. This is not an agent planning/execution error; it is an external/guardrail block.

9. **System Failure:** The agent faces a system connectivity issue while calling a particular tool like an endpoint not being reachable

10. **Inconclusive (USE SPARINGLY)** : If you are not able to classify the failure into any of the above categories, label it as inconclusive and create your own category.

## **How to Judge (Decision Procedure):**

1. **Step 1 - Locate the first failure:** Scan the trajectory step-by-step from the start. The first step where the agent deviates from the intended plan or emits an error is the first failure. Record the step index and a short failure note.

2. **Step 2 - Check if that failure was resolved** : Look ahead in the trajectory for evidence that the error was resolved. If yes → Resolved; if no such evidence → Not resolved.

3. **Step 3 - Decide and continue:**

   - If Resolved: continue scanning from the next step to find the next new failure, then repeat Step 2 for it.

   - If Not: treat this step as the root-cause failure for the run and assign the taxonomy at this step.

## **Output format (JSON):**

{

- "reason_for_failure": "string", "failure_case": "int 1-10",

- "reason_for_index": "string",

- "index": "int"

}

## **G.2. Checklist Baseline Judge Prompt**

You are an Expert Failure-Categorization Judge. You will be provided with a trajectory of an agent’s interaction with a user. **Given:** a full trajectory of an agent’s conversation with the user (step-indexed)

**YOUR TASK:** is determine why the agent failed, which failure category applies from the taxonomy below and exactly which step index the failure occurred at. The failure taxonomy has the following categories: YOUR TASK is determine why the agent failed, which failure category applies from the taxonomy below. and exactly which step index the failure occurred at. The failure taxonomy has the following categories:

1. **Instruction/Plan Adherence Failure** : Goal is correct, but the agent deviates from the required plan by ignoring directives and skipping steps despite having enough information. This covers both

under-execution (missed steps) and over-execution (unplanned or unnecessary actions, e.g., extra tool calls) that deviate from the static plan, domain policy or orchestrator plan. Checklist:

   - Can you state the user’s goal, and do the agent’s intent and end goal match that goal (i.e., the agent is not solving the wrong problem)?

   - Was all the required information already available at this step (user intent, required context, prior tool outputs)?

   - Is there a step where the ground-truth/policy requires an action (tool call, question, confirmation, ordering) and the agent did something different (skipped it / reordered it / added extra unneeded action)?

2. **Invention of New Information** : The agent introduces, removes, or alters information that is not grounded in any available input, context, or tool output. This includes fabricating unsupported facts, hallucinating details, or omitting relevant information. Checklist:

   - Can you pinpoint the exact invented/altered/omitted claim, value, or assumption the agent used?

   - Is that claim absent from all evidence available up to that step (user text, provided context, tool outputs)?

   - Did the agent rely on that claim to decide an

   - action or produce the failing conclusion (not just harmless wording)?

3. **Invalid Invocation** : Tool call fails because the request is ill-formed (missing args, wrong fields/types, malformed query, schema mismatch). Checklist:

   - At the failure step, did the agent attempt a

   - tool call with a concrete invocation payload/arguments?

   - Does the tool/runtime explicitly report a parse/validation/schema/syntax error for that call (e.g., missing field, invalid type, cannot parse, malformed query)?

   - Is the error NOT a network/timeout/service-

   - unavailable/endpoint-unreachable issue (infra/connectivity)?

   - Is the error NOT primarily a CAPTCHA/login/paywall refusal (access/guardrail block)?

4. **Misinterpretation of Tool Output / Handoff Failure** : The agent incorrectly reasons about its

own or another agent’s tool output, leading to incorrect assumptions or actions. This also includes cases where the agent considered only partial tool output. Checklist:

- Before (or at) the failure step, did the agent receive tool output or handoff output that is relevant to the failing decision?

   - Did the agent state or imply a specific reasoning derived from that tool output?

   - Does that reasoning contradict the tool output,

   - omit a crucial part, or reflect a clear computation/logic error relative to the output?

5. **Intent-Plan Misalignment** : Agent misunderstands the user’s intent/constraints and pursues the wrong objective or violates key constraints due to misunderstanding. Checklist:

   - Do the agent’s actions/plan optimize for a different goal OR violate a key constraint (not a minor wording/format issue)?

   - Is the misalignment due to misunderstanding of intent/constraints (rather than missing required info from the user/context/tool outputs)?

   - Is the misalignment not primarily caused by a

   - tool error (invalid invocation, infra failure, or access/guardrail block)?

6. **Underspecified User Intent** : The agent was unable to complete the task due to lack of complete information at any point in the trajectory/plan execution. Checklist:

   - Can you identify a specific missing piece of information that is required to proceed correctly (e.g., date, address, account id, item variant)?

   - Is that information absent from all evidence available up to that step (user text, provided context, and tool outputs)?

   - Did the agent fail because it proceeded without obtaining this information OR because it did not ask for it when needed?

7. **Intent Not Supported** : Requested action cannot be performed with available tools/capabilities. Checklist:

   - Is the user requesting an action that requires an external capability/tool (e.g., listen to audio, access a private system, perform a human action)?

   - Given the tool set available in this environment, is there no tool that can perform the requested action?

   - Is the failure not primarily caused by infrastructure/connectivity issues?

8. **Guardrails Triggered** : The agent is blocked by safety/RAI policies or by external site access restrictions, preventing execution despite a valid plan. Checklist: - Is there an explicit refusal/block signal (policy refusal, CAPTCHA, login required, 403, paywall, robots.txt, automation forbidden)?

   - Would the plan be feasible and correct if this

   - block were removed (i.e., the agent is not pursuing the wrong goal/constraints)?

   - Is the failure not primarily due to malformed tool invocation (schema/syntax/args validation error)?

   - Is the failure not primarily due to infrastructure/connectivity issues (timeouts, endpoint unreachable)?

9. **System Failure** : The agent faces a system connectivity issue while calling a particular tool like an endpoint not being reachable. Checklist:

   - At the failure step, did the agent attempt a

   - tool call or rely on a tool that should have been callable?

   - Is there an explicit infra/connectivity error signal (timeout, connection refused, DNS failure, endpoint unreachable, service unavailable, premature termination)?

   - Is the failure not primarily a parse/valida-

   - tion/schema/syntax error caused by malformed arguments?

10. **Inconclusive (USE SPARINGLY)** : None of 1- 10 clearly apply; must provide a custom category label. Checklist:

   - If labeling as 10, did you provide a non-empty custom category describing the failure type?

## **How to Judge (Decision Procedure):**

1. **Step 1 - Locate the first failure:** Scan the trajectory step-by-step from the start. The first step where the agent deviates from the intended plan or emits an error is the first failure. Record the step index and a short failure note.

2. **Step 2 - Check if that failure was resolved** : Look ahead in the trajectory for evidence that the error was resolved. If yes → Resolved; if no such evidence → Not resolved.

3. **Step 3 - Decide and continue:**

- If Resolved: continue scanning from the next step to find the next new failure, then repeat Step 2 for it.

- If Not: treat this step as the root-cause failure for the run and assign the taxonomy at this step.

## **Output format (JSON):**

{

- "taxonomy_checklist_reasoning":

- _�→_ "string",

- "reason_for_failure": "string",

- "failure_case": "int 1-10",

- "reason_for_index": "string", "index": "int"

}

## **G.3. AGENTRX Judge Prompt**

You are an Expert Failure-Categorization Judge. You will be provided with a trajectory of an agent’s interaction with a user. **Given:** a full trajectory of an agent’s conversation with the user (step-indexed)

**YOUR TASK:** is determine why the agent failed, which failure category applies from the taxonomy below and exactly which step index the failure occurred at. The failure taxonomy has the following categories:

1. **Instruction/Plan Adherence Failure** The agent fails to follow the directions or the agreed plan by ignoring directives and skipping policy steps. This covers both under-execution (missed steps) and over-execution (unplanned or unnecessary actions, e.g., extra tool calls) that deviate from the static plan, domain policy or orchestrator plan.

2. **Invention of New Information:** The agent introduces, removes, or alters information that is not grounded in any available input, context, or tool output. This includes fabricating unsupported facts, hallucinating details, or omitting relevant information without justification.

3. **Invalid Invocation:** The agent encounters errors triggered by inputs that cannot be parsed or validated e.g., Kusto syntax errors or tool calls with bad/missing arguments. Not involving wrong logic; just invalid inputs.

actions. This also includes cases where the agent considered only partial tool output.

5. **Intent-Plan Misalignment:** The agent misreads the user’s goal or constraints and produces the wrong step sequence or structure. Covers both bad ordering/structure and plans aimed at the wrong objective.

6. **Underspecified User Intent:** The agent was unable to complete the task due to lack of complete information at any point in the trajectory/plan execution.

7. **Intent Not Supported:** The agent/user is asking to perform an action for which a tool is not available, like listening to an audio file.

8. **Guardrails Triggered:** The agent is blocked by safety policies or by external site access restrictions, preventing execution despite a valid plan. Examples include policy refusals (e.g., unsafe content, privacy-protected data), CAPTCHA/robot blocks, login/paywall/403/robots.txt denials, or site forbids automation. This is not an agent planning/execution error; it is an external/guardrail block.

9. **System Failure:** The agent faces a system connectivity issue while calling a particular tool like an endpoint not being reachable

10. **Inconclusive (USE SPARINGLY)** : If you are not able to classify the failure into any of the above categories, label it as inconclusive and create your own category.

You are also provided a list of violations that have been generated through the trajectory through various constraints. Use these to help you identify the root cause category, failure step and agent. Static constraints have been generated through the domain policy and system prompt. Each static constraint is associated with a tool call to ensure it adheres to the domain policy. Dynamic constraints have been generated to cover computation checks, data accuracy, argument validity, and tool output consistency. Each constraints returns a boolean, and if it returns false, it indicates a violation. Note that some violations may be false positives and not all violations may be relevant to the root cause failure. Here are the list of violations:

4. **Misinterpretation of Tool Output / Handoff Failure:** The agent incorrectly reasons about its own or another agent’s tool output (like computation errors), leading to incorrect assumptions or

### **Executable check violation with grounded evidence**

================================ VIOLATION #1

2. **Step 2 - Check if that failure was resolved** : Look ahead in the trajectory for evidence that the error was resolved. If yes → Resolved; if no such evidence → Not resolved.

3. **Step 3 - Decide and continue:**

Step Index: 2

Assertion Name: kusto_invocation_r _⌋ �→_ equires_predefined_query_and_c _⌋ �→_ orrect_cluster Constraint Type: CAPABILITY Check Type: python_check Severity: medium

- If Resolved: continue scanning from the next step to find the next new failure, then repeat Step 2 for it.

- If Not: treat this step as the root-cause failure for the run and assign the taxonomy at this step.

Check Hint:

- When KustoAgent runs a query, it _�→_ must be a predefined query

- _�→_ present in the plan or prior _�→_ Orchestrator instruction, and

- _�→_ the query must be tailored to

   - the incident's cluster (no

- _�→_

   - placeholders like

- _�→_

   - TODO/TBD/<CLUSTER>). Verify

- _�→_

## **Output format (JSON):**

{

- "reason_for_failure": "string", "failure_case": "int 1-10",

- "reason_for_index": "string",

- "index": "int"

}

- _�→_ that a kusto code block exists

- _�→_ earlier and that the current

- _�→_ query's clusterName matches the

- _�→_ cluster parsed from the

- _�→_ incident description.

Evidence:

Current Event:

Role: KustoAgent Content:

**<sup>KustoQuery:</sup> ** let driftedSettingName =

- _�→_ 'VncEndpointCandidates';

semantic_query_matcher: True

Matched Substeps: Sub-index: 5 Role: KustoAgent

Taxonomy Targets:

- InvalidInvocation

- Instruction/PlanAdherenceFailu _⌋_

- _�→_ re

- IntentPlanMisalignment

## **How to Judge (Decision Procedure):**

1. **Step 1 - Locate the first failure:** Scan the trajectory step-by-step from the start. The first step where the agent deviates from the intended plan or emits an error is the first failure. Record the step index and a short failure note.
