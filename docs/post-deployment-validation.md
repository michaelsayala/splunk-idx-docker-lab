# Splunk Indexer Cluster – Post-Deployment Validation Guide

This guide provides validation, functional testing, cluster bundle verification, search head integration checks, security validation, and troubleshooting procedures after deploying a Splunk Indexer Cluster.

---

## Overview

Use this guide to ensure your cluster is:

* Healthy and stable
* Properly replicating data
* Fully searchable
* Operationally ready for production

---

## 1. Cluster Health Verification

### 1.1 Cluster Manager Status

```bash
splunk show cluster-status
```

**Expected:**

* All peers = `Up`
* Replication Factor (RF) = Met
* Search Factor (SF) = Met
* No peers in `Pending` or `Down`
* No fix-up tasks in progress
* Cluster state = Active

---

### 1.2 Peer Status (REST API)

```bash
curl -k -u <user>:<password> https://<cluster_manager_ip>:8089/services/cluster/master/peers?output_mode=json
```

**Expected:**

* Peer appears in response
* Status = Up
* Correct RF/SF values

---

### 1.3 Replication and Search Factor Check

```bash
splunk show cluster-status | grep -E "Replication|Search"
```

**Expected:**

* RF and SF match configured values
* Both show Met

---

### 1.4 Peer Registration

```bash
splunk list cluster-peers
```

**Expected:**

* All peers listed
* Status = Up

---

### 1.5 Replication Port Connectivity

(Default port: 9887 unless changed)

```bash
telnet <peer_ip> 9887
```

**Expected:**

* Successful connectivity between peers

---

## 2. Cluster Bundle Validation

### 2.1 Validate Before Apply

```bash
splunk validate cluster-bundle --check-restart
```

**Expected:**

* No validation errors
* Restart requirements displayed

---

### 2.2 Apply Bundle

```bash
splunk apply cluster-bundle
```

---

### 2.3 Verify Bundle Status

```bash
splunk show cluster-bundle-status
```

**Expected:**

* Successful deployment
* No errors
* All peers updated

---

### 2.4 Index Replication Validation

```bash
splunk list index
```

**Expected:**

* Index exists on all peers

---

## 3. Functional Testing

### 3.1 Data Ingestion Test

```bash
echo "Indexer cluster test event" >> /tmp/test.log
```

```spl
index=_internal | stats count by host
```

**Expected:**

* Events indexed successfully
* Data searchable across cluster

---

### 3.2 Bucket Replication Check

```bash
splunk show cluster-status --verbose
```

**Expected:**

* No under-replicated buckets
* No fix-up backlog
* Buckets distributed across peers

---

### 3.3 Peer Failure Test

Stop a peer:

```bash
splunk stop
```

Check cluster:

```bash
splunk show cluster-status
```

**Expected:**

* RF/SF temporarily degraded
* Fix-up process starts automatically
* Data remains searchable

Restart peer:

```bash
splunk start
```

**Expected:**

* Peer rejoins cluster
* RF/SF returns to Met

---

### 3.4 Rolling Restart Validation

```bash
splunk rolling-restart cluster-peers
```

**Expected:**

* Sequential restart
* No full downtime
* Searches continue
* RF/SF maintained

---

## 4. Search Head Integration

### 4.1 Verify Connected Indexers

```bash
splunk list search-server
```

**Expected:**

```
https://idx1:8089    Active
https://idx2:8089    Active
https://idx3:8089    Active
```

---

### 4.2 End-to-End Search Test

```spl
index=_internal | stats count by splunk_server
```

**Expected:**

* Results returned from all indexers
* Multiple splunk_server values

---

## 5. Index Configuration Validation

### 5.1 Verify indexes.conf

```bash
cat $SPLUNK_HOME/etc/master-apps/_cluster/local/indexes.conf
```

**Expected:**

* repFactor = auto
* Correct paths configured

---

### 5.2 Verify Index Availability

```bash
splunk list index
```

**Expected:**

* Index exists on all peers
* No missing indexes

---

## 6. Security Validation

Ensure the following are implemented:

* SSL enabled on all nodes
* Port 8089 (management) restricted
* Port 9887 (replication) restricted
* Strong pass4SymmKey
* Role-Based Access Control (RBAC) enforced
* Index-level access control configured
* Audit logging enabled
* Cluster Manager access restricted

---

## 7. Troubleshooting

### Peer Not Joining Cluster

```bash
cat $SPLUNK_HOME/etc/system/local/server.conf
```

```bash
telnet <cluster_manager_ip> 8089
telnet <cluster_manager_ip> 9887
```

```bash
tail -f $SPLUNK_HOME/var/log/splunk/splunkd.log
```

---

### Replication Factor Not Met

Common causes:

* Peer down
* Insufficient peers
* Disk issues

```bash
splunk show cluster-status
```

---

### Search Factor Not Met

Common causes:

* Peer failure
* Network instability

Resolution:

* Restore affected peer
* Allow fix-up to complete

---

### Excessive Fix-Up Activity

```bash
splunk show cluster-status --verbose
```

Possible causes:

* Frequent restarts
* Resource constraints

---

### Indexing Delays

```spl
index=_internal source=*metrics.log group=queue
```

Possible causes:

* High ingestion rate
* Resource bottlenecks

---

## 8. Operational Readiness Checklist

* [ ] All peers are Up
* [ ] Replication Factor met
* [ ] Search Factor met
* [ ] Cluster bundle validated and applied
* [ ] Index replication verified
* [ ] Data ingestion tested
* [ ] Bucket replication verified
* [ ] Peer failure recovery tested
* [ ] Rolling restart validated
* [ ] Search Head integration verified
* [ ] Logs reviewed (no critical errors)
* [ ] Security controls implemented

---

## 9. Expected Cluster Behavior

A properly functioning cluster should:

* Maintain Replication Factor (RF)
* Maintain Search Factor (SF)
* Automatically replicate buckets
* Perform fix-up during peer failure
* Support rolling restarts without downtime
* Maintain search availability during outages
* Synchronize indexes via cluster bundle
* Require no manual bucket management
