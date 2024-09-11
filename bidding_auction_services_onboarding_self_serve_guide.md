> FLEDGE has been renamed to Protected Audience API. To learn more about the name change, see the [blog post](https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge)

**Authors:** <br>
[Alek Kundla][4], Google Privacy Sandbox<br>
[Jaspreet Arora][3], Google Privacy Sandbox<br> 
[Daniel Kocoj][2], Google Privacy Sandbox<br>
[Priyanka Chatterjee][1], Google Privacy Sandbox

# Bidding and Auction services onboarding and self-serve guide

This document provides guidance to adtechs to onboard to [Bidding and Auction
services (B&A)][5].

You may refer to B&A services timeline and roadmap [here][6].
You may refer to the [high level architecture and design [here][7].
You may also refer to [privacy considerations][8], [security goals][9] and the
[trust model][10].

If you have any questions, please submit your feedback using the Privacy Sandbox
feedback [form][11] or file a github issue [here][12].

Following are the steps that adtechs need to follow to onboard, integrate, deploy,
test, scale and run B&A services in non-production and production environments.

## Step 1: Enroll with Privacy Sandbox and onboard to B&A
* Refer to the [guide][13] to enroll with Privacy Sandbox and enroll using the form 
  documented [here][124]. This is a prerequisite for onboarding to B&A.

* Onboard to B&A (Protected Auction service) and enroll with [Coordinators][37] by filling out this
  [form][125].

  _Note:_
   * _Adtechs need to choose one of the currently [supported cloud platforms][27] to run B&A services._
   * _Set up a cloud account on a preferred [cloud platform][27] that is supported._ 
   * _Provide the specific information related to their cloud account in the intake form._
   * _Adtechs can onboard more than one cloud account (e.g. AWS account IDs or GCP service accounts) for the
     same enrollment site._

## Step 2: Setup for ad auctions  

### Seller

  * Refer to [spec for SSP][15].
  * Develop [scoreAd][16]() for Protected Audience auction.
  * Develop [reportResult][17]() for event level reporting.
  * Setup [seller's key/value service][18].
  * Add support such that the seller's code on the publisher web page calls [browser API][19]
    and/or [Android API][20] to fetch encrypted [ProtectedAudienceInput][21]. Then includes
    encrypted [ProtectedAudienceInput][21] in the request to the seller's ad server.
  * Add support in the [seller's ad server][22] to send [SelectAd][23] requests to
    Bidding and Auction services for Protected Audience auctions.
  * Refer to [SelectAdRequest][68] and [SelectAdResponse][69] in [B&A service API][24].
  * Add support for [forwarding client metadata][25] in the request to Bidding
    and Auction services (SellerFrontEnd).
  * Review [logging][26].

### Buyer

Buyers could configure B&A to support Protected Audience (PA) both for Chrome and Android traffic 
and/or [Protected App Signals][120] (PAS) for Android only. 
Most of the configuration is shared between the two use cases. We will indicate what parts are specific 
to one or the other.

  * Refer to [spec for DSP][28] for the PA configuration and integrate with [the PAS specific changes][117] for PAS.
  * Develop [generateBid][29]() for PA bidding or [generateBid][118]() for PAS.
  * Develop [reportWin][30]() for event level reporting (for both PA and PAS).
  * For PA:
    * Setup [buyer's Key/Value service][31].
    * If your Key/Value server supports filtering of interest groups, refer to this
      section and [metadata forwarding][25].
  * For PAS:
    * If you are planning to use use contextual ads retrieval: 
      * Setup the [buyer's trusted Key/Value service][121] 
    * If you are planning TEE ads retrieval:
      * Setup the [buyer's trusted Ad Retrieval service][122]
      * Develop [prepareDataForAdRetrieval][123]()
      * Develop the [handleAdsFetchRequest][124][] UDF to retrieve the ads stored in the K/V server  
  * [Optimize payload][32].
  * Review [logging][26].

### Cloud platforms

  * Adtechs need to choose one of the currently [supported cloud platforms][27] to run
    B&A services. Refer to the corresponding cloud support explainer for details:
    * [AWS support][33]
      * Adtechs must set up an [AWS account][34], create IAM users and security
          credentials.
    * [GCP support][35]
      * Create a GCP Project and generate a cloud service account. Refer to the
          [GCP project setup][36] section for more details.

## Step 3: Coordinator integration

**_Step 1 covers enrollment with Coordinators as part of onboarding to B&A. This step
   provides further guidance around coordinator integration._**

The Coordinators run [key management systems (key services)][43] that provision keys to
Bidding and Auction services running in a [trusted execution environment (TEE)][44] after
service attestation. Integration of Bidding and Auction server workloads with
Coordinators would enable TEE server attestation and allow fetching live
encryption / decryption keys from [public or private key service endpoints][43] in
Bidding and Auction services.

_Note:_
 * _Refer to [B&A release notes][126] for the prod images allowed by Coordinators
    associated with a specific release._
 * _Adtechs can only use production images attested by the key management systems_
   _(Coordinators) in production. This is required during ([Beta testing][38],
    [Scale testing][39] and [beyond][40]) with live traffifc._
 * _Adtechs must use [Test mode](#test-mode) with debug (non-prod) images. This would
    disable attestation of TEEs._
 * _Without successfully onboarding to the Coordinator, adtechs will not be able to run_
   _attestable services in TEE and therefore will not be able to process production data_
   _using B&A services._ 
 * _Key management systems are not in the critical path of B&A services. The_
   _cryptographic keys are fetched from the key services in the non critical_
   _path at service startup and periodically every few hours, and cached in-memory_
   _server side._
 * _Sellers can call buyers on a different cloud platform. The SellerFrontEnd_
   _service fetches public keys from all supported cloud platforms, this is_
   _important to encrypt request payloads for BuyerFrontEnd service._

**After onboarding to B&A, Coordinators will provide url endpoints of key services to
the adtechs. Adtechs must incorporate those in B&A server configurations. Refer below
for more information.**

### Test mode

Test mode supports [cryptographic protection][45] with hardcoded public-private key
pairs, while disabling TEE server attestation. Adtechs can use this mode with debug / non-prod
B&A service images for debugging and internal testing.

During initial phases of onboarding, this would allow adtechs to test Bidding and Auction
server workloads even before integration with Coordinators.

 * Set TEST_MODE flag to `true` in seller's Bidding and Auction server configurations
   ([AWS][46], [GCP][47]) when using debug / non-prod service images.
 * _Note: TEST_MODE should be set to `false` in production with prod service images_.

### Amazon Web Services (AWS)

An adtech should provide their AWS Account Id to both the Coordinators.

The Coordinators would create IAM roles. After adtechs provide the AWS account Id, 
they would attach that information to the IAM roles and include in an allowlist. 
Then the Coordinators would let adtechs know about the IAM roles and that should be included
in the B&A server Terraform configs that fetch cryptographic keys from key management systems.

Adtechs must set IAM roles information provided by the Coordinators in the following parameters
in buyer or seller server configs for AWS deployment:
  * PRIMARY_COORDINATOR_ACCOUNT_IDENTITY
  * SECONDARY_COORDINATOR_ACCOUNT_IDENTITY

### Google Cloud Platform (GCP)

An adtech should provide IAM service account email to both the Coordinators.

The Coordinators would create IAM roles. After adtechs provide their service account email,
the Coordinators would attach that information to the IAM roles and include in an allowlist. 
Then the Coordinators would let adtechs know about the IAM roles and that should be included 
in the B&A server Terraform configs that fetch cryptographic keys from key management systems.

Adtechs must set IAM roles information provided by the Coordinators in the following parameters
in buyer or seller server configs for GCP deployment:
  * PRIMARY_COORDINATOR_ACCOUNT_IDENTITY
  * SECONDARY_COORDINATOR_ACCOUNT_IDENTITY

## Step 4: Build, deploy services

Follow the steps only for your preferred cloud platform to build and deploy B&A
services.

### B&A code repository

Bidding and Auction services code and configurations are open sourced to [Github repo][52].

The hashes of Coordinator approved images of B&A services will be published to
the [B&A release page][53]. 

### Prerequisites

The following prerequisites are required before building and packaging B&A services. 
 * AWS: Refer to the prerequisite steps [here][54].
 * GCP: Refer to the prerequisite steps [here][55].

### Build service images

To run B&A services locally, refer [here][56].

Adtechs must build images of B&A services to generate hashes specific to a
supported cloud platform and then verify that those match the ones published on
the [B&A release page][53]. 
 * AWS: Refer to the detailed instructions [here][57].
 * GCP: Refer to the detailed instructions [here][58].

### Deploy services 

_Note: The configurations set the default parameter values. The values that are_
_not set must be filled in by adtechs before deployment to the cloud._

#### AWS

 * Refer to understand Terraform configuration setup and layout here.
 * Refer to the [README][106] for deployment.
 * Follow the steps [here][107] to create configurations for different environments
   for deployment.
   * Seller: To deploy [SellerFrontEnd (SFE)][108] and [Auction][109] server
      instances, copy paste [seller/seller.tf][46] and update custom values. 
   * Buyer: To deploy [BuyerFrontEnd (BFE)][110] and [Bidding][111] server
      instances, copy paste [buyer/buyer.tf][87] and update custom values.
 * _Note: Adtechs are not required to update the default configurations before_
   _B&A services deployment._

#### GCP

 * Refer to understand Terraform configuration setup and layout [here][112].
 * Refer to the [README][113] for deployment.
 * Follow the steps [here][114] to create configurations for different environments
   for deployment.
   * Seller: Deploy [SellerFrontEnd (SFE)][108] and [Auction][109] server instances,
      copy paste [seller/seller.tf][47] and update custom values.
   * Buyer: Deploy [BuyerFrontEnd (BFE)][110] and [Bidding][111] server instances,
      copy paste [buyer/buyer.tf][89] and update custom values.
 * _Note: Adtechs are not required to update the [default configurations][115]_
   _before B&A services deployment._

## Step 5: Integration with clients 

### Integration with web browser

Refer to [browser and B&A integration][59] for more details.

#### Chrome web browser

Chrome web browser provides a flag `FledgeBiddingAndAuctionKeyURL` that should
be set to the public key endpoint that can serve a public key such that
corresponding private keys of the same version are known to the Bidding and
Auction services.

For end-to-end functional testing from the browser with B&A services running in
[Test mode][42], the endpoint can be set to the following:

```
FledgeBiddingAndAuctionKeyURL/http%3A%2F%2Flocalhost%3A8000%2Fkey
```

The above endpoint can be configured to serve a public key similar to the
following:

```
$ mkdir -p /tmp/bna && cd /tmp/bna && echo '{ "keys": [{ "id": "40", "key": "87ey8XZPXAd+/+ytKv2GFUWW5j9zdepSJ2G4gebDwyM="}]}' > key && python3 -m http.server 8000
```

When B&A services run in [production][60] and serve live traffic from Chrome browser,
the flag `FledgeBiddingAndAuctionKeyURL` needs to be set to the public key service
endpoint in [key management systems][43] in production, run by Coordinators. Note
the following:
 * The public key service endpoint would vary per cloud platform.
 * Clients (browser, Android) will configure the public key service endpoints
   corresponding to those cloud platforms where the sellers are running B&A
   services. This is because clients encrypt [ProtectedAudienceInput][21] data using
   public keys prefetched from the endpoint that would be decrypted in
   SellerFrontEnd service running in a trusted execution environment, run by the
   seller.
   * _Note: Chrome will enable multi cloud support by [Beta2][61]._

### Integration with Android

Refer to [Android and B&A integration][62] for more details. This section will be
updated at a later date.

## Step 6: Test

### Payload generation tool

The [secure invoke][63] tool is a payload generation tool that uses hardcoded public
keys to encrypt payloads and then sends requests to TEE-based B&A services. The
corresponding private keys of the same version are hardcoded / configured in B&A
services such that the encrypted payloads can be correctly decrypted. Refer to
the [README][64] for more information about the tool.
 * The tool works with B&A services running in [test mode][42].
 * The tool can also work with B&A services when [Coordinators][65] are enabled.
   The tool has the ability to use custom live public keys for encryption that
   can be fetched from the public key service endpoint of [key management systems][43]
   running on a supported cloud platform. Refer [here][67] for more details.

#### Seller

 * The tool can generate [SelectAdRequest][68] payload for communication with TEE
   based SellerFrontEnd if a plaintext request payload is supplied. The payload
   will include [ProtectedAudienceInput][21] ciphertext that is encrypted with
   hardcoded public keys.
 * The tool also has the capability to decrypt the response received from Bidding
   and Auction services and print out the out human readable plaintext response.

#### Buyer

 * The tool can generate [GetBidsRequest][70] payload for communication with TEE based
   BuyerFrontEnd if a plaintext request payload is supplied.
 * The tool also has the capability to decrypt the response received from TEE
   based BuyerFrontEnd and print out the out human readable plaintext response.

### Local testing

Adtechs can test the services locally. This step is optional.

Refer to the [README][71] to understand how B&A services are built and run locally.
 * Refer here to [test buyer services][72] (BuyerFrontEnd, Bidding) locally.
 * Refer here to [test seller services][73] (SellerFrontEnd, Auction) locally.

### Functional testing

It is recommended to do function testing during the initial [onboarding phase][74].

For functional testing, B&A services must be enabled in [Test mode][42] so that
Coordinators can disabled.

There are couple of options for functional testing:
 * Option 1: Using the [secure invoke][63] tool.
   In this case, B&A services can be tested without real client (web browser,
   Android app) integration. B&A services running in [Test mode][42] can receive
   encrypted requests generated by the tool and the response from B&A services
   can be decrypted by the tool.

 * Option 2: [End to end testing][75] from Chrome browser.

 ### Load testing

 To determine scalability requirements to support desired throughput, adtechs
 may run a few load tests.

It is recommended to do load testing before running services in production,
however this step is optional.

_NOTE: Buyers can run load tests independently for buyside auction (bid generation)._
_However, sellers may need to coordinate with at least one partner buyer to conduct_
_load tests for final auction of bids returned by buyer(s). Alternatively they can_
_deploy a fake BFE server for these load tests. We recommend setting up fake buyer_
_services so that they have the same expected range of latencies as with real buyer_
_services._ 

Steps:
 * Refer to the B&A [load testing guide][76], that includes guidance to conduct load
  tests along with recommended [load testing tools][77].
 * Gather a few encrypted sample requests using the [secure_invoke][63] tool.
   Refer to secure_invoke tool [guide][64] for more information.
 * Deploy B&A services on one of the [supported cloud platforms][27].
   * Start with about the largest instance type of each service with which you
     plan to run. 
   * If you don't have a preference initially, start with a VM instance of 32
     virtual CPUs (cores) with high memory. The Auction and Bidding servers
     especially require instances with >1 GB memory per vCPU.
   * Set Auction, BuyerFrontEnd, Bidding services to the same VM instance type initially. 
   * You can use the standard memory instances for SellerFrontEnd.
 * Bring up one instance for each service.
 * Make sure there is an adequate resource quota in the region you’re testing.

## Step 7: Traffic experiments

### Non-production traffic experiments 

Non-production traffic experiments can be enabled during the initial adtech
onboarding phase, also known as the Alpha testing phase.

**Alpha testing phase is a rolling window.** When an adtech onboards, they are
recommended to do the initial setup, integration, service deployment and functional
testing during this phase.

_Note: The timelines for testing for web and Android apps may be different._
_However, every adtech that onboards to B&A is recommended to do Alpha testing_
_before enabling production traffic experiments._

### Production traffic experiments 

#### Experiments for web

Adtechs can participate in B&A's [Beta 1 and Beta 2][38] testing with 
**limited stable production traffic** for Protected Audience auctions on the web.

Chrome browser has enabled [Origin Trial][78] tokens for testing with B&A. Refer
[here][79] to register.

#### Experiments for Android apps

This section will be updated at a later date.

## Step 8: Multi-region support

Multi-regional replication is recommended for scaling, fault tolerance and
achieving [higher service availability][85] in production.

### AWS

It is recommended to be familiar with [AWS regions and availability zones][86].
Region is a physical location where data centers are clustered. Availability
zone is one or more discrete data centers interconnected with high-bandwidth,
low-latency networking in an AWS Region.

Sellers can configure multiple regions in [seller][46] deployment configuration.
Buyers can configure multiple regions in [buyer][87] deployment configuration.
Refer to the following for an example:

```
locals {
  region      = ["us-central1", "us-west1"]
  environment = "prod"
}
```

### GCP

It is recommended to be familiar with [GCP regions and zones][88]. A region is a
specific geographical location where adtechs can host their resources. Regions
have three or more zones. 

Sellers can configure multiple regions in [seller][47] deployment configuration.
Buyers can configure multiple regions in [buyer][89] deployment configuration. Refer
to the following for an example:

```
# Example config provided cloud region us-central1 and "asia-south-1". 
# Similarly, more regions can be added.
"us-central1" = {
  collector = {
    ...
  }
  backend = {
    ...
  }
  frontend = {
    ...
  },
"asia-south1" = {
  collector = {
    ...
  }
  backend = {
    ...
  }
  frontend = {
    ...
  }
}
```

## Step 9: Customize load balancing

Following section provides an overview of cloud infrastructure setup for B&A
and provides scalability and performance recommendations.

### AWS

* Adtechs may refer to the [load balancing configuration][90].
  * _Note: Adtechs are not required to update this configuration._
* The frontend services (SellerFrontEnd, BuyerFrontEnd) are load balanced by
  [application load Balancer][91]. 
* The backend services (Auction, Bidding) are load balanced by internal load
  balancer. 
  * _As an optimization, frontend and backend services for both seller and buyer_
    _will be configured in [AWS App Mesh][92], this will eliminate the need of internal_
    _load balancers. This optimization will be available by [Scale Testing][39] phase._
* The default load balancing algorithm used is `least_outstanding_requests`.

### GCP

* Adtechs may refer to the [load balancing configuration][93].
  * _Note: Adtechs are not required to update this configuration._
* The frontend services (SellerFrontEnd, BuyerFrontEnd) are load balanced by
  [global external application load balancer][94]. 
* The backend services (Auction, SellerFrontEnd) are co-located in a
  [service mesh][95] that is facilitated by the [Traffic Director][96].
* The default locality load balancing algorithm is `ROUND_ROBIN`; this is set
  in [load balancing configuration][93]. This controls load balancing to machines
  within a specific zone. 
* Traffic distribution is based on **load balancing algorithm (policy) and load balancing mode.**

In a few specific scenarios, the type of request protocol is also a factor for traffic distribution. Following are the protocol types supported by B&A services.
  * SellerFrontEnd service can receive external traffic that may be HTTP or gRPC. 
    Since the service depends on the gRPC framework, HTTP traffic is translated to gRPC by Envoy Proxy.
  * The following server to server communication would always be gRPC. 
    * SellerFrontEnd and Auction services
    * SellerFrontEnd and BuyerFrontEnd services
    * BuyerFrontEnd and Bidding services

#### Load balancing modes

##### Utilization mode 

The recommended load balancing strategy is [utilization mode][97] which aims to
make each “server group” serve a fixed percentage of its capacity (defaults to 80%).
In this context, a “server group” refers to host machines in the same zone within
the same region. Capacity is calculated for each “server group,” but not for
individual machines (the locality load balancing policy distributes traffic to
individual host machines).

**Caveat**
*In our observations, utilization mode works best for traffic sent via gRPC.
However, for traffic sent via HTTP 2 or HTTP 1.1, utilization mode tends to favor
a specific zone for all load levels and that may in turn result in machines not
in the favorited zone sitting idle and their capacity being wasted.* 

*[As per GCP's documentation, Google Cloud may prefer a specific zone if average utilization of all VMs is less than 10% in the region.][98]*

**Recommendation**
 * _If SFE receives external traffic via gRPC from the seller's ad server, the_ 
   _recommendation is to configure utilization mode for the global load balancer_
   _for SFE._
 * _If SFE receives external traffic via HTTP, the recommendation is to manually_ 
   _[specify a single zone][100] for SFE instances in each region. This will reduce_
   _redundancy protection, but will result in even incoming traffic distribution_
   _across hosts. In this case, it is not required to specify a single zone for_
   _Auction service._
 * _Do not attempt to set `max_rate_per_instance` for utilization mode in the_
   _[seller's configuration][47]. This is a no-op._

The following is an example for the seller to specify zones in a given region
for the SellerFrontEnd service in the seller's configuration. The `frontend`
zone setting is highlighted.

```
# Example config provided for us-central1 region.
"us-central1" = {
  ...
  frontend = {
        machine_type          = "n2d-standard-64"
        min_replicas          = 1
        max_replicas          = 2
        zones                 = ['us-central1-a']
        max_rate_per_instance = null # Null signifies no max.
  }
}
```

##### Rate Mode

[Rate mode][101] allows the specification of a `max_rate_per_instance` in requests
per second (RPS), that would control the traffic that each server instance receives.

This is an alternate mode for load balancing that requires carefully determining
the rate. However, it doesn't lead to zone favoritism for HTTP traffic, that may
be observed with utilization mode.

**Recommendation**
If the SellerFrontEnd service receives HTTP traffic and the recommendation for
[utilization mode][99] doesn't suffice, the following are the steps to configure rate
mode for the global load balancing distributing traffic to SellerFrontEnd service.

Adtechs can set the `max_rate_per_instance` per service in the seller's
[configuration][47] for frontend (SellerFrontEnd) service only. In this case, it
would be important to determine the target RPS per SellerFrontEnd instance.

```
# Example config provided for us-central1 region.
"us-central1" = {
  ...
  frontend = {
        machine_type          = "n2d-standard-64"
        min_replicas          = 1
        max_replicas          = 2
        zones                 = ['us-central1-a']
       # Note: 1000 is random, not suggested rate. The rate needs to be
       # determined.
        max_rate_per_instance = 1000 
  }
}
```

###### Determine optimal maximum rate per virtual machine instance
Refer to the following steps to determine the maximum rate per virtual machine (VM)
instance.

Adtechs can refer to the [scalability and performance tuning][102] section to
choose an instance size.
 * Start with one VM instance each with a few different sizes (types) that you
   intend to run with.
 * Send a very low amount of traffic, say 30 QPS, through the system.
   * Use the metrics dashboards to record P95 and P99 latencies through the load
      balancer and through the frontend service. These would be the baseline
      latency numbers.
      * _Note: Latencies tend to go down when you add more instances, even when_
          _single instances are not overwhelmed or close to the threshold. So P95s_
          _and P99s may in practice be better than these numbers._
 * Ramp up traffic
   * If you are using a synchronous load-generation tool like [wrk2][103]:
      * Set the desired rate to something high.
      * Use the metrics dashboards to observe incoming and outgoing RPS. This
          will give you the maximum RPS that can be supported with this setup at
          the cost of higher latency.
   * If you are using an asynchronous load generation tool like [ghz][104]:
      * The goal is to send as much (but not more than) what the servers can
        handle.
      * Use the metrics dashboards to track responses with status OK (200)
        and the P95 latency. If you have a high proportion of non-200 responses
        or a P95 latency that is too high, dial back the QPS until requests are
        all succeeding.
   * Once you have an idea of what the servers can handle successfully at any
     latency, dial back the QPS until you see latency numbers you find acceptable.

_Note: It is not recommended to set this mode for all services in [main][93] load_
_balancing configuration. This is because the communication between other B&A services_
_would be gRPC, hence default utilization mode is expected to work better. Additionally,_
_not all types of services (frontend, backend) would require the same amount of capacity._

### Load Shedding

It is recommended for sellers to devise load shedding mechanisms in the seller's
ad server based on contextual signals, size of encrypted [ProtectedAudienceInput][21]
and other parameters. This can help determine whether to throttle traffic to B&A.

Note:
 * Within B&A, SellerFrontEnd service calls buyers only if there is corresponding
   buyer input (or Interest Groups) in [ProtectedAudienceInput][21] and the buyer
   is also present in `buyers_list` in `auction_config` in [SelectAdRequest][68] as
   suggested by the seller.
 * Other advanced throttling and replay attack mitigations are planned but not
   incorporated in B&A, refer here for more information.

## Step 10: Scalability and performance tuning

For scaling your services, and tuning latency and performance, it is important to
determine the following for every B&A service:
 * Optimal type of virtual machine (VM) instance
 * Minimum and maximum number of replicas per cloud region
 * Multi-regional replication

### Configuration update

After determining the values stated above, sellers need to update a few parameters
in [seller][47] deployment configuration. Buyers also need to update the same
parameters in [buyer][89] deployment configuration.

The parameters are as follows and need to be set for `backend` and `frontend`
services, per cloud region.
 * `machine_type` (type of VM instance)
   * Refer to the scaling strategies section to determine the instance type for
      different services.
 * `min_replicas`
 * `max_replicas`

The following parameters are optional, per cloud region:
 * Setting `zones` explicitly may be useful in a particular scenario, refer to
   [utilization mode][99] for more information.
 * Setting `max_rate_per_instance` may be useful when using rate mode for load balancers.  
   Refer to [rate mode][105] for more information.

```
# Example config provided cloud region us-central1. Multi regional 
# replication can be supported similarly.
"us-central1" = {
  ...
  backend = {
        machine_type          = "n2d-standard-64"
        min_replicas          = 1
        max_replicas          = 5
        zones                 = null # Null signifies no zone preference.
        max_rate_per_instance = null # Null signifies no max.
  }
  frontend = {
        machine_type          = "n2d-standard-16"
        min_replicas          = 1
        max_replicas          = 2
        zones                 = ['us-central1-a']
        max_rate_per_instance = null # Null signifies no max.
  },
"asia-south1" = {
  collector = {
    ...
  }
  backend = {
    ...
  }
  frontend = {
    ...
  }
}
```

### Scaling Strategies

The following scaling and latency strategy will help determine the size and
number of instances required to serve scaled traffic with an acceptable latency.
This would involve the following:
 * Vertical scaling
 * Horizontal scaling

#### Vertical scaling

Selecting a VM instance type (virtual CPUs, memory) to support a reasonable RPS.
We recommend scaling up till the point of diminishing returns in performance
(when the gain in RPS for an acceptable P95 latency is too little to justify the
increase in instance size/cost).

The instance types can be estimated and fine-tuned with load testing. The formulae
documented below can help ad-techs choose the types of instances you can use to
run the load tests.

**Recommendation**
*A good rule of thumb is to have smaller frontend instances serving larger backend instances.
Usually the backend services, that is Bidding Service or Auction service, will require more
compute power for the same RPS compared to the frontend services (SellerFrontEnd or BuyerFrontEnd services).
The Auction and Bidding services also require instances with >1 GB memory per vCPU.*

##### Determine baseline latency

We need to find the baseline latency of a B&A request for your payload and scripts.
This is the best case latency that is observed when the services are not loaded.
This can be used to determine when the server’s performance starts degrading.
 * Send a few individual requests (<=1RPS) to SFE/BFE. 
 * Note down the overall P95 latency from the Seller/Buyer dashboards on the cloud
   service or the load test client.
   * If `generateBid()` or `scoreAd()` latency seems too high, this would mean
     that your Bidding or Auction service instance needs more compute power. The
     recommendation is to use a larger instance type for Bidding or Auction services.
   * If buyer or seller key/value service lookup latency seems too high compared
     to expected or reported latency metrics, this would mean that the BFE or SFE
     services need more network bandwidth for downloading larger responses.

##### Determine backend instance size 

The goal is to determine the instance type of backend services (Auction, Bidding)
for 'x' RPS.

To determine the size of instances for your setup, you can start with a target RPS
of 50-200.
 * Choose frontend instances of the same type as backend instances. This helps
   ensure that frontend instances are not the bottleneck during this test.
 * Slowly ramp up load on this setup till the P95 latency is in an acceptable
   range. Once the latency suffers too much, ramp down the load till it becomes
   acceptable again. This is this setup’s maximum capacity.
 * This maximum capacity is usually driven by the size of the backend instances.
   We can then jump to the next section for determining the smallest frontend
   instances that can support this load.
 * Alternatively, if we want to continue scaling till we can achieve the maximum
   capacity equal to the  RPS target, we can start to scale up or scale down the
   server instances.
 * If the maximum capacity was less than the required RPS - monitor P95 latency
   metrics to determine which server is the bottleneck - probably the Bidding or
   Auction server. Try increasing the instance’s size and test again.
   * Repeat this process until you either:
      * Reach an instance so large that you do not want to run such an instance
          regardless of its performance.
      * Reach diminishing returns in performance (the gain in RPS for an acceptable
          P95 latency is too less for the increase in size/cost). At this point,
          we can refer to the horizontal scaling section to increase the number
          of instances instead of size. For eg. 2 Bidding service instances for
          1 Auction service instance.
   * If the maximum capacity is more than the required RPS - try decreasing the
      instance size and test again. Repeat this process until you reach a P95
      latency performance that is not acceptable.
 * Note the following:
   * `generateBid() `/ `scoreAd()` latency seems too high - This means that your
      Bidding/Auction service instance needs more compute power. Try to increase
      the size of the instance.
   * `generateBid()` / `scoreAd()` latency latency is fine but Bidding / Auction
      service latency is high - The requests are waiting in the ROMA queue. Use
      a small ROMA queue for best latency performance (<100).
   *  Throughput seems low - Increase the ROMA queue size if the tolerance for
      latency is higher.

##### Determine frontend instance size for ‘x’ RPS

The goal is to determine the instance type of frontend services (SellerFrontEnd, BuyerFrontEnd) for 'x' RPS.

After your backend instance size is fixed for a target RPS, we can determine the
smallest frontend instances that can support this RPS, since smaller B&A frontend
service instances can usually support larger B&A backend service instances.
 * Reduce the size of the instance for your frontend. Test again. Repeat this
   process until  you reach a P95 latency performance that is not acceptable.
 * If the latency for any of the following is high, this would imply corresponding
   frontend (BFE / SFE) instances need more compute power. Try to increase the
   size of the instance.
   * Real time signals from key/value service
   * Bidding initiated call
   * Auction initiated call

#### Estimate an instance size

##### Seller

###### SellerFrontEnd service

The SellerFrontEnd (SFE)  services’ memory requirements scale with the average
number of ads per request, and the average size of `trustedScoringSignals` per ad.

<table>
  <tr>
    <td><strong>Number of ads per ad request</strong></td>
    <td><strong>Target RPS per instance</strong></td>
    <td><strong>Size of `trustedScoringSignals` per ad</strong></td>
    <td><strong>Total memory required for a SellerFrontEnd instance</strong></td>
  </tr>
  <tr>
    <td>A</td>
    <td>R</td>
    <td>S</td>
    <td>~ A * R * S</td>
  </tr>
</table>

###### Auction Service

The Auction service usually requires higher compute power than the SFE to support
the same RPS, since every `scoreAd()` execution happens in isolation per ad.

<table>
  <tr>
    <td><strong>Number of ads per ad request</strong></td>
    <td><strong>Target RPS per instance</strong></td>
    <td><strong>p95 latency of  `scoreAd()` and reporting executions</strong></td>
    <td><strong>Total vCPUs required in an Auction instance</strong></td>
  </tr>
  <tr>
    <td>A</td>
    <td>R</td>
    <td>L (ms)</td>
    <td>~ A * R * L / 1000
        
    (Total scoring and reporting executions per second) * 
                (latency for each scoring execution)
         
    Note: Reporting urls are generated in Auction service after all scoring executions.
    The latency overhead is very small. Refer [here][116] for more details.
  </tr>
</table>

###### Determine SFE / Auction instances ratio

A **single** SFE instance can support a higher RPS with multiple Auction server
instances without any degradation in performance. To figure out how many Auction
service instances can be served by a single SFE service instance, you can run the
following test -
 * Deploy your services with instance sizes as determined in the last step. Set
   the min Auction service instances to 1 and max number of instances to 2 or 3.
   Set the min and max SFE service instances to 1. Use a low utilization number
   for the autoscaler (eg. 60%).
 * Set up a mock Buyer BFE service (or coordinate with a buyer) to support a high
   load (3-5X RPS) with a P95 latency range as expected for live traffic.
 * Start loading the service with a small RPS X (eg. 50 or 100 RPS).
 * Slowly start to ramp up the load from X to 2X or 3X RPS.
 * The autoscaler should automatically bring up more Auction service instances
   as required. The P95 latency might degrade while the autoscaler brings up more
   instances, but wait for the server performance to stabilize to an acceptable
   level of latency.
 * Keep increasing the load (and increasing the Bidding service instances) as
   long as the overall latency is within the acceptable range.
 * Note the number of Auction instances and the RPS that can be handled by a
   single SFE service instance.
 * This should give you the max RPS that a single SFE and Auction services
   combination can serve.
   * For example, a single small SFE instance might be able to serve 3X RPS
     with 3 Auction service instances.

##### Buyer

###### BuyerFrontEnd service

The BuyerFrontEnd (BFE) services’ memory requirements scale with the average
number of interest groups per request, and the average size of `trustedBiddingSignals`
per interest group.

<table>
  <tr>
    <td><strong>Number of Interest Groups per buyer per ad request</strong></td>
    <td><strong>Target RPS per instance</strong></td>
    <td><strong>Size of `trustedBiddingSignals` per Interest Group</strong></td>
    <td><strong>Total memory required for a BuyerFrontEnd instance (M)</strong></td>
  </tr>
  <tr>
    <td>I</td>
    <td>R</td>
    <td>S</td>
    <td>~ I * R * S</td>
  </tr>
</table>

###### Bidding service

The Bidding service usually requires higher compute power than the BFE to support
the same RPS, since every `generateBid()` execution happens in isolation per
interest group. 

<table>
  <tr>
    <td><strong>Number of Interest Groups per buyer per ad request</strong></td>
    <td><strong>Target RPS per instance</strong></td>
    <td><strong>p95 latency of a `generateBid()` execution</strong></td>
    <td><strong>Total vCPUs required in a Bidding instance</strong></td>
  </tr>
  <tr>
    <td>I</td>
    <td>R</td>
    <td>L (ms)</td>
    <td>~ I * R * L / 1000
        
    (Total `generateBid()` executions per second * latency for each
    `generateBid()` execution)
  </tr>
</table>

###### Determine BFE / Bidding instances ratio

A **single** BFE service instance can support a higher RPS with multiple Bidding
service instances without any degradation in performance. To figure out how many
Bidding service instances can be served by a single BFE service instance, you
can run the following test -
 * Deploy your services with instance sizes as determined in the last step.
   Set the min Bidding service instances to 1 and max number of instances to 2
   or 3. Set the min and max BFE service instances to 1. Use a low utilization
   number for the autoscaler (eg. 60%).
 * Start loading the service with a small RPS X (eg. 50 or 100 RPS).
 * Slowly start to ramp up the load from X to 2X or 3X RPS.
 * The autoscaler should automatically bring up more Bidding service instances
   as required. The P95 latency might degrade while the autoscaler brings up
   more instances, but wait for the server performance to stabilize to an
   acceptable level of latency.
 * Keep increasing the load (and increasing the Bidding service instances) as
   long as the overall latency is within the acceptable range.
 * Note the number of Bidding instances and the RPS that can be handled by a
   single BFE service instance.
 * This should give you the max RPS that a single BFE and Bidding services
   combination can serve.
   * For example, a single small BFE instance might be able to serve 3X RPS with
     3 Bidding service instances.

#### Horizontal Scaling 

After determining instance type and per instance RPS, adtechs can add more such
instances to support a higher RPS per cloud region and globally across cloud
regions. The SLO (Service Level Objective) for availability of B&A services run
by adtechs would be decided by them. Depending on the service availability SLO,
servers need to be replicated within a cloud region and in more than one region.

Once the size of instances and the ratio of frontend to backend services has been
determined using the previous steps, we can add more server instances as required
for supporting a higher RPS by configuring the autoscaler -

**Minimum instances** - you should bring up at least the number of B&A service
instance combinations required to serve your average QPS.

**Maximum instances** - This should be set to handle the peak QPS for your setup.

**Autoscaler threshold** - This is the metric used to determine when the autoscaler
should start adding more instances to handle the load on your setup. Ideally, the
autoscaler should bring up new instances to seamlessly handle any extra load
without performance degradation. Some cloud providers might also overflow some
traffic to other regions while the autoscaler brings up new instances. The threshold
is usually based on CPU utilization and can be determined using the following steps -

 * Set up your services with instance sizes as determined in the last step and
   allow the autoscaler to start more instances with a high threshold. We want
   the server performance to suffer before the autoscaler kicks in. You will
   need an asynchronous load generation tool to overwhelm the servers (like
   [ghz][104]) to send more requests than the servers can handle.
 * Start loading your servers up to the RPS determined in the last step (X). 
 * At this point, note the utilization of each service instance.
 * Slowly start to ramp up the load from X to 2X RPS.
 * Notice where the performance of the instances starts degrading or requests
   start getting dropped, and note down the utilization percent of each service
   at that point.
 * Figure out how long(t) the autoscaler takes to start up a new instance and
   recover. Note the utilization of the overwhelmed instances time before the
   instances were overwhelmed and/or requests started dropping.
 * Use this utilization threshold for the autoscaler to start bringing up new
   instances.

## Step 11: Determine service availability

Adtechs should determine service availability (uptime) of B&A services they
operate. Higher availability can be achieved with multi regional replication. 

Given B&A are real time services for ad selection, 3 nines uptime or above is
recommended. 

Adtechs would deploy, run and maintain their B&A instances in production. The
availability service level objective (SLO) of the instances need to be monitored
by adtechs.

## Step 12: Enable debugging and monitor services

Refer to the [debugging explainer][80] to understand how user consented debugging can
be used in production.

Refer to the [monitoring explainer][81] to understand how cloud monitoring is
integrated and service [metrics][82] that are exported for monitoring by adtechs.

## Known issues

Prewarmed connections have a keep alive timeout, and the host machine brings
down the connections. This means if the services are sitting idle and not serving
traffic for around 20 mins, the first request will result in a failed response.
 * B&A plans to handle warming up the connection in the critical path for such
   cases. However that is not supported now.
 * _Note: Setting up connections in the request path would lead to higher_
   _latency._

## Github issues

Adtechs can file issues and feature requests on [Github][83]. 

## Related publications

Refer to related publications on [Github][84].

[1]: https://github.com/chatterjee-priyanka
[2]: https://github.com/dankocoj-google
[3]: https://github.com/jasarora-google
[4]: https://github.com/akundla-google
[5]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md
[6]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#timeline-and-roadmap
[7]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#high-level-design
[8]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#privacy-considerations
[9]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#security-goals
[10]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trust-model
[11]: https://docs.google.com/forms/d/e/1FAIpQLSePSeywmcwuxLFsttajiv7NOhND1WoYtKgNJYxw_AGR8LR1Dg/viewform
[12]: https://github.com/privacysandbox/fledge-docs/issues
[13]: https://developers.google.com/privacy-sandbox/private-advertising/enrollment
[14]: #enrollwithcoordinators
[15]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#spec-for-ssp
[16]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#scoread
[17]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#reportresult
[18]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#seller-byos-keyvalue-service
[19]: https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md
[20]: https://developer.android.com/design-for-safety/privacy-sandbox/protected-audience-bidding-and-auction-integration
[21]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/api/bidding_auction_servers.proto#L58
[22]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#sellers-ad-service
[23]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/api/bidding_auction_servers.proto#L288
[24]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/api/bidding_auction_servers.proto
[25]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#metadata-forwarding
[26]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#logging
[27]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#supported-public-cloud-platforms
[28]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#spec-for-dsp
[29]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#generatebid
[30]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#reportwin
[31]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#buyer-byos-keyvalue-service
[32]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding-auction-services-payload-optimization.md
[33]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_aws_guide.md
[34]: https://docs.aws.amazon.com/signin/latest/userguide/introduction-to-iam-user-sign-in-tutorial.html
[35]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md
[36]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#gcp-project-setup
[37]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#deployment-by-coordinators
[38]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#beta-testing
[39]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#scale-testing
[40]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#fast-follow
[41]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#alpha-testing
[42]: #testmode
[43]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#key-management-systems
[44]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trusted-execution-environment
[45]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#client--server-and-server--server-communication
[46]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/seller/seller.tf
[47]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/seller/seller.tf
[48]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#enrollment-with-aws-coordinators
[49]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#enrollment-with-gcp-coordinators
[50]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#enroll-with-coordinators
[51]: https://docs.google.com/forms/d/e/1FAIpQLSduotEEI9h_Y8uEvSGdFoL-SqHAD--NVNaX1X1UTBeCeEM-Og/viewform
[52]: https://github.com/privacysandbox/bidding-auction-servers
[53]: https://github.com/privacysandbox/bidding-auction-servers/releases
[54]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_aws_guide.md#step-0-prerequisites
[55]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#step-0-prerequisites
[56]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/debug#running-servers-locally
[57]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_aws_guide.md#step-1-packaging
[58]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#step-1-packaging
[59]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#browser---bidding-and-auction-services-integration
[60]: #productiontrafficexperiments 
[61]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#beta-2-february-2024
[62]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#android---bidding-and-auction-services-integration
[63]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/secure_invoke
[64]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/secure_invoke/README.md
[65]: #step3:enrollwithcoordinators
[66]: #cloudplatforms
[67]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/secure_invoke/README.md#using-custom-keys
[68]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/api/bidding_auction_servers.proto#L300
[69]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/api/bidding_auction_servers.proto#L453
[70]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/api/bidding_auction_servers.proto#L491
[71]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/debug/README.md
[72]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/debug/README.md#test-buyer-stack
[73]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/debug/README.md#test-seller-stack
[74]: #non---productiontrafficexperiments
[75]: #integrationwithwebbrowser
[76]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/load_testing
[77]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/load_testing#recommended-load-testing-tool
[78]: https://chromestatus.com/feature/4649601971257344
[79]: https://developer.chrome.com/origintrials/#/view_trial/2845149064591310849
[80]: https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md
[81]: https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md
[82]: https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md#proposed-metrics
[83]: https://github.com/WICG/protected-auction-services-discussion
[84]: https://github.com/privacysandbox/protected-auction-services-docs
[85]: #step11:determineserviceavailability
[86]: https://aws.amazon.com/about-aws/global-infrastructure/regions_az/
[87]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/buyer/buyer.tf
[88]: https://cloud.google.com/compute/docs/regions-zones
[89]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/buyer/buyer.tf
[90]: https://github.com/privacysandbox/bidding-auction-servers/tree/c346ce5a79ad853c622f64ffd5082e3d1a4457d6/production/deploy/aws/terraform/services/load_balancing
[91]: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html
[92]: https://aws.amazon.com/app-mesh/
[93]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/production/deploy/gcp/terraform/services/load_balancing/main.tf
[94]: https://cloud.google.com/load-balancing/docs/https
[95]: https://en.wikipedia.org/wiki/Service_mesh
[96]: https://cloud.google.com/traffic-director
[97]: https://cloud.google.com/load-balancing/docs/backend-service#utilization_balancing_mode
[98]: https://cloud.google.com/load-balancing/docs/backend-service#:~:text=If%20the%20average,the%20load%20balancer
[99]: #utilizationmode
[100]: https://github.com/privacysandbox/bidding-auction-servers/blob/9a17ea6e12bdd0720c1a6ab3e4b4932ebd66621d/production/deploy/gcp/terraform/environment/demo/seller/seller.tf#L152
[101]: https://cloud.google.com/load-balancing/docs/backend-service#rate_balancing_mode
[102]:#step10:scalabilityandperformancetuning
[103]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/load_testing#wrk2
[104]: https://ghz.sh/
[105]: #ratemode
[106]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/README.md
[107]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/README.md#using-the-demo-configuration
[108]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#sellerfrontend-service
[109]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#auction-service
[110]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#buyerfrontend-service
[111]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#bidding-service
[112]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#step-2-deployment
[113]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/README.md
[114]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/README.md#using-the-demo-configuration
[115]: https://github.com/privacysandbox/bidding-auction-servers/tree/b27547a55f20021eb91e1e61b0d2175b4aee02ea/production/deploy/gcp/terraform/services
[116]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_event_level_reporting.md#rationale-for-the-design-choices
[117]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md#buyer-ba-services
[118]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md#preparedataforadretrieval-udf
[119]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md#generatebid-udf
[120]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md
[121]: https://github.com/privacysandbox/protected-auction-key-value-service/blob/main/docs/tee_kv_server_overview.md
[122]: https://github.com/privacysandbox/protected-auction-key-value-service/blob/main/docs/ad_retrieval_overview.md
[123]: https://github.com/privacysandbox/protected-auction-key-value-service/blob/main/docs/protected_app_signals/ad_retrieval_overview.md#udf-api
[124]: https://developers.google.com/privacy-sandbox/private-advertising/enrollment#how_to_enroll
[125]: https://docs.google.com/forms/d/e/1FAIpQLSduotEEI9h_Y8uEvSGdFoL-SqHAD--NVNaX1X1UTBeCeEM-Og/viewform
[126]: https://github.com/privacysandbox/bidding-auction-servers/releases
