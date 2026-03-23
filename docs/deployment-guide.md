# Splunk Search Head Cluster – Deployment Procedure

## Configure Deployer

`Note: This method is intended for deployer usage.`

### Method 1: Configuration File

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. Navigate to `$SPLUNK_HOME/etc/system/local`.
     ```bash
     cd /opt/splunk/etc/system/local
     ```
     
3. **Create or Edit server.conf:**
   
    - To create a new configuration file if it doesn't exist:
    ```bash
     touch server.conf
    ```
   - To edit a configuration file
    ```bash
     vi server.conf
    ```

4. **Add Configuration:**
   - Inside `server.conf`, add the following content:
     ```ini
     [shclustering]
     pass4SymmKey = <security_key>
     shcluster_label = <label_name>
     ```
     Replace `<security_key>` - This key authenticates communication among cluster members and between each member and the deployer. \
     Replace `<label_name>` - Use this for identifying the cluster in the monitoring console. This parameter is optional but, if configured on one member, it must be the same across all members and the deployer.

5. **Save Changes:**
   - Save the `server.conf` file.

6. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configure Cluster Members

`Note: This method is intended for search head cluster members. Execute this procedure on each search head cluster member.`

### Method 1: Command Line

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. **Navigate to `$SPLUNK_HOME/bin`**.
     ```bash
     cd /opt/splunk/bin
     ```

3. **Edit Splunk Cluster Configuration**
    - Use the `./splunk init shcluster-config` command to modify the cluster configuration.
    - Syntax:
        ```bash
        ./splunk init shcluster-config \
          -auth <username>:<password> \
          -mgmt_uri https://<sh_cluster_member_ipaddress>:8089 \
          -replication_port <replication_port> \
          -replication_factor <replication_number> \
          -conf_deploy_fetch_url https://<sh_deployer_ipaddress>:8089 \
          -secret <deployer_pass4SymmKey> \
          -shcluster_label <deployer_label_name>
        ```
    - Options:
        - `auth:` Your current login credentials for this instance.
        - `mgmt_uri:` URI for the search head cluster member.
        - `replication_port:` Port for replication.
        - `replication_factor:` Replication factor (n).
        - `conf_deploy_fetch_url:` URL for the deployer.
        - `secret:` Same security key used in the deployer.
        - `shcluster_label:` Same label name used in the deployer (optional if label is not specified on the deployer).

    Example:
    ```bash
    ./splunk init shcluster-config -auth admin:p@ssw0rd123 -mgmt_uri https://10.0.0.221:8089 -replication_port 8090 -replication_factor 3 -conf_deploy_fetch_url https://10.0.0.220:8089 -secret s3cr3t@123 -shcluster_label shcluster1
    ```

4. **Execute Cluster Configuration Command**
    - Run the command in the Splunk terminal to apply the configuration changes.

5. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```
### Method 2: Configuration File

`Note: This method is intended for search head cluster members. Execute this procedure on each search head cluster member.`

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. Navigate to `$SPLUNK_HOME/etc/system/local`.
     ```bash
     cd /opt/splunk/etc/system/local
     ```
     
3. **Create or Edit server.conf:**
   
    - To create a new configuration file if it doesn't exist:
    ```bash
     touch server.conf
    ```
   - To edit a configuration file
    ```bash
     vi server.conf
    ```

4. **Add Configuration:**
   - Inside `server.conf`, add the following content:
     ```ini
     [replication_port://8090]

     [shclustering]
     conf_deploy_fetch_url = https://<sh_cluster_member_ipaddress>:8089
     disabled = 0
     mgmt_uri = https://<deployer_ipaddress>:8089
     pass4SymmKey = <deployer_security_key>
     shcluster_label = <deployer_label_name>
     ```
     Replace `<sh_cluster_member_ipaddress>`: IP address of the search head cluster member. \
     Replace `<deployer_ipaddress>`: IP address of the deployer.\
     Replace `<security_key>`: Security key authenticating communication between cluster members and the deployer. \    
     Replace `<label_name>`: Identifies the cluster in the monitoring console (optional but should be consistent across members and the deployer).

5. **Save Changes:**
   - Save the `server.conf` file.

6. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configure Search Head Captain

`Note: This method is intended for designating one search head as the captain.`

### Method 1: Command Line

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. **Navigate to `$SPLUNK_HOME/bin`**.
     ```bash
     cd /opt/splunk/bin
     ```

3. **Edit Splunk Cluster Configuration**
    - Use the `./splunk bootstrap shcluster-captain` command to modify the cluster configuration.
    - Syntax:
        ```bash
        ./splunk bootstrap shcluster-captain \
        -servers_list "<https://sh_cluster_member1:8089,https://sh_cluster_member2:8089,https://sh_cluster_member3:8089>" \
        -auth <username>:<password>
        ```
        Replace `<username>` and `<password>` with your authentication details. Also `<https://sh_cluster_member1:8089,https://sh_cluster_member2:8089,https://sh_cluster_member3:8089>` with the correct ip address.
    - Options:
        - `servers_list:` List of cluster member URLs.
        - `auth:` Your current login credentials for this instance.
    Example:
    ```bash
    ./splunk bootstrap shcluster-captain -servers_list "https://10.0.0.221:8089,https://10.0.0.222:8089,https://10.0.0.223:8089" -auth admin:p@ssw0rd123
    ```

4. **Execute Cluster Configuration Command**
    - Run the command in the Splunk terminal to apply the configuration changes.

5. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configure Distributed Search

`Note: This step connects the Search Head Cluster to Indexers or an Indexer Cluster.`

---

### Method 1: Command Line (Recommended)

Run on **each Search Head member**:

1. Navigate:
   ```bash
   cd /opt/splunk/bin
   ```

2. Add search peers (Indexers):
   ```bash
   ./splunk add search-server https://<indexer_ip>:8089 \
   -auth <username>:<password>
   ```

Example:
```bash
./splunk add search-server https://10.0.0.101:8089 -auth admin:password
./splunk add search-server https://10.0.0.102:8089 -auth admin:password
./splunk add search-server https://10.0.0.103:8089 -auth admin:password
```

3. Verify:
   ```bash
   ./splunk list search-server
   ```

Expected:
```
https://10.0.0.101:8089    Active
https://10.0.0.102:8089    Active
https://10.0.0.103:8089    Active
```

---

### Method 2: Configuration File

1. Navigate:
   ```bash
   cd /opt/splunk/etc/system/local
   ```

2. Create or edit `distsearch.conf`:
   ```bash
   vi distsearch.conf
   ```

3. Add:
   ```ini
   [distributedSearch]
   servers = https://idx1:8089,https://idx2:8089,https://idx3:8089
   ```

4. Restart Splunk:
   ```bash
   /opt/splunk/bin/splunk restart
   ```

---

### Method 3: Splunk Web

`Note: This is the easiest method for manual configuration and validation.`

1. Open Splunk Web:
   ```
   http://<search_head_ip>:8000
   ```

2. Navigate to:
   ```
   Settings > Distributed Search
   ```

3. Under **Search Peers**, click:
   ```
   Add new
   ```

4. Enter the following details:
   - **Peer URI:**  
     ```
     https://<indexer_ip>:8089
     ```
   - **Username:**  
     Splunk admin username
   - **Password:**  
     Splunk admin password

5. Click **Save**

6. Repeat for all indexers.

7. Verify via Splunk Web

- Navigate to:
  ```
  Settings > Distributed Search
  ```

- Confirm:
  - All peers show **Status: Up**
  - No peers in `Down` state
  - No authentication errors

---
