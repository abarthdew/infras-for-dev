# AGENTS

## Goal
- The user wants to learn specific topics through interactive discussion and iterative questioning.
- The user plans to reinforce understanding by asking questions, receiving explanations, and refining their understanding through follow-up questions.
- The Agent must actively identify misunderstandings, incorrect assumptions, or flawed reasoning, and immediately correct them.
- Under no circumstances should the Agent allow the user to internalize incorrect knowledge.
- The Agent must always guide the user toward accurate understanding and sound learning practices.

## Pipeline Flow
- One cycle consists of: the user asks a question, the Agent answers and creates a note, commits and pushes the changes, then awaits the next question.
- When a markdown file for a topic is created or updated, the Agent should automatically commit and push the changes unless the user explicitly instructs otherwise.
- To distinguish Agent-generated commits from user-generated commits, the Agent must use a separate git identity.
- Commit messages should be concise while still clearly describing the core change.
- The Agent must commit using the following command format:
    ```bash
    git commit --author="{agent_name: claude | codex} <{agent_name}@example.com>" -m "{commit_message}"
    ```
- To minimize token consumption, the Agent must avoid reading the full contents of existing files whenever possible.
- The Agent should primarily append new content to the bottom of files instead of rewriting or restructuring existing content.
- If understanding file context is necessary, the Agent should inspect only section titles, headings, or file names, and avoid reading detailed content unless absolutely required.
- When deciding which file to append content to, the Agent must make the decision based only on file names and section headings whenever possible.
The Agent should avoid unnecessary repository-wide scans or full-file inspections.

## Directory Structure
```bash
programming-study/
├── 00-roadmap/
│   ├── overview.md
│   └── learning-log.md
│
├── 01-computer-science/
│   ├── operating-system.md
│   ├── network.md
│   ├── database-theory.md
│   └── distributed-system.md
│
├── 02-linux-and-shell/
│   ├── linux.md
│   ├── shell.md
│   ├── process.md
│   ├── file-system.md
│   └── ssh.md
│
├── 03-programming-languages/
│   ├── java/
│   ├── javascript/
│   ├── python/
│   └── sql/
│
├── 04-backend-development/
│   ├── spring/
│   ├── api-design/
│   ├── authentication/
│   └── web-server/
│
├── 05-database/
│   ├── sqlite/
│   ├── postgresql/
│   ├── redis/
│   ├── elasticsearch/
│   └── data-modeling/
│
├── 06-infrastructure/
│   ├── nginx/
│   ├── docker/
│   ├── kubernetes/
│   ├── networking/
│   └── load-balancing/
│
├── 07-devops-and-cicd/
│   ├── git/
│   ├── github-actions/
│   ├── codex-workflow/
│   └── deployment/
│
├── 08-observability/
│   ├── prometheus/
│   ├── grafana/
│   ├── datadog/
│   ├── logging/
│   ├── metrics/
│   └── tracing/
│
├── 09-data-engineering/
│   ├── hadoop/
│   ├── batch-processing/
│   ├── streaming/
│   └── data-pipeline/
│
├── 10-ai-and-llm/
│   ├── ai-basics/
│   ├── machine-learning/
│   ├── llm/
│   ├── rag/
│   ├── vector-database/
│   └── prompt-engineering/
│
└── 99-projects/
    ├── redis-with-docker/
    ├── spring-postgresql-docker/
    ├── nginx-reverse-proxy/
    └── rag-demo/
```
