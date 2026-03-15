# Splunk Search Head Cluster Lab (Docker)

## Overview

This repository provides a **Docker-based Splunk Search Head Cluster (SHC) environment** designed to simulate a distributed Splunk search layer.

The lab demonstrates how multiple **Search Heads operate as a cluster** to provide:

- High availability
- Search workload distribution
- Centralized app management using a **Deployer**

The environment includes:

- **3 Search Heads**
- **1 Deployer**

This lab is useful for understanding:

- Search Head Cluster architecture
- Captain election
- Deployer-based configuration management
- Search Head cluster formation

The repository supports **two deployment modes**:

1. **Base Environment (Unconfigured)**
2. **Preconfigured Search Head Cluster**
---
## Cluster Architecture

```mermaid
graph LR

%% =========================
%% Deployer
%% =========================
DEP[Deployer (dep1)]

%% =========================
%% Search Head Cluster
%% =========================
subgraph SHC["Search Head Cluster"]
    SH1[sh1]
    SH2[sh2]
    SH3[sh3]
end

%% =========================
%% Deployer pushes apps/config
%% =========================
DEP -.-> SH1
DEP -.-> SH2
DEP -.-> SH3

%% =========================
%% SHC member links
%% =========================
SH1 --- SH2
SH2 --- SH3
SH1 --- SH3
```
---
| Component | Hostname | Web Port | Management Port |
|-----------|----------|----------|----------------|
| Search Head 1 | sh1 | 8000 | 8089 |
| Search Head 2 | sh2 | 8000 | 8089 |
| Search Head 3 | sh3 | 8000 | 8089 |
| Deployer | dep1 | 8000 | 8089 |
---
All containers run on the external Docker network:

```
skynet
```

---

## Prerequisites

### 1 Install Docker

Install Docker and Docker Compose.

```
https://docs.docker.com/get-docker/
```

---

### 2 Create Docker Network

Create the external network used by the lab.

```
docker network create skynet
```

---

### 3 Create `.env` File

Create a `.env` file in the project root.

Example:

```
SPLUNK_PASSWORD=YourStrongPassword
SPLUNK_SHC_SECRET=SHClusterSecret123
```

---

## Deployment Modes

### 1 Base Environment

This deployment starts the following components:
- 3 Search Heads
- 1 Deployer

However, the Search Head Cluster is not automatically configured.

This mode allows you to manually practice:

Initializing a Search Head Cluster

- Setting the captain node
- Joining cluster members
- Configuring the deployer

This is useful for learning manual SHC configuration.

---

### 2 Preconfigured Search Head Cluster

This deployment automatically configures the Search Head Cluster during container startup.

The configuration includes:
- Creating a Search Head Cluster
- Setting sh1 as the initial captain
- Joining sh1, sh2, sh3 as cluster members
- Connecting the Deployer to the cluster

This mode is useful for:
- automated lab environments
- repeatable testing
- learning SHC architecture
