> FLEDGE has been renamed to Protected Audience API. To learn more about the name change, see the [blog post](https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge)

**Authors:** <br>
[Priyanka Chatterjee][26], Google Privacy Sandbox<br> 
Itay Sharfi, Google Privacy Sandbox

# Bidding and Auction Services High Level Design and API

[The Privacy Sandbox][4] aims to develop technologies that enable more private
advertising on the web and mobile devices. Today, real-time bidding and ad
auctions are executed on servers that may not provide technical guarantees of
security. Some users have concerns about how their data is handled to generate
relevant ads and in how that data is shared with ad providers. FLEDGE
([Android][24], [Chrome][5]) provides ways to preserve privacy and limit
third-party data sharing by serving personalized ads based on previous mobile
app or web engagement.

For all platforms, FLEDGE may require [real-time services][6]. In the initial
[FLEDGE proposal][7], bidding and auction for remarketing ad demand is
executed locally. This can demand computation requirements that may be
impractical to execute on devices with limited processing power, or may be
too slow to render ads due to network latency.

The Bidding and Auction Services proposal outlines a way to allow FLEDGE
computation to take place on cloud servers in a trusted execution environment,
rather than running locally on a user's device. Moving computations to cloud
servers can help optimize the FLEDGE auction, to free up computational cycles
and network bandwidth for a device.

