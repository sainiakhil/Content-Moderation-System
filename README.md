# AI-Powered Content Moderation Pipeline

This project implements a sophisticated, multi-step content moderation pipeline using Python and the [LangGraph](https://github.com/langchain-ai/langgraph) framework. It is designed to automatically check user-generated content against a variety of policies, providing transparent, explainable decisions and a clear audit trail for every piece of content processed.

The system is built to be modular, allowing for easy extension and integration into larger applications. It includes workflows for initial content analysis, parallel policy checks, final decision synthesis, and handling user appeals.

## Features

-   **Multi-Step Moderation Workflow:** Content passes through a graph of nodes, each performing a specific check.
-   **Parallel Policy Checks:** Simultaneously checks for:
    -   **Toxicity:** Insults, threats, harassment.
    -   **Spam:** Unsolicited advertising, repetitive text.
    -   **PII (Personally Identifiable Information):** Emails, phone numbers.
    -   **Custom Policies:** Hate speech, self-harm promotion, etc.
-   **Explainable AI Decisions:** For every moderation action, the system provides:
    -   **Clear Reasoning:** A human-readable explanation for the decision.
    -   **Severity Scoring:** A 0-10 score indicating the violation's severity.
    -   **Recommended Action:** `APPROVE`, `REJECT`, or `REVIEW`.
-   **Audit Trail:** A complete, step-by-step log of the path each piece of content took through the graph, ensuring full transparency.
-   **Human-in-the-Loop:** Automatically flags ambiguous or sensitive cases for human review.
-   **Appeal Handling:** Includes a workflow to re-evaluate content with additional context provided by a user appeal.
-   **Efficient Triage:** A quick initial check to bypass full analysis for obviously safe content, saving on processing costs.

## How It Works: The Moderation Graph

The pipeline is structured as a "state graph" where each piece of content is a state that evolves as it passes through different nodes.


*(This diagram shows the flow of content through the various checks and decision points.)*

1.  **Start:** New content enters the pipeline.
2.  **`triage_content`:** A fast, initial check. If content is very short or clearly harmless, it can be approved immediately. Otherwise, it proceeds to detailed checks.
3.  **Parallel Checks (`run_parallel_checks`):** The content is sent to multiple specialized "expert" models simultaneously.
    -   `toxicity_check`
    -   `spam_check`
    -   `pii_check`
    -   `custom_policy_check`
4.  **`synthesize_results`:** An LLM-powered "Head Moderator" node reviews the reports from all parallel checks. It aggregates the findings, calculates a final severity score, and generates a consolidated, user-facing reason for its final action recommendation.
5.  **`route_decision` (Conditional Routing):** Based on the synthesized action, the graph routes the content to one of three final states:
    -   `auto_approve`: The content is deemed safe.
    -   `auto_reject`: The content is a clear, high-severity violation.
    -   `request_human_review`: The case is ambiguous, low-severity, or requires a human touch.

## Technical Requirements

-   Python 3.8+
-   LangGraph
-   LangChain & `langchain-openai`
-   Python-dotenv

## Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/your-username/content-moderation-pipeline.git
    cd content-moderation-pipeline
    ```

2.  **Create and activate a virtual environment:**
    ```bash
    python3 -m venv venv
    source venv/bin/activate
    ```

3.  **Install the required packages:**
    ```bash
    pip install -r requirements.txt
    ```
    *(You may need to create a `requirements.txt` file with the necessary packages)*
    ```text
    # requirements.txt
    langgraph
    langchain
    langchain-openai
    python-dotenv
    pydantic
    ipython  # For displaying graphs in notebooks
    ```

4.  **Set up your environment variables:**
    Create a `.env` file in the root of the project directory and add your OpenAI API key.
    ```env
    # .env
    OPENAI_API_KEY="your-openai-api-key"
    ```

## Usage

The main script (`moderation_pipeline.py`) can be run directly from the command line. It simulates processing a queue of content items and demonstrates the appeal handling workflow.

```bash
python moderation_pipeline.py
```

### Example Output

The script will log detailed information for each piece of content, including the final decision and a full audit trail.

```text
================================================================================
Processing Content ID: post_002 | User: user_456
Content: 'You are a total idiot and I hate you.'
--------------------------------------------------------------------------------
[... logs of individual checks running ...]

--- FINAL DECISION ---
Action: REJECT
Severity: 9
Reasoning: The content contains a direct personal insult and expresses hate towards another person, which violates our toxicity and community guidelines.

--- AUDIT TRAIL ---
- Triage started.
- Triage determined detailed checks are required.
- Toxicity check completed: Violation=True, Severity=9
- Spam check completed: Violation=False, Severity=0
- PII check completed: Violation=False, Severity=0
- Custom Policy check completed: Violation=True, Severity=7
- Synthesizing results.
- Synthesis complete. Final Action: REJECT, Severity: 9
- Final state: AUTO_REJECTED
================================================================================
```

## How to Extend the Pipeline

This system is designed to be modular. To add a new policy check (e.g., for misinformation):

1.  **Create a New Node Function:** Write a Python function (e.g., `misinformation_check`) that takes the `state` and returns a dictionary with the results, similar to `toxicity_check`.
2.  **Add the Node to the Graph:** Use `workflow.add_node("misinformation_check", misinformation_check)`.
3.  **Wire it into the Parallel Flow:** Add an edge from the `run_parallel_checks` gateway to your new node: `workflow.add_edge("run_parallel_checks", "misinformation_check")`.
4.  **Connect it to the Synthesizer:** Add an edge from your new node to the `synthesize_results` node: `workflow.add_edge("misinformation_check", "synthesize_results")`.

The `synthesize_results` node will automatically incorporate the new report into its final decision without needing any code changes.
