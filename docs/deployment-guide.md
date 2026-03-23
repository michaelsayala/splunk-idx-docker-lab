# Splunk Indexer Cluster – Deployment Procedure

## Configuring Manager Node

`Note: This method is intended for a single manager node or cluster master.`

### Method 1: Splunk Web

1. **Access Splunk Web:**
   - Navigate to your Splunk instance URL.
   - Log in with appropriate credentials.

2. **Navigate to Indexer Clustering:**
   - Go to `Settings` -> `Indexer Clustering`.

3. **Enable Indexer Clustering:**
   - Click on "Enable indexer clustering" to begin the configuration.

4. **Configure Manager Node:**
   - Follow prompts to select the manager node and proceed by clicking "Next".

5. **Configure Manager Node Settings:**
   - Define configurations for "Manager Node Configuration":
     - **Replication Factor:** Determine the number of copies of each bucket to store.
     - **Search Factor:** Specify the number of searchable copies of indexed data.
     - **Security Key:** Set up authentication for cluster nodes.
     - **Cluster Label:** Configure cluster labeling for identification purposes.
   - Click on "Enable Manager Node" to apply settings.

6. **Restart Splunk:**
   - Go to "Settings" > "Server controls."
   - Click on "Restart Splunk" to apply the changes made.

### Method 2: Command Line

`Note: This method is intended for a single manager node or cluster master.`

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. **Navigate to `$SPLUNK_HOME/bin`**.
     ```bash
     cd /opt/splunk/bin
     ```

3. **Edit Splunk Cluster Configuration**
    - Use the `./splunk edit cluster-config` command to modify the cluster configuration.
    - Syntax:
        ```bash
        ./splunk edit cluster-config \
            -mode <manager> \
            -replication_factor <#_of_replication_factor> \
            -search_factor <#_of_search_factor> \
            -secret <Indexer_pass4SymmKey> \
            -cluster_label <label_name>
        ```
    - Options:
        - `-mode`: Sets the cluster mode (e.g., `manager`).
        - `-replication_factor`: Specifies the number of replication factors.
        - `-search_factor`: Specifies the number of search factors.
        - `-secret`: Sets the Indexer pass4SymmKey.
        - `-cluster_label`: Specifies the cluster name.

    Example:
    ```bash
    ./splunk edit cluster-config -mode manager -replication_factor 3 -search_factor 2 -secret s3cr3t@123 -cluster_label splk_cm
    ```
    *Replace `s3cr3t@123` with your preferred secret.*

4. **Execute Cluster Configuration Command**
    - Run the command in the Splunk terminal to apply the configuration changes.

5. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```
### Method 3: Configuration File

`Note: This method is intended for a single manager node or cluster master.`

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
     [clustering]
     cluster_label = <label_name>
     mode = manager
     pass4SymmKey = <Indexer_pass4SymmKey>
     ```
     Replace `<cluster_name>` Enter your desired cluster name \ 
     Replace `<Indexer_pass4SymmKey>` Use the actual pass4SymmKey for the indexer.

5. **Save Changes:**
   - Save the `server.conf` file.

6. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configuring Peer Node

`Note: This method is intended for each peer node or indexer cluster member.`

### Method 1: Splunk Web

1. **Access Splunk Web:**
   - Navigate to your Splunk instance URL.
   - Log in with appropriate credentials.

2. **Navigate to Indexer Clustering:**
   - Go to `Settings` -> `Indexer Clustering`.

3. **Enable Indexer Clustering:**
   - Click on "Enable indexer clustering" to begin the configuration.

4. **Configure Peer Node:**
   - Follow prompts to select the manager node and proceed by clicking "Next".

5. **Configure Peer Node Settings:**
   - Define configurations for "Peer Node Configuration":
     - **Manager URI**: Enter the Uniform Resource Identifier (URI) for the manager node.
     - **Peer replication port**: Specify the port used for peer replication.
     - **Security key**: Set up a security key for communication between nodes.
   - Click on "Enable Peer Node" to apply settings.

6. **Restart Splunk:**
   - Go to "Settings" > "Server controls."
   - Click on "Restart Splunk" to apply the changes made.

### Method 2: Command Line

`Note: This method is intended for each peer node or indexer cluster member.`

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. **Navigate to `$SPLUNK_HOME/bin`**.
     ```bash
     cd /opt/splunk/bin
     ```

3. **Edit Splunk Cluster Configuration**
    - Use the `./splunk edit cluster-config` command to modify the cluster configuration.
    - Syntax:
        ```bash
        ./splunk edit cluster-config \
          -mode peer \
          -manager_uri https://<master_node_ip>:8089 \
          -replication_port <port> \
          -secret <Indexer_pass4SymmKey>
        ```
    - Options:
        - `-mode`: Sets the cluster mode (e.g., `peer`).
        - `-manager_uri`: Specifies the URI for the master node.
        - `-replication_port`: Specifies the port for replication.
        - `-secret`: Sets the Indexer pass4SymmKey.

    Example:
    ```bash
    ./splunk edit cluster-config -mode peer -manager_uri https://10.0.0.220:8089 -replication_port 8090 -secret s3cr3t@123
    ```
    *Replace `s3cr3t@123` with your preferred secret.*

