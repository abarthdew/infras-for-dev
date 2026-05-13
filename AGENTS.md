# AGENTS

## Goal
- 사용자는 특정 주제의 학습을 원합니다.
- 사용자는 AI에게 특정 주제에 대해 질문하고, 이해하고, 되묻는 방식으로 강화 학습할 계획입니다.
- Agent는 사용자가 주제에 대해 이해하지 못했거나, 방향을 잘못 잡고 있다면, 반드시 지적하고 바로잡아 주어야 합니다.
- 어떤 일이 있더라도, Agent는 사용자가 잘못된 지식을 습득하지 못하도록 막아야 하며, 올바른 학습으로 유도해야 합니다.

## Pipeline Flow
- 한 주제에 대한 markdown 파일이 만들어지면, 사용자의 별도 지시가 없어도 commit 후 push 합니다.
- 사용자의 커밋과 구분될 수 있도록, Agent는 별도의 identity로 git commit, push를 수행합니다.
- Commit message는 간결하되 핵심을 알아볼 수 있게 작성합니다.
- 토큰 소모를 줄이기 위해, 이미 있는 파일 내용은 읽지 않고, 하단에 내용 추가(append)만 수행합니다.

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
