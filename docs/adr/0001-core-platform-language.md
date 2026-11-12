# ADR 0001: Use Java for the core RelayOps platform

## Status

Accepted

## Context

The developer's primary language is Java. RelayOps needs durable API handling, event processing, database transactions, security integration, and cloud deployment. Later AI/ML workflows may benefit from Python libraries.

## Decision

Build the core platform with Java, Spring Boot, and Maven. Introduce Python as an independent AI/ML service only when the project reaches model-serving, experimentation, or data-science requirements.

## Consequences

- The core platform builds on existing Java expertise.
- AI capabilities can use HTTP or gRPC across a clear service boundary.
- The team must manage an additional language and deployment artifact only when the benefit justifies it.

