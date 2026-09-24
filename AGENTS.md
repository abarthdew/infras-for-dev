# AGENTS

## Goal
- The user wants to learn specific topics through interactive discussion and iterative questioning.
- The user plans to reinforce understanding by asking questions, receiving explanations, and refining their understanding through follow-up questions.
- The Agent must actively identify misunderstandings, incorrect assumptions, or flawed reasoning, and immediately correct them.
- Under no circumstances should the Agent allow the user to internalize incorrect knowledge.
- The Agent must always guide the user toward accurate understanding and sound learning practices.

## Pipeline Flow
- One cycle consists of: the user asks a question, the Agent answers and creates a note, commits and pushes the changes, then awaits the next question.
- For Jira-tracked reorganizations or cross-repository work, use an issue-key branch and a pull request; do not push changes directly to the default branch.
- When a markdown file for a topic is created or updated, the Agent should automatically commit and push the changes unless the user explicitly instructs otherwise.
- To distinguish Agent-generated commits from user-generated commits, the Agent must use a separate git identity that includes its name and model.
- Commit messages should be concise while still clearly describing the core change.
- The Agent must commit using the following command format:
    ```bash
    git commit --author="{agent_name}-{model_name} <{agent_name}@users.noreply.github.com>" -m "{commit_message}"
    ```
- To minimize token consumption, the Agent must avoid reading the full contents of existing files whenever possible.
- The Agent should primarily append new content to the bottom of files instead of rewriting or restructuring existing content.
- If understanding file context is necessary, the Agent should inspect only section titles, headings, or file names, and avoid reading detailed content unless absolutely required.
- When deciding which file to append content to, the Agent must make the decision based only on file names and section headings whenever possible.
The Agent should avoid unnecessary repository-wide scans or full-file inspections.

## Read-Only Learning Mode
- If the user explicitly requests read-only learning mode, the Agent must follow that request.
- In read-only learning mode, the Agent must not modify files, create commits, or push changes.
- The Agent should first inspect only the directory structure and section headings to propose a top-down learning path, or maximize learning effectiveness through an interactive question-and-answer dialogue with the user.

## git issue registration instructions
- When the user requests it, the Agent must register the requested content as a git issue. This content will usually be what the user and the Agent discussed in a question-and-answer format while learning and refining the user's understanding of a concept.
- The Agent must use the following template:
    ```text
    - git issue title: file path/file name/section name (usually a `##` subsection title)
    - git issue body: the user's question and the Agent's answer. It must also mention what the user did not understand, where the user was confused, or what point the user was stuck on.
    ```

## User image generation requests
- The user may ask the Agent to generate an image for concepts that have order, sequence, or interaction, so the relationship can be understood efficiently at a glance.
- In that case, the user will request an image for content in a specific learning file. The Agent must generate a clear, concise learning image that explains the structure of the concept and how its parts interact, in order to support the user's understanding.
- Image storage path: create an `images/` directory for the relevant file and place the generated image there.
- The Agent must also attach the image directly below the relevant section in the file mentioned by the user so it can be viewed while studying, then commit and push the change.

## Directory Structure
```bash
infras-for-dev/
├── 01-computer-science/
├── 02-linux-and-shell/
├── 03-infrastructure/
├── 04-devops-and-cicd/
├── 05-observability/
├── 06-data-engineering/
├── 07-ai-and-llm/
└── 08-projects/
```

Frontend and backend notes belong in `abarthdew/back-and-forth`; database and SQL notes belong in `abarthdew/dbms-for-dev`. Keep relative links and colocated images intact when moving notes.
