


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

# Rate Limiting
Rate limiting helps protects services from being overwhelmed by too many requests from a single user or client.
## Rate Limiting Algorithms
### 1. Token Bucket Algorithm
![cea82243-07a3-48bc-a16b-1540cd175092_1204x1046](https://github.com/user-attachments/assets/73d72979-0dce-4bf3-852f-a136add7593b)
The Token Bucket algorithm is one of the most popular and widely used rate limiting approaches due to its simplicity and effectiveness.

###### How It Works:
Imagine a bucket that holds tokens.<br />
The bucket has a maximum capacity of tokens.<br />
Tokens are added to the bucket at a fixed rate (e.g., 10 tokens per second).<br />
When a request arrives, it must obtain a token from the bucket to proceed.<br />
If there are enough tokens, the request is allowed and tokens are removed.<br />
If there aren't enough tokens, the request is dropped.<br />

```
import { CronJob } from "cron";
import express from 'express';

const app = express();
app.use(express.json());

const RATE_LIMIT = 10;

const token_bucket=[];

//function to refill the bucket
const refillBucket = () =>{
    if(token_bucket.length < RATE_LIMIT){
        token_bucket.push(Date.now());
    }
}

// API endpoint to check the bucket's status
app.get('/bucket',(req,res)=>{
    res.json({
        bucketLimit:RATE_LIMIT,
        currentBucketSize: token_bucket.length,
        bucket:token_bucket
    })
})

//Middleware for rate limiting
const rateLimitMiddleWare=(req,res,next)=>{
    if(token_bucket.length>0){
        const token = token_bucket.shift(); //Shift (remove) the first element of the array:
        console.log(`Token ${token} is consumed...`);
        res.set('X-RateLimiting-Remaining',token_bucket.length);
        next();
    }
    else{
        res.status(429).set('X-RateLimit-Remaining', 0).set('Retry-After', 2).json({
            success: false,
            message: 'Too many requests'
        });
    }
}
app.use(rateLimitMiddleWare);

// Sample endpoint for testing rate limiting
app.get('/test',(req,res)=>{
    const ROCK_PAPER_SCISSORS = ['rock 🪨', 'paper 📃', 'scissors ✂️'];
    const index = Math.floor(Math.random()*3);
    const item = ROCK_PAPER_SCISSORS[index];

    res.json({
        success:true,
        message:`You got ${item}`
    })
})

// Cron job to periodically refill the bucket
const job = new CronJob('*/2 * * * * *',()=>{
    refillBucket();
    
})

app.listen(5000,()=>{
    console.log('server is running at port 5000');
    job.start();
})
```
###### Pros
Relatively straightforward to implement and understand. <br />
Allows bursts of requests up to the bucket's capacity, accommodating short-term spikes.
###### Cons:
The memory usage scales with the number of users if implemented per-user. <br />
It doesn’t guarantee a perfectly smooth rate of requests.

### 1. Leaky Bucket Algorithm
In the leaky bucket algorithm, requests are processed at a fixed rate. <br />
A queue holds the requests to be processed following a First-In-First-Out(FIFO) manner.
![79d555be-f951-4086-9fad-84884f21f517_1088x724](https://github.com/user-attachments/assets/0bab009a-a02d-4579-a258-7db0d3dbfe97)

The algorithm works as follows:

- When a request arrives, the system checks if the queue is full:
If it is not full, the request is added to the queue.
<br />
If the queue is full, the request is dropped.

- Requests are pulled from the queue and processed at regular intervals.
Leaky bucket algorithm takes **two parameters:**

- Bucket size: equal to the queue size.
- Outflow rate: how many requests can be processed at a fixed rate.

```
// Online C++ compiler to run C++ program online
#include <bits/stdc++.h>
#include <chrono>
#include <thread>
using namespace std;

class Packet {
    // Packet class to simulate data packets.
    int id, size;
public:
    Packet(int id, int size) {
        // Constructor to initialize packets.
        this->id = id;
        this->size = size;
    }
    int getSize() {
        return this->size;
    }
    int getId() {
        return this->id;
    }
};

class LeakyBucket {
    // Leaky Bucket class to simulate leaky bucket algorithm.
    int leakRate, bufferSizeLimit, currBufferSize;
    queue<Packet> buffer;
public:
    LeakyBucket(int leakRate, int size) {
        // Constructor to initialize leak rate, and maximum buffer size available.
        this->leakRate = leakRate;
        this->bufferSizeLimit = size;
        this->currBufferSize = 0;
    }
    void addPacket(Packet newPacket) {
        // Function to add new packets at the end of the buffer.
        if(currBufferSize + newPacket.getSize() > bufferSizeLimit) {
            // If the packet cannot fit in the buffer, then reject the packet.
            cout << "Bucket is full. Packet rejected." << endl;
            return ;
        }
        // Add packet to the buffer.
        buffer.push(newPacket);
        // Update current Buffer Size.
        currBufferSize += newPacket.getSize();
        cout<<"currBufferSize"<<currBufferSize<<endl;
        // Print out the appropriate message.
        cout << "Packet with id = " << newPacket.getId() <<  " added to bucket." << endl;
    }
    void transmit() {
        // Function to transmit packets. Called at each clock tick.
        if(buffer.size() == 0) {
            // Check if there is a packet in the buffer.
            cout << "No packets in the bucket." << endl;
            return ;
        }
        // Initialize n to the leak rate.
        int n = leakRate;
        while(buffer.empty() == 0) {
            Packet topPacket = buffer.front();
            int topPacketSize = topPacket.getSize();
            // Check if the packet can be transmitted or not.
            if(topPacketSize > n) break;
            // Reduce n by packet size that will be transmitted.
            n = n - topPacketSize;
            // Update the current buffer size.
            currBufferSize -= topPacketSize;
            // Remove packet from buffer.
            buffer.pop();
            cout << "Packet with id = " << topPacket.getId() << " transmitted." << endl;
        }
    }
};

int main() {
    LeakyBucket* bucket = new LeakyBucket(1000, 1500);
    bucket->addPacket(Packet(1, 200));
    bucket->addPacket(Packet(2, 500));
    bucket->addPacket(Packet(3, 400));
    bucket->addPacket(Packet(4, 500));
    bucket->addPacket(Packet(5, 200));
    
    while(true) {
        bucket->transmit();
        cout << "Waiting for next tick." << endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
    }
}
```


# Fault tolerence:
**Fault tolerence is the abolity of system to servive when a failure or error occurs.** <br/>
Example:here’s a simple demonstration of comparative fault tolerance in the database layer. In the diagram below,
![fault-tolerance-comparative-application-architecture-illustration](https://github.com/user-attachments/assets/988a5b94-5a87-4e73-9c6e-ef5f4d37f1ef)

In this scenario, Application 2 is more fault tolerant. If its primary database goes offline, it can switch over to the standby replica and continue operating as usual.
<br/>
Application 1 is not fault tolerant. If its database goes offline, all application features that require access to the database will cease to function.

*Fault tolerance can also be achieved in a variety of ways. <br/>
 **Hardware Faults:** suppose my cart fetching cart items from one DB. and my DB has been failed. <br/>
 Faults: DB Fails  <br/>
 Failure: Data Inaccessable <br/>
 solution: Make replicas of DB <br/>
**Software Faults**: Multiple instances of software capable of doing the same work. For example, many modern applications make use of containerization platforms such as Kubernetes so that they can run multiple instances of software services. One reason for this is so that if one instance encounters an error or goes offline, traffic can be routed to other instances to maintain application functionality. <br/>
**Human Errors**

# What is Consensus in Distributed System?
In a distributed system, multiple computers (known as nodes) are mutually connected with each other and collaborate with each other through message passing. Now, during computation, they need to agree upon a common value to coordinate among multiple processes. This phenomenon is known as Distributed Consensus.

<img  src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*6do-COi_H0dqaWtu7zVJCw.png" />
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*LXXq91obzfYaY-yQJ84_vA.png" />
<img  src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*s-6WGvm6vlIuEid4xMTSPg.png" />

### Why is Consensus required? <br/>
In a distributed system, it may happen that multiple nodes are processing large computations distributedly and they need to know the results of each node to keep them updated about the whole system. In such a situation, the nodes need to agree upon a common value. This is where the requirement for consensus comes into the picture.

### Challenges in Distributed Consensus
A distributed system can face mainly two types of failure.

1.**Crash failure** <br/>
2.**Byzantine failure**

**1.Crash failure** occurs when a node is not responding to other nodes of the system due to some hardware or software or network fault. This is a very common issue in distributed systems and it can be handled easily by simply ignoring the node’s existence.
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*6RePWyW9-V8n8aSenmP3vA.png"/>

**2.Byzantine failure** is a situation where one or more node is not crashed but behaves abnormally and forward a wrong message to different nodes, due to an internal or external attack on that node. Handling this kind of situation is complicated in the distributed system.
<img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*ra0k2v6m1wyOxwNTIZ_hdQ.png" />

### Consensus Algorithms
1. Paxos Algorithm (Learn in Later)
2. Raft Algorithm
3. Byzantine Fault Tolerance (BFT) Algorithms

### Application of Distributed Consensus
Consensus algorithms are used in many real-world applications in distributed or decentralized networks —

✅ Blockchain and cryptocurrencies <br/>
✅ Google page-rank <br/>
✅ Load balancing etc…

# Long-polling vs WebSockets
## Understanding Real-Time Communication
Real-time communication refers to the ability of a server to push information to a client as soon as it becomes available, without the client having to request it explicitly. <br/><br/>
Long-Polling and WebSockets are two strategies to overcome this limitation.
<br/>
Real-time communication between clients and servers is essential for applications like chat applications, live sports updates, stock tickers, online gaming, and collaborative tools.

## Long-polling
Long-Polling is a technique where the client makes an HTTP request to the server, and the server holds the request open until new data is available.
<br/>
Once the server has new data, it responds to the client, which then immediately makes a new request. This creates a continuous connection for real-time updates.
![3529dbe1-1e31-47ac-8e7b-955cc3803d5d_932x1004](https://github.com/user-attachments/assets/dc414077-1685-4085-bfe2-462981dc6e73)

<br/>
### How Long-polling Works
1.The client sends a request to the server.
<br/>
2.The server doesn't immediately respond. Instead, it holds the request open and waits for data to become available.
<br/>
3.When data becomes available (or after a timeout), the server responds to the request.
<br/>
4.The client immediately sends another request, restarting the process.
<br/>
### Advantages
**Simplicity:** Easy to implement with standard HTTP infrastructure.
<br/>
**Compatibility:** Works with existing firewalls and proxies without additional configuration.
<br/>


### Disadvantages
**Latency:** Higher latency compared to WebSockets due to the time required to establish new HTTP connections.
<br/>
**Overhead:** Increased overhead from frequent HTTP requests and responses.
<br/>
**Scalability:** Harder to scale due to the number of open HTTP connections and resource consumption on the server.
<br/>
### When to Use Long-polling
When you need to support older browsers or environments where WebSockets aren't available.
<br/>
For applications with infrequent updates where near real-time is sufficient.
<br/>
When working with existing infrastructure that doesn't support WebSockets.
<br/>
For simple applications where the added complexity of WebSockets isn't justified.

## WebSockets
WebSockets provide a full-duplex communication channel over a single, long-lived connection.
<br/>
Once the WebSocket connection is established, the server and client can send messages to each other independently and asynchronously, making it suitable for real-time, low-latency applications.
![3a6f0751-b4a6-486a-86cb-9552c605544e_700x700](https://github.com/user-attachments/assets/d2a28933-f202-4cad-941f-04189eff15b3)
### How WebSockets Work
1.The client initiates a WebSocket connection through a process called the WebSocket handshake.
<br/>
2.Once the handshake is successful, the connection is upgraded from HTTP to WebSocket.
<br/>
3.Both the client and server can send messages to each other at any time.
<br/>
4.The connection remains open until either party decides to close it.
<br/>
### Advantages
**True real-time:** Allows for genuine real-time communication with minimal latency.
<br/>
**Full-duplex communication:** Both client and server can send messages independently of each other.
<br/>
**Efficiency:** After the initial handshake, the overhead per message is very low.
<br/>
**Reduced server load**: The server doesn't need to handle and maintain multiple connections for the same client.
<br/>
### Disadvantages
**Potential lack of support:** Not supported in some older browsers (though this is becoming less of an issue).
<br/>
**Proxies and firewalls:** Some proxies and firewalls may not handle WebSocket connections correctly.
<br/>
**Stateful:** The server needs to maintain the state of each connection, which can be memory-intensive with many concurrent connections.
<br/>
**Complexity:** Implementing WebSockets can be more complex than traditional HTTP requests.
<br/>
### When to Use WebSockets
For applications requiring true real-time updates (e.g., live sports updates, real-time collaboration tools).
<br/>
When you need bi-directional communication between client and server.
<br/>
For applications with frequent updates where minimizing latency is crucial.
<br/>
When efficiency in terms of bandwidth and server resources is a priority for applications with many concurrent users.

## Comparison: Long-polling vs WebSockets
![2cec4136-ba2c-433c-af8a-d137eaf2a53a_1952x1872](https://github.com/user-attachments/assets/21d8dfcb-c682-4907-9c6c-df14550b703e)



# Synchronous vs Asynchronous Communication
![682466c0-3abb-4f0b-8787-89447c04f9c5_1532x972](https://github.com/user-attachments/assets/03872787-3d34-4e8d-a381-e25abb5ae34e)

## Synchronous Communications
Synchronous communication is a communication pattern where the sender waits for the receiver to acknowledge or respond to the message before proceeding.
<br/>
This type of communication is often referred to as blocking or request-response communication.
<br/>
### Advantages:
**Immediate Feedback:** Synchronous communications provide instant feedback, allowing for swift error detection and correction.
<br/>
**Simple Implementation:** Synchronous designs are often straightforward to implement, as the request and response occur in a single, continuous transaction.
<br/>
**Consistency:** Data consistency is easier to manage because updates are processed in order.
<br/>
### Disadvantages:
**Blocking:** The sender is blocked until a response is received, potentially leading to resource waste and decreased system performance.
<br/>
**Tight Coupling:** Synchronous communications can create tight coupling between components, making it challenging to evolve or replace individual components without affecting the entire system.
<br/>
**Resource Intensive:** Each request must be fully processed before moving to the next, potentially leading to resource underutilization.
<br/>
### Use Cases:
**Low-latency applications:** Synchronous communications are suitable for applications requiring real-time responses, such as video streaming or online gaming.
<br/>
<br/> Synchronous designs are ideal for straightforward transactions, like querying a database or fetching cached data.

## Asynchronous Communications
Asynchronous communication is a communication pattern where the sender does not wait for the receiver to process the message and can continue with other tasks. The receiver processes the message when it becomes available.
<br/>
### Advantages:
**Non-Blocking:** The sender does not block and can continue executing other tasks after sending the message, reducing resource waste and improving system performance.
<br/>
**Loose Coupling:** The sender and receiver are loosely coupled, allowing them to operate independently.
<br/>
**Scalability:** Asynchronous communication enables better scalability as the sender and receiver can process messages at their own pace.
<br/>


### Disadvantages:
**Complex Implementation:** Asynchronous designs can be more challenging to implement, as they require additional mechanisms for handling responses and errors.
<br/>
**Data Consistency:** Ensuring data consistency across different parts of the system can be more complex.

### Use Cases:
**High-throughput applications:** Asynchronous communications are suitable for applications requiring high throughput, such as message queues or task processing.
<br/>
**Decoupled systems:** Asynchronous designs are ideal for systems with multiple, independent components, like microservices architecture.
<br/>
**Long-running tasks:** Offloading non-urgent tasks to an asynchronous queue, like image processing or report generation, is ideal.
<br/>
**Event-driven architectures:** Asynchronous communication shines in systems where components react to real-time events, such as notifications.
<br/>

# What's the difference between RPC and REST?
Remote Procedure Call (RPC) and REST are two architectural styles in API design. APIs are mechanisms that enable two software components to communicate with each other using a set of definitions and protocols. Software developers use previously developed or third-party components to perform functions, so they don’t have to write everything from scratch. RPC APIs allow developers to call remote functions in external servers as if they were local to their software. For example, you can add chat functionality to your application by remotely calling messaging functions on another chat application. In contrast, REST APIs allow you to perform specific data operations on a remote server. For example, your application could insert or modify employee data on a remote server by using REST APIs.


# Content Delivery Networks (CDNs)
Imagine you have built an app used by millions of users worldwide.
<br/>
Your app also serves video content, but all your videos are hosted in one geographical location.
<br/>
Due to the large physical distance, users in other locations experience significant latency and buffering while watching videos. This is expected since the data takes time to travel from the server to the client.
<br/>
One popular solution to this issue is to place these video files closer to user locations.
<br/>
This is what a Content Delivery Network (CDN) in Picture.
![a09696e9-f98e-47bd-9eb9-08a20c60464d_5667x2834](https://github.com/user-attachments/assets/0c3996d9-7cde-480e-bbb7-ec43fd4838d3)
### What is a CDN?
A CDN is a geographically distributed network of servers that work together to deliver web content (like HTML pages, JavaScript files, stylesheets, images, and videos) to users based on their geographic location.
<br/>
The primary purpose of a CDN is to deliver content to end-users with high availability and performance by reducing the physical distance between the server and the user.
<br/>
When a user requests content from a website, the CDN redirects the request to the nearest server in its network, reducing latency and improving load times.
### How Does a CDN Work?
**Content Caching:** When a user requests content, the CDN caches this content in servers located closer to the user, known as edge servers. Subsequent requests for the same content are served from these edge servers, reducing the load on the origin server.
<br/>
**Content Delivery:** If the edge server has the requested content cached, it delivers it directly to the user. If not, it retrieves the content from the origin server, caches it for future requests, and then delivers it to the user.
<br/>
**Content is Refreshed:** CDNs periodically update cached content to ensure users receive the latest version.

### Benefits of Using a CDN
**Improved Website Load Times:** By serving content from servers closer to the user's geographic location, CDNs significantly reduce latency and improve website load times.
<br/>
**Reduced Bandwidth Costs:** By offloading traffic from the origin server, CDNs reduce bandwidth costs and server load, potentially lowering overall infrastructure expenses.
<br/>
**Increased Content Availability and Redundancy:** With content distributed across multiple servers, CDNs provide a failover mechanism that ensures content remains available even if one or more servers go offline.
<br/>
**Better User Experience:** Faster load times and increased reliability translate to a better user experience, which can lead to higher engagement and conversion rates.
<br/>
**Global Reach:** CDNs make it easier to deliver content to users worldwide, regardless of their location.
<br/>
**Scalability:** CDNs can handle traffic spikes more efficiently than traditional hosting, making them ideal for websites with fluctuating traffic patterns.
### Popular CDN Providers
Akamai, Clouflare, Google Cloud CDN,Amazon CloudFront

# HeartBeats: How Distributed Systems Stay Alive
In a distributed system,suppose things fail.
<br/>
But, how do we know if a particular service is alive and working as expected?
<br/>
This is where heartbeats come into play.
![ce69caff-3a6e-4a0d-ba9c-97f3c07d105d_1162x808](https://github.com/user-attachments/assets/0e81f645-137d-4e93-bb6f-2194322b61cd)

## What exactly is a Heartbeat?
> In distributed systems, a heartbeat is a periodic message sent from one component to another to monitor each other's health and status.
Its primary purpose is to signal, "Hey, I'm still here and working!"
<br />
This signal is usually a small packet of data transmitted at regular intervals, typically ranging from seconds to minutes, depending on the system's requirements.
<br />
Without a heartbeat mechanism, it's hard to quickly detect failures in a distributed system, leading to:
1.Delayed fault detection and recovery
<br />
2.Increased downtime and errors
<br />
3.Decreased overall system reliability
## How Do Heartbeats Work?
The heartbeat mechanism involves two primary components:
<br />
**Heartbeat sender (Node):** This is the node that sends periodic heartbeat signals.
<br />
**Heartbeat receiver (Monitor):** This component receives and monitors the heartbeat signals.
<br />
Here's a simplified overview of the process:
<br />
The node sends a heartbeat signal to the monitor at regular intervals (e.g., every 30 seconds).
<br />
The monitor receives the heartbeat signal and updates the node's status as "alive" or "available".
<br />
If the monitor doesn't receive a heartbeat signal within the expected timeframe, it marks the node as "unavailable" or "failed".
<br />
The system can then take appropriate actions, such as redirecting traffic, initiating failover procedures, or alerting administrators.
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

