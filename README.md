# Splunk Indexer Cluster Lab (Docker)

## Overview

This repository provides a Docker-based Splunk lab environment designed to simulate an Indexer Cluster (IDX) deployment.

It supports two deployment modes:

1. [Base Environment (Unconfigured)](https://github.com/michaelsayala/splunk-idx-docker-lab/blob/main/docker-compose_unconfigure.base.yml)
2. [Preconfigured Indexer Cluster with Search Head Cluster](https://github.com/michaelsayala/splunk-idx-docker-lab/blob/main/docker-compose-shc.yml)

****
This lab is intended for learning, testing, troubleshooting, and demonstrating Splunk indexer clustering in a controlled environment.

---

## Architecture

## Architecture

```text
Base Environment (Unconfigured)

+---------------------+      +---------------------+      +---------------------+
| Indexer 1 (idx1)    |      | Indexer 2 (idx2)    |      | Indexer 3 (idx3)    |
| Cluster Not Config  |      | Cluster Not Config  |      | Cluster Not Config  |
+---------------------+      +---------------------+      +---------------------+

+---------------------+      +---------------------+      +---------------------+
| Search Head 1 (sh1) |      | Search Head 2 (sh2) |      | Search Head 3 (sh3) |
| SHC Not Configured  |      | SHC Not Configured  |      | SHC Not Configured  |
+---------------------+      +---------------------+      +---------------------+

+---------------------+
| Deployer (dep1)     |
| Not Configured      |
+---------------------+

+---------------------+
| Cluster Master (cm1)|
| Not Configured      |
+---------------------+


Preconfigured Indexer Cluster Environment

                        +----------------------+
                        | Cluster Master (cm1) |
                        | Web: 8000            |
                        | Mgmt: 8089           |
                        +----------+-----------+
                                   |
                                   v

+---------------------+      +---------------------+      +---------------------+
| Indexer 1 (idx1)    |<---->| Indexer 2 (idx2)    |<---->| Indexer 3 (idx3)    |
| Web: 8000           |      | Web: 8000           |      | Web: 8000           |
| Mgmt: 8089          |      | Mgmt: 8089          |      | Mgmt: 8089          |
| Cluster Peer        |      | Cluster Peer        |      | Cluster Peer        |
+---------------------+      +---------------------+      +---------------------+


                        +----------------------+
                        | Deployer (dep1)      |
                        | Web: 8000            |
                        | Mgmt: 8089           |
                        +----------+-----------+
                                   |
                                   v

+---------------------+      +---------------------+      +---------------------+
| Search Head 1 (sh1) |<---->| Search Head 2 (sh2) |<---->| Search Head 3 (sh3) |
| Web: 8000           |      | Web: 8000           |      | Web: 8000           |
| Mgmt: 8089          |      | Mgmt: 8089          |      | Mgmt: 8089          |
| SHC Member          |      | SHC Member          |      | SHC Member          |
+---------------------+      +---------------------+      +---------------------+
```

### Base Environment

- 3 standalone Splunk **indexer instances**
- 3 standalone Splunk **search head instances**
- Cluster Master not configured
- Cluster Peers not configured
- Search Head Cluster configured
- Deployer configured

## Preconfigured Indexer Cluster Environment

- **1 Cluster Master** responsible for managing the indexer cluster
- **3 Indexers** configured as cluster peers
- **3 Search Heads** configured as a Search Head Cluster (SHC)
- **1 Deployer** responsible for managing SHC apps and configurations

---

## Components

| Component | Hostname | Web UI Port | Management Port | Role |
|-----------|----------|-------------|-----------------|------|
| Cluster Master | cm1 | 8000 | 8089 | Cluster Manager |
| Indexer 1 | idx1 | 8000 | 8089 | Cluster Peer |
| Indexer 2 | idx2 | 8000 | 8089 | Cluster Peer |
| Indexer 3 | idx3 | 8000 | 8089 | Cluster Peer |
| Deployer | dep1 | 8000 | 8089 | SHC Deployer |
| Search Head 1 | sh1 | 8000 | 8089 | SHC Member |
| Search Head 2 | sh2 | 8000 | 8089 | SHC Member |
| Search Head 3 | sh3 | 8000 | 8089 | SHC Member |

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
├── docker-compose_unconfigured.yml
├── docker-compose-idx.yml
├── .env.example
└── README.md
```

---

## Environment Variables

Create a `.env` file in the root directory. Do not commit this file to GitHub.

Example `.env`:

```
SPLUNK_PASSWORD=splunkpassword
SPLUNK_SHC_SECRET=splunkidxpassword
```

Add `.env` to your `.gitignore`.

---

## Deployment Instructions

### Deploy Base Environment (Unconfigured)

This deployment launches standalone Splunk instances without indexer clustering configured and search clustering configured already.

```bash
docker compose -f docker-compose_unconfigure.yml up -d
```

Access the instances:

- Cluster Master: http://localhost:8001
- Indexer 1: http://localhost:8002
- Indexer 2: http://localhost:8003
- Indexer 3: http://localhost:8004
- Deployer: http://localhost:8008
- Search Head 1: http://localhost:8005
- Search Head 2: http://localhost:8006
- Search Head 3: http://localhost:8007

Use this mode if you want to manually configure Indexer Clustering.

---

### Deploy Preconfigured Indexer Cluster with Search Head Cluster

This deployment launches:

- 1 Cluster Master
- 3 Indexer cluster peers
- 3 Search Head cluster members
- 1 Deployer

```bash
docker compose -f docker-compose-idx.yml up -d
```

Access the instances:

- Cluster Master: http://localhost:8001
- Indexer 1: http://localhost:8002
- Indexer 2: http://localhost:8003
- Indexer 3: http://localhost:8004
- Deployer: http://localhost:8008
- Search Head 1: http://localhost:8005
- Search Head 2: http://localhost:8006
- Search Head 3: http://localhost:8007
