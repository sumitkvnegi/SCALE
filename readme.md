### System Design

#### Distributed Computing / Systems
- Running a service across multiple systems to produce a unified result.

---

1. **Availability (Service Accessibility)**
   - **High Availability (HA)** measures how often a service is operational and accessible.
     - **Example:**  
       - **90% Availability:** Service downtime of ~36.5 days/year  
       - **99% Availability:** Service downtime of ~3.65 days/year  
       - **99.99% Availability:** (Industry benchmark) Less than 52 minutes of downtime per year.  
     - **Trade-off:** Higher availability generally increases costs and technical complexity.

2. **Reliability (Service Functionality)**
   - **Definition:** Whether the service consistently performs its intended functions without failure.
     - Does the application reliably meet its functional requirements?

3. **Consistency (Uniform Data Across Clients)**
   - **Definition:** Ensuring all clients access the same data at any given time.
     - Data must be synchronized across the system, providing a consistent experience for all users.

    1. **Strong Consistency**
       - **Definition:** Every read operation returns the most recent write for a given piece of data.
       - **How it works:** Changes are propagated from one node to others, ensuring that all nodes reflect the latest state before acknowledging the update.  
       - **Trade-off:** This can introduce latency, as the system waits for updates to propagate across all nodes, leading to slower responses.
       - **Example Scenario:** In banking applications, strong consistency is critical. Imagine a user transfers money from one account to another. If the system provides strong consistency, the system ensures that, across all nodes, the transfer is fully reflected everywhere before acknowledging it as complete. This prevents a situation where the user's balance is incorrectly shown as unchanged on one node after the transfer has occurred, avoiding issues like double-spending.

    2. **Eventual Consistency**
       - **Definition:** The system does not guarantee immediate consistency but ensures that all nodes will eventually converge to the same state.
       - **How it works:** Updates are confirmed on one node first, and the system then asynchronously updates other nodes. This can result in temporary inconsistencies, but eventual consistency is achieved over time.
       - **Trade-off:** Improved performance and lower latency, but with the risk of temporary data inconsistencies across nodes.
       - **Example Scenario:** In social media platforms like Facebook or Twitter, eventual consistency is often acceptable. When you post a status update, it doesn’t need to instantly appear on every server or node before you see it. Some users may see the update slightly later than others due to replication delays, but eventually, all users will see the same status. This is a trade-off for better performance and reduced latency, which is important in large-scale, globally distributed systems.

#### **Scalability**
- **Definition:** Scalability refers to a system's ability to handle an increasing workload or expand its capacity.
- **Key Aspects:**
  - **Workload and Computing:** Scalability deals with how a system can manage increased workload demands, both in terms of processing power and storage capacity.
  - **Handling Peak Usage:** A scalable system can handle **peak usage** efficiently, ensuring smooth performance during high-demand periods (e.g., Black Friday for an e-commerce site).
  - **Vertical Scaling:** Increasing the capacity of a single system (e.g., adding more CPU, RAM, or storage to a server). 
    - **Example:** Upgrading a web server from 16GB to 64GB of RAM.
  - **Horizontal Scaling:** Adding more systems (e.g., additional servers) to distribute the workload.
    - **Example:** Adding more web servers behind a load balancer to handle higher web traffic.
  - **Note:** Scalability doesn't inherently guarantee **availability**—a system can be scalable but still go down if not designed for redundancy and fault tolerance.

#### **Elasticity**
- **Definition:** Elasticity refers to a system's ability to **automatically scale up and down** in response to changing workloads.
- **Key Aspects:**
  - **Dynamic Adjustment:** An elastic system can automatically adjust computing resources in real-time to meet increasing or decreasing workload demands.
  - **Increase and Decrease in Workload:** When demand spikes, the system can quickly add resources (e.g., more instances or servers). When demand decreases, resources are scaled back to save cost.
  - **Example:** A cloud infrastructure like **AWS EC2** or **Azure VMs** can automatically add or remove virtual machines based on traffic patterns, ensuring cost efficiency while maintaining performance.

#### **Single Point of Failure (SPOF)**
- **Definition:** A **Single Point of Failure (SPOF)** is any part of a system that, if it fails, causes the entire system to fail.
- **Key Aspects:**
  - **Lack of Redundancy or High Availability:** A SPOF exists when a component (hardware, software, network link, etc.) does not have high availability or redundancy built in.
  - **Example:** A single web server or database that is critical to the system, with no failover or backup, becomes a SPOF. If that server goes down, the entire service goes offline.
  - **Mitigation:** To prevent SPOFs, systems should be designed with **redundancy**, **failover mechanisms**, and **high availability** features. For example, using multiple database replicas or deploying systems across multiple availability zones or data centers.

