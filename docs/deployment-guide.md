# Splunk Indexer Cluster – Deployment Guide

A step-by-step guide for deploying a **Splunk Indexer Cluster**, including configuration of the **Cluster Manager, Indexer Peers, Search Heads, and Indexes** using Splunk Web, CLI, and configuration files.

---

## 1. Cluster Manager Configuration

> Applies to the **Cluster Manager** (formerly Cluster Master)

### Method 1: Splunk Web

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

**File Location**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

**Configuration**

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

**File Location**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

**Configuration**

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

**File Location**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

**Configuration**

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

<img width="335" height="101" alt="image" src="https://github.com/user-attachments/assets/76a01deb-faca-45f8-876f-3b8b376da329" />

```ini
[<index_name>]
repFactor = auto
homePath = $SPLUNK_DB/<index_name>/db
coldPath = $SPLUNK_DB/<index_name>/colddb
thawedPath = $SPLUNK_DB/<index_name>/thaweddb
```
<img width="369" height="174" alt="image" src="https://github.com/user-attachments/assets/4b66257e-e541-42d7-9170-34a87cbb68c0" />


---

### Apply Changes

```bash
./splunk validate cluster-bundle --check-restart
./splunk apply cluster-bundle
./splunk show cluster-bundle-status
```
<img width="1217" height="78" alt="image" src="https://github.com/user-attachments/assets/3d7473fe-86cb-47c6-a7e9-b2137c296382" /> <br>
<img width="908" height="112" alt="image" src="https://github.com/user-attachments/assets/d341b5ec-7ca7-4ab5-b633-9f22d7d23411" /> <br>
<img width="960" height="611" alt="image" src="https://github.com/user-attachments/assets/a58ceabd-2c89-4aa0-bdf5-905f93665af4" />




---
