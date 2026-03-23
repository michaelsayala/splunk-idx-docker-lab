# Splunk Indexer Cluster Lab (Docker)

## Overview

This repository provides a **Docker-based Splunk clustered environment** designed to simulate a distributed Splunk deployment.

The lab demonstrates the following Splunk architectures:

- **Indexer Clustering**
- **Search Head Clustering (SHC)**
- **Search Head Deployer**
- **Cluster Manager**

In this lab:

- The **Search Head Cluster is already configured** in both deployment modes.
- The **Search Heads act as search peers to the Indexer Cluster**.
- The **difference between the two deployment modes only affects the Indexer Cluster configuration**.

This environment allows you to practice:

- Building an Indexer Cluster
- Connecting Search Heads to Indexer peers
- Managing Search Head Clusters
- Using a Deployer to manage Search Head apps
- Understanding distributed search architecture

---

## Architecture

```mermaid
flowchart TB

%% =========================
%% Search Head Cluster
%% =========================
subgraph SHC["Search Head Cluster"]
    SH1["Search Head 1 (sh1)"]
    SH2["Search Head 2 (sh2)"]
    SH3["Search Head 3 (sh3)"]
end

%% =========================
%% Deployer
%% =========================
DEP["Deployer (dep1)"]

%% =========================
%% Cluster Manager
%% =========================
CM["Cluster Manager (cm1)"]

%% =========================
%% Indexer Cluster
%% =========================
subgraph IDX_CLUSTER["Indexer Cluster"]
    IDX1["Indexer 1 (idx1)"]
    IDX2["Indexer 2 (idx2)"]
    IDX3["Indexer 3 (idx3)"]
end

%% =========================
%% Relationships
%% =========================

DEP --> SH1
DEP --> SH2
DEP --> SH3

SH1 --> IDX1
SH1 --> IDX2
SH1 --> IDX3

SH2 --> IDX1
SH2 --> IDX2
SH2 --> IDX3

SH3 --> IDX1
SH3 --> IDX2
SH3 --> IDX3

IDX1 --- CM
IDX2 --- CM
IDX3 --- CM
```
---

 Component | Hostname | Web Port | Management Port | Indexing Port |
|-----------|----------|----------|----------------|--------------|
| Cluster Manager | cm1 | 8000 | 8089 | N/A |
| Indexer 1 | idx1 | 8000 | 8089 | 9997 |
| Indexer 2 | idx2 | 8000 | 8089 | 9997 |
| Indexer 3 | idx3 | 8000 | 8089 | 9997 |
| Search Head 1 | sh1 | 8000 | 8089 | N/A |
| Search Head 2 | sh2 | 8000 | 8089 | N/A |
| Search Head 3 | sh3 | 8000 | 8089 | N/A |
| Deployer | dep1 | 8000 | 8089 | N/A |

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
SPLUNK_IDXC_SECRET=ClusterSecret123
SPLUNK_SHC_SECRET=SHClusterSecret123
```

---

## Deployment Modes

### 1 Base Environment (Manual Configuration)

This deployment starts all Splunk components, but the Indexer Cluster is not configured.

The **Search Head Cluster is already configured**, but the **Indexer Cluster is not yet configured**.

Components started:

- Cluster Manager (cm1)
- 3 Indexers (idx1, idx2, idx3)
- Search Head Cluster (sh1, sh2, sh3)
- Deployer (dep1)

Allows you to manually practice:

- Initializing the Indexer Cluster
- Peer node registration
- Setting replication and search factors
- Connecting Search Heads to Indexers as search peers

Start environment:

```
docker-compose -f docker-compose.manual.yml up -d
```

---

### 2 Preconfigured Indexer Cluster

This deployment automatically configures the Indexer Cluster during container startup:

- Sets cm1 as Cluster Manager
- Joins idx1, idx2, idx3 to the Indexer Cluster
- Connects the Search Head Cluster to the Indexers as search peers

Start environment:

```
docker-compose -f docker-compose.preconfigured.yml up -d
```

The Search Head Cluster is already configured and ready to perform distributed searches across the indexer cluster.

## Repository Structure

```
.
├── .env
├── docker-compose.manual.yml
├── docker-compose.preconfigured.yml
├── README.md
├── docs/
│   ├── deployment-guide.md
│   ├── post-deployment-validation.md
```