4. **Execute Cluster Configuration Command**
    - Run the command in the Splunk terminal to apply the configuration changes.

5. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

### Method 3: Configuration File

`Note: This method is intended for each peer node or indexer cluster member.`

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

     [clustering]
     manager_uri = https://<master_node_ip>:8089
     mode = peer
     pass4SymmKey = <Indexer_pass4SymmKey>
     ```
     Replace `<master_node_ip>` with ip of manager node. \
     Replace `<Indexer_pass4SymmKey>` with the actual pass4SymmKey for the indexer.

5. **Save Changes:**
   - Save the `server.conf` file.

6. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configuring Search Head Node

### Method 1: Splunk Web

`Note: This method is intended for each search head cluster member.`

1. **Access Splunk Web:**
   - Navigate to your Splunk instance URL.
   - Log in with appropriate credentials.

2. **Navigate to Indexer Clustering:**
   - Go to `Settings` -> `Indexer Clustering`.

3. **Enable Indexer Clustering:**
   - Click on "Enable indexer clustering" to begin the configuration.

4. **Configure Search Head Node:**
   - Follow prompts to select the manager node and proceed by clicking "Next".

5. **Configure Search Head Node Settings:**
   - Define configurations for "Peer Node Configuration":
     - **Manager URI**: Enter the Uniform Resource Identifier (URI) for the manager node.
     - **Security key**: Set up a security key for communication between nodes.
   - Click on "Enable Peer Node" to apply settings.

6. **Restart Splunk:**
   - Go to "Settings" > "Server controls."
   - Click on "Restart Splunk" to apply the changes made.

### Method 2: Command Line

`Note: This method is intended for each search head cluster member.`

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. **Navigate to `$SPLUNK_HOME/bin`**.
     ```bash
     cd /opt/splunk/bin
     ```

3. **Edit Splunk Cluster Configuration**
    - Use the `./splunk edit cluster-config` command to modify the cluster configuration.
    - Syntax:
        ```bash
        ./splunk edit cluster-config \
          -mode searchhead \
          -master_uri https://<master_node_ip>:8089 \
          -secret <Indexer_pass4SymmKey>
        ```
    - Options:
        - `-mode`: Sets the cluster mode (e.g., `peer`).
        - `-master_uri`: Specifies the URI for the master node.
        - `-secret`: Sets the Indexer pass4SymmKey.

    Example:
    ```bash
    ./splunk edit cluster-config -mode searchhead -master_uri https://10.0.0.220:8089 -secret s3cr3t@123
    ```
    *Replace `s3cr3t@123` with your preferred secret.*

4. **Execute Cluster Configuration Command**
    - Run the command in the Splunk terminal to apply the configuration changes.

5. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```
### Method 3: Configuration File

`Note: This method is intended for each search head cluster member.`

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
     [clustering]
     master_uri = https://<master_node_ip>:8089
     mode = searchead
     pass4SymmKey = <Indexer_pass4SymmKey>
     ```
     Replace `<master_node_ip>` with ip of manager node. \
     Replace `<Indexer_pass4SymmKey>` with the actual pass4SymmKey for the indexer.

5. **Save Changes:**
   - Save the `server.conf` file.

6. **Restart Splunk:**
   - Restart Splunk to apply the changes made in the configuration file:
     ```bash
     /opt/splunk/bin/./splunk restart
     ```

## Configuring Indexes

1. **Access Command Line Interface:**
   - Open a terminal or command prompt.
     
2. Navigate to `$SPLUNK_HOME/etc/master-apps/_cluster/local`.
     ```bash
     cd /opt/splunk/etc/master-apps/_cluster/local
     ```
     
3. **Create or Edit indexes.conf:**
   
    - To create a new configuration file if it doesn't exist:
    ```bash
     touch indexes.conf
    ```
   - To edit a configuration file
    ```bash
     vi indexes.conf
    ```

4. **Add Configuration:**
   - Inside `indexes.conf`, add the following content:
     ```ini
     [<index_name>]
     repFactor = auto
     homePath = $SPLUNK_DB/<index_name>/db
     coldPath = $SPLUNK_DB/<index_name>/colddb
     thawedPath = $SPLUNK_DB/<index_name>/thaweddb
     ```
     Replace `<index_name>`: Enter the desired index name..

5. **Validate restart before applying changes:**
    ```bash
    /opt/splunk/bin/splunk validate cluster-bundle --check-restart
    ```

6. **Apply the cluster bundle changes:**
    ```bash
    /opt/splunk/bin/splunk apply cluster-bundle
    ```
7. **Check the status of the cluster bundle:**
    ```bash
    /opt/splunk/bin/splunk show cluster-bundle-status
    ```
8. **Verification**
   - Verify that the index is reflected on each indexer in the cluster.
