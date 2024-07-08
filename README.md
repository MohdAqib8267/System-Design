


<img width="586" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/0abd6e8b-946d-4f4d-8a33-ef19ee666a59">
So we have one service which is answer to everything, and that is Amazon EC2.
<img width="532" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/2dd25545-4337-415c-a311-9be2166599ff">
There is a misconception that you cannot run microservices on Amazon EC2, which is not correct. So as we saw before, each microservices can be running in their own Amazon EC2 and each of those Amazon EC2 could be part of different scaling groups with different scaling criteria.
You can select appropriate Amazon EC2 family based on the nature of the microservices.So some microservices memory intensive, you can select a higher memory EC2 and you can scale best of memory instead of CPU.

<img width="517" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/c42219ee-8b5e-4f1a-ac81-796e96d5e3e3">
You can use Elastic Load Balancer or Amazon API Gateway to host those microservices beyond EC2, for modern application development, we can look at the serverless alternative, which is AWS Lambda, where each microservices backend will be implemented in separate lambda functions. Remember that lambda scales automatically.
So there is no scaling group in this case, and lambda could also be fronted by Elastic Load Balancer or Amazon API Gateway.
<img width="550" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/7f4c65ff-58af-42e4-b31d-3afbddffeeb1">


<img width="568" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/8b75a723-839b-42d3-b3c2-ea2896b3b36a">
Now, another popular choice is using containers. 
You can containerised your application and run within Amazon Elastic Kubernets Service or Amazon Elastic Container Service or ECS. And in this case as well, you're confronted by using Elastic Load Balancer or Amazon API Gateway. So let's zoom in into the Kubernetes part because Kubernetes is super popular right now, and this might come up in your interview.

<img width="582" alt="image" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/dae0fe5a-1e66-4778-a59a-23f840f99553">
So for Kubernetes, each micro service will be fronted by different services, so shall we say,we have Service A, B and Service C and the code for each microservice will be running in a container and this container will run in a pod and all these will be fronted by a single ingress, which will be an application load balancer.

![image](https://github.com/MohdAqib8267/System-Design/assets/106628860/c056e07e-3010-45a9-b242-3491cc1c02e6)

since each microservices is independent of each other and can even be coded in different programming languages. You can mix and match all these services. So, for example, store/get can run on a pod when your back-end of the microservices is Dockerised into a container store.store/post can be hosted on Amazon EC2 and stored/delete can run on A.W.S lambda.


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
</div>

# 9. What is Caching? What are the various cache update strategies available in caching?
Caching refers to the process of storing file copies in a temporary storage location called cache which helps in accessing data more quickly thereby reducing site latency. The cache can only store a limited amount of data. Due to this, it is important to determine cache update strategies that are best suited for the business requirements. Following are the various caching strategies available:

### Cache-aside: 
In this strategy, our application is responsible to write and read data from the storage. Cache interaction with the storage is not direct.

<div align="center">
<img width="300" alt="Screenshot 2023-10-01 000412" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/c2365f9b-dd4b-4ef0-af72-b2a6dde74968">
</div>

### Write-through: 
In this strategy, the cache will be considered as the main data store by the system and the system reads and writes data into it. 

<div align="center">
<img width="300" alt="Screenshot 2023-10-01 000412" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/bb6a3e56-09d5-4be7-bd6c-abe3f93b8254">
</div>

### Write-behind (write-back):
In this strategy, the application does the following steps:
<div align="center">
<img width="400" alt="Screenshot 2023-10-01 000412" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/ef6de0f3-7e67-4393-a52e-6ace8046e339">
</div>

### Refresh-ahead:
Using this strategy, we can configure the cache to refresh the cache entry automatically before its expiration.
<div align="center">
<img width="300" alt="Screenshot 2023-10-01 000412" src="https://github.com/MohdAqib8267/System-Design/assets/106628860/7d9da52d-e2e5-422e-821e-b40365e3686d">
</div>
