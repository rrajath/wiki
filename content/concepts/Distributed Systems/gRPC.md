---
title: gRPC
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - raw/notes/20260513210409-grpc.org
tags: [distributed-systems, grpc, networking, rpc, system-design]
---

# gRPC

A high-performance RPC (Remote Procedure Call) framework from Google. Primary use case: **internal service-to-service communication**.

## Why it's fast

gRPC uses **binary serialization** (Protocol Buffers / protobuf) instead of JSON. Binary encoding is significantly more compact and faster to serialize/deserialize than text-based formats like JSON.

| Format | Encoding | Typical use |
|--------|----------|-------------|
| REST/JSON | Text (verbose) | Public APIs, browser clients |
| gRPC/protobuf | Binary (compact) | Internal microservice calls |

## Why it's not used for public APIs

Most browsers do not natively support gRPC. Public-facing APIs therefore use REST over HTTP/1.1 or HTTP/2 with JSON, which every browser understands out of the box.

## When to use in system design

- **Internal microservice communication**: when latency and throughput between services matter (e.g., a payment service calling a fraud detection service).
- **Streaming**: gRPC supports bidirectional streaming natively, unlike traditional REST.
- **Strongly typed contracts**: protobuf schemas enforce the interface between services at compile time.

See also: [[Distributed Systems/index|Distributed Systems]]
