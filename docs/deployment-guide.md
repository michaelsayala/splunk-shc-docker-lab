# Splunk Search Head Cluster – Deployment Guide

A step-by-step guide for deploying a **Splunk Search Head Cluster (SHC)**, including configuration of the **Deployer, Cluster Members, Captain Election, and Distributed Search** using Splunk CLI and configuration files.

---

## 1. Deployer Configuration

> Applies to the **Search Head Cluster Deployer**

The deployer is responsible for distributing configurations to all Search Head Cluster members.

### Method 1: Configuration File

**File Location**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

**Configuration**

```ini
[shclustering]
pass4SymmKey = <security_key>
shcluster_label = <label_name>
```

---

## 2. Search Head Cluster Members Configuration

> Applies to all Search Head Cluster Members

---

### Method 1: CLI (Recommended)

```
cd /opt/splunk/bin

./splunk init shcluster-config \
  -auth <username>:<password> \
  -mgmt_uri https://<sh_member_ip>:8089 \
  -replication_port <replication_port> \
  -replication_factor <replication_number> \
  -conf_deploy_fetch_url https://<deployer_ip>:8089 \
  -secret <pass4SymmKey> \
  -shcluster_label <cluster_label>
```

---

### Method 2: Configuration File

**File Location**

```
$SPLUNK_HOME/etc/system/local/server.conf
```

**Configuration**

```ini
[replication_port://8090]

[shclustering]
conf_deploy_fetch_url = https://<deployer_ip>:8089
disabled = 0
mgmt_uri = https://<sh_ip>:8089
pass4SymmKey = <security_key>
shcluster_label = <label_name>
```

---

## 3. Search Head Captain Configuration

> Used to manually elect a Search Head Cluster Captain

---

### Method 1: CLI (Recommended)

```
cd /opt/splunk/bin

./splunk bootstrap shcluster-captain \
-servers_list "https://<sh1>:8089,https://<sh2>:8089,https://<sh3>:8089" \
-auth <username>:<password>
```

## 4. Configure Distributed Search

> Connects Search Head Cluster to Indexers

---

### Method 1: CLI (Recommended)

Run on each Search Head:

```"
./splunk add search-server https://<indexer_ip>:8089 -auth <username>:<password>
```

Verify

```
./splunk list search-server
```

Expected output:

```
https://10.0.0.101:8089    Active
https://10.0.0.102:8089    Active
https://10.0.0.103:8089    Active
```

---

### Method 2: Configuration File

**File Location**

```
$SPLUNK_HOME/etc/system/local/distsearch.conf
```

**Configuration**

```ini
[distributedSearch]
servers = https://idx1:8089,https://idx2:8089,https://idx3:8089
```

---

### Method 3: Splunk Web

1. Open Splunk Web:

```
http://<search_head_ip>:8000
```

2. Navigate:

```
Settings → Distributed Search
```

3. Add Search Peers:

* Enter Indexer URL: `https://<indexer_ip>:8089`
* Provide credentials

4. Verify:

* Status = Up
* No authentication errors
