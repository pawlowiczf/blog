---
# draft: true 
date: 2025-08-21
categories:
  - DevOps
authors:
  - pawlowiczf
slug: proto-repository

pin: true

links:
  - GCP Compute Engine: https://cloud.google.com/products/compute
---

# Setup remote repository for `.proto` and `gRPC` files 
In a microservices architecture, Protobuf files and generated stubs are often used by different services located in separate repositories. How can we **share** them across applications while keeping them **consistent**? How can we establish a **single source of truth**, streamline modifications, and automate deployment?

<!-- more -->

Project files: [schema-definitions-repository](../files/p2025-08-21-proto-repository/schema-definitions.zip)