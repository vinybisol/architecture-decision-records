---
number: 9
title: Return successful collection responses as HTTP 200
date: 2026-10-24
status: accepted
tags:
- api
- http
- response-contract
---

# 9. Return successful collection responses as HTTP 200

Date: 2026-10-24

## Status

Accepted

## Context

Collection queries can succeed with zero or more items. Changing the status code and omitting the body when no items match forces clients to handle two success representations for the same operation.

## Decision

Successful HTTP queries that return a collection use 200 OK, including when the collection is empty. Do not return 204 solely because no items match.

Return the collection directly, without introducing a standard response envelope or mandatory metadata. A non-empty JSON collection is an array of items; an empty JSON collection is `[]`, not null or an omitted body.

This rule applies to successful collection queries, not to failures such as an inaccessible or nonexistent parent resource. Other operations' status codes and error formats are outside this decision's scope.

## Consequences

- Clients consume the same status code and representation regardless of the number of returned items.
- Empty results are valid data, not a separate success condition.
- No envelope, timestamp, invocation identifier or count is required by this contract.
- Existing endpoints returning 204 for empty collections require a coordinated contract change.
