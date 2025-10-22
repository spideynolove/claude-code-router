
# AI Coding Team: Implementation Plan

This document outlines the detailed plan to build a multi-agent AI coding system, as per the vision described in the `materials` directory. The system will feature a **Conductor** agent for orchestration and specialized **Musician** agents for execution, with seamless human-in-the-loop (HITL) capabilities, all orchestrated by the `claude-code-router`.

## Milestone 1: Foundational Setup (2 Weeks)

**Goal:** Establish the core infrastructure, including the router, a basic Conductor agent, and two Musician agents (Coder and Tester).

| # | Task | Objective | Acceptance Criteria |
|---|---|---|---|
| 1.1 | **Configure Advanced Router** | Set up `claude-code-router` with multiple providers (Anthropic, DeepSeek, OpenAI) and scenario-based routing for "Conductor" and "Musician" roles. | Router starts and correctly routes requests based on the `CCR_SCENARIO` environment variable. |
| 1.2 | **Implement Base Agent Framework** | Create an abstract `Agent` class with `execute(task, context)` and a `Musician` subclass. This will serve as the foundation for all specialized agents. | Base classes are defined, tested, and can be extended. |
| 1.3 | **Build Coder Musician** | Develop an agent that takes a natural language task and generates code using a powerful model (e.g., DeepSeek Coder). | Given the prompt "Implement a function to calculate Fibonacci," the agent writes the correct Python code to a file. |
| 1.4 | **Build Tester Musician** | Develop an agent that generates unit tests for a given piece of code. | Given the Fibonacci function from 1.3, the agent generates a valid `pytest` test file. |
| 1.5 | **Create Conductor v1** | A simple CLI tool that takes a task, sequentially invokes the Coder and Tester musicians, and saves the resulting files. | `python conductor.py "Implement and test a Fibonacci function"` results in both a `fibonacci.py` and `test_fibonacci.py` file. |
| 1.6 | **End-to-End Integration Test** | Create an automated test that runs the full workflow from task submission to file generation and validation. | The test suite successfully simulates a complete task and verifies the output artifacts. |

## Milestone 2: Orchestration & State Management (3 Weeks)

**Goal:** Evolve from a simple sequential CLI to a stateful, event-driven system capable of managing complex workflows and agent states.

| # | Task | Objective | Acceptance Criteria |
|---|---|---|---|
| 2.1 | **Introduce an Orchestrator** | Implement a state machine using a workflow engine (e.g., Temporal or a lightweight asyncio-based equivalent) to manage the lifecycle of a task. | The Orchestrator can successfully execute the plan from Milestone 1, managing the state of the Coder and Tester agents. |
| 2.2 | **Implement Message Bus** | Set up an asynchronous message bus (e.g., Redis Pub/Sub or RabbitMQ) for inter-agent communication, decoupling the Conductor from Musicians. | The Conductor can publish a `CODE_TASK` event, and the Coder agent will consume it and publish a `CODE_COMPLETE` event upon finishing. |
| 2.3 | **Agent State & Memory** | Implement a simple state management system for agents, allowing them to maintain context within a single task workflow. This could use Redis or a similar key-value store. | An agent can store and retrieve its `current_task` and `session_history` from the state store. |
| 2.4 | **Develop Reviewer Musician** | Create an agent that reviews code for quality, style, and potential bugs, using a high-reasoning model like Claude Opus or GPT-4. | The Reviewer agent can provide meaningful feedback on the code generated in Milestone 1. |
| 2.5 | **Expand Conductor Logic** | The Conductor can now generate a multi-step plan (Code -> Test -> Review) and publish the corresponding events to the message bus. | The Conductor successfully orchestrates a three-agent workflow. |

## Milestone 3: Human-in-the-Loop (HITL) Integration (2 Weeks)

**Goal:** Build the interface and logic for humans to seamlessly interact with the agent team, providing oversight and making critical decisions.

| # | Task | Objective | Acceptance Criteria |
|---|---|---|---|
| 3.1 | **Design HITL Gateway** | Create a simple API (e.g., FastAPI) that exposes endpoints for the review queue (`/reviews`, `/reviews/{id}/approve`). | The API is documented (Swagger/OpenAPI) and can be used to approve or reject a task. |
| 3.2 | **Implement HITL Triggers** | The Orchestrator can now pause a workflow and send a task to the HITL queue based on triggers (e.g., low confidence score, failing tests). | When tests from the Tester agent fail, the workflow is paused and a review item is created via the HITL Gateway. |
| 3.3 | **Build Simple Review UI** | A basic web interface that lists pending reviews and allows a human to view the code, test results, and approve/reject the task. | A user can log in, see a list of pending tasks, click on one, and approve it, which resumes the workflow. |
| 3.4| **Implement Notification System** | Integrate a notification service (e.g., via Discord or Slack webhooks) to alert the human operator when a review is required. | When a task is sent to the HITL queue, a message is posted to a designated channel with a link to the review UI. |

## Milestone 4: Advanced Features & Scalability (3 Weeks)

**Goal:** Add more sophisticated agents, improve the system's intelligence, and prepare it for deployment and scaling.

| # | Task | Objective | Acceptance Criteria |
|---|---|---|---|
| 4.1 | **Context Store (RAG)** | Implement a Retrieval-Augmented Generation system using a vector database (e.g., Qdrant, ChromaDB) to provide agents with relevant project context. | When given a task, the Conductor first queries the vector store for relevant code snippets and documentation to include in the prompt. |
| 4.2 | **Develop Documenter Musician** | An agent that can generate documentation (e.g., READMEs, API docs, code comments) based on the generated code. | The Documenter can create a well-formatted Markdown file explaining the Fibonacci function and its usage. |
| 4.3 | **Dockerize the System** | Create `Dockerfile`s for each component (Conductor, Musicians, Router, HITL Gateway) and a `docker-compose.yml` file for local deployment. | `docker-compose up` successfully starts all services, and they can communicate with each other. |
| 4.4 | **Enhance Router Logic** | Implement the `custom-router.js` with dynamic, task-based model selection logic, as described in your provided materials. | The router automatically selects a powerful model for the Conductor and a cost-effective model for the Documenter based on keyword analysis. |
| 4.5 | **Cost Tracking & Budgeting** | Implement middleware in the router to track token usage and estimated cost for each API call, storing it in a database. | After a workflow completes, the total cost and token count are logged and associated with the task ID. |

## Milestone 5: Polishing and Productionizing (Ongoing)

**Goal:** Refine the system, improve observability, and add feedback loops for continuous improvement.

| # | Task | Objective | Acceptance Criteria |
|---|---|---|---|
| 5.1 | **Monitoring & Observability** | Integrate Prometheus and Grafana to monitor key system metrics (e.g., task throughput, HITL queue length, agent error rates). | A Grafana dashboard displays real-time metrics of the multi-agent system. |
| 5.2 | **Feedback & Learning Loop** | Human feedback from the HITL system is captured and stored as labeled data for future fine-tuning. | When a human edits and approves code, the original output and the corrected version are saved as a training example. |
| 5.3 | **Implement Security Guardrails** | Add safety checks to detect and flag prompt injection, PII in outputs, and potentially destructive tool use. | The system correctly flags a prompt that tries to reveal environment variables and sends it to the HITL queue. |
| 5.4 | **Expand Agent Toolset** | Equip agents with more tools, such as file system access, web search, and the ability to run shell commands within a sandboxed environment. | The Coder agent can create a new directory and write multiple files to it as part of its task execution. |
