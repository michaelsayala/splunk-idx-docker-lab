# Splunk Indexer Cluster – Deployment Guide

A step-by-step guide for deploying a **Splunk Indexer Cluster**, including configuration of the **Cluster Manager, Indexer Peers, Search Heads, and Indexes** using Splunk Web, CLI, and configuration files.

---

## 1. Cluster Manager Configuration

> Applies to the **Cluster Manager** (formerly Cluster Master)

### Method 1: Splunk Web (Recommended)

1. Go to **Settings → Indexer Clustering**
2. Click **Enable Indexer Clustering**
3. Configure:

   * Replication Factor
   * Search Factor
   * Security Key (`pass4SymmKey`)
   * Cluster Label
4. Click **Enable Cluster Manager**
5. Restart Splunk

---

### Method 2: CLI

```bash
cd /opt/splunk/bin

./splunk edit cluster-config \
  -mode manager \
  -replication_factor 3 \
  -search_factor 2 \
  -secret <pass4SymmKey> \
  -cluster_label <cluster_name>

./splunk restart
```

---

### Method 3: Configuration File

**Path:**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

```ini
[clustering]
mode = manager
replication_factor = 3
search_factor = 2
cluster_label = <cluster_name>
pass4SymmKey = <pass4SymmKey>
```

---

## 2. Indexer Peer Configuration

> Apply to all indexer nodes

### Method 1: Splunk Web

1. Go to **Settings → Indexer Clustering**
2. Enable clustering
3. Configure:

   * Manager URI
   * Replication Port
   * Security Key
4. Restart Splunk

---

### Method 2: CLI

```bash
./splunk edit cluster-config \
  -mode peer \
  -manager_uri https://<cluster_manager_ip>:8089 \
  -replication_port 8090 \
  -secret <pass4SymmKey>

./splunk restart
```

---

### Method 3: Configuration File

```ini
[replication_port://8090]

[clustering]
mode = peer
manager_uri = https://<cluster_manager_ip>:8089
pass4SymmKey = <pass4SymmKey>
```

---

## 3. Search Head Configuration

> Applies to Search Heads connected to the cluster

### Method 1: Splunk Web

1. Go to **Settings → Indexer Clustering**
2. Configure:

   * Manager URI
   * Security Key
3. Restart Splunk

---

### Method 2: CLI

```bash
./splunk edit cluster-config \
  -mode searchhead \
  -master_uri https://<cluster_manager_ip>:8089 \
  -secret <pass4SymmKey>

./splunk restart
```

---

### Method 3: Configuration File

```ini
[clustering]
mode = searchhead
master_uri = https://<cluster_manager_ip>:8089
pass4SymmKey = <pass4SymmKey>
```

---

## 4. Index Configuration (Cluster Bundle)

> Run on the Cluster Manager

### Navigate

```bash
cd /opt/splunk/etc/master-apps/_cluster/local
```

### Create or Edit `indexes.conf`

```ini
[<index_name>]
repFactor = auto
homePath = $SPLUNK_DB/<index_name>/db
coldPath = $SPLUNK_DB/<index_name>/colddb
thawedPath = $SPLUNK_DB/<index_name>/thaweddb
```

---

### Apply Changes

```bash
./splunk validate cluster-bundle --check-restart
./splunk apply cluster-bundle
./splunk show cluster-bundle-status
```

---
