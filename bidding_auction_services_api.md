> FLEDGE has been renamed to Protected Audience API. To learn more about the name change, see the [blog post](https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge)

**Authors:** <br>
[Priyanka Chatterjee][26], Google Privacy Sandbox<br> 
Itay Sharfi, Google Privacy Sandbox

# Bidding and Auction Services High Level Design and API

[The Privacy Sandbox][4] aims to develop technologies that enable more private
advertising on the web and mobile devices. Today, real-time bidding and ad
auctions are executed on servers that may not provide technical guarantees of
security. Some users have concerns about how their data is handled to generate
relevant ads and in how that data is shared. Protected Audience API
([Android][24], [Chrome][5]) provides ways to preserve privacy and limit
third-party data sharing by serving personalized ads based on previous mobile
app or web engagement.

For all platforms, Protected Audience may require [real-time services][6].
In the initial [proposal by Chrome][7], bidding and auction for Protected
Audience ad demand is executed locally. This can demand computation requirements
that may be impractical to execute on devices with limited processing power, or
may be too slow to render ads due to network latency.

The Bidding and Auction Services proposal outlines a way to allow Protected
Audience computation to take place on cloud servers in a trusted execution
environment, rather than running locally on a user's device. Moving computations
to cloud in a [Trusted Execution Environment (TEE)][29] have the following
benefits:

  * Scalable auctions.
    * A scalable ad auction may include several buyers and sellers and that can
      demand more compute resources and network bandwidth.        

  * System health of the user's device.
    * Ensure better system health of user's device by freeing up computational
      cycles and network bandwidth.

  * Better latency of ad auctions.
    * Server to server communication on the cloud is faster than multiple
      device to server calls. 
    * Adtech code can execute faster on servers with higher computing power
      compared to a device.

  * Servers have better processing power.
    * Adtechs can run more compute intensive workloads on a server compared to
      a device to unblock better utility.

  * [Trusted Execution Environment][29] can protect confidentiality of adtech
    code and signals.

