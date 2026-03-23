# Splunk Indexer Cluster – Post-Deployment Validation

This document provides **validation, functional testing, cluster bundle verification, search head integration, security checks, and troubleshooting procedures** after deploying a Splunk Indexer Cluster (Cluster Manager and Peer Nodes).

---

## 1. Cluster Health Verification

### 1.1 Verify Cluster Status (Cluster Manager)

Run on the Cluster Manager:

```bash
splunk show cluster-status
```

**Expected Results:**
- All peers show `Status: Up`
- Replication Factor (RF) = Met
- Search Factor (SF) = Met
- No peers in `Pending` or `Down`
- No fix-up tasks in progress
- Cluster state = `Active`

---

### 1.2 Verify Peer Node Status

Run on each Indexer Peer:

```bash
splunk show cluster-status
```

**Expected:**
- Peer connected to Cluster Manager
- Status = `Up`
- No heartbeat failures

---

### 1.3 Verify Replication and Search Factor

```bash
splunk show cluster-status | grep -E "Replication|Search"
```

**Expected:**
- Replication Factor = configured value
- Search Factor = configured value
- Both = `Met`

---

### 1.4 Verify Peer Registration

```bash
splunk list cluster-peers
```

**Expected:**
- All peers listed
- Status = `Up`
- No missing peers

---

### 1.5 Verify Replication Port Connectivity

(Default replication port: **9887** unless changed)

```bash
telnet <peer_ip> 9887
```

**Expected:**
- Successful connectivity between all peers

---

## 2. Cluster Bundle Validation (CRITICAL)

### 2.1 Validate Cluster Bundle Before Apply

```bash
splunk validate cluster-bundle --check-restart
```

**Expected:**
- No validation errors
- Restart requirements clearly indicated

---

### 2.2 Apply Cluster Bundle

```bash
splunk apply cluster-bundle
```

---

### 2.3 Verify Bundle Status

```bash
splunk show cluster-bundle-status
```

**Expected:**
- Bundle applied successfully
- No errors
- All peers updated

---

### 2.4 Validate Index Replication

Check on peers:

```bash
splunk list index
```

**Expected:**
- Newly created indexes present on all peers

---

## 3. Functional Testing

### 3.1 Ingestion Test

Generate test data:

```bash
echo "Indexer cluster test event" >> /tmp/test.log
```

Search:

```spl
index=_internal | stats count by host
```

**Expected:**
- Events indexed successfully
- Data searchable across cluster

---

### 3.2 Verify Bucket Replication

```bash
splunk show cluster-status --verbose
```

**Expected:**
- No under-replicated buckets
- No fix-up backlog
- Buckets distributed across peers

---

### 3.3 Peer Failure Test

Stop one peer:

```bash
splunk stop
```

Check cluster:

```bash
splunk show cluster-status
```

**Expected:**
- RF/SF temporarily degraded
- Fix-up starts automatically
- Data remains searchable

Restart peer:

```bash
splunk start
```

**Expected:**
- Peer rejoins cluster
- RF/SF returns to `Met`

---

### 3.4 Rolling Restart Validation

```bash
splunk rolling-restart cluster-peers
```

**Expected:**
- Sequential restart
- No full downtime
- Searches continue
- RF/SF maintained

---

## 4. Search Head Integration Validation

### 4.1 Verify Search Head Connection to Cluster Manager

Run on Search Head:

```bash
splunk show cluster-status
```

**Expected:**
- Connected to Cluster Manager
- No connection errors

---

### 4.2 Verify Distributed Search to Indexers

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

### 4.3 End-to-End Search Validation

```spl
index=_internal | stats count by splunk_server
```

**Expected:**
- Results from all indexers
- Multiple `splunk_server` values

---

## 5. Index Configuration Validation

### 5.1 Verify Index Configuration

```bash
cat $SPLUNK_HOME/etc/master-apps/_cluster/local/indexes.conf
```

**Expected:**
- `repFactor = auto` set
- Paths correctly configured

---

### 5.2 Verify Index Availability

```bash
splunk list index
```

**Expected:**
- Index exists on all peers
- No missing indexes

---

## 6. Security Validation

Ensure the following:

- SSL enabled on Cluster Manager and Peers
- Port 8089 restricted (management)
- Port 9887 restricted (replication)
- Strong `pass4SymmKey`
- RBAC enforced
- Index-level access control configured
- Audit logs enabled
- Cluster Manager access restricted

---

## 7. Troubleshooting Guide

### Issue: Peer Not Joining Cluster

**Check Config:**
```bash
cat $SPLUNK_HOME/etc/system/local/server.conf
```

**Check Connectivity:**
```bash
telnet <manager_ip> 8089
telnet <manager_ip> 9887
```

**Check Logs:**
```bash
tail -f $SPLUNK_HOME/var/log/splunk/splunkd.log
```

---

### Issue: Replication Factor Not Met

**Causes:**
- Peer down
- Insufficient peers
- Disk full

**Fix:**
```bash
splunk show cluster-status
```

- Restart peers
- Add capacity if needed

---

### Issue: Search Factor Not Met

**Causes:**
- Peer failure
- Network instability

**Fix:**
- Restore peer
- Allow fix-up to complete

---

### Issue: Excessive Fix-Up Activity

**Check:**
```bash
splunk show cluster-status --verbose
```

**Causes:**
- Frequent restarts
- Hardware bottlenecks

---

### Issue: Indexing Delays

**Check:**
```spl
index=_internal source=*metrics.log group=queue
```

**Causes:**
- High ingestion
- Resource constraints

---

## 8. Operational Readiness Checklist

- [ ] All peers show `Up`
- [ ] Replication Factor met
- [ ] Search Factor met
- [ ] Cluster bundle validated and applied
- [ ] Index replication verified
- [ ] Data ingestion tested
- [ ] Bucket replication verified
- [ ] Peer failure recovery tested
- [ ] Rolling restart validated
- [ ] Search Head integration verified
- [ ] Logs reviewed (no critical errors)
- [ ] Security controls implemented

---

## 9. Expected Stable Cluster Behavior

A properly functioning Indexer Cluster should:

- Maintain configured Replication Factor (RF)
- Maintain configured Search Factor (SF)
- Automatically replicate buckets across peers
- Automatically perform fix-up during peer failure
- Support rolling restarts without downtime
- Maintain searchable data during outages
- Synchronize indexes via cluster bundle
- Require no manual bucket management
