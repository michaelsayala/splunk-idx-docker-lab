# Splunk Indexer Cluster Lab (Docker)

## Overview

This repository provides a Docker-based Splunk lab environment designed to simulate an Indexer Cluster (IDX) architecture.

It supports two deployment modes:

1. [Base Environment (Unconfigured)](https://github.com/michaelsayala/splunk-idx-docker-lab/blob/main/docker-compose.base.yml)
2. [Preconfigured Search Head Cluster (SHC)](https://github.com/michaelsayala/splunk-idx-docker-lab/blob/main/docker-compose.shc.yml)

****
This lab is intended for learning, testing, troubleshooting, and demonstrating Splunk indexer clustering in a controlled environment.

---

## Architecture

### Base Environment

- 3 standalone Splunk Enterprise indexers  
- 1 Cluster Master (CM)  
- No clustering enabled  
- Suitable for manual cluster configuration practice  

### Preconfigured Indexer Cluster

- 3 Indexers  
- 1 Cluster Master  
- Shared cluster secret (`pass4SymmKey`) across all members  
- Automatic cluster formation during container startup  

---

## Components

| Component       | Hostname | Web UI Port | Management Port | Role           |
|-----------------|----------|------------|----------------|----------------|
| Indexer 1       | idx1     | 8001       | 8090           | Member         |
| Indexer 2       | idx2     | 8002       | 8091           | Member         |
| Indexer 3       | idx3     | 8003       | 8092           | Member         |
| Cluster Master  | cm1      | 8004       | 8093           | Cluster Master |

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
├── docker-compose.base-indexer.yml
├── docker-compose.indexer-cluster.yml
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

This deploys standalone Splunk instances without clustering.

```bash
docker compose -f docker-compose.base.yml up -d
```

Access the instances:

- IDX1: http://localhost:8001
- IDX2: http://localhost:8002
- IDX3: http://localhost:8003
- CM1: http://localhost:8004

Use this mode if you want to manually configure Indexer Clustering.

---

### Deploy Search Head Cluster (Preconfigured)

This deployment automatically forms a Indexer Cluster.

```bash
docker compose -f docker-compose.idx.yml up -d
```
