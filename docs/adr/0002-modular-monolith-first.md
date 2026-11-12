# ADR 0002: Start as a modular monolith

## Status

Accepted

## Context

RelayOps will eventually demonstrate queues, workers, containers, Kubernetes, and AI services. Introducing all of them before the first API is usable would add operational complexity without proving a need for it.

## Decision

Start with one Spring Boot deployable application. Keep its source organized into modules for ingress, events, identity, processing, and notifications. Extract a component into an independent service only after a measurable problem requires independent scaling, deployment, security isolation, or technology choice.

## Consequences

- Local development and debugging remain simple in Sprint 1.
- Business logic is not duplicated across prematurely separated services.
- Spring 4 will first introduce asynchronous processing within the modular application, then demonstrate a worker boundary when the queue is added.
- Later microservice extraction remains possible because module boundaries are explicit.

