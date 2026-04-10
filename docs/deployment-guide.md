# Splunk Search Head Cluster – Deployment Procedure

## Overview

This guide provides a step-by-step procedure for deploying and configuring a **Splunk Search Head Cluster (SHC)** in a Docker-based or virtual environment.

It covers:

* Deployer configuration
* Search Head cluster initialization
* Captain election
* Cluster member configuration
* Distributed search setup

---

## 1. Configure Deployer

> This step is performed on the **Deployer node only**.

### Method 1: Configuration File

#### 1. Access configuration directory

```bash id="p1x9kd"
cd /opt/splunk/etc/system/local
```

#### 2. Create or edit `server.conf`

```bash id="k3m8lp"
touch server.conf
vi server.conf
```

#### 3. Add configuration

```ini id="shc1"
[shclustering]
pass4SymmKey = <security_key>
shcluster_label = <label_name>
```

#### Notes:

* `pass4SymmKey` → Authentication key for SHC communication
* `shcluster_label` → Must be consistent across all cluster members (optional but recommended)

#### 4. Restart Splunk

```bash id="r8x2qv"
/opt/splunk/bin/splunk restart
```

---

## 2. Configure Search Head Cluster Members

> This step must be executed on **each Search Head node**.

---

### Method 1: CLI Initialization (Recommended)

#### 1. Navigate to Splunk bin directory

```bash id="m2q7lt"
cd /opt/splunk/bin
```

#### 2. Initialize SHC configuration

```bash id="v9c1hd"
./splunk init shcluster-config \
-auth <username>:<password> \
-mgmt_uri https://<sh_ip>:8089 \
-replication_port <port> \
-replication_factor <n> \
-conf_deploy_fetch_url https://<deployer_ip>:8089 \
-secret <pass4SymmKey> \
-shcluster_label <label_name>
```

#### Example

```bash id="e4x8nf"
./splunk init shcluster-config \
-auth admin:p@ssw0rd123 \
-mgmt_uri https://10.0.0.221:8089 \
-replication_port 8090 \
-replication_factor 3 \
-conf_deploy_fetch_url https://10.0.0.220:8089 \
-secret s3cr3t@123 \
-shcluster_label shcluster1
```

#### 3. Restart Splunk

```bash id="u1k9qz"
/opt/splunk/bin/splunk restart
```

---

### Method 2: Configuration File

#### 1. Navigate to configuration directory

```bash id="d7m3xz"
cd /opt/splunk/etc/system/local
```

#### 2. Create or edit `server.conf`

```bash id="q8v2lp"
touch server.conf
vi server.conf
```

#### 3. Add configuration

```ini id="shc2"
[replication_port://8090]

[shclustering]
conf_deploy_fetch_url = https://<deployer_ip>:8089
disabled = 0
mgmt_uri = https://<sh_ip>:8089
pass4SymmKey = <security_key>
shcluster_label = <label_name>
```

#### Notes:

* Ensure `pass4SymmKey` matches the deployer
* All nodes must share the same `shcluster_label`

#### 4. Restart Splunk

```bash id="z3p8mt"
/opt/splunk/bin/splunk restart
```

---

## 3. Configure Search Head Captain

> This step designates a captain for the cluster.

---

### Method 1: CLI Bootstrap (Recommended)

#### 1. Navigate to bin directory

```bash id="c1m9qx"
cd /opt/splunk/bin
```

#### 2. Bootstrap captain

```bash id="t6v3kd"
./splunk bootstrap shcluster-captain \
-servers_list "https://<sh1>:8089,https://<sh2>:8089,https://<sh3>:8089" \
-auth <username>:<password>
```

#### Example

```bash id="n8x2qp"
./splunk bootstrap shcluster-captain \
-servers_list "https://10.0.0.221:8089,https://10.0.0.222:8089,https://10.0.0.223:8089" \
-auth admin:p@ssw0rd123
```

#### 3. Restart Splunk

```bash id="y4k8mz"
/opt/splunk/bin/splunk restart
```

---

## 4. Configure Distributed Search

> This step connects the Search Head Cluster to Indexers.

---

### Method 1: CLI (Recommended)

#### Add search peers

Run on each Search Head:

```bash id="f2x9pl"
./splunk add search-server https://<indexer_ip>:8089 -auth <username>:<password>
```

#### Example

```bash id="a9v3kt"
./splunk add search-server https://10.0.0.101:8089 -auth admin:password
./splunk add search-server https://10.0.0.102:8089 -auth admin:password
./splunk add search-server https://10.0.0.103:8089 -auth admin:password
```

#### Verify

```bash id="k5m2qv"
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

#### 1. Create or edit configuration

```bash id="p7x1ld"
cd /opt/splunk/etc/system/local
vi distsearch.conf
```

#### 2. Add configuration

```ini id="ds1"
[distributedSearch]
servers = https://idx1:8089,https://idx2:8089,https://idx3:8089
```

#### 3. Restart Splunk

```bash id="r3v8mz"
/opt/splunk/bin/splunk restart
```

---

### Method 3: Splunk Web (Optional)

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