There are other ideas, similar to Protected Audience, that propose server-side
auction and bidding, such as [Microsoft Edge's PARAKEET][8] proposal.

This document focuses on the high level design and API for Protected Audience's
Bidding and Auction services. Based on adtech feedback, further changes may be
incorporated in the design.

### Chrome and Android Announcement

Chrome and Android announced to integrate with Bidding and Auction services.
See [blog][27] for more details.

## Background

Read the [Protected Audience services overview][6] to learn about the
environment, trust model, server attestation, request-response encryption, and
other details.

Each Protected Audience service is hosted in a virtual machine (VM) within a
secure, hardware-based [Trusted Execution Environment][29]. Adtech platforms
operate and deploy Protected Audience services on a public cloud. Adtechs may
choose the cloud platform from one of the options that are planned. As cloud
customers, adtechs are the owners and only tenants of such VM instances.

## Supported Public Cloud Platforms

Bidding and Auction services will be available within the [Trusted Execution
Environment][29](TEE) on AWS and GCP in 2023. More cloud platforms may be
supported eventually.

### AWS Support
Bidding and Auction services will run in [Nitro Enclaves][30] on AWS. Refer
[here][52] for more details.

### GCP Support
Bidding and Auction services will run in [Confidential Space][31]
([Confidential Computing][32]) on GCP. Refer [here][53] for more details.

_Note: SSP and DSP can operate services on different cloud platforms that are
supported._

Bidding and Auction services will not support a “Bring Your Own Server” model,
similar to what is made available to Protected Audience’s Key/Value server.
**Bidding and Auction services can only be deployed within approved TEE cloud
environments.** This is valid for Alpha, Beta, Scale testing programs and
through 3PCD.

## Types of Auctions

Bidding and Auction services plan to support single-seller and all types of
multi-seller auctions including [Component Auctions][25]. Refer to the
[Multi seller auctions][55] explainer for more details.

## Related Material

### Bidding and Auction services documents

All documents related to Bidding and Auction services are available [here][57].

  * [Payload optimization][51]
    This is a recommended read for buyers / DSPs.

  * [Multi seller auctions][55]
    This is a recommended read for sellers / SSPs.

  * [AWS deployment guide][52]
    This is a recommended read for adtechs who would opt for AWS.

  * [GCP deployment guide][53]
    This is a recommended read for adtechs who would opt for GCP.

  * [System design][56]
    This provides detailed design of Bidding and Auction services.    

### Server productionisation documents

All documents related to server productionisation are available [here][58] 

### Browser - Bidding and Auction services integration 

Refer to the [browser API][54] for Bidding and Auction services integration.

### Android - Bidding and Auction services Integration 

This space will be updated soon.

## Open Source Repository

Open sourced Bidding and Auction [service code](#service-code-and-framework),
binaries, cloud deployment [configurations](#service-configuration) will be
available in [Github repo][59].

Adtechs will have to deploy the binaries and configurations to a supported
public cloud platform.

## Timeline and roadmap

Following are the timelines for Adtechs interested in testing Bidding and
Auction services.

### Open Sourcing

Bidding and Auction services will be open sourced to [Github repo][59] by 
the end of June 2023. Beyond that, there will be releases every week or a 
continuous sync to Github will be setup.

### Alpha Testing

Alpha Testing includes running services on non-production user opt-in traffic.
Alpha testing will be available starting July 2023. Alpha program is a **rolling
window**; when an adtech onboards, we will support the adtech with initial
integration. During Alpha, the MVP of Bidding and Auction services will be 
available. 

On a high level, there will be support for the following:
  * Bidding and Auction services running in [Trusted Execution Environment][29].
  * Bidding and Auction services and Key Management Systems integration.
  * Privacy and security protections:
    * Encryption of request / response payload between client and server and
      TEE based servers.
    * Padding of request / response payload between client and server.
  * [Adtech code blob fetch][64] from adtech provided endpoints.
  * Javascript and WASM support for [Adtech code execution in a sandbox][61].
  * AWS support for Bidding and Auction services.
  * GCP support for Bidding and Auction services.
  * Multi cloud regional support.
  * Support for [Payload optimization][51].
  * Generation of event level reporting URLs and registered beacons for
    Fenced Frame reporting in Bidding and Auction services. Refer here for
    more details.
  * Bidding and Auction services supporting [Chrome][54] and Android APIs.
  * Component Ads.
  * Production binary build of servers will be used for Alpha testing. 
  * For debugging, debug binary build of servers will be available that can
    provide access to TEE server logs of different verbosity level. For GCP,
    these logs will be exported to [Cloud Logging][65].

### Beta Testing

Beta testing includes running services on limited stable, production traffic.
Beta testing will be available starting October 2023.

During Beta, there will be support for the following additional features:
  * [Adtech code blob fetch][64] from Cloud Storage buckets.
  * Data version header.
  * Multi seller auctions for web.
  * [Privacy safe debugging][60] and [monitoring support][62] for
    productionisation of servers. Refer [here][58] for up-to-date information.

### Scale Testing

Available for full stable, production Scale testing starting February 2024. At
that point, there will be **General Availability (GA)** of all features.

At GA, there will be support for the following additional features:
  * Multi seller auctions for Android / app.
  * [Priority Vector][44] : This can help filter Interest Groups and reduce unnecessary 
    executions in Bidding service.
  * Support for bid currency.
  * K-Anonymity Integration.
  * Productionisation of servers. Refer [here][58] for up-to-date information.

Note: 
  * [Chaffing][63], anti-abuse mitigations will be available by 3PCD.
  * There may be additional features supported in Bidding and Auction services
    beyond GA.

## Onboarding and alpha testing guide

Following is the guide for onboarding to Bidding and Auction services and
participating in Alpha testing.
 
### Guidance to sellers / SSPs:
  * Refer to [Spec for SSP][88] section.
  * Develop [ScoreAd][67]() for Protected Audience auction.
  * Develop [ReportResult][75]() for event level reporting.
  * Setup [seller's key/value server][68].
  * [Chrome browser][54] will support a flag. Users' browsers that enable the
    flag can be targeted. 
  * Add support such that seller's code on publisher web page calls
    [browser API][54] to fetch encrypted [ProtectedAudienceInput][66]. Then
    includes encrypted ProtectedAudienceInput in the request to seller's ad
    server.
  * Add support in [seller's ad server][20] to send [SelectAd][35] request to
    Bidding and Auction services for Protected Audience auctions.
  * Deploy [SellerFrontEnd][21] and [Auction][23] server instances to your
    preferred cloud platform that is supported.
  * Update parameter values in [SellerFrontEnd][71] and [Auction][72] server
    configurations.
  * Set up experiments for ad auctions and target user opt-in traffic. Include
    one or more partner buyers in the same experiment.
  * [Enroll with the Coordinator][85]. During Alpha, Google Privacy Sandbox
    Engineers will act as Coordinators and operate the Key Management systems.

### Guidance to buyers / DSPs:
  * Refer to [Spec for DSP][89] section.
  * Develop [GenerateBid][69]() for bidding.
  * Develop [ReportWin][76]() for event level reporting.
  * Setup [buyer's key/value server][70].
  * [Optimise payload][51]. 
  * Deploy [BuyerFrontEnd][22] and [Bidding][42] server instances to your
    preferred cloud platform that is supported.
  * Update parameters in [BuyerFrontEnd][73] and [Bidding][74] server
    configurations.
  * [Enroll with the Coordinator][85]. During Alpha, Google Privacy Sandbox
    Engineers will act as Coordinators and operate the Key Management systems.
  * Reach out to partner SSPs to include in experiments for Protected Audience
    auctions.
    * _Note: Buyers can also independently start integrating and testing the
    bidding flow before they are included in a seller supported ad auction
    experiment._

## Specifications for adtechs

### Near drop-in replacement

Bidding and Auction services integrate into [Protected Audience API for browsers][28] and 
can scale as a near drop-in replacement for adtechs who already adopted Protected Audience
API and are interested in exploring a server side solution.

  * Interest Groups (Custom Audience) creation and management can stay the same.
    * For [Payload Optimization][51], some additional fields will be supported
      in Interest Group.

  * Key-value services can stay nearly the same.
    * Seller's key-value service can stay the same.
    * Buyer's key-value service instances can stay the same; however to support
      [Payload optimization][51], `trusted_bidding_signals` may need to include
      additional data.
      
  *  Code developed by adtechs following the guidance in [Protected Audience API for browsers][28]
     will mostly work with Bidding and Auction services. **The function signatures can stay exactly the same.**
    * Seller's code for [ScoreAd][67]() can stay the same.
    * Seller's code for [ReportResult][75]() can stay the same.
    * Buyer's code for [GenerateBid][69]() would mostly work. However, certain 
      updates will be required for [Payload Optimization][51].
    * Buyer's code for [ReportWin][76]() can stay the same.

### Spec for SSP

#### ScoreAd()

The [Auction service][23] exposes an API endpoint ScoreAds. The [SellerFrontEnd service][21] sends a
ScoreAdsRequest to the Auction service for running an auction. ScoreAdsRequest includes bids from each buyer
and other required signals. The code for auction, i.e. ScoreAd() is prefetched from Cloud Storage and cached in
Auction service. After all ads are scored, the Auction service picks the highest scored ad candidate
and returns the score and other related data for the winning ad in ScoreAdsResponse.

_Note: If you are developing scoreAd() following the web platform's [Chrome Protected Audience explainer][37], 
that also should work as-is for execution in Bidding and Auction services._

Adtech's scoreAd function signature is as follows.

```
scoreAd(adMetadata, bid, auctionConfig, trustedScoringSignals, bid_metadata) {
  ...
  return {desirability: desirabilityScoreForThisAd,
              allowComponentAuction: true_or_false};
}
```

##### Arguments

* `adMetadata`: Arbitrary metadata provided by the buyer.
* `bid`: A numerical bid value.
* `auctionConfig`: This would include `sellerSignals` (auctionConfig.sellerSignals) and
  `auctionSignals` (auctionConfig.auctionSignals).
* `trustedScoringSignals`: trustedScoringSignals fetched from Seller Key/Value service
    * Note: Only the signals required for scoring the ad / bid is passed to scoreAd().
* `bid_metadata`: This refers to an object created in the Auction service based on the render_url
   of the bid and other information known to the Auction service. 

```
{ 'topWindowHostname': 'www.example-publisher.com',
  'interestGroupOwner': 'https://www.example-dsp.com',
  'renderUrl': 'https://cdn.com/render_url_of_bid',
  'adComponents': ['https://cdn.com/ad_component_of_bid',
                   'https://cdn.com/next_ad_component_of_bid',
                   ...],
 'dataVersion': 1, /* Data-Version value from the trusted scoring signals server's response */
}
```

#### Seller BYOS Key / Value Service

The [SellerFrontEnd service][21] looks up `trustedScoringSignals` from seller's Key/Value service. The 
base url (domain) for Key/Value service is configured in [SellerFrontEnd service][21] so that the
connection can be prewarmed. All `render_urls` corresponding to all bids from buyers participating
in an auction are encoded, then batched and looked up in a single request. The lookup url and
response are in the same format as described in [Chrome Protected Audience explainer][40].

The lookup url is in the following format:

```
<base_url>?renderUrls=<url_1>..<url_n>&adComponentRenderUrls=<url1>,..<urln>

Where <base_url>, <url1>... are substituted
```

The response is in the following format:

```
{ 'renderUrls': {
      'https://cdn.com/render_url_of_some_bid': arbitrary_json,
      'https://cdn.com/render_url_of_some_other_bid': arbitrary_json,
      ...},
  'adComponentRenderUrls': {
      'https://cdn.com/ad_component_of_a_bid': arbitrary_json,
      'https://cdn.com/another_ad_component_of_a_bid': arbitrary_json,
      ...}
}
```

The `trustedScoringSignals` passed to scoreAd() is as follows:

```
{ 
  'renderUrl': {'https://cdn.com/render_url_of_bidder': arbitrary_value_from_signals},
  'adComponentRenderUrls': {
      'https://cdn.com/ad_component_of_a_bid': arbitrary_value_from_signals,
      'https://cdn.com/another_ad_component_of_a_bid': arbitrary_value_from_signals,
      ...}
}
```

#### ReportResult()

Event level win reporting would work with Bidding and Auction services and function signatures
can be the same as described in [Chrome Protected Audience explainer][43]. The reporting urls 
for the seller and registered ad beacons (for Fenced Frame reporting) would be generated in Auction 
service and returned to the client in encrypted [AuctionResult][84]. The client will ping the 
seller's reporting endpoint using the reporting url.

_Note: A detailed design covering reporting will be published separately._

```
reportResult(auctionConfig, reporting_metadata) {
  ...
  registerAdBeacon({"click", clickUrl,"view", viewUrl});
  sendReportTo(reportResultUrl);
  return signalsForWinner;
}

```

##### Arguments

* `auctionConfig`: This would include `sellerSignals` (auctionConfig.sellerSignals) and
  `auctionSignals` (auctionConfig.auctionSignals).
* `reporting_metadata`: This refers to an object created in the Auction service based
   on the information known to the Auction service.

#### Seller service configurations

Server configurations are based on [Terraform][16] and will be open sourced in [Github repo][59]
for cloud deployment. 

The configurations will include environment variables and parameters that may vary per seller.
These can be set by the seller in the configuration before deployment. The configurations also 
include urls that can be ingested when the service starts up for prewarming the connections.  

Following are some examples of data configured in service configurations. Comprehensive examples
will be published along with the deployment configs in [Github repo][59].

##### SellerFrontEnd service configurations

* _Seller Key/Value service endpoint (scoring_signals_url)_: This endpoint is configured in SellerFrontEnd
  service configuration and ingested at service startup to prewarm connections to seller's Key/Value service.

* _BuyerFrontEnd endpoint_: The domain address of BuyerFrontEnd services operated by Buyers that this Seller
  has partnered with. This is ingested at service startup to prewarm connection.

* _Auction service endpoint_:  The domain address of Auction service. This is ingested at service startup to
  prewarm connection.

* _Seller origin_: The origin of the seller.
     * _Note: The seller origin information is also passed by the seller's ad Service in SelectAd request and
       SellerFrontEnd validates that with the origin information configured._
       
* _Map of {InterestGroupOwner, BuyerFrontEnd endpoint}_: Map of InterestGroupOwner to BuyerFrontEnd domain
  address.
  
* _Global timeout for Buyer_: This information can be used to set a timeout on each buyer; however, will be
  overridden by the `buyer_timeout_ms` passed by the Seller Ad service to SellerFrontEnd in SelectAd request.

* _prefetch_bidding_signals_buyers_list_: The list of Buyers (InterestGroupOwners) that have opted-in for
  the [PrefetchBiddingSignals][36] lookup.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

##### Auction service configurations

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where Seller code is hot reloaded by the
  Auction service. 
  
* Private Key Hosting service and Public Key Hosting service endpoints in [Key Management System][10].

### Spec for DSP

#### GenerateBid()

The [Bidding service][42] exposes an API endpoint GenerateBids. The [BuyerFrontEnd service][22] sends
GenerateBidsRequest to the Bidding service, that includes required input for bidding. The code
for bidding, i.e. GenerateBid() is prefetched from Cloud Storage and cached in Bidding service.
After processing the request, the Bidding service returns the GenerateBidsResponse which includes 
bids that correspond to each ad, i.e. [AdWithBid][49].

_Note: If you are developing generateBid() following the web platform's [Chrome Protected Audience explainer][41], 
that also should work for execution in Bidding and Auction services._

Adtech's generateBid function signature is as follows.

```
generateBid(interestGroup, auctionSignals, perBuyerSignals, trustedBiddingSignals,  deviceSignals) {
  ...
  return {'ad': adObject,
          'bid': bidValue,
          'render': renderUrl,
          'allowComponentAuction': false};
 } 
```

##### Arguments

* `interestGroup`: The InterestGroup (Custom Audience) object. Refer InterestGroup data structure to
  understand what is sent in this object from the client.
    * _Note: To reduce payload over the network and further optimize latency, our goal is to minimize
      the information sent in this object. We will work with Adtechs for the long term to reduce the
      amount of information sent in this object and try to find a solution to fetch those on the server
      side._

* `auctionSignals`: Contextual signal that is passed from Seller Ad service to SellerFrontEnd in SelectAd
  request.

* `perBuyerSignals`: Contextual signal generated by the Buyer during Real Time Bidding that is passed
  from Seller Ad service to SellerFrontEnd in SelectAd request. 

* `trustedBiddingSignals`: Real time signals fetched by BuyerFrontEnd service from Buyer's Key/Value
  service. 
  * _Note: Only the trustedBiddingSignals required for generating bid(s) for the interestGroup are passed
    to generateBid()_.

* `deviceSignals`: This refers to "browserSignals" or "androidSignals",  built by the client (Browser,
  Android). This includes Frequency Cap (statistics related to previous win of ads) for the user's device.

#### ReportWin()

Event level win reporting would work with Bidding and Auction services and function signatures
can be the same as described in [Chrome Protected Audience explainer][43]. ReportWin()
will be executed in Auction service, reporting url for the buyer and registered ad beacons 
(for Fenced Frame reporting) would be generated in Auction service and returned to the client
in encrypted [AuctionResult][84]. The client will ping the buyer's reporting endpoint
using the reporting url.

_Note: A detailed design covering reporting will be published separately._

```
reportWin(auctionSignals, perBuyerSignals, signalsForWinner, reporting_metadata) {
  ...
  registerAdBeacon({"click", clickUrl,"view", viewUrl});
  sendReportTo(reportWinUrl);
  return;
}

```

##### Arguments

* `auctionSignals`: Contextual signal generated by the seller.
* `perBuyerSignals`: Contextual signal generated by the buyer.
* `signalsForWinner`: Object returned by seller's ReportResult().
* `reporting_metadata`: This refers to an object created in the Auction service
   based on the information known to the Auction service.   

#### Buyer BYOS Key/Value Service

The [BuyerFrontEnd service][22] looks up biddingSignals from Buyer's BYOS Key / Value service. The base url
(domain) for Key/Value service is configured in BuyerFrontEnd service so that the connection can be
prewarmed. All lookup keys are batched together in a single lookup request. The lookup url and response
are in the same format as described in [Chrome Protected Audience explainer][40].

The lookup url is the following format:

```
<base_url>/getvalues?hostname=<publisher.com>&experimentGroupId=<kv_exp_id>&keys=<key_1>,..
<key_n>

Where <base_url>, <publisher.com>, <kv_exp_id>, key_1>,...<key_n> are substituted

Note: If keys are the same as InterestGroups names, then those are not looked up more than once.
```

The response is in the following format:

```
{ 'keys': {
      'key1': arbitrary_json,
      'key2': arbitrary_json,
      ...},
  'perInterestGroupData': {
      'name1': {
      },
      ...
  }
}
```

_Note: The trustedBiddingSignals passed to generateBid() for an Interest Group (Custom Audience) is
the value corresponding to each lookup key in the Interest Group but not the entire response.
Following is an example, if key1 is the lookup key in an InterestGroup, then the following is passed
to generateBid() in trustedBiddingSignals._

```
  'key1': arbitrary_json
```

##### Metadata for Filtering in Buyer Key/Value Service

To support filtering in Buyer BYOS Key / Value service, the following metadata will be forwarded in the
HTTP request header of the lookup request.

###### Client (e.g. Browser)

* Geo Information
* User Agent 
* Language

###### Service (BuyerFrontEnd)

 IP address : This may be ingested by buyer's Key/Value service to monitor
 requests from Bidding and Auction services.
 
###### Metadata Forwarding 
 
 * The metadata will be sent from device to seller's ad service in HTTP request header.
 * Seller's ad service will [forward](#metadata-forwarding-by-seller-ad-service) that to SellerFrontEnd service.
 * SellerFrontEnd will [add metadata to gRPC][47] request sent to BuyerFrontEnd service.
 * BuyerFrontEnd will [forward][46] the metadata in the request header to buyer's Key/Value service.

#### Buyer service configurations

Server configurations are based on [Terraform][16] and will be open sourced in [Github repo][59]
for cloud deployment. 

The configurations will include environment variables and parameters that may vary per buyer.
These can be set by the buyer in the configuration before deployment. The configurations also 
include urls that can be ingested when the service starts up for prewarming the connections.  

Following are some examples of data configured in service configurations. Comprehensive examples
will be published along with the deployment configs in [Github repo][59].

##### BuyerFrontEnd service configurations

* _Buyer Key/Value service endpoint (bidding_signals_url)_: This endpoint is configured in SellerFrontEnd
  service configuration and ingested at service startup to prewarm connections to Seller Key/Value service.

* _Bidding service endpoint_: The domain address of Bidding service. This is ingested at service startup to
  prewarm connection.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

##### Bidding service configurations

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where Buyer code is hot reloaded by the
  Bidding service.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

## High level design

Following is the architecture of Bidding and Auction Services.

![Architecture diagram.](images/unified-contextual-remarketing-bidding-auction-services.png)

_Note:_ 
* _All arrows in the diagram are bidirectional, implying request and response._
* _Based on feedback from Adtechs, we may incorporate further optimization in
  the flow while ensuring it is privacy safe._

Seller's code (in the Publisher web page or app) sends one [unified request][83]
for contextual and Protected Audience auctions to seller's ad service. 

Then the seller’s ad service makes two *sequential requests*.
  * [Existing flow] The seller may send real-time bidding (RTB) requests to
    a select partner buyers for contextual bids, then conducts the contextual
    auction.
  * [Server side Protected Audience flow] The seller sends a [SelectAd][35]
    request to SellerFrontEnd service to start the Protected Audience auction if
    seller determines there is incremental value in conducting the auction. The
    request payload includes encrypted [ProtectedAudienceInput][9], AuctionConfig
    and other required information. The encrypted [ProtectedAudienceInput][9] can 
    only be decrypted by an attested service running in [Trusted Execution Environment][29],
    in this case the [SellerFrontEnd service][21].

_Unified Contextual and Protected Audience Auction Flow_ is important to
optimize e2e auction latency.

### Unified request

A seller’s single request for the contextual and server-side Protected Audience
auction, from their code on a publisher site or app. The request includes
contextual payload and encrypted [ProtectedAudienceInput][9]. The seller's code
on the publisher site or app calls the client API to get the encrypted
[ProtectedAudienceInput][9] before sending the unified request.

### Components

#### Browser API for Bidding and Auction services

Refer to [browser API and integration][54] design.

#### Sell-side platform (SSP) system

The following are the Protected Audience services that will be operated by an SSP,
also referred to as a Seller. 

##### Seller Ad service

With the *Unified Contextual and Protected Audience Auction* flow, the seller's
ad service will receive one request from the client. The request would include
contextual request payload and encrypted [ProtectedAudienceInput][9] from the
client.

The encrypted [ProtectedAudienceInput][9] includes Interest Group (Custom
Audience) information on the user's device. The size of [ProtectedAudienceInput][9]
is required to be small; and there may be a per-buyer size limit set by the
client or the seller. Refer to [Payload optimization][51] explainer for the
guidance around optimizing [ProtectedAudienceInput][9] payload size.

Refer to more details [here][77].

##### SellerFrontEnd service

The front-end service of the system that runs in the [Trusted Execution Environment][29]
on a supported cloud platform. The service receives requests from [Seller Ad service][20]
to initiate Protected Audience auction flow. Then the service orchestrates
requests (in parallel) to Buyers / DSPs participating in the auction for bidding. 

This service also fetches real-time scoring signals required for the auction and
calls [Auction service][23] for Protected Audience auction.

Refer to more details [here][78].

##### Auction service

The Auction service runs in the [Trusted Execution Environment][29] on a supported
cloud platform. This service responds to requests from the [SellerFrontEnd service][21]
and doesn't have access to arbitrary untrusted endpoints. 

The Auction service prefetches [code blobs](#adtech-code) owned by seller from 
Cloud Storage (or an endpoint provided by the seller). The code is prefetched
at service startup, periodically thereafter and cached. More than one code
version can be supported to facilitate experimentation by adtechs.

SSP's code for scoring ads can be written in JavaScript and / or WebAssembly (WASM). 
The code runs in a custom V8 sandbox within the TEE that has tighter security
restrictions; that can not log information or has no disk or network access in 
production mode. For a ScoreAds request from SellerFrontEnd, SSP's scoring
code is executed per ad within a separate V8 worker thread but all execution can 
happen in parallel. Between two executions in the service, there is no state saved.

_Note: The hosting environment protects the confidentiality of the Seller's code,
if the execution happens only in the cloud._

Refer to more details [here][79].

##### Seller's key/value service

A seller's Key/Value service is a critical dependency for the auction system.
The Key/Value service receives requests from the [SellerFrontEnd service][21]. 
The service returns real-time seller data required for auction that corresponds
to lookup keys available in buyers' bids (such as `ad_render_urls`
or `ad_component_render_urls`).

_Note: The Seller Key/Value system may be BYOS Key/Value Service or Trusted
Key/Value Service depending on the timeline._

#### Demand-side platform (DSP) system

This section describes Protected Audience services that will be operated by a
DSP, also called a buyer. 

##### BuyerFrontEnd service

The front-end service of the system that runs in the [Trusted Execution Environment][29]
on a supported cloud platform. This service receives requests to generate bids
from a [SellerFrontEnd service][21]. This service fetches real-time bidding
signals that are required for bidding and calls Bidding service.

Refer to more details [here][80].

##### Bidding service

The Bidding service runs in the [Trusted Execution Environment][29] on a supported
cloud platform. This service responds to requests from [BuyerFrontEnd service][22]
and doesn't have access to arbitrary untrusted endpoints. 

The Bidding service prefetches [code blobs](#adtech-code) owned by the buyer from
Cloud Storage (or an endpoint provided by the seller). The code is prefetched at
service startup, periodically thereafter and cached. More than one code version
can be supported to facilitate experimentation by adtechs.

Buyer's code for generating bids can be written in JavaScript and / or WebAssembly (WASM). 
The code runs in a custom V8 sandbox within the TEE that has tighter security restrictions; 
that can not log information or has no disk or network access in production. For a 
GenerateBids request from BuyerFrontEnd, buyer's code is executed per Interest Group
(Custom Audience) within a separate V8 worker thread but all execution can happen in
parallel. Between two executions in the service, there is no state saved.

_Note: This environment protects the confidentiality of a buyer's code, if the
execution happens only in the cloud._

Refer to more details [here][81].

##### Buyer’s key/value Service

A buyer's Key/Value service is a critical dependency for the bidding system. The 
Key/Value service receives requests from the [BuyerFrontEnd service][22]. The
service returns real-time buyer data required for bidding, corresponding to
lookup keys.

_Note: The buyer’s key/value system may be BYOS Key/Value Service or Trusted Key/Value 
service depending on timeline._

### Flow

* Clients (browser, Android) builds encrypted [ProtectedAudienceInput][9].
    * Client prefetch a set of public keys from the [Key Management Systems][10]
      in the non request path every 7 days. The public keys are used for encrypting
      [ProtectedAudienceInput][9].

    * This encrypted data includes Interest Group (Custom Audience) information
      for different buyers, i.e. [BuyerInput][82]. On the client, the BuyerInput(s)
      are compressed, then ProtectedAudienceInput is encrypted and padded. The
      padding may be done such that the size of padded ProtectedAudienceInput
      ciphertext falls in one of the 7 different size buckets.
      * From privacy perspective, 7 size buckets is chosen because this would
        leak less than 3 bits of information in the worst case.

* Seller's code in publisher page on the browser sends to seller's ad service.
    * Web: Seller's code in publisher webpage on the browser sends HTTP request to (untrusted)
      seller's ad service. 
        * __[Existing request, but excluding 3P Cookie]__ Contextual payload. 
        * Seller's code in publisher webpage asks browser for encrypted [ProtectedAudienceInput][9]
          to be included in request.
        * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) is
          added to HTTP request header by the client.
          
    * Android: Seller's code in publisher SDK in Android sends HTTP request to (untrusted)
      seller's ad service.
        * __[Existing request]__ Contextual payload.
        * Seller's code in publisher SDK asks Android for encrypted [ProtectedAudienceInput][9]
          to be included in request.
        * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) may
          be added to HTTP request header by the client.

* Seller Ad Service makes two sequential requests.
    * __[Existing flow]__ May send Real Time Bidding (RTB) requests to partner buyers
      for contextual bids and then conduct a contextual auction to select a
      contextual ad winner.

    * Sends SelectAd request to SellerFrontEnd service if there is incremental value
      in conducting the Protected Audience auction. The request payload includes 
      encrypted [ProtectedAudienceInput][9], AuctionConfig and other
      required information. 
      * Encrypted [ProtectedAudienceInput][9] should be Base64 encoded string of bytes
        if SelectAd request is sent as 
      * AuctionConfig includes contextual signals like seller_signals, auction_signals;
        per buyer signals / configuration and other data. Contextual ad winner
        may be part of seller_signals.
      * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) is [forwarded](#metadata-forwarding-by-seller-ad-service) to SellerFrontEnd service.
      
* Protected Audience bidding and auction kicks off in Bidding and Auction Services.
    * The SellerFrontEnd service decrypts encrypted [ProtectedAudienceInput][9] using
      decryption keys prefetched from [Key Management Systems][10].

    * The SellerFrontEnd service orchestrates GetBids requests to participating buyers’ 
      BuyerFrontEnd services in parallel.
      * Buyers in `buyer_list` (in AuctionConfig) as passed by the seller and that
        have non empty [BuyerInput][82] in [ProtectedAudienceInput][9] receive
        GetBids request.

    * Within each buyer system:
        * The BuyerFrontEnd service decrypts GetBidsRequest using decryption keys
          prefetched from [Key Management System][10].
        * The BuyerFrontEnd services fetch real-time data (`trustedBiddingSignals`) from
          the buyer’s Key/Value service required for generating bids.
        * The BuyerFrontEnd service sends a GenerateBids request to the Bidding service.
        * The Bidding service returns ad candidates with bid(s) for each `InterestGroup`.
          * The Bidding service deserializes and splits `trustedBiddingSignals` such that
            GenerateBid() execution for an `InterestGroup` can only ingest
            `trustedBiddingSignals` for the interest group / bidding signal key.
        * The BuyerFrontEnd returns all bid(s) ([AdWithBid][49]) to SellerFrontEnd.

    * Once SellerFrontEnd has received bids from all buyers, it requests real-time data
      (`trustedScoringSignals`) for all `ad_render_urls` and `ad_component_render_urls`
      (corresponding to ad with bids) from the seller’s Key/Value service. These
      signals are required to score the ads during Protected Audience auction.

    * SellerFrontEnd sends a ScoreAdsRequest to the Auction service to score ads
      and selects a winner. 
        * Auction service deserializes and splits scoring signal such that ScoreAd()
          execution for an ad can only ingest `trustedScoringSignals` for the ad.

    * The Auction service selects the winning ad, generates reporting urls and
      returns ScoreAdsResponse to SellerFrontEnd service. 

    * SellerFrontEnd returns winning ad, other metadata and reporting urls as an 
      encrypted [AuctionResult][84] back to seller's ad service.

    * There are a few cases when a fake, padded, encrypted [AuctionResult][84] will 
      be returned to the client. For such cases, contextual ad winner should be
      rendered on the client.

      * The `is_chaff` field in AuctionResult will be set and that would indicate 
        to the client that this is a fake Protected Audience auction result. Following
        are the possible scenarios when this would be set.
        1. Protected Audience auction returns no winner; this may be a possible
           scenario when contextual ad winner (part of `seller_signals`)
           participate in Protected Audience auction to filter all bids. Therefore,
           the contextual ad wins and should be rendered on the client.
        2. When none of the participating buyers in the auction return any bid.

      * The `error` field in AuctionResult is for the client to ingest and handle.
        Following are the possible scenarios when this would be set.
        1. The required fields in [ProtectedAudienceInput][9] doesn't pass validation.
        2. There is failure in de-compression of [BuyerInput][82]. 
        3. The private keys (decryption keys) are compromised and revoked. In this  
           scenario, [ProtectedAudienceInput][9] ciphertext can be decrypted with a
           compromised key but the request won't be processed further. The error field 
           would be set in AuctionResult to indicate to the client this corresponding 
           public key with the same version shouldn't be used again for encryption. 
           The AuctionResult will be encrypted and returned in the response.

    * SellerFrontEnd will propagate downstream service errors back to seller's ad service
      and will also return gRPC / HTTP errors when required fields in SelectAdRequest.AuctionConfig
      are missing. 
     
* Seller's ad service returns the encrypted [AuctionResult][84] back to the client.
    * Contextual ad and / or encrypted ProtectedAudienceInput will be sent back 
      to the client. In case the contextual ad wins, a fake encrypted [AuctionResult][84]
      will be sent in response.

* Seller code in publisher web page or app receives the response from seller's
  ad service and passes the encrypted [AuctionResult][84] (`auction_result_ciphertext`)
  to the client.

* Only the client (browser, Android) would be able to decrypt [AuctionResult][84]
  ciphertext. 

* Ad is rendered on the device.

### Client <> Server and Server <> Server Communication

#### Client <> Seller Ad Service Communication

[Client to Seller Ad Service communication][19] for the *Unified Contextual and Protected Audience Auction request*
would be HTTPS. The [ProtectedAudienceInput][9] included in the unified request will be encrypted on the client 
using a protocol called [Oblivious HTTP][50] that is based on bidirectional [Hybrid Public Key Encryption][48](HPKE). 
The Protected Audience response, i.e. [AuctionResult][84] will also be encrypted in SellerFrontEnd using 
[Oblivious HTTP][50].

The seller's ad service will not be able to decrypt or have access to [ProtectedAudienceInput][9] or 
[AuctionResult][84] in plaintext.

#### Seller Ad Service <> SellerFrontEnd Communication

[Seller Ad service][20] can send gRPC or HTTPS to SellerFrontend service. There would be an [Envoy Proxy][45]
service hosted before [SellerFrontEnd][21]. If HTTPS is sent to SellerFrontEnd, Envoy Proxy would translate
that to gRPC.  

The communication between seller's ad service and SellerFrontEnd service would be over TLS / SSL that
provide communications security by encrypting data sent over the untrusted network to an authenticated peer.

##### Metadata Forwarding by Seller Ad Service

To forward metadata received in request header from the client to SellerFrontEnd, seller's ad service can do
one of the following:
  * If the seller's ad service sends SelectAd as HTTPS to SFE, the header can be [forwarded][46].
  * If the seller's ad service sends SelectAd as gRPC, metadata needs to be [created and added][47].

#### Communication between Bidding and Auction Services

All communication between services running in [Trusted Execution Environment][29] is over TLS / SSL
and the request and response is end-to-end encrypted using bidirectional [Hybrid Public Key Encryption][48](HPKE). 
The TLS / SSL session terminates at the load balancer in-front of a service, therefore the data over the wire from
the load balancer to service needs to be protected; hence the request-response is end-to-end encrypted using
bidirectional HPKE.

### Payload Compression

Most request/response payload sent over the wire should be compressed. 
* The [BuyerInput(s)][82] in [ProtectedAudienceInput][9] will be compressed on the client using `gzip`. 
  The payload needs to be compressed first and then encrypted. In SellerFrontEnd, [ProtectedAudienceInput][9] 
  will be decrypted first and then decompressed.

* Seller Ad Service can compress [SelectAd][35] request payload when calling SellerFrontEnd service to save
  network bandwidth cost and reduce latency; `gzip` is accepted by SellerFrontEnd service. The 
  [AuctionResult][84] will be compressed using `gzip`, then encrypted and returned in SelectAdResponse.

* Request to BuyerFrontEnd service from SellerFrontEnd will be compressed first using `gzip` and then encrypted.
  In BuyerFrontEnd service, the request will have to be decrypted first and then decompressed.
    * _Note: This would be similar for communication between BuyerFrontEnd <> Bidding services and
      SellerFrontEnd <> Auction services._

* For Key/Value Server HTTP lookup, the [Accept-Encoding HTTP request header][38] would be set to specify the
  correct compression algorithm, i.e. `gzip`. The Key/Value server may set the [Content-Encoding Representation Header][39]
  to specify the content encoding (i.e. `gzip`) used to compress the response payload. 
    * _Note:_
         * It is recommended to compress Key/Value server response to optimize Key/Value lookup latency and reduce
           network bandwidth cost for adtechs.
         * The request payload to Key/Value service need not be compressed given the size is expected to be small.
         * The request-response payload between SellerFrontEnd / BuyerFrontEnd and Seller / Buyer Key/Value services
           do not require additional encryption using [HPKE][48]. However, the communication between these services
           is over TLS that provide communications security by encrypting data sent over the untrusted network to
           an authenticated peer. 

### Service Code and Framework

Bidding and Auction services are developed in C++. The service configurations required for cloud deployment
are based on [Terraform][16].

The service framework is based on gRPC. [gRPC][12] is an open source, high performance RPC framework built
on top of HTTP2 that is used to build scalable and fast APIs. gRPC uses [protocol buffers][13] as the
[interface description language][14] and underlying message interchange format.

Bidding and Auction services code will be open sourced to [Privacy Sandbox github][17].

### Adtech Code 

Adtech code for generateBid(), scoreAd(), ReportResult(), ReportWin() can follow
the same signature as described in the [Protected Audience API for the browser][28]. 

_Note:_
  * Code can be Javascript only or WASM only or WASM instantiated with Javascript.
  * If the code is in Javascript, then Javascript context is initialized before every execution.
  * No limit on code blob size.
  * More than one version of code can be supported to facilitate adtech experimentation.
  * Adtech can upload their code to Cloud Storage supported by the Cloud Platform.
  * Code is prefetched by Bidding / Auction services running in [Trusted Execution Environment][29]
    from the Cloud Storage bucket owned by adtech.

Refer to more details [here][86].

### Cloud Deployment

Protected Audience Bidding and Auction services are deployed by adtechs to a [public cloud platform](#supported-public-cloud-platforms) so that they are co-located within a cloud region. 
Servers can be replicated in multiple cloud regions and the availability Service-Level-Objective (SLO)
will be decided by adtechs. 

#### SSP System

There will be a Global Load balancer for managing / routing public traffic to [SellerFrontEnd service][21].
Traffic between SellerFrontEnd and Auction service would be over private VPC network. To save cost,
SellerFrontEnd and Auction server instances will be configured in a [service mesh][87].

#### DSP System

There will be a Global Load balancer for managing / routing public traffic to [BuyerFrontEnd services][22].
Traffic between BuyerFrontEnd and Bidding service would be over private VPC network. To save cost,
BuyerFrontEnd and Bidding server instances will be configured in a [service mesh][87].

### Dependencies

Through techniques such as prefetching and caching, the following dependencies are in the non-critical
path of ad serving.

#### Key Management System

A [Key Management System][10] is required for Protected Audience service attestation and cryptographic key generation. 
Learn more in the [Overview of Protected Audience Services Explainer][6]. The Key Management System will be deployed to
all supported public clouds. Services in the Key Management System will be replicated in multiple cloud
regions. 

All services running in TEE prefetch encryption and decryption keys from Key Management System at service
startup and periodically in the non critical path. All communication between a service in TEE and another
service in TEE is end-to-end encrypted using Hybrid Public Key Encryption and TLS. Refer [here][11] for more
details.

## Service APIs

### Client <> Server Data

Following section describes the data that flows from client (e.g. browser, Android) to Bidding and Auction
Services through [Seller Ad service][20] and the data received by client from Bidding and Auction Services.

_Note: We will update the API code to include additional data for Component Ads and Event level reporting._

#### ProtectedAudienceInput

ProtectedAudienceInput is built and encrypted by client (browser, Android). Then sent to [Seller Ad service][20] in
the [unified request][83]. This includes per [BuyerInput](#buyer-input) and other required data.

```
syntax = "proto3";

// ProtectedAudienceInput is generated and encrypted by the client,
// passed through the untrusted Seller service, and decrypted by the
// SellerFrontEnd service.
// It is the wrapper for all of BuyerInput and other information required
// for the Protected Audience auction.
message ProtectedAudienceInput {
  // Input per buyer.
  // The key in the map corresponds to IGOwner (Interest Group Owner) that
  // is the Buyer / DSP. This  string that can identify a
  // buyer participating in the auction. The value corresponds to compressed
  // BuyerInput data. BuyerInput is ingested by the Buyer for bidding.
  map<string, bytes> buyer_input = 1;

  // Publisher website or app.
  // This is required to construct browser signals for web.
  // It will also be passed via GetBids to buyers for their Buyer KV lookup
  // to fetch trusted bidding signals.
  string publisher_name = 2;

  // A boolean value which indicates if event level debug reporting should be
  // enabled or disabled for this request.
  bool enable_debug_reporting = 3;

  // Globally unique identifier for the client request.
  string generation_id = 4;
}
```

#### BuyerInput

BuyerInput is part of [ProtectedAudienceInput][9]. This includes data for each buyer / DSP.

```
syntax = "proto3";

// A BuyerInput includes data that a buyer (DSP) requires to generate bids.
message BuyerInput {
  // InterestGroup (a.k.a CustomAudience) information passed from the client.
  message InterestGroup {
    // Required.
    // Name or tag of Interest Group (a.k.a Custom Audience).
    string name = 1;

    // Required to fetch real time bidding signals from buyer's key/value
    // server.
    repeated string bidding_signals_keys = 2;

    // Optional.
    // Ids of ad_render_urls generated by the DSP / Buyer and passed to the
    // client. Then client passes this in InterestGroup if available.
    // Note: If the Buyer doesn't generate the ad_render_id, then their
    // GenerateBid() should dynamically generate the url for the bid. The
    // winning ad render url returned back to the client will be validated with
    // the Interest Group information on the client.
    repeated string ad_render_ids = 3;

    // Optional.
    // Ids of ad_component_render_url(s) generated by the DSP / Buyer and passed
    // to the client.
    //
    // Note: If the Buyer doesn't generate the ad_component_render_id, device
    // will not pass ads to Bidding and Auction services to ensure payload size
    // is small. In this case, GenerateBid() should dynamically generate the
    // urls for component ads.The winning ad render url returned back to the
    // client will be validated with the Interest Group information on the
    // client.
    repeated string component_ads = 4;

    // Optional.
    // User bidding signal that may be ingested during bidding.
    // This is a JSON array.
    // NOTE: If this is used by the Buyer for bidding, it is recommended to
    // fetch this server side from Buyer Key / Value server to keep request
    // payload size small.
    string user_bidding_signals = 5;

    // Required for bidding.
    // Contains filtering data, like Frequency Cap.
    oneof DeviceSignals {
      // Information passed by Android.
      AndroidSignals android_signals = 6;

      // Some information that the browser knows about that is required for
      // bidding.
      BrowserSignals browser_signals = 7;
    }
  }
  // The Interest Groups (a.k.a Custom Audiences) owned by the buyer.
  repeated InterestGroup interest_groups = 1;
}
```

#### BrowerSignals
Information about an Interest Group known to the browser. These are required to
generate bid.

```
syntax = "proto3";

// Information about an Interest Group passed by the browser.
message BrowserSignals {
  // Number of times the group was joined in the last 30 days.
  int64 join_count = 1;

  // Number of times the group bid in an auction in the last 30 days.
  int64 bid_count = 2;

  // The most recent join time for this group expressed in seconds
  // before the containing auctionBlob was requested.
  int64 recency = 3;

  // Tuple of time-ad pairs for a previous win for this interest group
  // that occurred in the last 30 days. The time is specified in seconds
  // before the containing auctionBlob was requested.
  string prev_wins = 4;
}
```

#### AndroidSignals
Information passed by Android for Protected Audience auctions. This will be 
updated later.

```
syntax = "proto3";

// Information passed by Android.
message AndroidSignals {}
```

#### AuctionResult

FLEDGE auction result returned from SellerFrontEnd service to the client through the Seller
Ad service. The data is encrypted by SellerFrontEnd service and decrypted by the client. The
Seller Ad service will not be able to decrypt the data. 

In case the contextual ad wins, an AuctionResult will still be returned that includes fake data
and has is_chaff field set to true. Clients should ignore AuctionResult after decryption if
is_chaff is set to true.

```
syntax = "proto3";

// Protected Audience auction result returned from SellerFrontEnd to the client
// through the Seller service. It is encrypted by the SellerFrontEnd, passed
// through the untrusted Seller service and decrypted by the client. Note that
// untrusted Seller service will be unable to determine if there was a
// successful auction result, so the client must check the value of is_chaff.
message AuctionResult {
  // The ad that will be rendered on the end user's device.
  string ad_render_url = 1;

  // Render URLs for ads which are components of the main ad.
  repeated string ad_component_render_urls = 2;

  // Name of the InterestGroup (Custom Audience), the remarketing ad belongs to.
  string interest_group_name = 3;

  // Domain of the Buyer who owns the interest group that includes the ad.
  string interest_group_owner = 4;

  // Score of the ad determined during the auction. Any value that is zero or
  // negative indicates that the ad cannot win the auction. The winner of the
  // auction would be the ad that was given the highest score.
  // The output from ScoreAd() script is desirability that implies score for an
  // ad.
  float score = 5;

  // Bid price corresponding to an ad.
  float bid = 6;
  
  // Boolean to indicate that there is no remarketing winner from the auction.
  // AuctionResult may be ignored by the client (after decryption) if this is
  // set to true.
  bool is_chaff = 7;

  // The reporting urls registered during the execution of reportResult() and
  // reportWin().
  WinReportingUrls win_reporting_urls = 8;

  // Debugging URLs for the Buyer. This information is populated only in case of
  // component auctions.
  DebugReportUrls buyer_debug_report_urls = 9;

  // Debugging URLs for the Seller. This information is populated only in case
  // of component auctions.
  DebugReportUrls seller_debug_report_urls = 10;

  // List of interest group indices that generated bids.
  message InterestGroupIndex {
    // List of indices of interest groups. These indices are derived from the
    // original ProtectedAudienceInput sent from the client.
    repeated int32 index = 1;
  }

  // Map from the buyer participating origin (that participated in the auction)
  // to interest group indices.
  map<string, InterestGroupIndex> bidding_groups = 11;

  // In the event of an error during the SelectAd request, an Error object will
  // be returned as a part of the AuctionResult to indicate what went wrong.
  message Error {
    // Status code.
    int32 code = 1;

    // Message containing the failure reason.
    string message = 2;
  }

  // Error thrown during the SelectAd request. If there is no error and the
  // request completes successfully, this field will be empty.
  Error error = 12;
}
```

### Public APIs

#### SellerFrontEnd Service and API Endpoints

The SellerFrontEnd service exposes an API endpoint (SelectAd). The Seller Ad service would send
a SelectAd RPC or HTTPS request to SellerFrontEnd service. After processing the request,
SellerFrontEnd would return a SelectAdResponse that includes an encrypted AuctionResult. 

The AuctionResult will be encrypted in SellerFrontEnd using [Oblivious HTTP][50] that is based on
bidirectional [HPKE][48].

```
syntax = "proto3";

// SellerFrontEnd service (also known as SFE) operated by SSP / Seller.
service SellerFrontEnd {
  // Selects a winning remarketing ad for the Publisher ad slot that may be
  // rendered on the user's device.
  rpc SelectAd(SelectAdRequest) returns (SelectAdResponse) {
    option (google.api.http) = {
      post: "/v1/selectAd"
      body: "*"
    };
  }
}

// SelectAdRequest is sent by the untrusted Seller service to SellerFrontEnd
// (SFE) once it receives an encrypted ProtectedAudienceInput from a client.
// SelectAdRequest would also include contextual signals and other data
// passed by untrusted Seller service for the auction.
message SelectAdRequest {
  message AuctionConfig {
    // Contextual signals that include information about the context
    // (e.g. Category blocks Publisher has chosen and so on). This is passed by
    // untrusted Seller service to SellerFrontEnd service.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "sellerSignals".
    // The serialized string can be deserialized to a JSON object.
    string seller_signals = 1;

    // Contextual signals that are passed by untrusted Seller service to
    // SellerFrontEnd service.
    // Information about auction (ad format, size). This information
    // is available both to the seller and all buyers participating in
    // auction.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "auctionSignals".
    // The serialized string can be deserialized to a JSON object.
    string auction_signals = 2;

    // List of buyers participating in FLEDGE auctions.
    // Buyers are identified by buyer domain (i.e. Interest Group owner).
    repeated string buyer_list = 3;

    // Seller origin / domain.
    string seller = 4;

    // Per buyer configuration.
    message PerBuyerConfig {
      // Contextual signals corresponding to each Buyer in auction that could
      // help in generating bids.
      string buyer_signals = 1;

      // Optional.
      // The Id is specified by the buyer to support coordinated experiments
      // with the buyer's Key/Value services.
      int32 buyer_kv_experiment_group_id = 2;

      // Optional.
      // Version of buyer's GenerateBid() code.
      // The string must be an object name belonging to the
      // Cloud Storage bucket specified at Bidding service startup.
      // A buyer can pass this information to the Seller in RTB response.
      // If a version is not specified, the default version
      // (specified in the service startup config) will be used.
      int32 generate_bid_code_version = 3;
      
      // Optional.
      // A debug id passed by the buyer that will be logged with VLOG, if
      // available. This can help adtech oncallers to map an ad request
      // with their internal log / query id.
      // Buyer can pass this information to the Seller in RTB response.
      // Note: The VLOGs are only accessible in TEE debug mode. In TEE
      // production mode, additional user consent would be required to access
      // these.
      string buyer_debug_id = 4;
    }

    // The key in the map corresponds to Interest Group Owner (IGOwner), a
    // string that can identify a buyer participating in the auction. The
    // SellerFrontEnd server configuration, has the mapping of IGOwner to a
    // public load balancer address in front of BuyerFrontEnd. IGOwners that the
    // SFE has not been configured to communicate with will simply be ignored.
    map<string, PerBuyerConfig> per_buyer_config = 5;

    // Contains information about all code module versions to be used for
    // bidding, auctions, and reporting. This supports the seller and buyers in
    // maintaining multiple versions of their ScoreAd and GenerateBid modules,
    // respectively, which may be used for experimentation. The desired code
    // module version can be specified here per ad selection request.
    message SellerCodeExperimentSpecification {
      // The Id is specified by the seller to support coordinated experiments
      // with the seller's Key/Value services.
      int32 seller_kv_experiment_group_id = 1;

      // The code version of the score ad module provided by the seller.
      // The string must be an object name belonging to the
      // Cloud Storage bucket specified at Auction service startup.
      // If a version is not specified, the default version
      // (specified in the service startup config) will be used.
      string score_ad_version = 2;
    }

    // Specifications about code modules that are passed by
    // the Seller Ad service in a SelectAd request.
    SellerCodeExperimentSpecification code_experiment_spec = 6;
    
    // Optional.
    // A debug id passed by the seller that will be logged with VLOG, if
    // available. This can help adtech oncallers to map an ad request
    // with their internal log / query id.
    // Note: The VLOGs are only accessible in TEE debug mode. In TEE
    // production mode, additional user consent would be required to access
    // these.
    string seller_debug_id = 7;

    // Optional.
    // Timeout is milliseconds specified by the seller that applies to total
    // time to complete GetBids.
    // If no timeout is specified, the Seller's default maximum Buyer timeout
    // configured in SellerFrontEnd service configuration, will apply.
    int32 buyer_timeout_ms = 8;
  }

  // Encrypted ProtectedAudienceInput generated by the device.
  bytes protected_audience_ciphertext = 1;

  // Plaintext. Passed by the untrusted Seller service.
  AuctionConfig auction_config = 2;

  enum ClientType {
    UNKNOWN = 0;

    // An Android device with Google Mobile Services (GMS).
    // Note: This covers apps on Android and browsers on Android.
    ANDROID = 1;

    // Any browser.
    BROWSER = 2;
  }

  // Type of end user's device / client, that would help in validating the
  // client integrity.
  // Note: Not all types of clients can be attested.
  ClientType client_type = 3;
}

// SelectAdResponse is sent from the SellerFrontEndService to the Seller
// service. auction_result_ciphertext can only be decrypted by the client device
// that initiated the original SelectAdRequest. The untrusted Seller service may
// send the contextual winner back to the client in addition to the
// auction_result_ciphertext to allow the client to pick the final winner.
message SelectAdResponse {
  // Encrypted AuctionResult from FLEDGE auction. May  contain a real candidate
  // or chaff, depending on ScoreAd() outcomes.
  bytes auction_result_ciphertext = 1;
}
```

#### LogContext 

Context for logging requests in Bidding and Auction servers. This includes `generation_id`
passed by the client in encrypted [ProtectedAudienceInput][9] and optional
(per) `buyer_debug_id` and `seller_debug_id` passed in [`SelectAdRequest.AuctionConfig`][35].
The `adtech_debug_id` (`buyer_debug_id` or `seller_debug_id`) can be an internal log / query id
used in an adtech's non TEE based systems and if available can help the adtech trace the ad request 
log in Bidding and Auction servers and map with the logs in their non TEE based systems.

```
// Context useful for logging and debugging requests.
message LogContext {
  // UUID for the request (as originating from client).
  string generation_id = 1;

  // Adtech debug id that can be used for correlating the request with the
  // adtech. This will contain `buyer_debug_id` when used in context of buyer
  // services and `seller_debug_id` when used in context of seller services.
  string adtech_debug_id = 2;
}
```

#### BuyerFrontEnd Service and API Endpoints

The BuyerFrontEnd service exposes an API endpoint GetBids. The SellerFrontEnd service sends
encrypted GetBidsRequest to the BuyerFrontEnd service that includes BuyerInput and other data.
After processing the request, BuyerFrontEnd returns GetBidsResponse, which includes bid(s) for
each Interest Group. Refer to [AdWithBid][49] for more information.

The communication between the BuyerFrontEnd service and the SellerFrontEnd service is TEE to TEE
communication and is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication will happen
over public network and that can also be cross cloud networks.

```
syntax = "proto3";

// Buyer's FrontEnd service.
service BuyerFrontEnd {
  // Returns bids for each Interest Group / Custom Audience.
  rpc GetBids(GetBidsRequest) returns (GetBidsResponse) {
    option (google.api.http) = {
      post: "/v1/getbids"
      body: "*"
    };
  }
}

// GetBidsRequest is sent by the SellerFrontEnd Service to the BuyerFrontEnd
// service.
message GetBidsRequest {
  // Unencrypted request.
  message GetBidsRawRequest {
    // Whether this is a fake request from SellerFrontEnd service
    // and should be dropped.
    // Note: SellerFrontEnd service will send chaffs to a very small set of
    // other buyers not participating in the auction. This is required for
    // privacy reasons to prevent seller from figuring the buyers by observing
    // the network traffic to `BuyerFrontEnd` Services, outside TEE.
    bool is_chaff = 1;

    // Buyer Input for the Buyer that includes keys for Buyer Key Value lookup
    // and other signals for bidding. In the case of is_chaff = true, this will
    // be noise.
    BuyerInput buyer_input = 2;

    // Information about auction (ad format, size) derived contextually.
    // Represents a serialized string that is deserialized to a JSON object
    // before passing to Adtech script. Copied from contextual signals sent to
    // SellerFrontEnd service.
    string auction_signals = 3;

    // Buyer may provide additional contextual information that could help in
    // generating bids. This is Copied from contextual signals sent to
    // SellerFrontEnd service.
    // The value represents a serialized string that is deserialized to a JSON
    // object before passing to Adtech script.
    string buyer_signals = 4;

    // Seller origin.
    // Used to verify that a valid seller is sending the request.
    string seller = 5;

    // Publisher website or app that is part of Buyer KV lookup url.
    string publisher_name = 6;

    // A boolean value which indicates if event level debug reporting should be
    // enabled or disabled for this request.
    bool enable_debug_reporting = 7;

    // Helpful context for logging and tracing the request.
    LogContext log_context = 8;
  }

  // Encrypted GetBidsRawRequest.
  bytes request_ciphertext = 1;

  // Version of the public key used for request encryption. The service
  // needs use private keys corresponding to same key_id to decrypt
  // 'request_ciphertext'.
  string key_id = 2;
}

// Response to GetBidsRequest.
message GetBidsResponse {
  // Unencrypted response.
  message GetBidsRawResponse {
    // Includes ad_render_url and corresponding bid value pairs for each IG.
    // Represents a JSON object.
    repeated AdWithBid bids = 1;
  }

  // Encrypted GetBidsRawResponse.
  bytes response_ciphertext = 1;
}
```

##### AdWithBid

The AdWithBid for an ad candidate, includes `ad` (i.e. ad metadata), `bid`, `render` (i.e. ad render url),
`allow_component_auction` and `interest_group_name`. This is returned in GetBidsResponse by
BuyerFrontEnd to SellerFrontEnd.

```
syntax = "proto3";

// Bid for an ad candidate.
message AdWithBid {
  // Metadata of the ad, this will be passed to Seller's scoring function.
  // Represents a serialized string that is deserialized to a JSON object
  // before passing to Adtech script.
  // Note: API will be updated separately for Component Ads.
  google.protobuf.Value ad = 1;

  // Bid price corresponding to an ad.
  float bid = 2;

  // Ad render url that identifies an ad creative.
  string render = 3;

  // List of ad render urls that identifies ad components.
  // This field must not be present if no component_ad_render_id is passed in
  // Interest Group to GenerateBid().
  repeated string ad_component_render = 4;

  // Whether component auction is allowed.
  bool allow_component_auction = 5;

  // Name of the Custom Audience / Interest Group this ad belongs to required
  // by the device to validate that a winning remarketing ad actually belongs
  // to the InterestGroup / CustomAudience as stored on-device.
  string interest_group_name = 6;

  // A numerical value used to pass reporting advertiser click or conversion
  // cost from generateBid to reportWin. The precision of this number is
  // limited to an 8-bit mantissa and 8-bit exponent, with any rounding
  // performed stochastically.
  double ad_cost = 7;

  // Optional field for debug report URLs.
  DebugReportUrls debug_report_urls = 8;

  // A 12 bit integer signal used as input to win reporting url generation for
  // the Buyer.
  int32 modeling_signals = 9;

  // Indicates the currency used for the bid price.
  string bid_currency = 10;
}
```

### Internal API

Internal APIs refer to the interface for communication between FLEDGE services within a SSP
system or DSP system.

#### Bidding Service and API Endpoints

The Bidding service exposes an API endpoint GenerateBids. The BuyerFrontEnd service sends
GenerateBidsRequest to the Bidding service, that includes required input for bidding. The code for
bidding is prefetched from Cloud Storage and cached in Bidding service. After processing the request,
i.e. generating bids, the Bidding service returns the GenerateBidsResponse to BuyerFrontEnd service.

The communication between the BuyerFrontEnd service and Bidding service occurs between each service’s TEE
and request-response is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication also happens
over a private VPC network.

```
syntax = "proto3";

// Bidding service operated by buyer.
service Bidding {
  // Generate bids for ads in Custom Audiences (a.k.a InterestGroups) and
  // filters ads.
  rpc GenerateBids(GenerateBidsRequest) returns (GenerateBidsResponse) {
    option (google.api.http) = {
      post: "/v1/generatebids"
      body: "*"
    };
  }
}

// Generate bids for all Custom Audiences (a.k.a InterestGroups) corresponding
// to the Buyer.
message GenerateBidsRequest {
  // Unencrypted request.
  message GenerateBidsRawRequest {
    // Custom Audience (a.k.a Interest Group) for bidding.
    message InterestGroupForBidding {
      // Unique string that identifies the Custom Audience (a.k.a Interest
      // Group) for a buyer.
      // The object "name" is part of InterestGroup JSON object that is an
      // argument to GenerateBid.
      string name = 1;

      // Used to fetch real time bidding signals from buyer's key/value server
      // included in the request. The value of each key in this list will be
      // passed from the bidding signals dictionary to the Interest Group's
      // GenerateBid() function as the trustedBiddingSignals parameter.
      repeated string bidding_signals_keys = 2;

      // Optional.
      // Id of ad_render_url generated by the DSP / Buyer and passed to the
      // client. Then client passes this in InterestGroup if available.
      // Note: If the Buyer doesn't generate the ad_render_id, then their
      // GenerateBid() should dynamically generate the url for the bid. The
      // winning ad render url returned back to the client will be validated
      // with the Interest Group information on the client.
      repeated string ad_render_ids = 3;

      // Optional.
      // Id of ad_component_render_url generated by the DSP / Buyer and passed
      // to the client.
      repeated string ad_component_render_ids = 4;

      // Optional.
      // User bidding signal that may be ingested during bidding and/or
      // filtering. This should be any serializable object.
      string user_bidding_signals = 5;

      // Required for bidding.
      // Contains filtering data, like Frequency Cap.
      oneof DeviceSignals {
        // A JSON string constructed by Android that includes Frequency Cap
        // information.
        AndroidSignals android_signals = 6;

        // An object constructed by the browser, containing information that
        // the browser knows like previous wins of ads / Frequency Cap
        // information.
        BrowserSignals browser_signals = 7;
      }
    }

    // Interest Group is an input to bidding code.
    repeated InterestGroupForBidding interest_group_for_bidding = 1;

    /********************* Common inputs for bidding ***********************/
    // Information about auction (ad format, size) derived contextually.
    // Represents a JSON object. Copied from Auction Config in SellerFrontEnd
    // service.
    // Represents a serialized string that is deserialized to a JSON object
    // before passing to Adtech script.
    string auction_signals = 2;

    // Buyer may provide additional contextual information that
    // could help in generating bids. Not fetched real-time.
    // Represents a serialized string that is deserialized to a JSON object
    // before passing to Adtech script.
    //
    // Note: This is passed in encrypted BuyerInput, i.e.
    // buyer_input_ciphertext field in GetBidsRequest. The BuyerInput is
    // encrypted in the client and decrypted in `BuyerFrontEnd` Service.
    // Note: This is passed in BuyerInput.
    string buyer_signals = 3;

    // Real Time signals fetched from buyer's Key/Value service.
    string bidding_signals = 4;

    // A boolean value which indicates if event level debug reporting should be
    // enabled or disabled for this request.
    bool enable_debug_reporting = 5;
    
    // Seller origin.
    // Sent to generateBid script.
    string seller = 6;

    // Publisher website or app that is part of Buyer KV lookup url.
    string publisher_name = 7;

    // Helpful context for logging and tracing the request.
    LogContext log_context = 8;
  }

  // Encrypted GenerateBidsRawRequest.
  bytes request_ciphertext = 1;

  // Version of the public key used for request encryption. The service
  // needs use private keys corresponding to same key_id to decrypt
  // 'request_ciphertext'.
  string key_id = 2;
}

// Encrypted response to GenerateBidsRequest with bid prices corresponding
// to all eligible Ad creatives.
message GenerateBidsResponse {
  // Unencrypted response.
  message GenerateBidsRawResponse {
    // Bids corresponding to ads. Each AdWithBid object contains bid for ad per
    // IG (CA). Note GenerateBid() per IG returns bid for one ad per IG (though
    // for component auction this would be slightly different).
    repeated AdWithBid bids = 1;
  }

  // Encrypted GenerateBidsRawResponse.
  bytes response_ciphertext = 1;
}
```

#### Auction Service and API Endpoints

The Auction service exposes an API endpoint ScoreAds. The SellerFrontEnd service sends a
ScoreAdsRequest to the Auction service for running auction; ScoreAdsRequest includes bids from
each buyer and other required signals. The code for auction is prefetched from Cloud Storage and
cached in Auction service. After all ads are scored, the Auction service picks the highest scored
ad candidate and returns the score and other related data for the winning ad in ScoreAdsResponse.

The communication between the SellerFrontEnd service and Auction service occurs within each service’s
TEE and request-response is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication also
happens over a private VPC network.

```
syntax = "proto3";

// Auction service operated by the seller.
service Auction {
  // Scores all top ad candidates returned by each buyer participating
  // in the auction.
  rpc ScoreAds(ScoreAdsRequest) returns (ScoreAdsResponse) {
    option (google.api.http) = {
      post: "/v1/scoreads"
      body: "*"
    };
  }
}

// Scores top ad candidates of each buyer.
message ScoreAdsRequest {
  // Unencrypted request.
  message ScoreAdsRawRequest {
    // Bid for an ad along with other information required to score the ad.
    message AdWithBidMetadata {
      // Metadata of the ad, this will be passed to Seller's scoring function.
      // Represents a serialized string that is deserialized to a JSON object
      // before passing to Adtech script.
      google.protobuf.Value ad = 1;

      // Bid price corresponding to an ad.
      float bid = 2;

      // Ad render url that identifies an ad creative.
      string render = 3;

      // Optional.
      // List of ad render urls that identifies ad components.
      // This field must not be present if no component_ad_render_id is passed
      // in Interest Group for bidding.
      repeated string ad_component_render = 4;

      // Whether component auction is allowed.
      bool allow_component_auction = 5;

      // Name of the Custom Audience / Interest Group this ad belongs to
      // required by the device to validate that a winning remarketing ad
      // actually belongs to the InterestGroup / CustomAudience as stored
      // on-device.
      string interest_group_name = 6;

      // Domain of Buyer who owns the interest group that includes the ad.
      string interest_group_owner = 7;

      // The number of times this device has joined this interest
      // group in the last 30 days while the interest group has been
      // continuously stored (that is, there are no gaps in the storage of the
      // interest group on the device due to leaving or membership expiring)
      int32 join_count = 8;

      // Duration of time (in minutes) from when this device joined the interest
      // group until the current time.
      int64 recency = 9;

      // A signal used as input to win reporting url generation for the Buyer.
      int32 modeling_signals = 10;

      // A numerical value used to pass reporting advertiser click or conversion
      // cost from generateBid to reportWin. The precision of this number is
      // limited to an 8-bit mantissa and 8-bit exponent, with any rounding
      // performed stochastically.
      double ad_cost = 11;
    }
    // Ad with bid.
    repeated AdWithBidMetadata ad_bids = 1;

    /*....................... Contextual Signals .........................*/
    // Contextual Signals refer to seller_signals and auction_signals
    // derived contextually.

    // Seller specific signals that include information about the context
    // (e.g. Category blocks Publisher has chosen and so on). This can
    // not be fetched real-time from Key-Value Server.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "sellerSignals".
    // Note: This is passed by client in AuctionConfig in
    // SelectAdRequest to SellerFrontEnd service. This data is copied
    // from AuctionConfig. The serialized string can be deserialized to a JSON
    // object.
    string seller_signals = 2;

    // Information about auction (ad format, size). This information
    // is available both to the seller and all buyers participating in
    // auction.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "auctionSignals".
    // Note: This is passed by client in AuctionConfig
    // in SelectAdRequest to SellerFrontEnd service. This data is copied
    // from AuctionConfig. The serialized string can be deserialized to a JSON
    // object.
    string auction_signals = 3;

    /*....................... Real time signals .........................*/
    // Real-time signals fetched from seller Key Value Service.
    // Represents a JSON string as fetched from Seller Key Value service.
    // Note: The keys used to look up scoring signals are ad_render_urls and
    // ad_component_render_urls that are part of the bids returned by buyers
    // participating in the auction.
    string scoring_signals = 4;

    // Publisher website or app included in device signals.
    string publisher_hostname = 5;

    // A boolean value which indicates if event level debug reporting should be
    // enabled or disabled for this request.
    bool enable_debug_reporting = 6;

    // Helpful context for logging and tracing the request.
    LogContext log_context = 7;
  }

  // Encrypted ScoreAdsRawRequest.
  bytes request_ciphertext = 1;

  // Version of the public key used for request encryption. The service
  // needs use private keys corresponding to same key_id to decrypt
  // 'request_ciphertext'.
  bytes key_id = 2;
}

// Encrypted response that includes winning ad candidate.
message ScoreAdsResponse {
  // Identifies the winning ad belonging to a Custom Audience / Interest Group.
  message AdScore {
    // Rejection reasons provided by seller should be one of the following.
    // Details: https://github.com/WICG/turtledove/blob/main/Proposed_First_FLEDGE_OT_Details.md#reporting.
    enum RejectionReason {
      NOT_AVAILABLE = 0;
      INVALID_BID = 1;
      BID_BELOW_AUCTION_FLOOR = 2;
      PENDING_APPROVAL_BY_EXCHANGE = 3;
      DISAPPROVED_BY_EXCHANGE = 4;
      BLOCKED_BY_PUBLISHER = 5;
      LANGUAGE_EXCLUSIONS = 6;
      CATEGORY_EXCLUSIONS = 7;
    }  
    
    // This captures the rejection reason provided by the seller for an Ad.
    // An ad is identified by the interest group owner and name.
    message AdRejectionReason {
      // Name of the Custom Audience / Interest Group.
      string interest_group_name = 1;

      // Domain of Buyer who owns the interest group that includes the ad.
      string interest_group_owner = 2;

      // Rejection reason provided by the seller for this ad.
      RejectionReason rejection_reason = 3;
    }
  
    // Score of the ad determined during the auction. Any value that is zero or
    // negative indicates that the ad cannot win the auction. The winner of the
    // auction would be the ad that was given the highest score.
    // The output from ScoreAd() script is desirability that implies score for
    // an ad.
    float desirability = 1;

    // Ad creative render url.
    string render = 2;

    // Ad creative render url.
    repeated string component_renders = 3;

    // Name of Custom Audience / Interest Group the ad belongs to.
    string interest_group_name = 4;

    // Bid corresponding to the winning ad.
    float buyer_bid = 5;

    // Domain of Buyer who owns the interest group that includes the ad.
    string interest_group_owner = 6;

    /***************** Only relevant to Component Auctions *******************/
    // Additional fields for Component Auctions.

    // Optional. Arbitrary metadata to pass to top level seller.
    // This is also optional for Component Auctions.
    string ad_metadata = 7;

    // Optional for Android, required for Web in case of component auctions.
    // If the bid being scored is from a component auction and this value is not
    // true, the bid is ignored. If not present, this value is considered false.
    // This field must be present and true both when the component seller scores
    // a bid, and when that bid is being scored by the top-level auction.
    bool allow_component_auction = 8;

    // Optional for Android, required for Web in case of component auctions.
    // Modified bid value to provide to the top-level seller script. If
    // present, this will be passed to the top-level seller's scoring function
    // instead of the original bid, if the ad wins the component auction and
    // top-level auction respectively.
    // This is also optional for Component Auctions.
    float bid = 9;

    // The reporting urls registered during the execution of reportResult() and
    // reportWin().
    WinReportingUrls win_reporting_urls = 10;

    // Optional field for debug report URLs.
    DebugReportUrls debug_report_urls = 11;

    // Map of the interest group owners to the list of bids that got the second
    // highest score. This is used for debugging reporting.
    map<string, google.protobuf.ListValue>
        ig_owner_highest_scoring_other_bids_map = 12;
        
    // List of rejection reasons for scored ads. For ads which are not rejected,
    // we do not return any rejection reason. The reporting client will
    // substitute the default rejection reason.
    repeated AdRejectionReason ad_rejection_reasons = 13;
  }

  // The response includes the top scored ad along with other related data.
  // Unencrypted response.
  message ScoreAdsRawResponse {
    // Score of the winning ad in the auction.
    AdScore ad_score = 1;
  }

  // Encrypted ScoreAdsRawResponse.
  bytes response_ciphertext = 1;
}
```

#### WinReporting Urls

```
// The reporting urls registered during the execution of reportResult() and
// reportWin(). These urls will be pined from the client.
message WinReportingUrls {
  message ReportingUrls {
    // The url to be pinged for reporting win to the Buyer or Seller.
    string reporting_url = 1;

    // The map of (interactionKey, URI).
    map<string, string> interaction_reporting_urls = 2;
  }

  // The reporting urls registered during the execution of
  // reportWin(). These urls will be pinged from client.
  ReportingUrls buyer_reporting_urls = 1;

  // The reporting urls registered during the execution of reportResult() of the
  // seller in case of single seller auction and component seller in case of
  // multi seller auctions. These urls will be pinged from client.
  ReportingUrls component_seller_reporting_urls = 2;

  // The reporting urls registered during the execution of reportResult() of the
  // top level seller in case of multi seller auction. These urls will be pinged
  // from client. This will not be set for single seller auctions.
  ReportingUrls top_level_seller_reporting_urls = 3;
}
```

#### DebugReporting Urls

```
// Urls to support debug reporting, when auction is won and auction is lost.
message DebugReportUrls {
  // URL to be triggered if the interest group wins the auction.
  // If undefined or malformed url it will be ignored.
  string auction_debug_win_url = 1;

  // URL to be triggered if the interest grou losses the auction.
  // If undefined or malformed url it will be ignored.
  string auction_debug_loss_url = 2;
}
```

[4]: https://privacysandbox.com
[5]: https://developer.chrome.com/docs/privacy-sandbox/fledge/
[6]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md
[7]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md
[8]: https://github.com/microsoft/PARAKEET
[9]: #protectedaudienceinput
[10]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#key-management-systems
[11]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#fledge-services
[12]: https://grpc.io
[13]: https://developers.google.com/protocol-buffers
[14]: https://en.wikipedia.org/wiki/Interface_description_language
[15]: https://developers.google.com/protocol-buffers/docs/proto3
[16]: https://www.terraform.io/
[17]: https://github.com/privacysandbox
[18]: https://developers.google.com/protocol-buffers/docs/proto3#json
[19]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#client-to-service-communication
[20]: #seller-ad-service
[21]: #sellerfrontend-service
[22]: #buyerfrontend-service
[23]: #auction-service
[24]: https://developer.android.com/design-for-safety/privacy-sandbox/fledge
[25]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#21-initiating-an-on-device-auction
[26]: https://github.com/chatterjee-priyanka
[27]: https://developer.chrome.com/blog/bidding-and-auction-services-availability
[28]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md
[29]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trusted-execution-environment
[30]: https://aws.amazon.com/ec2/nitro/nitro-enclaves/
[31]: https://cloud.google.com/blog/products/identity-security/announcing-confidential-space
[32]: https://cloud.google.com/confidential-computing
[33]: https://github.com/privacysandbox
[34]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md
[35]: #sellerfrontend-service-and-api-endpoints
[36]: #buyerfrontend-service-and-api-endpoints
[37]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#23-scoring-bids
[38]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding
[39]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding
[40]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#31-fetching-real-time-data-from-a-trusted-server
[41]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#32-on-device-bidding
[42]: #bidding-service
[43]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#5-event-level-reporting-for-now
[44]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#35-filtering-and-prioritizing-interest-groups
[45]: https://www.envoyproxy.io/
[46]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded
[47]: https://github.com/grpc/grpc-go/blob/master/Documentation/grpc-metadata.md#constructing-metadata
[48]: https://datatracker.ietf.org/doc/rfc9180/
[49]: #adwithbid
[50]: https://datatracker.ietf.org/wg/ohttp/about/
[51]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding-auction-services-payload-optimization.md
[52]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_aws_guide.md
[53]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md
[54]: https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md
[55]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md
[56]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md
[57]: https://github.com/privacysandbox/fledge-docs#bidding-and-auction-services
[58]: https://github.com/privacysandbox/fledge-docs#server-productionization
[59]: https://github.com/privacysandbox/bidding-auction-servers
[60]: https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md
[61]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#adtech-code-execution-engine
[62]: https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md
[63]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#fake-requests--chaffs-to-dsp
[64]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#code-blob-fetch-and-code-version
[65]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#cloud-logging
[66]: #protectedaudienceinput
[67]: #scoread
[68]: #seller-byos-key--value-service
[69]: #generatebid
[70]: #buyer-byos-keyvalue-service
[71]: #sellerfrontend-service-configurations
[72]: #auction-service-configurations
[73]: #buyerfrontend-service-configurations
[74]: #bidding-service-configurations
[75]: #reportresult
[76]: #reportwin
[77]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#sellers-ad-service
[78]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#sellerfrontend-service
[79]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#auction-service
[80]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#buyerfrontend-service
[81]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#bidding-service
[82]: #buyerinput
[83]: #unified-request
[84]: ##auctionresult
[85]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#adtech-authentication-by-coordinator
[86]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#adtech-code-execution-engine
[87]: https://en.wikipedia.org/wiki/Service_mesh
[88]: #spec-for-ssp
[89]: #spec-for-dsp
