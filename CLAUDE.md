# CLAUDE.md — notes-to-cards

## What this repo is
A **teaching repository**. We are building an AI flashcard app (paste notes → AI generates flashcards → save and review them) one lesson at a time. Each lesson ends with a git tag (`lesson-01`, `lesson-02`, …) and a matching `lessons/NN-topic.md`, so a reader can check out any tag and follow along.

The person driving this repo is learning React, AWS and AI-assisted development. **Teaching matters more than speed.**

## Stack
- Frontend: React + TypeScript + Vite (`/web`)
- Data fetching: TanStack Query; routing: React Router
- Backend: AWS Lambda (Node/TypeScript) behind API Gateway (`/api`)
- Data: DynamoDB
- AI: Amazon Bedrock (called from a Lambda, never from the browser)
- Infrastructure: AWS CDK in TypeScript (`/infra`)
- Tests: Vitest + React Testing Library
- CI/CD: GitHub Actions, deploying to AWS via OIDC (no stored access keys)

## How to work in this repo
1. **Small steps.** One concept per change. Don't scaffold a whole feature in one go.
2. **Explain before you write.** Before writing code, say in 2–4 sentences what you're about to do and which concept it demonstrates.
3. **No mystery lines.** If you introduce a library, pattern or config option, explain why it's there. Prefer readable code over clever code.
4. **Leave the person room to do it.** When a step is a good exercise (a component, a hook, a CDK construct), offer to let them write it first and then review it.
5. **Commits teach too.** Commit messages use the format `lesson-NN: <what changed and why>`.
6. **Update the lesson file** in `lessons/` as part of each lesson: goal, concept, steps, and a "check yourself" puzzle.

## Hard rules
- Never commit secrets, `.env` files, or AWS credentials.
- Never call AI models from the browser; all model calls go through the API.
- Keep AWS costs near zero: on-demand DynamoDB, no NAT gateways, no always-on servers.
- Every CDK stack must be destroyable with `cdk destroy`.
