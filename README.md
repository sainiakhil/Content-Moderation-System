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