### Summary:
- **Scalability** is about expanding the system’s capacity to handle more load but does not guarantee availability. It can be achieved through **vertical scaling** (upgrading resources) or **horizontal scaling** (adding more resources).
- **Elasticity** is the ability to dynamically adjust resources to meet workload demands, automatically increasing or decreasing resources in real-time.
- **SPOF** refers to critical parts of a system that, if they fail, will cause the whole system to fail. Designing a system without SPOFs is essential for ensuring **reliability** and **availability**.

### **Database Cluster**

A **database cluster** typically consists of a **master node** and **slave nodes** (also known as replicas), which work together to provide high availability, fault tolerance, and scalability. The communication between these nodes can be **synchronous** or **asynchronous**, depending on the desired level of consistency and performance.

#### **Master Node (Write Node)**
- **Role:** The master node handles all **write operations** (INSERT, UPDATE, DELETE).
  - **Data Consistency:** Since it is the authoritative source of data changes, the master node ensures that all write operations are applied to the database.
  - **Writes Only:** While the master node handles both read and write operations, it is primarily responsible for writes, with reads often directed to slave nodes to balance the load.
  - **Data Synchronization:** The master node propagates changes to the slave nodes to ensure redundancy and data availability.

#### **Slave Nodes (Read Nodes / Replicas)**
- **Role:** The slave nodes are **read-only copies** of the master node that replicate its data.
  - **Read Operations:** Slave nodes handle **read operations** (SELECT), reducing the load on the master node and improving overall read performance.
  - **Replication:** Slave nodes maintain copies of the master’s data, either through **synchronous** or **asynchronous replication**.
  - **Fault Tolerance:** In the event of a master node failure, a slave node can be promoted to the master role, maintaining availability and fault tolerance.
  
#### **Replication Types**

1. **Synchronous Replication (Consistent)**
   - **How it Works:** In synchronous replication, the master node only acknowledges a write operation after the data has been successfully written to the master **and** all slave nodes.
   - **Consistency:** Guarantees **strong consistency** because all nodes are in sync with the latest data before a write operation is confirmed.
   - **Latency:** The trade-off is **latency**. Since the master has to wait for all replicas to confirm the write, this can slow down response times, especially if replicas are geographically distributed or under heavy load.
   - **Use Case Example:** Critical applications, such as **banking systems** or **financial transactions**, where data consistency is paramount, and it’s more important to avoid stale data than to optimize for performance.
   
2. **Asynchronous Replication (Eventually Consistent)**
   - **How it Works:** In asynchronous replication, the master node acknowledges a write operation as soon as the data is written to the master, without waiting for slave nodes to confirm.
   - **Consistency:** Provides **eventual consistency**, meaning that while the data may not be immediately available on all replicas, they will eventually catch up.
   - **Latency:** Reduces latency since the master doesn’t need to wait for the replicas to confirm the write operation. This leads to faster write response times.
   - **Trade-Off:** There is a possibility of **data inconsistency** for a short period, as some slave nodes may not have received the most recent write when read requests are made.
   - **Use Case Example:** Systems where **performance** is more important than immediate consistency, such as **social media platforms** or **e-commerce sites** where slight staleness in data is acceptable, but fast response times are crucial.

#### **Latency in Synchronous Replication**
- **Definition:** Latency refers to the delay that occurs when the system waits for data to be replicated and acknowledged across nodes.
- **Impact in Synchronous Replication:** In synchronous replication, the master node must wait for all slave nodes to confirm that they have successfully received and applied the write operation. If the nodes are located far apart (e.g., in different geographic regions), this can introduce significant **network latency**, slowing down the overall system performance.

---

### **Summary:**
- **Master Node (Write Node):** Handles all write operations and ensures data consistency across the system.
- **Slave Nodes (Read Nodes / Replicas):** Replicate data from the master and handle read requests, offering load balancing and fault tolerance.
- **Synchronous Replication (Consistent):** Ensures strong consistency but may introduce latency as the system waits for confirmation from all replicas.
- **Asynchronous Replication (Eventually Consistent):** Provides faster response times but may result in temporary data inconsistency until the replicas catch up.

### **ACID Compliance in Databases**