This contrasts with [Google Chrome's initial FLEDGE proposal][7], which proposes
bidding and auction execution to occur locally. There are other ideas, similar
to FLEDGE, that propose server-side auction and bidding, such as [Microsoft Edge's
PARAKEET][8] proposal.

This document focuses on the high level design and API for FLEDGE's Bidding and
Auction services. Based on adtech feedback, further changes may be incorporated in
the design. A document that details system design for Bidding and Auction services
will also be published at a later date.

## Background

Read the [FLEDGE services overview][6] to learn about the environment, trust
model, server attestation, request-response encryption, and other details.

Each FLEDGE service is hosted in a virtual machine (VM) within a secure,
hardware-based trusted execution environment (TEE). Adtech platforms operate
and deploy FLEDGE services on a public cloud. Adtechs may choose the cloud
platform from one of the options that are planned. As cloud customers,
adtechs are the owners and only tenants of such VM instances.

## Chrome and Android Announcement 

Chrome and Android announced to integrate with Bidding and Auction services.
See [blog][27] for more details.  

## Near drop-in replacement

Bidding and Auction services integrate into [FLEDGE design][28] and can scale
as a near drop-in replacement for Adtechs who are interested in exploring a
server side solution. In particular, scripts that currently run for Chrome
on-device FLEDGE will work as-is running in the cloud. Key-value services can
stay the same; Interest groups creation and management can stay as is.

## Supported Public Cloud Platforms

Bidding and Auction services will be available within the [Trusted Execution Environment][29](TEE)
on AWS and GCP in 2023. More cloud platforms may be supported eventually. 

Bidding and Auction services will run in [Nitro Enclaves][30] on AWS and [Confidential
Space][31] ([Confidential Computing][32]) on GCP. 

Bidding and Auction services will not support a “Bring Your Own Server” model,
similar to what is made available to FLEDGE’s Key/Value server. **Bidding and Auction
services can only be deployed within approved TEE cloud environments.** This is valid
for Alpha, Beta, Scale testing programs and through 3PCD.

Solution on AWS and GCP will be open sourced ([service code](#service-code-and-framework),
[configurations](#service-configuration)) on [github][33] by the middle of the year to facilitate
early testing by interested Adtechs. Adtechs will have to deploy the binaries to a supported public
cloud platform. We will also publish explainers and user guides for cloud deployment and provide
recommendations on Virtual Machine instance types.

_Note: SSP and DSP can operate services on different cloud platforms that are supported._

## Timeline

Following are the timelines for Adtechs interested in testing Bidding and Auction services.

### Alpha Testing

Alpha Testing includes running services on non-production user opt-in traffic. Alpha testing
will be available starting July 2023. 

### Beta Testing

Beta testing includes running services on limited stable, production traffic. Beta testing
will be available starting October 2023.

### Scale Testing 

Available for full stable, production Scale testing starting January 2024. 

At that point, there will be **General Availability** of all features available with
[Chrome FLEDGE][34].

## Types of Auctions

Bidding and Auction services plan to support single-seller and all types of multi-seller
auctions including [Component Auctions][25].

Details about multi seller auctions will be published in a separate explainer.

## Unified Contextual and FLEDGE Auction Flow with Bidding and Auction Services

Seller's code (in the Publisher web page or app) sends one unified request for contextual
and FLEDGE auctions to Seller's Ad Service. The unified request includes contextual payload
and encrypted [remarketing data][9] that can only be decrypted by a Service running in the
[Trusted Execution Environment][29]. Seller initiates real time bidding requests to partner Buyers 
and then conducts contextual auction (existing flow today). After contextual auction concludes,
Seller's Ad Service calls Bidding and Auction services for FLEDGE auctions. 

_Unified Contextual and FLEDGE Auction Flow_ can further optimize e2e latency.

### Architecture

Following is the architecture of Bidding and Auction Services incorporating *Unified
Contextual and FLEDGE Auction Flow*.

![Architecture diagram.](images/unified-contextual-remarketing-bidding-auction-services.png)

_Note:_ 
* _All arrows in the diagram are bidirectional, implying request and response._
* _Sequence diagrams will be published in a detailed system design explainer._
* _Based on feedback from Adtechs, we may incorporate further optimization in the flow
  while ensuring it is privacy safe._

### Components

_Note: This explainer doesn't include client side design details; that will be published separately._

#### Sell-side platform (SSP) system

The following are the FLEDGE services that will be operated by an SSP, also referred to as a Seller. 

##### Seller Ad Service

With the *Unified Contextual and FLEDGE Auction flow, the Seller Ad Service will receive one request
from the client. The request would include contextual request payload and encrypted [remarketing data][9]
from the client. The encrypted [remarketing data][9] needs to be uploaded in the HTTP body, therefore
the Seller needs to support the HTTP POST method in the contextual request path.

The encrypted [remarketing data][9] (a.k.a remarketing ciphertext) includes Interest Group (Custom
Audience) information on the user's device. 

_**Note: Not all data in Interest Group would be sent over the wire in [remarketing data][9]. 
Also, our goal is to continue optimizing / reducing the payload size by collaborating closely with
DSPs. This is important to further optimize latency and reduce network cost for Adtechs. We will
publish payload optimization ideas separately.**_

##### SellerFrontEnd service

The front-end service of the system that runs in the [Trusted Execution Environment][29] on a
supported cloud platform. The service receives requests from [Seller Ad service][20] to initiate
FLEDGE auction flow. Then the service orchestrates requests (in parallel) to Buyers / DSPs
participating in the auction for bidding. 

This service also fetches real-time scoring signals required for the auction and calls 
[Auction service][23] for FLEDGE auction.

##### Auction service

The Auction service runs in the [Trusted Execution Environment][29](TEE) on a supported cloud
platform. This service responds to requests from the [SellerFrontEnd service][21]
and doesn't have access to arbitrary untrusted endpoints. 

The Auction service prefetches [code blobs](#adtech-code) owned by Seller from a Cloud Storage instance.
The code is prefetched periodically and cached. If required, more than one code version can be supported
at any point to facilitate experimentation by Adtechs.

SSP's code for scoring ads can be written in JavaScript and / or WebAssembly (WASM). The code runs
in a custom V8 sandbox within the TEE that has tighter security restrictions; that can not log 
information or has no disk or network access. For a ScoreAds request from SellerFrontEnd, SSP's scoring
code is executed per ad within a separate V8 worker thread but all execution can happen in parallel.
Between two executions in the service, there is no state saved.

_Note: The hosting environment protects the confidentiality of the Seller's code, if the execution
happens only in the cloud._

##### Seller's key/value service

A seller's Key/Value service is a critical dependency for the auction system. The Key/Value service
receives requests from the [SellerFrontEnd service][21] in this architecture. The service returns
real-time Seller data required for auction that corresponds to lookup keys available in buyers' bids
(such as ad_render_urls or ad_component_render_urls).

_Note: The Seller Key/Value system may be BYOS Key/Value Service or Trusted Key/Value Service depending
on the timeline._

#### Demand-side platform (DSP) system

This section describes FLEDGE services that will be operated by a DSP, also called a buyer. 

##### BuyerFrontEnd service

The front-end service of the system that runs in the [Trusted Execution Environment][29] on a
supported cloud platform. This service receives requests to generate bids from a [SellerFrontEnd
service][21]. This service fetches real-time bidding signals that are required for bidding and calls
Bidding service.

##### Bidding service

The Bidding service runs in the [Trusted Execution Environment][29] on a supported cloud platform. 
This service responds to requests from [BuyerFrontEnd service][22] and doesn't have access to
arbitrary untrusted endpoints. 

The Bidding service prefetches [code blobs](#adtech-code) owned by the Buyer from a Cloud Storage instance.
The code is prefetched periodically and cached. If required, more than one code version can be supported
at any point to facilitate experimentation by Adtechs.

DSP's code for generating bids can be written in JavaScript and / or WebAssembly (WASM). The code 
runs in a custom V8 sandbox within the TEE that has tighter security restrictions; that can not log
information or has no disk or network access. For a GenerateBids request from SellerFrontEnd, DSP's
code is executed per Interest Group (Custom Audience) within a separate V8 worker thread but all
execution can happen in parallel. Between two executions in the service, there is no state saved.

_Note: This environment protects the confidentiality of a buyer's code, if the execution happens only
in the cloud._

##### Buyer’s key/value Service

A buyer's Key/Value service is a critical dependency for the bidding system. The Key/Value service
receives requests from the [BuyerFrontEnd service][22] in this architecture. The service returns
real-time buyer data required for bidding, corresponding to lookup keys.

_Note: The buyer’s key/value system may be BYOS Key/Value Service or Trusted Key/Value Service
depending on timeline._

### Flow

* Seller's code in publisher page on the browser sends *Unified Contextual and FLEDGE Auction request* to
  Seller Ad Service.
    * Web: SSP's code in publisher webpage on the browser sends HTTP POST request to (untrusted) Seller
      Ad Service. 
        * __[Existing request, but excluding 3P Cookie]__ Contextual payload. 
        * SSP's code in publisher webpage asks browser for encrypted [remarketing data][9] to be included in
          request.
        * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) is added to HTTP request header.
          
    * Android: SSP's code in publisher SDK in Android sends HTTP POST request to (untrusted) Seller Ad
      Service.
        * __[Existing request]__ Contextual payload.
        * SSP's code in publisher SDK asks Android for encrypted [remarketing data][9] to be included in request.
        * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) is added to HTTP request header.
        
    * _Note:_
         * Encrypted remarketing data includes Interest Group (Custom Audience) information on the user's
           device and may be noised on the client to ensure it is always a fixed size. Refer [RemarketingInput][9]
           structure.
         * The [encrypted remarketing data][9] __may__ be required to be uploaded in the body of HTTP request,
           hence HTTP POST needs to be supported in the existing contextual path. 

* Seller Ad Service makes two sequential requests.
    * __[Existing flow]__ Sends Real Time Bidding (RTB) requests to buyers for contextual bids, conducts
      contextual auction.
    * Sends SelectAd request to SellerFrontEnd service. The request payload includes *encrypted remarketing
      data*, contextual signals, buyer_list, seller_origin and per_buyer_timeout.
      * Contextual signals include seller_signals, auction_signals and per_buyer_signals.
      * [Device metadata](#metadata-for-filtering-in-buyer-keyvalue-service) is [forwarded](#metadata-forwarding-by-seller-ad-service)
        to SellerFrontEnd service.
      
* Remarketing Bidding and Auction kicks off in Bidding and Auction Services.
    * The SellerFrontEnd service decrypts *encrypted remarketing data* using decryption keys prefetched
      from Key Management System (Refer [here][10] for more details).
    * The SellerFrontEnd service orchestrates GetBids requests to participating buyers’ (*buyer_list*)
      BuyerFrontEnd services in parallel.
    * Within each Buyer system:
        * The BuyerFrontEnd service decrypts GetBidsRequest using decryption keys prefetched from
          [Key Management System][10].
        * The BuyerFrontEnd services fetch real-time data from the Buyer’s Key/Value service required
          for generating bids.
        * The BuyerFrontEnd service sends a GenerateBids request to the Bidding service. The Bidding
          service returns ad candidates with bid(s) for each IG.
        * The BuyerFrontEnd returns all bid(s) in IG ([AdWithBid][49]) to SellerFrontEnd.
    * Once SellerFrontEnd has received bids from all buyers, it requests real-time data
      (*trustedScoringSignals*) for all render_urls (corresponding to ad with bids) from the Seller’s
      Key/Value service required to score the ads for auction.
    * SellerFrontEnd sends a ScoreAds request to the auction service to score ads and select a winner. 
        * Auction service deserializes and splits scoring signal such that ScoreAd() for an ad can
          only ingest *trustedScoringSignals* for the ad.
    * The Auction service selects the winning ad and additional data.
    * SellerFrontEnd returns winning ad, other metadata and reporting urls as an *encrypted remarketing
      response* back to Seller Ad Service.
     
* Seller Ad service returns the encrypted remarketing response back to the client.
    * Contextual ad and / or encrypted remarketing data will be sent back to the client. In case
      contextual ad wins (i.e. filters all remarketing bids during FLEDGE auction), a fake encrypted
      remarketing data will be sent in response.

* Client (browser, Android) decrypts the encrypted remarketing response.

* Ad is rendered on the device.

#### InitiateBiddingSignalsLookup

_Note: This is optional and not included in the above architecture diagram to keep the diagram simple.
The request path will be disabled by default and it is up to the Seller to enable it, depending on
whether a partner Buyer opts-in. However, this is expected to double the QPS on SellerFrontEnd even
if one Buyer opts-in and also double the QPS for BuyerFrontEnd of the Buyer._

[InitiateBiddingSignalsLookup][35] request may be sent by [Seller Ad service][20]
to [SellerFrontEnd][21] when the Real Time Bidding flow starts in Seller Ad Service. The
SellerFrontEnd may support an optional pathway for Buyers who may want to parallelize *trustedBiddingSignals*
lookup with contextual auction.

This is an optional optimization for Buyers and that may help optimize subsequent [GetBids][36] request
latency. Buyers may opt-in to this with the Sellers that they have partnered with. The SellerFrontEnd
service configuration must include the Buyers who opted-in to receive this request. When SellerFrontEnd
service receives InitiateBiddingSignalsLookup request from Seller Ad Service, that would lead SellerFrontEnd
to orchestrate [PrefetchBiddingSignals][36] requests (in parallel) to [BuyerFrontEnd services][22] operated by
Buyers who opted-in for this bidding signals prefetch flow. InitiateBiddingSignalsLookup or PrefetchBiddingSignals
do not initiate any bidding and auction. Also, if enabled by Seller, InitiateBiddingSignalsLookup must happen
before SelectAd.

This optional path can help Buyers optimize subsequent GetBids request (overall bidding for FLEDGE)
latency whose Buyer Key/Value server side latency may be high. If the Buyer Key/value service supports
server side cache, PrefetchBiddingSignals can help Buyers prefetch *trustedBiddingSignals* and prewarm
Key/Value caches while the contextual auction is happening in Seller Ad Service. When a [SelectAd][35] request
is received by a SellerFrontEnd server instance, there is another *trustedBiddingSignals*
lookup request sent to Buyer Key/Value Service. At that point, Buyer Key/Value service can serve the request
from the hot cache. 

## Cloud Deployment

FLEDGE Bidding and Auction services are deployed by Adtechs to a [public cloud platform](#supported-public-cloud-platforms)
so that they are co-located in a data center within a cloud region. Servers will be replicated in multiple cloud
regions and the availability Service Level Objective will be decided by respective Adtech. 

### SSP System

There will be a Global Load balancer for public traffic management for traffic to [SellerFrontEnd service][21].
There will be regional Virtual Private Cloud (VPC) load balancers for traffic management to Auction
service given traffic to Auction service will be over a private VPC network on the cloud.

### DSP System

There will be a Global Load balancer for public traffic management for traffic to [BuyerFrontEnd services][22].
There will be regional Virtual Private Cloud (VPC) load balancers for traffic management to Bidding
service given traffic to Bidding service will be over a private VPC network on the cloud.

## Specifications for Adtechs

### Spec for SSP

#### ScoreAd()

The [Auction service][23] exposes an API endpoint ScoreAds. The [SellerFrontEnd service][21] sends a
ScoreAdsRequest to the Auction service for running an auction. ScoreAdsRequest includes bids from each buyer
and other required signals. The code for auction, i.e. ScoreAd() is prefetched from Cloud Storage and cached in
Auction service. After all ads are scored, the Auction service picks the highest scored ad candidate
and returns the score and other related data for the winning ad in ScoreAdsResponse.

_Note: If you are developing scoreAd() following the web platform's [Chrome FLEDGE explainer][37], that
also should work as-is for execution in Bidding and Auction services._

Adtech's scoreAd function signature is as follows.

```
scoreAd(adMetadata, bid, auctionConfig, trustedScoringSignals, bid_metadata) {
  ...
  return {desirability: desirabilityScoreForThisAd,
              allowComponentAuction: true_or_false};
}
```

##### Arguments

* _adMetadata_: Arbitrary metadata provided by the buyer.
* _bid_: A numerical bid value.
* _auctionConfig_: This would include sellerSignals and auctionSignals, i.e. auctionConfig.sellerSignals
  and auctionConfig.auctionSignals that can be derived in scoreAd script.
* _trustedScoringSignals_: trustedScoringSignals fetched from Seller Key/Value service
    * Note: Only the signals required for scoring the ad / bid is passed to scoreAd().
* _bid_metadata_: This refers to an object created in the Auction service based on the render_url of the
  bid and other information known to the Auction service. 

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

The [SellerFrontEnd service][21] looks up *trustedScoringSignals* from Seller's Key/Value service. The 
base url (domain) for Key/Value service is configured in [SellerFrontEnd service][21] so that the
connection can be prewarmed. All render_urls corresponding to all bids from buyers participating
in an auction are encoded, then batched and looked up in a single request. The lookup url and
response are in the same format as described in [Chrome FLEDGE explainer][40].

The lookup url is the following format:

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

The trustedScoringSignals passed to scoreAd() is as follows:

```
{ 
  'renderUrl': {'https://cdn.com/render_url_of_bidder': arbitrary_value_from_signals},
  'adComponentRenderUrls': {
      'https://cdn.com/ad_component_of_a_bid': arbitrary_value_from_signals,
      'https://cdn.com/another_ad_component_of_a_bid': arbitrary_value_from_signals,
      ...}
}
```

### Spec for DSP

#### GenerateBid()

The [Bidding service][42] exposes an API endpoint GenerateBids. The [BuyerFrontEnd service][22] sends
GenerateBidsRequest to the Bidding service, that includes required input for bidding. The code
for bidding, i.e. GenerateBid() is prefetched from Cloud Storage and cached in Bidding service.
After processing the request, the Bidding service returns the GenerateBidsResponse which includes 
bids that correspond to each ad, i.e. [AdWithBid][49].

_Note: If you are developing generateBid() following the web platform's [Chrome FLEDGE explainer][41], 
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

* interestGroup: The InterestGroup (Custom Audience) object. Refer InterestGroup data structure to
  understand what is sent in this object from the client.
    * _Note: To reduce payload over the network and further optimize latency, our goal is to minimize
      the information sent in this object. We will work with Adtechs for the long term to reduce the
      amount of information sent in this object and try to find a solution to fetch those on the server
      side._

* auctionSignals: Contextual signal that is passed from Seller Ad service to SellerFrontEnd in SelectAd
  request.

* perBuyerSignals: Contextual signal generated by the Buyer during Real Time Bidding that is passed
  from Seller Ad service to SellerFrontEnd in SelectAd request. 

* trustedBiddingSignals: Real time signals fetched by BuyerFrontEnd service from Buyer's Key/Value
  service. 
  * _Note: Only the trustedBiddingSignals required for generating bid(s) for the interestGroup are passed
    to generateBid()_.

* deviceSignals: This refers to "browserSignals" or "androidSignals",  built by the client (Browser,
  Android). This includes Frequency Cap (statistics related to previous win of ads) for the user's device.
  

#### Buyer BYOS Key/Value Service

The [BuyerFrontEnd service][22] looks up biddingSignals from Buyer's BYOS Key / Value service. The base url
(domain) for Key/Value service is configured in BuyerFrontEnd service so that the connection can be
prewarmed. All lookup keys are batched together in a single lookup request. The lookup url and response
are in the same format as described in [Chrome FLEDGE explainer][40].

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

* IP / Geo Information
* User Agent 
* Language

###### Service (BuyerFrontEnd)

 IP address
 
###### Metadata Forwarding 
 
 * The metadata will be sent from device to Seller Ad Service in HTTP request header.
 * Seller Ad Service will [forward](#metadata-forwarding-by-seller-ad-service) that to SellerFrontEnd service.
 * SellerFrontEnd will [add metadata to gRPC][47] request sent to BuyerFrontEnd service.
 * BuyerFrontEnd will [forward][46] the metadata in the request header to Buyer Key/Value service.

#### Priority Vector

[Priority Vector][44] can help filter Interest Groups and reduce unnecessary executions in Bidding service.
This feature will be supported by Beta testing, will not be available by Alpha testing.

## Event Level Reporting

Event level win reporting would work with Bidding and Auction services and function signatures are the
same as described in [Chrome FLEDGE explainer][43]. Execution for win reporting can happen server side.
More information will be updated soon.

## Adtech Code 

Adtech code for generateBid(), scoreAd(), ReportResult(), ReportWin() (for event level win reporting) can
follow the same signature as described in the [Chrome FLEDGE explainer][28]. 

_Note:_
  * Code can be Javascript only or Javascript with WASM embedded or WASM only.
  * If the code is in Javascript, then Javascript context is initialized before every execution.
  * No limit on code blob size.
  * If required, more than one version of code can be supported at any point to support Adtech
    experimentation.
  * Adtech can upload their code to Cloud Storage supported by the Cloud Platform.
  * Code is prefetched by Bidding / Auction services running in [Trusted Execution Environment (TEE)][29]
    from the Cloud Storage bucket owned by Adtech.

## Dependencies

Through techniques such as prefetching and caching, the following dependencies are in the non-critical
path of ad serving.

### Key Management System

A [Key Management System][10] is required for FLEDGE service attestation and cryptographic key generation. 
Learn more in the [Overview of FLEDGE Services Explainer][6]. The Key Management System will be deployed to
all supported public clouds. Services in the Key Management System will be replicated in multiple cloud
regions. 

All services running in TEE prefetch encryption and decryption keys from Key Management System at service
startup and periodically in the non critical path. All communication between a service in TEE and another
service in TEE is end-to-end encrypted using Hybrid Public Key Encryption and TLS. Refer [here][11] for more
details.

## Service Code and Framework

Bidding and Auction services are developed in C++. The service configurations required for cloud deployment
are based on [Terraform][16].

The service framework is based on gRPC. [gRPC][12] is an open source, high performance RPC framework built
on top of HTTP2 that is used to build scalable and fast APIs. gRPC uses [protocol buffers][13] as the
[interface description language][14] and underlying message interchange format.

Bidding and Auction services code will be open sourced to [Privacy Sandbox github][17].

## Client <> Server and Server <> Server Communication

### Client <> Seller Ad Service Communication

[Client to Seller Ad Service communication][19] for the *Unified Contextual and FLEDGE Auction request* would
be HTTPS. Sellers would need to support the HTTP POST method in the contextual request path. The
[remarketing data][9] included in the unified request will be encrypted on the client using a protocol called
[Oblivious HTTP][50] that is based on bidirectional [Hybrid Public Key Encryption][48](HPKE). The FLEDGE
response, i.e. [AuctionResult](#auctionresult) will also be encrypted in SellerFrontEnd using [Oblivious HTTP][50].

The Seller Ad Service will not be able to decrypt or have access to [remarketing data][9] or [AuctionResult](#auctionresult)
in plaintext.

### Seller Ad Service <> SellerFrontEnd Communication

[Seller Ad service][20] can send gRPC or HTTPS to SellerFrontend service. There would be an [Envoy Proxy][45]
service hosted before [SellerFrontEnd][21]. If HTTPS is sent to SellerFrontEnd, Envoy Proxy would translate
that to gRPC.  

#### Metadata Forwarding by Seller Ad Service

To forward metadata received in request header from the client to SellerFrontEnd, Seller Ad Service can do one of
the following:
  * If the Seller Ad Service sends SelectAd as HTTPS to SFE, the header can be [forwarded][46].
  * If the Seller Ad Service sends SelectAd as gRPC, metadata needs to be [created and added][47].

### Communication between Bidding and Auction Services

All communication between services running in [Trusted Execution Environment (TEE)][29] is over TLS / SSL
and the request and response is end-to-end encrypted using bidirectional  [Hybrid Public Key Encryption][48](HPKE). 
The TLS / SSL session terminates at the load balancer in-front of a service, therefore the data over the wire from
the load balancer to service needs to be protected; hence the request-response is end-to-end encrypted using
bidirectional HPKE.

### Payload Compression

Most request/response payload sent over the wire should be compressed. 
* The [RemarketingInput][9] will be compressed on the client using `gzip`. The payload needs to be 
  compressed first and then encrypted. In SellerFrontEnd, [RemarketingInput][9] will be decrypted first
  and then decompressed.
* Seller Ad Service should compress [SelectAd][35] request payload when calling SellerFrontEnd service to save
  network bandwidth cost and reduce latency; `gzip` is recommended for payload compression. The selectAd response
  from SellerFrontEnd will be compressed using `gzip`.
* Request to BuyerFrontEnd service from SellerFrontEnd will be compressed first using `gzip` and then encrypted.
  In BuyerFrontEnd service, the request will have to be decrypted first and then decompressed.
    * _Note: This would be similar for communication between BuyerFrontEnd <> Bidding services and
      SellerFrontEnd <> Auction services._
* For Key/Value Server HTTP lookup, the [Accept-Encoding HTTP request header][38] would be set to specify the
  correct compression algorithm, i.e. `gzip`. The Key/Value server may set the [Content-Encoding Representation Header][39]
  to specify the content encoding (i.e. `gzip`) used to compress the response payload. 
    * _Note:_
         * It is recommended to compress Key/Value server response to optimize Key/Value lookup latency and reduce
           network bandwidth cost for Adtechs.
         * The request payload to Key/Value service need not be compressed given the size is expected to be small.
         * The request-response payload between SellerFrontEnd / BuyerFrontEnd and Seller / Buyer Key/Value services
           do not require additional encryption using [HPKE][48]. However, the communication between these services
           is over TLS that provide communications security by encrypting data sent over the untrusted network to
           an authenticated peer. 

## Service Configuration

Server configurations are based on [Terraform][16]. This should be best utilized for configuration data
that can be ingested when the service starts up. The configurations will include environment variables
and parameters that may vary per Adtech; these are set by the Adtech in the configuration before
deployment. Following are some examples of data configured in service configurations. 

### SellerFrontEnd service

* _Seller Key/Value service endpoint (scoring_signals_url)_: This endpoint is configured in SellerFrontEnd
  service configuration and ingested at service startup to prewarm connections to Seller Key/Value service.

* _BuyerFrontEnd endpoint_: The domain address of BuyerFrontEnd services operated by Buyers that this Seller
  has partnered with. This is ingested at service startup to prewarm connection.

* _Auction service endpoint_:  The domain address of Auction service. This is ingested at service startup to
  prewarm connection.

* _Seller origin_: The origin of the Seller.
     * _Note: The seller origin information is also passed by the Seller Ad Service in SelectAd request and
       SellerFrontEnd validates that with the origin information configured._
       
* _Map of {InterestGroupOwner, BuyerFrontEnd endpoint}_: Map of InterestGroupOwner to BuyerFrontEnd domain
  address.
  
* _Global timeout for Buyer_: This information can be used to set a timeout on each Buyer; however, will be
  overridden by the per_buyer_timeout passed by the Seller Ad service to SellerFrontEnd in SelectAd request.

* _prefetch_bidding_signals_buyers_list_: The list of Buyers (InterestGroupOwners) that have opted-in for
  the [PrefetchBiddingSignals][36] lookup.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

### Auction service

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where Seller code is hot reloaded by the
  Auction service. 
  
* Private Key Hosting service and Public Key Hosting service endpoints in [Key Management System][10].

### BuyerFrontEnd service

* _Buyer Key/Value service endpoint (bidding_signals_url)_: This endpoint is configured in SellerFrontEnd
  service configuration and ingested at service startup to prewarm connections to Seller Key/Value service.

* _Bidding service endpoint_: The domain address of Bidding service. This is ingested at service startup to
  prewarm connection.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

### Bidding service

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where Buyer code is hot reloaded by the
  Bidding service.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [Key Management System][10].

## Service APIs

### Client <> Server Data

Following section describes the data that flows from client (e.g. browser, Android) to Bidding and Auction
Services through [Seller Ad service][20] and the data received by client from Bidding and Auction Services.

_Note: We will update the API code to include additional data for Component Ads and Event level reporting._

#### ProtectedAudienceInput

ProtectedAudienceInput is generated and encrypted by client and sent to [Seller Ad service][20] in *Unified Contextual
and FLEDGE Auction request*. This includes per [BuyerInput](#buyer-input), `client_type` (i.e. Browser or Android),
`publisher_name` (i.e. website or app name) and an `encrypted_nonce`. 

```
syntax = "proto3";

// ProtectedAudienceInput is generated and encrypted by the client,
// passed through the untrusted Seller service, and decrypted by the
// SellerFrontEnd service.
// It is the wrapper for all of BuyerInput and other information required
// for the FLEDGE auction.
message ProtectedAudienceInput {
  // Input per buyer.
  // The key in the map corresponds to IGOwner (Interest Group Owner) that
  // is the Buyer / DSP. This  string that can identify a
  // buyer participating in the auction. The value corresponds to plaintext
  // BuyerInput ingested by the buyer for bidding.
  map<string, BuyerInput> buyer_input = 1;

  // Publisher website or app.
  // This is required to construct browser signals for web.
  // It will also be passed via GetBids to buyers for their Buyer KV lookup
  // to fetch trusted bidding signals.
  string publisher = 2;

  // Nonce passed by the client and sent back to the client in encrypted
  // response.
  string nonce = 3;

  // A boolean value which indicates if event level debug reporting should be
  // enabled or disabled for this request.
  bool enable_debug_reporting = 4;

  // Globally unique identifier for the client request.
  string generation_id = 5;
}
```

#### BuyerInput

BuyerInput is part of [RemarketingInput][9]. This includes data for each DSP / Buyer for server
side execution.

**_Note: We plan to publish ideas of BuyerInput optimization and collaborate with DSPs for
the longer term to fetch some of the InterestGroup data on the server side and / or pass ads in
an optimal way from the device. This can help reduce [RemarketingInput][9] payload size,
optimize latency even further and reduce network bandwidth cost for both SSPs and DSPs._**

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
      // A JSON string constructed by Android that includes Frequency Cap
      // information.
      string android_signals = 6;

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

  // Nonce decrypted by the SellerFrontEnd and sent back to the client.
  string nonce = 7;

  // Boolean to indicate that there is no remarketing winner from the auction.
  // AuctionResult may be ignored by the client (after decryption) if this is
  // set to true.
  bool is_chaff = 8;

  // The reporting urls registered during the execution of reportResult() and
  // reportWin().
  WinReportingUrls win_reporting_urls = 9;

  // Debugging URLs for the Buyer. This information is populated only in case of
  // component auctions.
  DebugReportUrls buyer_debug_report_urls = 10;

  // Debugging URLs for the Seller. This information is populated only in case
  // of component auctions.
  DebugReportUrls seller_debug_report_urls = 11;

  // List of interest group indices that generated bids.
  message InterestGroupIndex {
    // List of indices of interest groups. These indices are derived from the
    // original ProtectedAudienceInput sent from the client.
    repeated int32 index = 1;
  }

  // Map from the buyer participating origin (that participated in the auction)
  // to interest group indices.
  map<string, InterestGroupIndex> bidding_groups = 12;

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
  Error error = 13;
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
    // Required.
    // Contextual signals that include information about the context
    // (e.g. Category blocks Publisher has chosen and so on). This is passed by
    // untrusted Seller service to SellerFrontEnd service.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "sellerSignals".
    // The serialized string can be deserialized to a JSON object.
    string seller_signals = 1;

    // Required.
    // Contextual signals that are passed by untrusted Seller service to
    // SellerFrontEnd service.
    // Information about auction (ad format, size). This information
    // is available both to the seller and all buyers participating in
    // auction.
    // This is passed to ScoreAd() in AuctionConfig JSON object, the key in JSON
    // being "auctionSignals".
    // The serialized string can be deserialized to a JSON object.
    string auction_signals = 2;

    // Required.
    // List of buyers participating in FLEDGE auctions.
    // Buyers are identified by buyer domain (i.e. Interest Group owner).
    repeated string buyer_list = 3;

    // Required.
    // Seller origin / domain.
    string seller = 4;

    // Per buyer configuration.
    message PerBuyerConfig {
      // Required.
      // Contextual signals corresponding to each Buyer in auction that could
      // help in generating bids.
      string buyer_signals = 1;

      // Optional.
      // Timeout is milliseconds and applies to total time to complete GetBids.
      // If no timeout is specified, the Seller's default maximum Buyer timeout
      // configured in SellerFrontEnd service configuration, will apply.
      int32 buyer_timeout = 2;

      // Optional.
      // The Id is specified by the buyer to support coordinated experiments
      // with the buyer's Key/Value services.
      int32 buyer_kv_experiment_group_id = 3;

      // Optional.
      // Version of buyer's GenerateBid() code.
      // The string must be an object name belonging to the
      // Cloud Storage bucket specified at Bidding service startup.
      // If a version is not specified, the default version
      // (specified in the service startup config) will be used.
      int32 generate_bid_code_version = 4;

      // The domain of the Buyer. This is used as key for the map of
      // event and win reporting urls returned in SelectAdResponse.
      string buyer_origin = 5;
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
    bool enable_event_level_debug_reporting = 7;
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

  // A signal used as input to win reporting url generation for the Buyer.
  string modeling_signals = 9;
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
        string android_signals = 6;

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
    bool enable_event_level_debug_reporting = 5;
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
      string modeling_signals = 10;

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
    bool enable_event_level_debug_reporting = 8;
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
