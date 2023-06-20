# **Sharding the Key Value Service**


# **Overview**

The Protected Audience Key Value Service stores an in-memory data set of all key/value pairs needed to support end users queries.  AdTechs have expressed that their data sets are larger than what can fit in a single machine's memory.  We will need to support data sharding while maintaining user privacy.


# **Requirements**

The following high level requirements are broken down into two sections: feature set and privacy.  The utility section breaks down the work into everything needed to support a sharded peer to peer data set.  The privacy section defines what is required to maintain user privacy while querying this sharded data set.


## **Feature set**

The goal is to be able to scale our Key Value service to support arbitrarily large data sets without sacrificing any of the other requirements we have.


### **Functionality**

The sharding concept is internal to the system. Devices and external clients should not know that the service is sharded.


##### **Static sharding**

Shard numbers can be fixed and set at system startup. No dedicated hotspot detection. Service operator is responsible for determining hotspots by checking metrics like QPS, error rate, RAM usage, and etc.

In the event that AdTech demands dynamic resharding of data, we can address this later.  It is not currently considered a requirement.


### **Reliability**


#### _High availability_

We will offer the ability for the AdTech to operate shard replicas.  The failure of a single replica should not be detrimental.  The number of replicas is up to the AdTech.


#### _In-Flight Resharding_

The reason we need in-flight resharding is that, as the amount of data grows AdTechs will need to be able to add more shards.

It should be possible to reshard without a spike in latency.  An AdTech needs to be able to reshard without taking down the entire system.


#### _Consistency_

All peers should converge towards data consistency. They do not all need to share the same data freshness at the same time.


#### _Goal_

No central point of failures. Have a clear understanding of what effect a degradation of each component has on the system.


### **Minimal overhead**


#### _Infrastructure Complexity_

Avoid new cloud service requirements where possible. This helps with the cost, ease of support and debugging. All peer servers within the network should have the same specification.


#### _Latency_

Additional serving latency is constant with the launching of shards, but does not grow with the number of shards.  Latency will grow as shards-queried-per-request goes up.

Support batching of cross peer lookups to match the potential batching of lookups queried by the UDF.


#### _Optional_

Sharding is optional, in the event AdTech can fit all data within a single shard, it should work without any additional overhead.


#### _Cost_

Cost should be estimated and be predictable.


### **Scaling**

The system owner can change the total shard number to scale horizontally as resources are added.  It is a problem where you can throw additional peer instances at the problem.


## **Privacy**

Adding sharding exposes the attack vectors that use the fact that an AdTech controls a UDF -- user defined function. All such attacks use the fact that when lookup requests are made to data servers, an AdTech can observe those encrypted downstream requests. Since an AdTech controls the load balancer, an AdTech could route only one request out of all requests to a dedicated EC2 box for which the downstream requests are observed.

### The attacks we are currently aware of are:
#### 1. Number of lookup requests

A UDF can be created that queries data servers once for key A, and twice for key B. The number of roundtrips per UDF can reveal information about the key.
#### 2. Timing of lookup requests
UDF allows an AdTech to wait for extra time between the requests. E.g. if a key is A wait for 20 ms, before making one more query to the data server, and for key B wait 40 ms.
#### 3. Payload size of lookup requests
UDF controls the payload size of downstream requests which is one more way to reveal what's being queried.
It would be possible to mitigate some of those attacks by creating dummy network requests to shards that do not contain relevant data, adding noise to the response time, or padding queries with dummy data.

Such mitigations would be prohibitively expensive for server operators and may not fully address the issues. For instance, side-channel attacks on the bidding server itself might still be possible. In addition, they would not be meaningful [until TEE usage is required for K/V servers](https://developer.chrome.com/docs/privacy-sandbox/fledge-api/feature-status/#trusted-execution-environment-usage-for-keyvalue-service). We will continue to investigate potential mitigations for the known attacks until then.

Note that because it will not be possible to do network level replay attacks, and there are certain limitations and additional cost and constraints to "client side" replay attacks (such as triggering an auction multiple times), the practicality and scalability of known attacks remains to be proven.


## **Future Plans**

We are exploring options to make the data storage have lower cost in the future, by potentially allowing some data to be stored on disc. Currently, we’re trying to assess the feasibility of this approach and the associated tradeoffs, e.g. how much extra latency that would add. Please open a [GitHub issue](https://github.com/privacysandbox/fledge-key-value-service/issues) or reach out to us if you would like to provide us with feedback regarding your scale.