Databases are often described as **ACID-compliant**, which means they adhere to a set of properties that ensure reliable processing of database transactions. The acronym **ACID** stands for:

1. **Atomicity**
   - **Definition:** A transaction is treated as a single, indivisible unit of work, meaning that either all operations within the transaction are completed successfully, or none are. If any part of the transaction fails, the entire transaction is rolled back, leaving the database unchanged.
   - **Example:** If a bank transfer involves deducting money from one account and adding it to another, atomicity ensures that both operations happen together, or neither happens if there's an error (e.g., insufficient funds).

2. **Consistency**
   - **Definition:** Every transaction must move the database from one valid state to another, ensuring that all data obeys the rules and constraints defined in the schema. After a transaction, the database must remain in a consistent state, with no invalid data.
   - **Example:** If a bank's rule is that the total sum of all accounts must equal a certain amount, consistency ensures that no transaction violates this rule, even in the case of failures or crashes.

3. **Isolation**
   - **Definition:** Each transaction must be executed as if it is the only transaction in the system, even when multiple transactions are happening concurrently. The operations of one transaction should not be visible to others until the transaction is complete.
   - **Example:** If two users are trying to withdraw money from the same account simultaneously, isolation ensures that the account balance is consistent, and no transaction is able to "see" the other transaction until it is finished.

4. **Durability**
   - **Definition:** Once a transaction has been committed, its results are permanent, even if the system crashes immediately afterward. The changes made by the transaction are stored in a non-volatile medium and will survive power outages or system failures.
   - **Example:** After completing a bank transfer, even if the server crashes, the transaction's effects (i.e., the deduction and addition of amounts) will still be reflected when the system is restored.

---

### **Sharding (Database Partitioning)**

**Sharding** refers to the practice of **partitioning** a large database into smaller, more manageable pieces called **shards**. Each shard is an independent subset of the data, and together, they form a complete dataset. Sharding is often used to improve **scalability** and **performance** by distributing the database load across multiple servers.

#### **Key Aspects of Sharding:**

1. **Horizontal Partitioning**  
   - **Definition:** Sharding is a form of horizontal partitioning, where data is distributed across different databases (shards) based on certain criteria (e.g., user ID, geographic location).
   - **Example:** In an e-commerce platform, user accounts with IDs 1-1000 might be stored in **Shard A**, accounts 1001-2000 in **Shard B**, and so on.

2. **Sharding Key**  
   - **Definition:** A **shard key** is the attribute used to determine how data is partitioned among shards. It should be chosen carefully to balance the data distribution and avoid skewed workloads.
   - **Example:** If you shard a database based on user ID, you could use the **user ID** as the shard key, ensuring that related data (like user profiles or orders) is stored together in the same shard.

3. **Benefits of Sharding:**
   - **Scalability:** By distributing data across multiple servers, sharding allows databases to handle large-scale applications and high traffic loads that a single server cannot manage.
   - **Performance:** Queries can be processed faster because they are distributed across multiple shards, reducing the burden on any single server.
   - **Availability & Redundancy:** By splitting data into independent shards, it is possible to replicate each shard for fault tolerance. If one shard goes down, the others can still function, increasing availability.

4. **Challenges of Sharding:**
   - **Complexity:** Managing multiple shards increases complexity in terms of data distribution, querying, and maintaining consistency across shards.
   - **Joins Across Shards:** Since data is partitioned across multiple shards, performing joins across shards can be complicated and inefficient.
   - **Rebalancing:** As data grows, the system may need to rebalance the distribution of data among shards, which can be resource-intensive.

#### **Sharding Strategies:**

1. **Range-based Sharding:**  
   - **Description:** Data is partitioned based on a range of values for a specific shard key (e.g., user IDs from 1 to 1000 go to one shard, and 1001 to 2000 go to another).
   - **Example:** A database might be partitioned based on a customer ID range, where all customers with IDs 1–10000 go into **Shard 1**, customers 10001–20000 go into **Shard 2**, etc.

2. **Hash-based Sharding:**  
   - **Description:** The shard key is passed through a hash function to determine which shard the data should reside in.
   - **Example:** If you are sharding based on user ID, you could apply a hash function to the user ID and use the hash value to decide which shard stores that user's data.

3. **Directory-based Sharding:**  
   - **Description:** A lookup table or directory is used to map each shard key to a specific shard.
   - **Example:** A directory keeps track of which user IDs belong to which shard, so when a request for a specific user comes in, the system looks up the directory to find the right shard.
