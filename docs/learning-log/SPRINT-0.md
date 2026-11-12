# Sprint 0 — Foundation and Architecture

## Status

In progress. Completion requires review of the architecture blueprint and a score of at least 90% in the Sprint 0 quiz.

## Goal

Define a production scenario and establish the project structure before application code is written.

## Scenario

RelayOps AI accepts webhook events from external systems. It must validate the sender, avoid duplicate processing, preserve accepted events, process costly work asynchronously, and provide operators with event status.

## Initial production constraints

- Average target traffic: one million events per hour.
- Burst target: two thousand events per second.
- Acknowledgement target: p95 under 200 ms.
- An accepted event must be durable before acknowledgement.
- Tenants must be isolated from each other.

## Decisions recorded so far

- The core application will use Java and Maven.
- Python remains available for a later, separate AI/ML workload.
- We start as a modular monolith and introduce distributed components only after reproducing the problem they solve.

## Current blockers

- No product or technical-design blockers are recorded for Sprint 0.

## Sprint 0 deliverables

- [x] Project repository and documentation structure
- [x] Initial technology and language decision
- [x] Product requirements, architecture, failure scenarios, and operational targets documented
- [ ] Sprint 0 learner quiz passed with at least 90%

