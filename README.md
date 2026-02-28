# Splunk Search Head Cluster Lab (Docker)

## Overview

This repository provides a Docker-based Splunk lab environment designed to simulate a Search Head Cluster (SHC) architecture.

It supports two deployment modes:

1. [Base Environment (Unconfigured)](https://github.com/michaelsayala/splunk-shc-docker-lab/blob/main/docker-compose.base.yml)
2. [Preconfigured Search Head Cluster (SHC)](https://github.com/michaelsayala/splunk-shc-docker-lab/blob/main/docker-compose.shc.yml)

****
This lab is intended for learning, testing, troubleshooting, and demonstrating Splunk clustering architecture in a controlled environment.

---

## Architecture

### Base Environment

- 3 standalone Splunk Enterprise instances
- 1 deployer instance (not configured for SHC)
- No clustering enabled
- Suitable for manual SHC configuration practice

### Search Head Cluster Environment

- 3 Search Heads
- 1 Deployer
- SH1 configured as cluster captain
- Shared SHC secret across all members
- Automatic SHC formation during container startup

---

## Components

| Component       | Hostname | Web UI Port | Management Port | Role     |
|---------------|----------|------------|------------------|----------|
| Search Head 1 | sh1      | 8005       | 8094             | Captain  |
| Search Head 2 | sh2      | 8006       | 8095             | Member   |
| Search Head 3 | sh3      | 8007       | 8096             | Member   |
| Deployer      | dep1     | 8008       | 8097             | Deployer |

---

## Prerequisites

- Docker
- Docker Compose
- External Docker network named `skynet`

Create the required Docker network before deployment:

```bash
docker network create skynet
```

---

## Repository Structure

```
.
├── docker-compose.base.yml
├── docker-compose.shc.yml
├── .env.example
└── README.md
```

---

## Environment Variables

Create a `.env` file in the root directory. Do not commit this file to GitHub.

Example `.env`:

```
SPLUNK_PASSWORD=splunkpassword
SPLUNK_SHC_SECRET=splunkshcpassword
```

Add `.env` to your `.gitignore`.

---

## Deployment Instructions

### Deploy Base Environment (Unconfigured)

This deploys standalone Splunk instances without clustering.

```bash
docker compose -f docker-compose.base.yml up -d
```

Access the instances:

- SH1: http://localhost:8005
- SH2: http://localhost:8006
- SH3: http://localhost:8007
- DEP1: http://localhost:8008

Use this mode if you want to manually configure Search Head Clustering.

---

### Deploy Search Head Cluster (Preconfigured)

This deployment automatically forms a Search Head Cluster.

```bash
docker compose -f docker-compose.shc.yml up -d
```
