# Splunk Search Head Cluster – Post-Deployment Validation

This document provides **validation, functional testing, distributed search verification, security checks, and troubleshooting steps** after deploying a Splunk Search Head Cluster (SHC).

---

## 1. Cluster Health Verification

### 1.1 Verify Cluster Status (All Members)

Run on each Search Head member:

```bash
splunk show shcluster-status
```
<img width="782" height="565" alt="image" src="https://github.com/user-attachments/assets/94b18e0f-8984-4bba-8ac3-be26bbdbe42f" />

**Expected Results:**
- All members show `Status: Up`
- Exactly one Captain
- No members in `Pending` or `Down`
- No rolling restart in progress
- Replication status successful

---

### 1.2 Verify KV Store Status

```bash
splunk show kvstore-status
```

**Expected:**
- Status = `Ready`
- KV Store running
- No replication lag

---

### 1.3 Verify Replication Port Connectivity

```bash
telnet <peer_ip> 8090
```

**Expected:**
- Successful connectivity between all members

---

## 2. Distributed Search Validation (Critical)

### 2.1 Verify Search Peers via CLI

```bash
splunk list search-server
```

**Expected:**

```
https://idx1:8089    Active
https://idx2:8089    Active
https://idx3:8089    Active
```

- All peers must be `Active` or `Up`
- No peers in `Down` state

---

### 2.2 Verify via Splunk Web

Access:
```
http://<search_head>:8000
```

Navigate:
```
Settings > Distributed Search
```

**Validate:**
- All peers show `Up`
- No authentication errors

---

### 2.3 Validate distsearch.conf (If Used)

```bash
cat $SPLUNK_HOME/etc/system/local/distsearch.conf
```

**Expected:**

```ini
[distributedSearch]
servers = https://idx1:8089,https://idx2:8089,https://idx3:8089
```

---

### 2.4 Perform End-to-End Search Test

```spl
index=_internal | stats count by splunk_server
```

**Expected:**
- Results returned from all indexers
- Multiple `splunk_server` values

---

## 3. Functional Testing

### 3.1 Deploy Apps from Deployer

```bash
splunk apply shcluster-bundle -target https://<sh>:8089
```

Verify:

```bash
splunk list shcluster-bundle -member_uri https://<sh>:8089
```

**Expected:**
- Bundle applied successfully
- No errors
- Configuration replicated

---

### 3.2 Knowledge Object Replication Test

Create on one Search Head:
- Dashboard
- Saved Report
- Alert
- Lookup
- Macro

**Validate on other members:**
- Objects are visible
- No manual sync required

---

### 3.3 Captain Failover Test

```bash
splunk show shcluster-status
splunk stop
```

Wait 30–60 seconds:

```bash
splunk show shcluster-status
```

**Expected:**
- New Captain elected automatically
- No service disruption

---

### 3.4 Rolling Restart Validation

```bash
splunk rolling-restart shcluster-members
```

**Expected:**
- No downtime
- Sequential restart
- Cluster remains searchable

---

## 4. Deployer Validation

### 4.1 Verify Deployer Connectivity

```bash
curl -k https://<deployer_ip>:8089
```

---

### 4.2 Verify App Sync

```bash
ls $SPLUNK_HOME/etc/shcluster/apps/
```

**Expected:**
- Deployer apps are present

---

## 5. Security Validation

Ensure the following:

- SSL enabled on all nodes (8089)
- Strong `pass4SymmKey`
- Firewall rules configured:
  - 8089 (management)
  - 8090 (replication)
- RBAC implemented
- Admin accounts secured
- Audit logging enabled
- Deployer access restricted
- Secure distributed search communication

---

## 6. Troubleshooting Guide

### Issue: Member Not Joining Cluster

**Check Configuration:**
```bash
cat $SPLUNK_HOME/etc/system/local/server.conf
```

**Check Connectivity:**
```bash
telnet <member_ip> 8089
```

**Check Logs:**
```bash
tail -f $SPLUNK_HOME/var/log/splunk/splunkd.log
```

---

### Issue: Distributed Search Peers Down

**Fix:**
```bash
splunk remove search-server https://<indexer_ip>:8089
splunk add search-server https://<indexer_ip>:8089 -auth admin:password
```

**Logs:**
```bash
tail -f splunkd.log | grep DistributedSearch
```

---

### Issue: No Captain Elected

**Resolution:**
- Restart nodes one at a time
- Verify consistent configuration across members

---

### Issue: Bundle Push Fails

```bash
splunk validate files
```

- Fix errors and reapply bundle

---

### Issue: KV Store Not Ready

```bash
splunk show kvstore-status
```

```bash
tail -f $SPLUNK_HOME/var/log/splunk/mongod.log
```

---

## 7. Operational Readiness Checklist

- [ ] All SH members are `Up`
- [ ] Captain elected
- [ ] Distributed search peers are `Up`
- [ ] End-to-end search working
- [ ] Bundle deployment verified
- [ ] Knowledge object replication working
- [ ] Captain failover tested
- [ ] Rolling restart tested
- [ ] KV Store operational
- [ ] Deployer connectivity verified
- [ ] Logs reviewed (no critical errors)
- [ ] Security controls implemented

---

## 8. Expected Stable Cluster Behavior

A properly functioning Search Head Cluster should:

- Maintain one active Captain
- Automatically replicate knowledge objects
- Automatically re-elect Captain during failure
- Maintain distributed search connectivity
- Accept deployer bundle updates
- Maintain KV Store consistency
- Require no manual synchronization
- Provide consistent search results across all members
