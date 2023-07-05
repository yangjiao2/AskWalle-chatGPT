---
title: Glass iOS Splunk request/response header logging
navTitle: Header Logging
---

# Glass iOS Splunk request/response header logging

This document contains all approved and denied requests/response to log headers. The approved header keys will be logging and added under `platform.networking.header.logging.allowed.list` allowed list in the CCM.

## Process

- Platform support request with the following info:
  - Header in question
  - Why it needs to be tracked
  - What the OL uses the header for
- Platform evaluates the request and either denies or approves.
- If approved, platform updates documentation and makes CCM change following current change request guidance.
- If denied, platform updates documentation to capture the header info and mark the request as denied. 

_Note_: Mark the Status with ✅(Approved) / ❌(Denied) / 🔍(In-Review)
| Header Key | Description | Status (Approved/Denied) | Denial/Approve context/reasoning
|---|---|---|---|
| `Content-Length` | Indicates the size of the message body | 🔍 | | 
| `Content-Type` | Indicates the original media type of the resource | 🔍 | |
| `traceparent` | Indicates the uniquely identify a request | 🔍 |  |
| `gqlOperationName` | Indicates the GQL operation's name | 🔍 |  |
| `gqlOperationId` | Indicates the GQL operation's unique identifier | 🔍 |  |
| `server-timing` | Indicates how time is spent while processing the request | 🔍 |  |
| `akam-parent-rtt` | Indicates the parent round trip time | 🔍 |  |
| `akam-origin-latency` | Indicates the origin Latency Header | 🔍 |  |
| `akam-child-rtt` | Indicates the child round trip time Header | 🔍 |  |
| `correlationId` | Indicates the unique identifier value that is attached to requests and messages | 🔍 |  |
