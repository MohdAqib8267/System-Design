# System-Design
The process of establishing system aspects such as modules, architecture, components and their interfaces, and data for a system based on specified requirements is known as system design. It is the process of identifying, creating, and designing systems that meet a company’s or organization’s specific objectives and expectations. Systems design is more about system analysis, architectural patterns, APIs, design patterns, and glueing it all together than it is about coding.
# 1. What is CAP theorem?
CAP(Consistency-Availability-Partition Tolerance) theorem says that a distributed system cannot guarantee C, A and P simultaneously. It can at max provide any 2 of the 3 guarantees. Let us understand this with the help of a distributed database system.

>
<b> Consistency</b>: This states that the data has to remain consistent after the execution of an operation in the database. For example, post database updation, all queries should retrieve the same result.
>
  <b>Availability</b>: The databases cannot have downtime and should be available and responsive always.
 >
<b> Partition Tolerance</b>: The database system should be functioning despite the communication becoming unstable.
>
The following image represents what databases guarantee what aspects of the CAP Theorem simultaneously. We see that RDBMS databases guarantee consistency and Availability simultaneously. Redis, MongoDB, Hbase databases guarantee Consistency and Partition Tolerance. Cassandra, CouchDB guarantees Availability and Partition Tolerance.

<div align="center">
  <img width="600" height="50%" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/d10235b3-f353-462d-9493-71efd26a4c36" />
</div>


# 2. How is Horizontal scaling different from Vertical scaling?
> <b>Horizontal scaling</b> refers to the addition of more computing machines to the network that shares the processing and memory workload across a distributed network of devices. In simple words, more instances of servers are added to the existing pool and the traffic load is distributed across these devices in an efficient manner.

> <b>Vertical scaling</b> refers to the concept of upgrading the resource capacity such as increasing RAM, adding efficient processors etc of a single machine or switching to a new machine with more capacity. The capability of the server can be enhanced without the need for code manipulation.

<div align="center">
  <img width="600" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/a0261bb3-4e67-41d0-a752-4a308c14a148" />
</div>


Category            | Horizontal Scalling                                                               | Vertical Scalling
------------------- |-----------------------------------------------------------------------------------|----------------------------------------------------------------
Load Balancing      | Requires load balancing for distributing request traffic across multiple machines.| Since there is just one single machine, the load balancer is not required.
Failure Resilience  |This is more resistant to application failure because if one server fails, traffic is routed to other servers.| This is more resistant to application failure because if one server fails, traffic is routed to other servers.
Machine Communication | Since there are multiple machines being involved, it is very much necessary to have network communication. | Vertical scaling makes use of inter-process communication within the machine which makes it quite fast.
Data Consistency    | There exist possibilities of data inconsistencies here because there are different machines for handling different requests which might result in data being out of sync. | As there is only one machine, there is no issue of data inconsistency.
Limitations        | Since this scaling requires multiple servers, there might be concerns on budget and space but the scaling of the application can be done as much as needed based on the business needs. | Vertical scaling has a limit on the capacity of the resources that are achievable. If the resources are scaled up above this limit, then the application might crash and result in downtime.

# 3. What do you understand by load balancing? Why is it important in system design?
Load balancing refers to the concept of distributing incoming traffic efficiently across a group of various backend servers. These servers are called server pools. Modern-day websites are designed to serve millions of requests from clients and return the responses in a fast and reliable manner. In order to serve these requests, the addition of more servers is required. In such a scenario, it is essential to distribute request traffic efficiently across each server so that they do not face undue loads. Load balancer acts as a traffic police cop facing the requests and routes them across the available servers in a way that not a single server is overwhelmed which could possibly degrade the application performance.
<div align="center">
<img src="https://github.com/MohdAqib8267/System-Design/assets/106628860/75d00f2f-2ebf-4b44-8e50-741874fd6eee" width="60%" alt="Load Balancing">
</div>

# 4. What do you understand by Latency, throughput, and availability of a system?
Performance is an important factor in system design as it helps in making our services fast and reliable. Following are the three key metrics for measuring the performance:

Latency: This is the time taken in milliseconds for delivering a single message.

Throughput: This is the amount of data successfully transmitted through a system in a given amount of time. It is measured in bits per second.

Availability: This determines the amount of time a system is available to respond to requests. It is calculated: System Uptime / (System Uptime+Downtime)

# 5. What is Sharding?
Sharding is a process of splitting the large logical dataset into multiple databases. It also refers to horizontal partitioning of data as it will be stored on multiple machines. By doing so, a sharded database becomes capable of handling more requests than a single large machine. Consider an example - in the following image, assume that we have around 1TB of data present in the database, when we perform sharding, we divide the large 1TB data into smaller chunks of 256GB into partitions called shards.

<div align="center">
<img src="https://github.com/MohdAqib8267/System-Design/assets/106628860/9c5b75d8-2bad-46d0-9eb9-6db9a241bb95" width="60%" alt="Load Balancing">
</div>
Sharding helps to scale databases by helping to handle the increased load by providing increased throughput, storage capacity and ensuring high availability

# sharding vs partitioning
### Partitioning
In database systems, partitioning refers to splitting data based on a predefined set of attributes. Think of partitioning as a generic term for all data-splitting methods in a database. The purpose of partitioning is to increase the performance of database queries while ensuring the maintainability of the data.

### Sharding
While partitioning is a generic term for data splitting in a database, sharding is used for a specific type of partitioning, popularly known as horizontal partitioning.

In sharding, data is split horizontally into multiple shards. The schema of the table is replicated in every shard, and a unique portion of the whole table lives in each of them. A key is predefined to determine which shard a piece of data lives. The database checks the key whenever a new row comes and writes the data to a specific shard.

Example

Consider an example of a database table with a created_at column. The table gets populated every day and is growing. We have decided to split the table horizontally into multiple shards based on the created_at column to prevent the table from becoming too big.

One strategy could be having a shard for each month. All the rows for a specific month will be persisted on the shard for that particular month.

shard for that particular month.

<div align="center">
<img width="601" alt="Screenshot 2023-10-01 000412" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/ce53b502-9f6f-4947-b888-55373cd4713c">
</div>div>






