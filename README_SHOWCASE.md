# Multi-Agent Customer Support System — LangGraph + AWS Bedrock

> **Extended fork** of [aws-solutions-library-samples/guidance-for-multi-agent-orchestration-langgraph-on-aws](https://github.com/aws-solutions-library-samples/guidance-for-multi-agent-orchestration-langgraph-on-aws) — enhanced with a Bedrock Knowledge Base RAG layer, MCP-compatible tool gateway, and enterprise security (Cognito, IAM isolation, VPC PrivateLink).

---

## Overview

A **production-grade multi-agent system** that orchestrates 5 specialized AI agents using **LangGraph** and **AWS Bedrock (Claude 3.5 Sonnet)** to handle complex, multi-domain customer support tasks autonomously.

This architecture directly mirrors the **Supervisor + Specialized Agents (Subagents pattern)** used in the PennyMac AI Platform, where a centralized supervisor dispatches stateless specialized agents — enabling parallel execution, strong context isolation, and measurable reduction in end-to-end processing time.

---

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│           React Frontend (S3 + CloudFront + Cognito)      │
└──────────────────────┬───────────────────────────────────┘
                       │ WebSocket / GraphQL (AppSync)
┌──────────────────────▼───────────────────────────────────┐
│              Supervisor Agent (LangGraph)                  │
│   Plans, routes, and consolidates across all sub-agents   │
└──┬──────────┬────────────┬─────────────┬─────────────────┘
   │          │            │             │
┌──▼──┐  ┌───▼───┐  ┌─────▼────┐  ┌────▼──────┐  ┌──────────────┐
│Order│  │Product│  │Personal- │  │Trouble-   │  │  RAG Agent   │
│Mgmt │  │Recom. │  │ization   │  │shooting   │  │(Knowledge    │
│Agent│  │Agent  │  │Agent     │  │Agent      │  │ Base Bedrock)│
└──┬──┘  └───────┘  └──────────┘  └───────────┘  └──────────────┘
   │
┌──▼─────────────────────────────────────────────────────────────┐
│         MCP Tool Gateway (Lambda + API Gateway)                 │
│   Aurora PostgreSQL → Agent-ready tools via semantic routing    │
└────────────────────────────────────────────────────────────────┘
   │
┌──▼─────────────────────────────────────────────────────────────┐
│         Security & Observability Layer                          │
│  Cognito Auth │ IAM Role Isolation │ VPC PrivateLink │          │
│  CloudWatch Dashboards │ OpenTelemetry Traces                   │
└────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer | Technologies |
|---|---|
| Agent Orchestration | LangGraph (stateful graph-based workflows) |
| Foundation Model | Claude 3.5 Haiku / Sonnet via AWS Bedrock (cross-region inference) |
| RAG Layer | Bedrock Knowledge Base (unstructured), Aurora PostgreSQL (structured) |
| Tool Gateway | Model Context Protocol (MCP), AWS Lambda, API Gateway |
| Frontend | React (TypeScript), AWS AppSync (GraphQL + WebSocket streaming) |
| Authentication | Amazon Cognito (user pools + identity pools) |
| Data Layer | Aurora PostgreSQL, Amazon S3, Bedrock Knowledge Base |
| Compute | Amazon ECS (Fargate) — one container per specialized agent |
| Observability | Amazon CloudWatch (dashboards + alarms), OpenTelemetry |
| Security | AWS IAM (per-agent role isolation), VPC PrivateLink, Secrets Manager |
| IaC | AWS CDK (Python) |

---

## Agent Roles

| Agent | Responsibility | Tools |
|---|---|---|
| **Supervisor** | Plans tasks, routes to sub-agents, consolidates responses | All agent tools |
| **Order Management** | Order tracking, returns, inventory queries | Aurora PostgreSQL, Lambda |
| **Product Recommendation** | Personalized product suggestions | Bedrock Knowledge Base, embeddings |
| **Personalization** | Customer profile management, preference tracking | DynamoDB, Aurora |
| **Troubleshooting** | Technical issue resolution and support | Knowledge Base RAG, Lambda |
| **RAG Agent** *(added)* | Unstructured knowledge retrieval from PDFs and docs | Bedrock KB, Pinecone |

---

## Key Features

- **Subagents Pattern** — Centralized supervisor with stateless specialized agents; supports parallel tool calling across agents
- **Bedrock Knowledge Base RAG** — Unstructured document retrieval layer integrated alongside structured Aurora PostgreSQL queries
- **MCP Tool Gateway** — Converts existing Lambda functions and Aurora queries into agent-compatible tools with semantic discovery
- **Real-time Streaming** — Token-level response streaming via AWS AppSync WebSocket to React frontend
- **Enterprise Security** — Cognito authentication, per-agent IAM role isolation, VPC PrivateLink for Bedrock, Secrets Manager for credentials
- **CloudWatch Observability** — Per-agent dashboards tracking latency, tool selection accuracy, error rates, and session duration

---

## Extensions Added (Beyond Upstream)

| Extension | Description |
|---|---|
| Bedrock Knowledge Base RAG Agent | 6th specialized agent for unstructured document retrieval |
| MCP Tool Gateway | Converts Aurora PostgreSQL queries into MCP-compatible agent tools |
| CloudWatch Dashboards | Per-agent metric dashboards (latency, tool accuracy, error rate) |
| VPC PrivateLink | Private Bedrock access — no public internet exposure for LLM calls |
| IAM Role Isolation | Separate execution role per agent container for least-privilege access |

---

## Quick Start

```bash
git clone https://github.com/parikshit06reddy-cloud/multi-agent-langgraph-bedrock
cd multi-agent-langgraph-bedrock

# Install dependencies
pip install -r requirements.txt
npm install --prefix frontend

# Enable Bedrock model access (Claude 3.5 Haiku)
# AWS Console → Bedrock → Model Access → Enable Claude 3.5 Haiku

# Deploy with CDK
cd infrastructure
pip install -r requirements.txt
cdk bootstrap
cdk deploy --all
```

See [DEPLOYMENT.md](DEPLOYMENT.md) for full step-by-step setup.

---

## Observability

Each agent emits metrics to CloudWatch:

| Metric | Description |
|---|---|
| `AgentLatencyP95` | 95th percentile response time per agent |
| `ToolSelectionAccuracy` | % of tool calls selecting the correct tool |
| `ErrorRate` | Failed agent runs per minute |
| `SessionDuration` | Average multi-turn session length |
| `RAGFaithfulness` | RAGAS faithfulness score on sampled responses |

---

## Author

**Parikshit Reddy** — Principal Applied AI Engineer  
[LinkedIn](https://www.linkedin.com/in/parikshitr/) · [GitHub](https://github.com/parikshit06reddy-cloud)

> Extended with RAG agent, MCP gateway, and enterprise security patterns reflecting production multi-agent architecture.
