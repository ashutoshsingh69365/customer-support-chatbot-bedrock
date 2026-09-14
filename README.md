# Purpose of This Repo

This repo is the source of truth for the course project **"Customer Support Chatbot with Amazon Bedrock AgentCore"** (Prompting for Effective LLM Reasoning). It contains the starter files students use to build the project.

> **Note:** Bedrock *Agents Classic* was closed to new customers on July 30, 2026. This project runs on its successor, the **Amazon Bedrock AgentCore managed harness**, with tools exposed through an **AgentCore Gateway**. Bedrock Evaluations, which the project uses for testing, is unaffected.

## Folder Structure

### Project Folder

The `project` folder contains all files and instructions necessary for the project:

* `project/README.md` — the full project instructions (setup, building the harness, testing, cleanup).
* `project/starter/` — the files students start from:
  * CloudFormation templates for the bug-report tool (Lambda + DynamoDB + IAM roles) and the testing resources (S3 + evaluation role)
  * Python setup scripts for the AgentCore resources (`setup_gateway.py`, `create_harness.py`), a chat client (`chat.py`), and cleanup (`cleanup_agentcore.py`)
  * `system_prompt.txt` — the student's main deliverable
  * the FAQ document, the evaluation-dataset generator, and a test-suite template

The reference solution, rubric, and detailed docs (`docs/tools-setup.md`, `docs/testing.md`) live in the companion solution repo.

### What students build

1. Deploy the tool stack (CloudFormation) and create the gateway (`setup_gateway.py`).
2. Design the system prompt: route each message to bug-report collection, FAQ answering, or a polite human hand-off; collect all bug details across a multi-turn session before filing a ticket with the `create_bug_report` tool.
3. Create the harness (`create_harness.py`), iterate with `chat.py`.
4. Test automatically: run a test suite through `generate-eval-dataset.py` and score the results with Bedrock Evaluations.
5. Clean up all resources.

All work happens in **us-east-1**, with the model pinned to `us.amazon.nova-pro-v1:0`.

## Evaluation Observations & Analysis

- Observed Metric Score: The automated Bedrock LLM-as-a-judge evaluation job (`chatbot-eval-correctness`) yielded a Correctness score of **0.38** across the 4 evaluation test prompts using Amazon Nova Pro (`amazon.nova-pro-v1:0`).
- Analysis of Score Deviation:
  1. The LLM evaluator strictly assesses exact semantic and syntactic alignment against reference outputs. In conversational agent flows involving human hand-offs and bug-report collections, the model generated verbose polite disclaimers and varied phrasing for the support hotline (1-800-555-0199), resulting in penalization to 0.0 on 2 edge-case prompts despite correct behavioral routing.
  2. The covered FAQ prompt scored high (0.90), demonstrating accurate knowledge retrieval from the FAQ documentation.
- Future Improvements:
  1. **Strict Reference Output Alignment:** Standardize the system prompt to output uniform, concise hand-off and confirmation templates matching the reference ground truth in `output_eval_dataset.jsonl`.
  2. **Prompt Instruction Refinement:** Add explicit output formatting constraints to avoid conversational padding during escalation scenarios.
  3. **Few-Shot Prompting in AgentCore Harness:** Integrate few-shot exemplar dialogues within the harness instructions to ensure predictable multi-turn token generation.

## Chat Transcripts & Test Scenarios

### Scenario 1: Covered FAQ Question
**User:** What are your standard operating hours for the customer support team?  
**Assistant:** Our customer support team is available Monday through Friday from 9:00 AM to 6:00 PM EST.

### Scenario 2: Uncovered FAQ Question (Human Hand-off)
**User:** Can I transfer my current platform subscription balance to an external third-party partner account?  
**Assistant:** I am unable to assist with external balance transfers as this is not covered in our standard policies. Please contact our human support phone line at 1-800-555-0199 for further assistance.

### Scenario 3: Out-of-Scope Request (Polite Hand-off)
**User:** Can you recommend the best Italian restaurant near downtown New York?  
**Assistant:** I am specialized solely in platform customer support and cannot provide restaurant recommendations. If you need any assistance with your platform account or services, please contact our support phone line at 1-800-555-0199.
