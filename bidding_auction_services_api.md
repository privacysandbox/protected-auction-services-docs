**Authors:** <br>
[Priyanka Chatterjee][26], Google Privacy Sandbox (Emeritus) <br> 
Itay Sharfi, Google Privacy Sandbox (Emeritus)

# Bidding and Auction services

[The Privacy Sandbox][4] aims to develop technologies that enable more private
advertising on the web and mobile devices. Today, real-time bidding and ad
auctions are executed on servers that may not provide technical guarantees of
security. Some users have concerns about how their data is handled to generate
relevant ads and in how that data is shared.

Protected Auctions API provides ways to preserve privacy and limit third-party
data sharing by serving personalized ads based on previous mobile app or web
engagement. Protected Auctions include:
  * Protected Audience ([Chrome][5], [Android][7]) auctions for web browser and app
  * [Protected App Signals][142] for Android app

The Bidding and Auction Services (B&A) proposal outlines a way to allow Protected
Auction computation to take place on cloud servers in a trusted execution
environment, rather than running locally on a user's device. Running workloads
in a [Trusted Execution Environment (TEE)][29] in cloud have the following benefits:

  * Scalable ad auctions.
    * A scalable ad auction may include several buyers and sellers and that can
      demand more compute resources and network bandwidth.

  * Lower latency of ad auctions.
    * Server to server communication on the cloud is faster than multiple
      device to server calls. 
    * Adtech code can execute faster on servers with higher computing power.

  * Higher utility of ad auctions.
    * Servers have better processing power, therefore adtechs can run compute
      intensive workloads on a server for better utility.
    * Lower latency of ad auctions also positively impact utility. 

  * Security protection
    * [trusted execution environment][29] can protect confidentiality of adtech
      code and signals.

  * System health of the user's device.
    * Ensure better system health of user's device by freeing up computational
      cycles and network bandwidth.

This document focuses on [timeline and roadmap][127], [high level design][130],
[API][131] for Bidding and Auction services.


## Useful information

### Client integration
Clients imply web browsers and Android platforms. In this context:
  * Web browsers imply browsers on desktop and Android devices.
  * Android implies Android apps.

_Chrome and Android announced to integrate with Bidding and Auction services.
See [blog][27]._

#### Browser
Refer to [browser API for Bidding and Auction services][54].

In case of browser, B&A request and response payload are [Concise Binary Object Representation (CBOR)][122]
encoded. Following are the web schemas used by browsers.
* [auction_request.json][123] is the web schema corresponding to [ProtectedAudienceInput][9] message (B&A request). 
  * [interest_group.json][124] is the web schema for [BuyerInput.InterestGroup][82] message.
* [auction_response.json][125] is the web schema corresponding to [AuctionResult][84] message (B&A response).

##### Near drop-in replacement

Bidding and Auction services integrate into [Protected Audience API for browsers][28] and 
can scale as a near drop-in replacement for adtechs who already adopted Protected Audience
API for on-device execution and are interested in exploring a server side solution.

  * [Interest Groups (Custom Audience) creation and management][158] can stay the same.
  * User defined functions (UDF) developed by adtechs:
    * The function signatures of UDFs for bid generation, scoring and reporting can
      stay exactly the same.
    * Buyer's code for [generateBid][69]() would mostly work. However, certain 
      updates will be required for [payload optimization][51], see [here][159] for
      more details.
  * Key-value services:
    * Seller key-value service do not require additional updates.
    * Buyer key-value service require some additional updates. The trusted bidding signals
      may need to include more information to support [payload optimization][51], see
      [here][159] for more details.
  * Seller integration:
    * Seller's code on client would require additional changes to call an API (provided
      by client) to fetch encrypted B&A request payload (`B&A request ciphertext`) and
      include that in ad request to seller's ad service.
    * Seller's ad service would require additional changes to call B&A.
    * Seller's ad service would require additional changes to handle encrypted B&A response
      (`B&A response ciphertext`) and send that back to the client.
    * Seller's ad service would require additional changes to call an API (provided by
      client) to decrypt `B&A response ciphertext`.

#### Android app
  * Protected Audience: Refer to [Android's integration][7] document.
  * Protected App Signals: Refer to [Protected App Signals][152] document.

In case of Android app, the B&A request and response payload are based on protobufs.

#### Privacy Preserving Ads
[Ad Selection API][8] is a proposal by Microsoft Edge browser and similar to Protected
Audience. The proposal supports [server-side bidding and auction][141] in [trusted execution environment][29].


### Related documents
Following explainers are recommended to adtechs onboarding to B&A. You may see
also see the [full list][147] of documents related to Protected Auctions and service
trust model.

  * [Protected Audience auctions mixed mode][143]:
    Mixing of on-device and B&A auctions together is referred as mixed mode.
    This explainer is a highly recommended read for sellers ([SSPs][148]).

  * [Bidding and Auction services self-serve guide][144]:
    The guide for adtech onboarding, coordinator enrollment, user-defined-function specifications,
    cloud deployment, testing and scalability guidance.
    **Adtechs who wants to onboard to B&A, must refer to the self-serve guide.**

  * [Payload optimization][51]:
    Buyers ([DSPs][149]) onboarding to B&A must refer to this explainer for payload
    optimization strategies. **Payload optimization is a requirement for buyers using B&A**. 
    This is also recommended to the sellers ([SSPs][148]) as an optimization for
    configuring [per buyer payload][150] limits.

  * Adtech code execution
    * [Roma Bring Your Own Binary (BYOB)][151]: Recommended for buyers ([DSPs][149])
      who may want to use BYOB for generateBid() execution.
      _Note: We will publish the guidance for incorporating BYOB for generateBid()._

    * [Javascript and WASM based execution in Roma][153]: Adtechs may refer to the system
      design of code fetch and code execution engine. 

  * [Multi seller auctions design][55]:
    Sellers ([SSPs][148]) may refer to this explainer to understand multi seller
    auctions design with B&A.

  * Reporting
    * [Event level reporting design][135]: Adtechs may refer to this explainer to
      understand event level reporting url generation with B&A.

  * K-Anonymity Integration
    * [B&A and K-Anonymity integration][168]

  * [Chaffing][169]
    
  * Monitoring and TEE debugging
    * [Monitoring design][145]: Describes the monitoring infra design and
      [list of metrics][139] available for adtechs.

    * [Adtech consented debugging design][146]: Decribes design and provides guidance
      to adtechs for using consented debugging.

  * Cost guidance
    * [B&A cost guidance][154]: Provides an overview of cost breakdown of the
      system and cost estimation guidance.

    * [Cost guidance for Protected App Signals][155] 

### Github discussion

Please file an issue in [Github/WICG/protected-auction-services-discussion][156]
for feedback and discussion.

### Code repository

Bidding and Auction services code is available in [Github](https://github.com/privacysandbox/bidding-auction-servers)
and a new code version is released every few weeks.
 * Refer to [releases page][157] for the available code versions, prod and non-prod image hashes
   of the services corresponding to each released version.
 * The prod images are approved by the Coordinators and must be used in production environment. 
 * The non-prod images do not require Coordinator approval and must be used in
   B&A services that enables TEST_MODE and disables attestation. 

Adtechs will have to deploy the binaries and configurations to a supported
public cloud platform.

### Supported public cloud platforms

Bidding and Auction services will be available within the [Trusted Execution Environment][29](TEE)
on AWS and GCP 2023 onwards. 

More cloud platforms may be supported eventually. See the [Public Cloud TEE requirements explainer][138]
for more details.

_Note: SSP and DSP can operate Bidding and Auction services on different cloud
platforms that are supported. For multi seller auctions, SSPs can operate on
different cloud platforms._

Bidding and Auction services will not support a “Bring Your Own Server” model,
similar to what is made available to Protected Audience’s Key/Value server.
**Bidding and Auction services can only be deployed within approved TEE cloud
environments.** This is valid for Alpha, Beta, Scale testing programs and beyond.

#### AWS support
Bidding and Auction services runs in [Nitro Enclaves][30] on AWS. Refer
[here][52] for more details.

#### GCP support
Bidding and Auction services runs in [Confidential Space][31]
([Confidential Computing][32]) on GCP. Refer [here][53] for more details.

#### Azure support
Subject to additional diligence and coordination with Microsoft, we expect to reach the Alpha milestone for B&A on 
[Azure][167] in September 2025, and the Beta milestone 6 months after ad techs test the Alpha version and provide feedback.

### Types of ad auctions

Bidding and Auction services supports single-seller and [Multi seller auctions][55]
for web and app traffic.


## Timeline and roadmap

The following sections include the timelines for ad techs interested in testing
Bidding and Auction services for Protected Auctions. Protected Auctions refer to 
[Protected Audiences](https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md)
and [Protected App Signals](https://developer.android.com/design-for-safety/privacy-sandbox/protected-app-signals#timeline)
ad targeting products.


#### **Timelines**

The following timelines show the availability of Bidding and Auction services. See [Launch and testing phases of Protected Auction services in TEE](/protected_auction_services_launch_testing_phases.md) for details.


<table>
  <tr>
   <td style="background-color: #efefef">API
   </td>
   <td style="background-color: #efefef"><strong>Alpha testing </strong>
   </td>
   <td style="background-color: #efefef"><strong>Beta testing </strong>
   </td>
   <td style="background-color: #efefef"><strong>Scale testing </strong>
   </td>
  </tr>
  <tr>
   <td>Protected Audience 
<p>
(web browser on desktop and Android)
   </td>
   <td>July 2023 onwards
<p>
Available
   </td>
   <td><ul>

<li>Beta 1: Nov 2023 onwards
<li>Beta 2: April 2024 onwards
<strong><em>Production experiment ramp:</em></strong><ul>

<li><strong><em>Chrome browser enabled B&A APIs for 1% stable user traffic. </em></strong></li></ul>
</li></ul>

   </td>
   <td>Jan 2025
<p>
<strong><em>Production experiment ramp:</em></strong><ul>

<li><strong><em>Chrome browser will enable B&A APIs for 100% stable user traffic. </em></strong></li></ul>

   </td>
  </tr>
  <tr>
   <td>Protected Audience (Android)
   </td>
   <td>Available
   </td>
   <td>April 2024 onwards
   </td>
   <td>Jan 2025
   </td>
  </tr>
  <tr>
   <td>Protected App Signals (Android)
   </td>
   <td>Available
   </td>
   <td>September 2024 onwards
   </td>
   <td>Jan 2025 
   </td>
  </tr>
</table>



#### **Roadmap**

_Note: The common server side features are common to all types of Protected Auctions on web and Android apps._


<table>
  <tr>
   <td>Timeline
   </td>
   <td>Common server side features
   </td>
   <td>Protected Audience for web browser on desktop and Android 
   </td>
   <td>Protected Audience and Protected App Signals (PAS) on Android Apps
   </td>
  </tr>
  <tr>
   <td>July 2023
   </td>
   <td><ul>

<li>Privacy and security protections: <ul>
 <li>Bidding and Auction services running in a <a href="https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trusted-execution-environment">trusted execution environment</a>.
 <li>Encryption of request or response payload between client and server and TEE based servers.
 <li>Padding of request or response payload between client and server.</li></ul>
  
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/trusted_services_overview.md">Server trust model</a>: <ul>
 <li>Bidding and Auction services and <a href="https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#key-management-systems">key management systems</a> integration.</li></ul>
 
<li>Adtech UDF execution support <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#adtech-code-execution-engine">Ad tech code execution in a sandbox</a>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#code-blob-fetch-and-code-version">Ad tech code blob fetch</a> from ad tech provided endpoints.
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#code-blob-fetch-and-code-version">Ad tech code blob fetch</a> from Cloud Storage buckets.
 <li>Javascript and WASM support for Protected Audience.</ul>
 
<li>Public cloud support: <ul>
 <li><strong>AWS support</strong> for Bidding and Auction services.
 <li><strong>GCP support</strong> for Bidding and Auction services.
 <li>Cloud regional support.</ul>
 
<li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding-auction-services-payload-optimization.md">Payload optimization</a>
 
<li>Reporting: <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_event_level_reporting.md">Event level reporting</a>: Generation of event level reporting URLs and registered beacons for Fenced Frame reporting in Bidding and Auction services.</li></ul>
 
<li><a href="https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/debug">Local server debugging</a>: <ul>
 <li>A debug binary build of servers will be available that can provide access to TEE server logs of different verbosity levels. For Google Cloud Platform, these logs will be exported to <a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_gcp_guide.md#cloud-logging">Cloud Logging</a>.</li></ul>
 
<li>Testing tools: <ul>
 <li><a href="https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/secure_invoke">Secure invoke</a>: Encrypted payload generation for e2e functional testing
 <li><a href="https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/load_testing">Load testing</a> </li> </ul>


</td>
<td><ul>

<li>Data format: <ul>
 <li><a href="https://cbor.io/">Concise Binary Object Representation (CBOR)</a> encoded request / response payload to support the <a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#web-platform-schemas">web platform</a>.</li></ul>
 
<li>Component ads
<li>Multi slot ads</li></ul>
</li></ul>

   </td>
   <td><ul>

<li>Data format for <a href="https://developer.android.com/design-for-safety/privacy-sandbox/protected-audience-bidding-and-auction-integration">Android</a> APIs: <ul>

 <li><a href="https://protobuf.dev/">Protobufs</a></li> </ul>
</li> </ul>

   </td>
  </tr>
  <tr>
   <td>November 2023
   </td>
   <td><ul>

<li>Public cloud support: <ul>
 <li>Cross cloud communication between SSP and DSPs.</li></ul>
  
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/trusted_services_overview.md">Server trust model</a>: <ul>
 <li>Coordinator integration on GCP</li></ul>
 
<li>Productionisation of servers; refer<a href="https://github.com/privacysandbox/fledge-docs#server-productionization"> to our Github</a> for up-to-date information. <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/monitoring_protected_audience_api_services.md">Monitoring support.</a></li> </ul>

   </td>
   <td><ul>

<li>Debugging: <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md">Ad tech / user consented debugging</a> for web traffic</li> </ul>

   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>January 2024
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><ul>

 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a><ul>
 <li>Client and Bidding and Auction integration to support Protected App Signals.
 <li>Combined PAS and PA support. 
 <li>Reporting: Event-level <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md#reportwin-udf">win reporting</a>.  
 <li>Cloud support: GCP support.</li> </ul>

   </td>
  </tr>
  <tr>
   <td>March 2024
   </td>
   <td><ul>

<li>Experiment group Id support
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_bid_currency.md">Multi currency support (bid currency)</a>
 
<li>Reporting: <ul>
 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_event_level_reporting.md">Noising for event level reporting</a> for Protected Audience</li></ul>
 
<li>Multiseller auctions for Protected Audience on web and Android: <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#server-orchestrated-component-auction">Server orchestrated Component Auctions</a>
 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_multiseller_event_level_reporting.md#server-orchestrated-component-auctions-1">Event level reporting for server orchestrated Component Auctions</a></li></ul>
 
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/trusted_services_overview.md">Server trust model</a>: <ul>
 <li>Coordinator integration on AWS.</li> </ul>

   </td>
   <td><ul>

<li>Multi seller auctions: <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#device-orchestrated-component-auctions">Device orchestrated Component Auctions</a>
 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_multiseller_event_level_reporting.md#device-orchestrated-component-auctions-1">Event level reporting for device orchestrated Component Auctions</a></li></ul>

<li>Multi cloud support on browser.  <ul>
 <li>This implies the browser can fetch public (encryption) keys for more than one cloud platform to encrypt the `ProtectedAuctionInput` for Bidding and Auction.</li></ul>
 
<li>Security protections: <ul>
 <li>OHTTP Encapsulation Media Type</li> </ul>

   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>April 2024
   </td>
   <td>
   </td>
   <td><ul>

<li>Reporting: <ul>
 <li>BuyerReportingId</li></ul>
 
<li>Reporting and debugging: <ul>
 <li>forDebugOnly (debug reporting) for single seller auctions.</li> </ul>

   </td>
   <td><ul>

<li><a href="https://developer.android.com/design-for-safety/privacy-sandbox/protected-audience-bidding-and-auction-integration">Protected Audience</a>: <ul>
 <li>Custom audience (interest group) origin support</li></ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>
 <li>B&A and <a href="https://github.com/privacysandbox/fledge-key-value-service/blob/main/docs/ad_retrieval_overview.md">Ad Retrieval service</a> integration:  <ul>
  <li>TEE to TEE communication.
  <li>Support for ad retrieval through contextual path.  </li></ul>
  
 <li>Adtech UDF execution support:  <ul>
  <li>Javascript and Javascript with inlined WASM support.</li>  </ul>
</li>  </ul>

   </td>
  </tr>
  <tr>
   <td>July 2024
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><ul>

<li>Security protections: <ul>
 <li>OHTTP Encapsulation Media Type</li></ul>
  
<li>Multi cloud support on Android <ul>
 <li>Android devices can fetch public / encryption keys for more than one cloud platform to encrypt the ProtectedAuctionInput for B&A.</li></ul>
 
<li>Multiseller auction: <ul>
 <li>Waterfall mediation</li></ul>
  
<li>Debugging: <ul>
 <li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md">Adtech / user consented debugging</a></li></ul>
  
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>
 <li>Cloud support: AWS support.
 <li>B&A and <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference_overview.md">Inference</a> integration</li> </ul>

   </td>
  </tr>
  <tr>
   <td>August 2024
   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#fake-requests--chaffs-to-dsp">Chaffing</a></li></ul>

   </td>
   <td><ul>

<li>Prebid and Chrome integration
<li>Protected Audience auctions mixed mode: <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/protected_audience_auctions_mixed_mode.md#sequential-ba-auction-and-on-device-auction">Sequential on-device and server side auctions</a></li></ul>

   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>September 2024
   </td>
   <td>
   </td>
   <td>
   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>

 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals_egress.md">Egress</a> with limited validation support</li> </ul>
</li> </ul>

   </td>
  </tr>
  <tr>
   <td>October 2024
   </td>
   <td><ul>

<li>Adtech UDF execution support for Protected Audience <ul>
 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/roma_bring_your_own_binary.md">BYOB (Bring Your Own Binary)</a> for generateBid() execution in Bidding server</li> </ul>
</li> </ul>

   </td>
   <td>
   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>November 2024
   </td>
   <td><ul>

<li>Adtech UDF execution support <ul>
 <li>Multiple versions of adtech code blobs</li></ul>
 
<li>Data version header
</li></ul>

   </td>
   <td><ul>

<li>Advanced <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding-auction-services-payload-optimization.md">payload optimizations</a></li></ul>

   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>Jan 2025
   </td>
   <td><ul>

<li><a href="https://wicg.github.io/turtledove/#k-anonymity">K-Anonymity Integration</a> <ul>
 <li>Including K-Anon status in reporting 
 <li>K-Anon support for Private Aggregate reporting</li></ul>
 
<li><a href="https://github.com/WICG/turtledove/blob/main/FLEDGE.md#35-filtering-and-prioritizing-interest-groups">Priority vector</a><span style="text-decoration:underline;">:</span>  <ul>
 <li>This can help filter interest groups and reduce unnecessary executions in Bidding service to optimize latency and cost.</li></ul>
 
<li><a href="https://github.com/privacysandbox/protected-auction-key-value-service/blob/release-0.17/docs/protected_audience/integrating_with_fledge.md">TEE key / value service</a> integration for Protected Audience</li></ul>
</li></ul>
</li></ul>

   </td>
   <td><ul>

<li><a href="https://groups.google.com/a/chromium.org/g/blink-dev/c/eXJLbFAuSU8/m/WzCpcHaZAgAJ">Interest Group Updates triggered by trustedBiddingSignals</a>

<li>Reporting and debugging: <ul>

 <li><a href="https://github.com/WICG/turtledove/pull/1237/files">buyerAndSellerReportingId</a>
 <li><a href="https://github.com/WICG/turtledove/pull/1237/files">SelectableBuyerAndSellerReportingId</a>
 <li>forDebugOnly (debug reporting) for <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#device-orchestrated-component-auctions">device orchestrated component auctions</a>
 <li>Private Aggregation API for single seller auctions</li></ul>
 
<li><a href="https://github.com/WICG/turtledove/pull/1237/files">Deals support</a>
</li></ul>

   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a><ul>
 <li>Server Orchestrated component auctions.
 <li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals_egress.md">Egress</a> with full schema validation support</li> </ul>

</li> </ul>

   </td>
  </tr>
  <tr>
   <td>March 2025
   </td>
   <td>
   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#device-orchestrated-component-auctions-with-prebidjs-with-browser-api-optimizations">Optimized browser API for publisher bidding framework integration</a>
 
<li>Protected Audience auctions mixed mode: <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/protected_audience_auctions_mixed_mode.md#recommended-parallelization-of-on-device-and-server-side-auctions">Parallelization of on-device and server side auctions</a></li> </ul>


</li> </ul>

   </td>
  </tr>
  <tr>
   <td>April 2025
   </td>
   <td>
   </td>
   <td><ul>

<li><a href="https://wicg.github.io/turtledove/#negative-targeting-section">Negative targeting</a>
 
<li>Reporting and debugging : <ul>
 <li><a href="https://wicg.github.io/turtledove/#downsampling-header">Down sampled forDebugOnly (debug reporting</a>)</li> </ul>
</li> </ul>

   </td>
   <td>
   </td>
  </tr>
  <tr>
   <td>June 2025
   </td>
   <td><ul>

<li>Adtech UDF execution support: <ul>
 <li>Publisher / subscriber message queue - B&A integration: This will support emergency rollouts of adtech code stored in cloud storage. </li> </ul>

   </td>
   <td><ul>

<li><a href="https://github.com/WICG/turtledove/blob/main/FLEDGE.md#311-cross-origin-trusted-server-signals">Cross origin trusted server signals</a>
 
<li>Reporting and debugging: <ul>
 <li>Private Aggregation API for multiple seller auctions</li> </ul>
</li> </ul>

   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>

 <li>Metrics for AWS and GCP</li> </ul>
</li> </ul>

   </td>
  </tr>

  <tr>
   <td>September 2025
   </td>
   <td>
    <ul>
      <li>Azure Alpha support for Bidding and Auction Services</li>
    </ul>
   </td>
   <td>
   </td>
   <td><ul>

<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>

 <li>Noising for event level <a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals_egress.md">egress</a>
  </li> </ul>
</li> </ul>

   </td>
  </tr>


  <tr>
   <td>October 2025 and beyond
   </td>
   <td><ul>

<li>Adtech UDF execution support: <ul>
 <li>Code blob signing and verification</li></ul>
 
<li>Parallelization of contextual and Bidding and Auction auctions
</li>

</ul>

**Further details to be added in future updates**.</li></ul>

   </td>
   <td><ul>

<li>Optimizations related to multi slot ads
<li>Support for ad size
<li>Reporting and debugging: <ul>
 <li>forDebugOnly (debug reporting) for server orchestrated component auctions</li> </ul>
</li> </ul>

   </td>
   <td><ul>

<li>Waterfall Mediation optimization with server side truncation</li>
<li><a href="https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md">Protected App Signals</a>: <ul>

 <li>Protected App Signals and Protected Audience isolation support in Bidding service on supported cloud platforms.</li> </ul>
</li> </ul>

   </td>
  </tr>
</table>


## Onboarding and alpha testing guide

Following is the guide for onboarding to Bidding and Auction services and
participating in Alpha testing.
 
### Guidance to sellers / SSPs:
  * Refer to [Spec for SSP][88] section.
  * Develop [ScoreAd][67]() for Protected Audience auction.
  * Develop [ReportResult][75]() for event level reporting.
  * Setup [Seller's Key/Value service][68].
  * Add support such that seller's code on publisher web page calls
    [browser API][54] to fetch encrypted [ProtectedAudienceInput][66]. Then
    includes encrypted ProtectedAudienceInput in the request to seller's ad
    server.
  * Add support in [Seller's ad server][20] to send [SelectAd][35] request to
    Bidding and Auction services for Protected Audience auctions.
  * Add support for [forwarding client metadata][90] in the request to Bidding
    and Auction services (SellerFrontEnd).
  * Review [Logging][93] section.
  * Bidding and Auction services code and configurations is open sourced to
    [Github repo][59].
     * Refer to the [README][104] for build / packaging information.
     * Refer to the [README for deployment on AWS][106] or [README for deployment on GCP][105]. 
     * Refer to [example config on AWS][101] or [example config on GCP][103] for the
       Terraform config required for cloud deployment. The config requires update of some parameter
       values (that vary per adtech) before deployment to cloud.
  * Deploy [SellerFrontEnd][21] and [Auction][23] server instances to your
    preferred [cloud platform that is supported][98].
  * Set up experiments for ad auctions and target user opt-in traffic. Include
    one or more partner buyers in the same experiment.
  * [Chrome browser][54] supports a flag `FledgeBiddingAndAuctionKeyURL`. Users' browsers that
    enable the flag can be targeted.
    * The flag would point to the public key service endpoint in [key management systems][10]. This
      is required to fetch public keys to encrypt [ProtectedAudienceInput][66]. 
  * [Enroll with coordinators][85] and / or run servers in `TEST_MODE`.

    During onboarding, it is recommended to start with Option 1 and then switchover to Option 2.
    However, getting started with Option 2 is also feasible. 

     * Option 1: Run Bidding and Auction services in `TEST_MODE`.
       * `TEST_MODE` supports [cryptographic protection](#client--server-and-server--server-communication)
         with hardcoded public-private key pairs, while disabling TEE server attestation.
         During initial phases of onboarding, this would allow adtechs test Bidding and Auction server workloads
         even before integration with Coordinators.
       * Set `TEST_MODE` flag to `false` in seller's Bidding and Auction server configurations ([AWS][101], [GCP][103]).
       * The following options are available for testing the auction flow with Bidding and Auction services.
         * Option A: Using [secure invoke][136] tool.
           * The tool uses hardcoded public keys to encrypt payloads and then sends requests to TEE based Bidding and Auction services.
             The corresponding private keys of the same version are hardcoded / configured in Bidding and Auction services such that
             the encrypted payloads can be correctly decrypted.
           * The tool can generate [SelectAdRequest][35] payload for communication with TEE based SellerFrontEnd if a plaintext
             request payload is supplied. The payload will include [ProtectedAudienceInput][66] ciphertext that is encrypted with
             hardcoded public keys.
           * The tool also has the capability to decrypt the response received from Bidding and Auction services and print out the
             out human readable plaintext response.
           * Refer to the [README][137] for more information about the tool.

         * Option B: End to end flow from Chrome browser.
           * With this option, the `FledgeBiddingAndAuctionKeyURL` flag should be set to the following endpoint. The
             endpoint would serve a public key such that corresponding private keys of the same version are known to
             the Bidding and Auction services.

              ```FledgeBiddingAndAuctionKeyURL/http%3A%2F%2Flocalhost%3A8000%2Fkey```

             The above endpoint can be configured to serve a public key with similar to the following.
             
             ```$ mkdir -p /tmp/bna && cd /tmp/bna && echo '{ "keys": [{ "id": "40", "key": "87ey8XZPXAd+/+ytKv2GFUWW5j9zdepSJ2G4gebDwyM="}]}' > key && python3 -m http.server 8000```

     * Option 2: [Enroll with coordinators][85].
       * During Alpha, Google Privacy Sandbox Engineers will act as Coordinators and operate the
         [key management systems][10]. Reach out to your Privacy Sandbox partner to get enrolled with
         Alpha Coordinators.
       * Integration of Bidding and Auction server workloads with Alpha Coordinators would enable TEE server attestation
         and allow fetching live encryption / decryption keys from [public or private key service endpoints][10] in Bidding
         and Auction services.
       * The flag supported by Chrome browser `FledgeBiddingAndAuctionKeyURL` should point to the
         public key service endpoint of [key management systems][10] run by Alpha Coordinators. This would allow the
         browser to fetch live public keys for encryption of [ProtectedAudienceInput][66] payload.

### Guidance to buyers / DSPs:
  * Refer to [Spec for DSP][89] section.
  * Develop [GenerateBid][69]() for bidding.
  * Develop [ReportWin][76]() for event level reporting.
  * Setup [Buyer's Key/Value service][70].
    * If your Key/Value server supports filtering of interest groups, refer to
      this [section][91] and [metadata forwarding][90].
  * [Optimise payload][51].
  * Review [Logging][93] section.
  * Bidding and Auction services code and configurations is open sourced to
    [Github repo][59].
    * Refer to the [README][104] for build / packaging information.
    * Refer to the [README for deployment on AWS][106] or [README for deployment on GCP][105]. 
    * Refer to [example config on AWS][100] or [example config on GCP][102] for the Terraform config
      required for cloud deployment. The config requires update of some parameter values (that vary
      per adtech) before deployment to cloud.
  * Deploy [BuyerFrontEnd][22] and [Bidding][42] server instances to your
    preferred [cloud platform that is supported][98].
  * [Enroll with coordinators][85] or run servers in `TEST_MODE`.

    During onboarding, it is recommended to start with Option 1 and then switchover to Option 2.
    However, getting started with Option 2 is also feasible. 

     * Option 1: Run Bidding and Auction services in `TEST_MODE`.
       * `TEST_MODE` supports [cryptographic protection](#client--server-and-server--server-communication)
         with hardcoded public-private key pairs, while disabling TEE server attestation.
         During initial phases of onboarding, this would allow adtechs test Bidding and Auction server workloads
         even before integration with Coordinators.
       * Set `TEST_MODE` flag to `false` in buyer's Bidding and Auction server configurations ([AWS][100], [GCP][102]).
       * **The following tool can facilitate testing buyer's Bidding and Auction services ([BuyerFrontEnd][22], [Bidding][42])
         and bid generation flow independently.**
         * [Secure invoke][136] tool.
           * The tool uses hardcoded public keys to encrypt payloads and then sends requests to TEE based Bidding and Auction services.
             The corresponding private keys of the same version are hardcoded / configured in Bidding and Auction services such that
             the encrypted payloads can be correctly decrypted.
           * The tool can generate [GetBidsRequest][36] payload for communication with TEE based BuyerFrontEnd if a plaintext
             request payload is supplied. 
           * The tool also has the capability to decrypt the response received from TEE based BuyerFrontEnd and print out the
             out human readable plaintext response.
           * Refer to the [README][137] for more information about the tool.

     * Option 2: [Enroll with coordinators][85].
       * During Alpha, Google Privacy Sandbox Engineers will act as Coordinators and operate the
         [key management systems][10]. Reach out to your Privacy Sandbox partner to get enrolled with
         Alpha Coordinators.
       * Integration of Bidding and Auction server workloads with Alpha Coordinators would enable TEE server attestation
         and allow fetching live encryption / decryption keys from [public or private key service endpoints][10] in Bidding
         and Auction services.
  * Reach out to partner SSPs to include in experiments for Protected Audience auctions.
    * _Note: Buyers can also independently start integrating and testing the bidding flow before they are included in a
      seller supported ad auction experiments._

### Enroll with coordinators

Adtechs would have to enroll with two Coordinators running [key management systems][10] that
provision keys to Bidding and Auction services after server attestion. 

Adtechs should only enroll with the Coordinators for the specific cloud platform where
they plan to run Bidding and Auction services. 

#### Enrollment with AWS coordinators

An adtech should provide their **AWS Account Id** to both the Coordinators.

The Coordinators would create IAM roles. After adtechs provide the AWS account Id, they would
attach that information to the IAM roles and include in an allowlist. Then the Coordinators would 
let adtechs know about the IAM roles and that should be included in the B&A server Terraform
configs that fetch cryptographic keys from [key management systems][10]. 

Following config parameters in [buyer][100] or [seller][101] server configs would include the IAM
roles information provided by the Coordinators.
 * PRIMARY_COORDINATOR_ACCOUNT_IDENTITY
 * SECONDARY_COORDINATOR_ACCOUNT_IDENTITY

#### Enrollment with GCP coordinators

An adtech should provide [**IAM service account email**][107] to both the Coordinators.

The Coordinators would create IAM roles. After adtechs provide their service account email, the Coordinators
would attach that information to the IAM roles and include in an allowlist. Then the Coordinators would let 
adtechs know about the IAM roles and that should be included in the B&A server Terraform configs that 
fetch cryptographic keys from [key management systems][10]. 

Following config parameters in [buyer][102] or [seller][103] server configs would include the IAM roles
information provided by the Coordinators.
  * PRIMARY_COORDINATOR_ACCOUNT_IDENTITY
  * SECONDARY_COORDINATOR_ACCOUNT_IDENTITY

## Specifications for adtechs

### Near drop-in replacement

Bidding and Auction services integrate into [Protected Audience API for browsers][28] and 
can scale as a near drop-in replacement for adtechs who already adopted Protected Audience
API and are interested in exploring a server side solution.

  * Interest Groups (Custom Audience) creation and management can stay the same.
    * For [payload optimization][51], some additional fields will be supported
      in Interest Group.

  * Key-value services can stay nearly the same.
    * Seller's key-value service can stay the same.
    * Buyer's key-value service instances can stay the same; however to support
      [payload optimization][51], `trusted_bidding_signals` may need to include
      additional data.
      
  *  Code developed by adtechs following the guidance in [Protected Audience API for browsers][28]
     will mostly work with Bidding and Auction services. **The function signatures
     can stay exactly the same.**
    * Seller's code for [ScoreAd][67]() can stay the same.
    * Seller's code for [ReportResult][75]() can stay the same.
    * Buyer's code for [GenerateBid][69]() would mostly work. However, certain 
      updates will be required for [payload optimization][51].
    * Buyer's code for [ReportWin][76]() can stay the same.

### Spec for SSP

#### scoreAd()

The [Auction service][23] exposes an API endpoint ScoreAds. The [SellerFrontEnd service][21] sends a
ScoreAdsRequest to the Auction service for running an auction. ScoreAdsRequest includes bids from each buyer
and other required signals. The code for auction, i.e. `ScoreAd()` is prefetched from Cloud Storage, cached
and precompiled in Auction service. After all ads are scored, the Auction service picks the highest scored ad
candidate and returns the score and other related data for the winning ad in ScoreAdsResponse.

_Note: If an SSP develops `scoreAd()` following web platform's [Protected Audience API explainer][37], 
that should also work as-is for execution in the Auction service._

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
* `trustedScoringSignals`: trustedScoringSignals fetched from seller's Key/Value service
    * Note: Only the signals required for scoring the ad / bid is passed to `scoreAd()`.
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

#### Seller BYOS Key/Value service

_Note: BYOS Key/Value is only supported for Chrome. Protected Auctions using data from Android devices
are required to use the [Protected Auction Key/Value service][166]._

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

The `trustedScoringSignals` passed to `scoreAd()` is as follows:

```
{ 
  'renderUrl': {'https://cdn.com/render_url_of_bidder': arbitrary_value_from_signals},
  'adComponentRenderUrls': {
      'https://cdn.com/ad_component_of_a_bid': arbitrary_value_from_signals,
      'https://cdn.com/another_ad_component_of_a_bid': arbitrary_value_from_signals,
      ...}
}
```

#### reportResult()

Event level win reporting would work with Bidding and Auction services and function signatures
can be the same as described in web platform's [Protected Audience API explainer][43]. 

The reporting urls for the seller and registered ad beacons (for Fenced Frame reporting) would be
generated in Auction service and returned to the client in encrypted [AuctionResult][84]. The client
will ping the seller's reporting endpoint using the reporting url.

_Note: Refer to the [event level reporting explainer][135] for the detailed design._

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

Server configurations are based on [Terraform][16] and is open sourced to [Github repo][59]
for cloud deployment. 

The configurations include environment variables and parameters that may vary per seller.
These can be set by the seller in the configuration before deployment. The configurations also 
include urls that can be ingested when the service starts up for prewarming the connections.  

Refer to the [README for deployment on AWS][106] or [README for deployment on GCP][105]. Refer to
[example config on AWS][101] or [example config on GCP][103] for the Terraform config required
for deployment to the cloud. The config requires update of some parameter values (that vary
per adtech) before deployment to cloud.

Following are some examples of data configured in service configurations.

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
       
* _Map of {InterestGroupOwner, BuyerFrontEnd endpoint}_: Map of InterestGroupOwner (buyer origin) to
  BuyerFrontEnd domain address.
  
* _Global timeout for Buyer_: This information can be used to set a timeout on each buyer; however, will be
  overridden by the `buyer_timeout_ms` passed by the seller's ad service to SellerFrontEnd in SelectAd request.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [key management systems][10].

##### Auction service configurations

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where seller's code is hot reloaded by the
  Auction service. 
  
* Private Key Hosting service and Public Key Hosting service endpoints in [key management systems][10].

### Spec for DSP

#### generateBid()

The [Bidding service][42] exposes an API endpoint GenerateBids. The [BuyerFrontEnd service][22] sends
GenerateBidsRequest to the Bidding service, that includes required input for bidding. The code
for bidding, i.e. `generateBid()` is prefetched from Cloud Storage, cached and precompiled in Bidding service.
After processing the request, the Bidding service returns the GenerateBidsResponse which includes 
bids that correspond to each ad, i.e. [AdWithBid][49]. 

The function can be implemented in Javascript (or WASM driven by Javascript) or compiled into a [standalone binary][162]. The specifcation for both is described in detail below.

##### generateBid() Javascript/WASM spec 

_Note: If a DSP develops `generateBid()` following web platform's [Protected Audience API explainer][41], 
that should also execute in Bidding service. However, certain updates will be required for [payload optimization][51]._

```
generateBid(interestGroup, auctionSignals, perBuyerSignals, trustedBiddingSignals,  deviceSignals) {
  ...
  return {'ad': adObject,
          'bid': bidValue,
          'render': renderUrl,
          'adComponents': ["adComponentRenderUrlOne", "adComponentRenderUrlTwo"],
          'allowComponentAuction': false};
 } 
```

###### Arguments

* `interestGroup`: The InterestGroup (Custom Audience) object. Refer InterestGroup data structure to
  understand what is sent in this object from the client.
    * The InterestGroup is serialized and passed to generateBid() exactly as-sent, _except_ for the following divergences:
      ** `DeviceSignals` are serialized and passed separately, see below.
      ** `component_ads` are serialized in a field named `adComponentRenderIds`
      ** `bidding_signals_keys` are serialized in a field named `trustedBiddingSignalsKeys` to align with the On-Device interestGroup spec.
    * _Note: To reduce payload over the network and further optimize latency, our goal is to minimize
      the information sent in this object. We will work with Adtechs for the long term to reduce the
      amount of information sent in this object and try to find a solution to fetch those on the server
      side._

* `auctionSignals`: Contextual signal that is passed from seller's ad service to SellerFrontEnd in SelectAd
  request.

* `perBuyerSignals`: Contextual signal generated by the buyer during Real Time Bidding that is passed
  from seller's ad service to SellerFrontEnd in SelectAd request. 

* `trustedBiddingSignals`: Real time signals fetched by BuyerFrontEnd service from Buyer's Key/Value
  service. 
  * _Note: Only the `trustedBiddingSignals` required for generating bid(s) for the `interestGroup` are
    passed to `generateBid()`_.

* `deviceSignals`: This refers to `browserSignals` or `androidSignals`,  built by the client (browser,
   Android). This includes Frequency Cap (statistics related to previous win of ads) for the user's device.

##### generateBid() BYOB (Bring Your Own Binary) spec 
The signature for the GenerateBid binary is specified as proto objects. That means, the input to the GenerateBid binary will be a proto object and the output will be a proto object. The function signature looks like this - 

```
GenerateProtectedAudienceBidResponse generateBid(GenerateProtectedAudienceBidRequest);
```

The definition for these high level protos along with nested types is specified in the [API code][160]. This is different from the Javascript signature - the parameters and return values are encapsulated in high level proto objects. These differences are discussed as follows.

###### Arguments
* `GenerateProtectedAudienceBidRequest`: This is the request object that encapsulates all the arguments for the generateBid() UDF (similar to the parameters in the JS spec for generateBid like interestGroup, deviceSignals, etc.). 
```
message GenerateProtectedAudienceBidRequest {
  ProtectedAudienceInterestGroup interest_group = 1 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'This will be prepared by the Bidding service based on the data received'
      ' in the BuyerInput from the device.'
}];

  string auction_signals = 2 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'Auction signals are sent by the seller in the Auction Config. This can'
      ' be encoded any way by the seller and will be passed as-is to the'
      ' generateBid() UDF.'
}];

  string per_buyer_signals = 3 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'Per buyer signals are sent by the seller in the Auction Config. This can'
      ' be encoded any way by the seller and will be passed as-is to the'
      ' generateBid() UDF.'
}];

  string trusted_bidding_signals = 4 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'This will be passed as the JSON response received from the buyer\'s'
      ' key/value server.'
}];

  oneof ProtectedAudienceDeviceSignals {
    ProtectedAudienceAndroidSignals android_signals = 5 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
        'This will be prepared by the Bidding server based on information'
        ' passed by the Android app.'
}];

    ProtectedAudienceBrowserSignals browser_signals = 6 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
        'This will be prepared by the Bidding server based on information'
        ' passed by the browser on desktop or Android.'
}];

    ServerMetadata server_metadata = 7 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
        'This will be prepared by the Bidding server and will contain config'
        ' information about the current execution environment.'
}];
  }
}
```
  * `ServerMetadata`: The server passes additional config information for the current execution in the ServerMetadata message. This will inform the binary if logging or debug reporting functionality is available for the current execution.
```
message ServerMetadata {
  option (privacysandbox.apis.roma.app_api.v1.roma_mesg_annotation) = {description:
      'Config information about the current execution environment for a'
      ' GenerateBidRequest.'
};

  bool debug_reporting_enabled = 1 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'A boolean value which indicates if event level debug reporting is'
      ' enabled or disabled for the request. Adtechs should only return debug'
      ' URLs if this is set to true, otherwise the URLs will be ignored and'
      ' creating these will be wasted compute.'
}];

  bool logging_enabled = 2 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'A boolean value which indicates if logging is enabled or disabled for'
      ' the request. If this is false, the logs returned from the RPC in the'
      ' response will be ignored. Otherwise, these will be outputted to the'
      ' standard logs or included in the response.'
}];
}
```

* `GenerateProtectedAudienceBidResponse`: This is the response field expected from the generateBid UDF. It contains the bid(s) for ad candidate(s) corresponding to a single Custom Audience (a.k.a Interest Group) (similar to the return values from the JS spec for generateBid).

```
message GenerateProtectedAudienceBidResponse {
  repeated ProtectedAudienceBid bids = 1 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'The generateBid() UDF can return a list of bids instead of a single bid.'
      ' This is added for supporting the K-anonymity feature. The maximum'
      ' number of bids allowed to be returned is specified by the seller. When'
      ' K-anonymity is disabled or not implemented, only the first candidate'
      ' bid will be considered.'
}];

  LogMessages log_messages = 2 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {description:
      'Adtechs can add logs to the response if logging was enabled in the'
      ' request. Logs will be printed out to the console in case of non-prod'
      ' builds and added to the server response in case of debug consented'
      ' requests.'
}];
}
```
  * `DebugReportUrls`: URLs to support debug reporting, when auction is won and auction is lost. There is no [forDebuggingOnly][163] method/API and the debug URLs for a bid have to be directly included by the binary in the proto response. These can be added in the DebugReportUrls field in the [ProtectedAudienceBid][165] proto. The [format][164] for the URLs stays the same as the browser definition and they will be pinged in the exact same way.
```
message DebugReportUrls {
   string auction_debug_win_url = 1 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = 
   {description:'URL to be triggered if the Interest Group wins the auction. If undefined'
      ' or malformed, it will be ignored.'
  }];

    string auction_debug_loss_url = 2 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = {
      description:'URL to be triggered if the Interest Group loses the auction. If'
        ' undefined or malformed, it will be ignored.'
  }];
}
```

  * `LogMessages`: The standard logs from the binary [are not exported for now][161] (This will be added later on in 2025). For now, any logs from the binary will be discarded. As a workaround, the GenerateProtectedAudienceBidResponse proto includes the log_messages field  for logs and error messages. 
```
message LogMessages {
  option (privacysandbox.apis.roma.app_api.v1.roma_mesg_annotation) = 
  {description: 'Logs, errors, and warnings populated by the generateBid() UDF.'};

  repeated string logs = 1 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = 
  {description: 'Optional list of logs.'}];

  repeated string errors = 2 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = 
  {description: 'Optional list of errors.'}];

  repeated string warnings = 3 [(privacysandbox.apis.roma.app_api.v1.roma_field_annotation) = 
  {description: 'Optional list of warnings.'}];
}

```
The logs, errors and warnings in this proto will be printed to the cloud logs in non_prod builds, and included in the server response in case of consented debug requests. 
 

#### reportWin()

Event level win reporting would work with Bidding and Auction services and function signatures
can be the same as described in web platform's [Protected Audience API explainer][43]. 

**ReportWin() will be executed in Auction service**, reporting url for the buyer and registered ad
beacons (for Fenced Frame reporting) would be generated in Auction service and returned to the client
in encrypted [AuctionResult][84]. The client will ping the buyer's reporting endpoint using the
reporting url.

_Note: Refer to the [event level reporting explainer][135] for the detailed design._

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

#### Buyer BYOS Key/Value service

_Note: BYOS Key/Value is only supported for Chrome. Protected Auctions using data from Android
devices are required to use the [Protected Auction Key/Value service][166]._

The [BuyerFrontEnd service][22] looks up biddingSignals from Buyer's BYOS Key/Value service. The base url
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

_Note: The `trustedBiddingSignals` passed to `generateBid()` for an Interest Group (Custom Audience) is
the value corresponding to each lookup key in the Interest Group but not the entire response.
Following is an example, if key1 is the lookup key in an interest group, then the following is passed
to `generateBid()` in `trustedBiddingSignals`._

```
  'key1': arbitrary_json
```

##### Filtering in buyer's Key/Value service

Filtering interest groups in buyer's Key/Value service can help reduce number
of interest groups for bidding; and therefore optimize latency and reduce cost of 
Bidding service.

To support filtering of interest groups in buyer's BYOS Key/Value service, metadata
received from the client will be forwarded in the HTTP request headers of the
`trustedBiddingSignals` lookup request.

Refer to [metadata forwarding][90] for more details.

#### Buyer service configurations

Server configurations are based on [Terraform][16] and is open sourced to [Github repo][59] for
cloud deployment. 

The configurations will include environment variables and parameters that may vary per buyer.
These can be set by the buyer in the configuration before deployment. The configurations also 
include urls that can be ingested when the service starts up for prewarming the connections.  

Refer to the [README for deployment on AWS][106] or [README for deployment on GCP][105]. Refer to 
[example config on AWS][100] or [example config on GCP][102] for the Terraform config required
for deployment to the cloud. The config requires update of some parameter values (that vary
per adtech) before deployment to cloud.

Following are some examples of data configured in service configurations. 

##### BuyerFrontEnd service configurations

* _Buyer's Key/Value service endpoint (bidding_signals_url)_: This endpoint is configured in BuyerFrontEnd
  service configuration and ingested at service startup to prewarm connections to buyer's Key/Value service.

* _Bidding service endpoint_: The domain address of Bidding service. This is ingested at service startup to
  prewarm connection.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [key management systems][10].

##### Bidding service configurations

* _Cloud Storage endpoint_: The endpoint of Cloud Storage from where buyer's code is hot reloaded by the
  Bidding service.

* _Private Key Hosting service_ and _Public Key Hosting service_ endpoints in [key management systems][10].

### Metadata forwarding

#### Metadata added by client

Browser will forward the following metadata in the request headers of [unified request][83].

* `Accept-Language`
* `User-Agent`
* `IP / geo information`

#### Metadata forwarded by seller's ad service

Seller's ad service will forward the metadata in the following non-standard HTTP
headers in the request to SellerFrontEnd service.

* `X-Accept-Language`
* `X-User-Agent`
* `X-BnA-Client-IP`

_Note: If the seller's ad service uses a gRPC client to send request to SellerFrontEnd,
the standard `User-Agent` header may be altered by gRPC. Due to that reason, non-standard
HTTP headers are required so that it cannot be overridden or altered by the gRPC client._

Seller's ad service can do one of the following to forward the metadata: 
  * If the seller's ad service sends SelectAd as HTTPS to SFE, the header can 
    be [forwarded][46].
  * If the seller's ad service sends SelectAd as gRPC, metadata needs to be 
    [created and added][47].

 
#### Metadata forwarded by SellerFrontEnd service

SellerFrontEnd will [add metadata to gRPC][47] request sent to BuyerFrontEnd service.

* `X-Accept-Language`
* `X-User-Agent`
* `X-BnA-Client-IP`

SellerFrontEnd will [forward][46] the metadata in the request headers to seller's
Key/Value service. Seller's Key/Value service may ingest this information to
generate scoring signals. Seller's Key/Value service may use `X-BnA-Client-IP`
header to monitor requests from TEE based SellerFrontEnd service.

* `Accept-Language`
* `User-Agent`
* `X-BnA-Client-IP`

#### Metadata forwarded by BuyerFrontEnd service

BuyerFrontEnd will [forward][46] the metadata in the request headers to buyer's
Key/Value service. 

This may help with filtering of interest groups (custom audiences) in buyer's 
Key/Value service. Buyer's Key/Value service may use `X-BnA-Client-IP` header to 
monitor requests from TEE based BuyerFrontEnd service.

* `Accept-Language`
* `User-Agent`
* `X-BnA-Client-IP`


## High level design

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
    only be decrypted by an attested service running in [trusted execution environment][29],
    in this case the [SellerFrontEnd service][21].

_Unified Contextual and Protected Audience Auctions Flow_ is important to
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

##### Seller's ad service

With the *Unified Contextual and Protected Audience Auction* flow, the seller's
ad service will receive one request from the client. The request would include
contextual request payload and encrypted [ProtectedAudienceInput][9] from the
client.

The encrypted [ProtectedAudienceInput][9] includes Interest Group (Custom
Audience) information on the user's device. The size of [ProtectedAudienceInput][9]
is required to be small; and there may be a per-buyer size limit set by the
client or the seller. Refer to [payload optimization][51] explainer for the
guidance around optimizing [ProtectedAudienceInput][9] payload size.

Refer to more details [here][77].

##### SellerFrontEnd service

The front-end service of the system that runs in the [trusted execution environment][29]
on a supported cloud platform. The service receives requests from [seller's ad service][20]
to initiate Protected Audience auction flow. Then the service orchestrates
requests (in parallel) to Buyers / DSPs participating in the auction for bidding. 

This service also fetches real-time scoring signals required for the auction and
calls [Auction service][23] for Protected Audience auction.

Refer to more details [here][78].

##### Auction service

The Auction service runs in the [trusted execution environment][29] on a supported
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

##### Seller's Key/Value service

A seller's Key/Value service is a critical dependency for the auction system.
The Key/Value service receives requests from the [SellerFrontEnd service][21]. 
The service returns real-time seller data required for auction that corresponds
to lookup keys available in buyers' bids (such as `ad_render_urls`
or `ad_component_render_urls`).

_Note: For a Protected Auction using data from Chrome, the sellers’s Key/Value
system may be BYOS Key/Value Service or trusted Key/Value service depending on timeline.
For a Protected Auction using data from Android, the seller must use the trusted
Key/Value service._

#### Demand-side platform (DSP) system

This section describes Protected Audience services that will be operated by a
DSP, also called a buyer. 

##### BuyerFrontEnd service

The front-end service of the system that runs in the [trusted execution environment][29]
on a supported cloud platform. This service receives requests to generate bids
from a [SellerFrontEnd service][21]. This service fetches real-time bidding
signals that are required for bidding and calls Bidding service.

Refer to more details [here][80].

##### Bidding service

The Bidding service runs in the [trusted execution environment][29] on a supported
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

##### Buyer’s Key/Value service

A buyer's Key/Value service is a critical dependency for the bidding system. The 
Key/Value service receives requests from the [BuyerFrontEnd service][22]. The
service returns real-time buyer data required for bidding, corresponding to
lookup keys.

_Note: For a Protected Auction using data from Chrome, the buyer’s Key/Value
system may be BYOS Key/Value Service or trusted Key/Value service depending on timeline.
For a Protected Auction using data from Android, the buyer must use the trusted
Key/Value service._

### Flow

* Clients (browser, Android) builds encrypted [ProtectedAudienceInput][9].
    * Client prefetch a set of public keys from the [key management systems][10]
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
        * [Client adds metadata][94] to HTTP request headers.
          
    * Android: Seller's code in publisher SDK in Android sends HTTP request to (untrusted)
      seller's ad service.
        * __[Existing request]__ Contextual payload.
        * Seller's code in publisher SDK asks Android for encrypted [ProtectedAudienceInput][9]
          to be included in request.

* Seller's ad service makes two requests.
    * __[Existing flow]__ May send Real Time Bidding (RTB) requests to partner buyers
      for contextual bids and then conduct a contextual auction to select a
      contextual ad winner.

    * Sends SelectAd request to SellerFrontEnd service if there is incremental value
      in conducting the Protected Audience auction. The request payload includes 
      encrypted [ProtectedAudienceInput][9], AuctionConfig and other
      required information. 
      * Encrypted [ProtectedAudienceInput][9] should be `Base64` encoded string of bytes
        if SelectAd request is sent as HTTPS request.
      * AuctionConfig includes contextual signals like seller_signals, auction_signals;
        per buyer signals / configuration and other data. Contextual ad winner
        may be part of seller_signals.
        * __If the SelectAd request is sent after contextual auction concludes, the
          `seller_signals` may include information about the contextual auction winner that
          can filter Protected Audience bids during Protected Audience auction.__
      * [Forwards client metadata][95] in non-standard HTTP headers in the request to
        SellerFrontEnd service.

   __Note: It is upto the seller / SSP to decide whether SelectAd request should be sent
   after contextual / RTB auction concludes or in parallel with contextual auction. The seller's
   ad server may incorporate traffic shaping and determine incremental value / demand for
   calling SellerFrontEnd for an ad request. The seller's ad server may call SellerFrontEnd while
   the contextual auction is running and this can optimize overall auction latency even further;
   however in this case contextual ad winner can not take part in Protected Audience auction to
   filter Protected Audience bids. If contextual signals (`buyer_signals`, `seller_signals` and
   `auction_signals`) are required for bidding, auction and reporting, that should be sent in
   SelectAdRequest.__
      
* Protected Audience auction kicks off in Bidding and Auction Services.
    * The SellerFrontEnd service decrypts encrypted [ProtectedAudienceInput][9] using
      decryption keys prefetched from [key management systems][10].

    * The SellerFrontEnd service orchestrates GetBids requests to participating buyers’ 
      BuyerFrontEnd services in parallel.
      * [SellerFrontEnd adds metadata to gRPC request][96] sent to BuyerFrontEnd service.
      * Buyers in `buyer_list` (in AuctionConfig) as passed by the seller and that
        have non empty [BuyerInput][82] in [ProtectedAudienceInput][9] receive
        GetBids request.

    * Within each buyer system:
        * The BuyerFrontEnd service decrypts GetBidsRequest using decryption keys
          prefetched from [key management systems][10].
        * The BuyerFrontEnd service fetches real-time data (`trustedBiddingSignals`) from
          the buyer’s Key/Value service required for generating bids.
          * [Forwards the metadata][97] in the request header to buyer's Key/Value service. 
            Buyer's Key / Value service may ingest this information for optional
            [filtering of interest groups][91] and / or monitoring requests from Bidding and
            Auction services.
        * The BuyerFrontEnd service sends a GenerateBids request to the Bidding service.
        * The Bidding service returns ad candidates with bid(s) for each `InterestGroup`.
          * The Bidding service deserializes and splits `trustedBiddingSignals` such that
            `generateBid()` execution for an `InterestGroup` can only ingest
            `trustedBiddingSignals` for the interest group / bidding signal key.
        * The BuyerFrontEnd returns all bid(s) ([AdWithBid][49]) to SellerFrontEnd.

    * Once SellerFrontEnd has received bids from all buyers, it requests real-time data
      (`trustedScoringSignals`) for all `ad_render_urls` and `ad_component_render_urls`
      (corresponding to ad with bids) from the seller’s Key/Value service. These
      signals are required to score the ads during Protected Audience auction.

    * SellerFrontEnd sends a ScoreAdsRequest to the Auction service to score ads
      and selects a winner. 
        * Auction service deserializes and splits scoring signal such that `scoreAd()`
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
      to the client. In case the contextual ad wins, a padded (chaff) but encrypted
      [AuctionResult][84] will be sent in response.

* Seller's code in publisher web page or app receives the response from seller's
  ad service and passes the encrypted [AuctionResult][84] (`auction_result_ciphertext`)
  to the client.

* Only the client (browser, Android) would be able to decrypt [AuctionResult][84]
  ciphertext. 

* Ad is rendered on the device.

### Client <> server and server <> server communication

#### Client <> seller ad service communication

Client would send a HTTPS request ([unified request][83]) to seller's ad service. 

The [ProtectedAudienceInput][9] included in the unified request will be encrypted on the client 
using a protocol called [Oblivious HTTP][50] that is based on bidirectional [Hybrid Public Key Encryption][48](HPKE). 
The Protected Audience response, i.e. [AuctionResult][84] will also be encrypted in SellerFrontEnd using 
[Oblivious HTTP][50].

The seller's ad service will not be able to decrypt or have access to [ProtectedAudienceInput][9] or 
[AuctionResult][84] in plaintext.

##### Data format
* For the web platform, request ([ProtectedAudienceInput][9]) and response ([AuctionResult][84])
payload will be [Concise Binary Object Representation (CBOR)][122] encoded. Refer to
[web platform schemas][126]. The request will be CBOR encoded on the browser and decoded in
TEE based SellerFrontEnd service. The response will be CBOR encoded in SellerFrontEnd service
and decoded on the browser.

* For android, request ([ProtectedAudienceInput][9]) and response ([AuctionResult][84])
payload will be binary protobuf.

##### Compression, encryption, padding
* For both web and Android, the BuyerInput(s) in [ProtectedAudienceInput][9] will be compressed.
  Then ProtectedAudienceInput will be encrypted and padded. An exponential padding scheme will be
  used. 

#### Seller ad service <> SellerFrontEnd communication

[Seller's ad service][20] can send gRPC or HTTPS request to SellerFrontEnd service. There would be an [Envoy Proxy][45]
service instance hosted with [SellerFrontEnd][21] for HTTPS to gRPC translation. 
  * If seller's ad service sends HTTPS request to SellerFrontEnd, [ProtectedAudienceInput][9] ciphertext should
    be `Base64` encoded; similarly the response to seller's ad service would be `Base64` encoded. This encoding is not
    required if seller's ad service and SellerFrontEnd communication is gRPC.

The communication between seller's ad service and SellerFrontEnd service would be protected by TLS / SSL that
provide communications security by encrypting data sent over the untrusted network to an authenticated peer.

#### Communication between Bidding and Auction Services

All communication between services running in [trusted execution environment][29] is protected by TLS / SSL
and the request and response payloads are encrypted using bidirectional [Hybrid Public Key Encryption][48](HPKE). 

**The TLS / SSL session terminates at the load balancer or the first hop in-front of a service, therefore the data over
the wire from the load balancer to service needs to be protected; hence the request-response is end-to-end encrypted using
bidirectional HPKE.**

_For client metadata forwarded in the requests, refer [here][90]._

### Payload compression

Most request/response payload sent over the wire should be compressed. 
* The [BuyerInput(s)][82] in [ProtectedAudienceInput][9] will be compressed on the client using `gzip`. 
  The payload needs to be compressed first and then encrypted. In SellerFrontEnd, [ProtectedAudienceInput][9] 
  will be decrypted first and then decompressed.

* Seller's ad service can compress [SelectAd][35] request payload when calling SellerFrontEnd service to save
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
         * The request-response payload between SellerFrontEnd <> seller's Key/Value services and
           BuyerFrontEnd <> buyer's Key/Value services do not require additional encryption using [HPKE][48]. 
           However, the communication between these services is over TLS that provide communications security
           by encrypting data sent over the untrusted network to an authenticated peer. 

### Payload optimization

The size of compressed [ProtectedAudienceInput][9] should be small. Refer to
[payload optimization][51] explainer for more details.

### Service code and framework

Bidding and Auction services are developed in C++. The service configurations required for cloud deployment
are based on [Terraform][16].

The service framework is based on gRPC. [gRPC][12] is an open source, high performance RPC framework built
on top of HTTP2 that is used to build scalable and fast APIs. gRPC uses [protocol buffers][13] as the
[interface description language][14] and underlying message interchange format.

Bidding and Auction services code and configurations are open sourced to [Github repo][59].

### Adtech code 

Adtech code for `generateBid()`, `scoreAd()`, `reportResult()`, `reportWin()` can follow
the same signature as described in the [Protected Audience API for the browser][28]. 

_Note:_
  * Code can be Javascript only or WASM only or WASM instantiated with Javascript.
  * If the code is in Javascript, then Javascript context is initialized before every execution.
  * No limit on code blob size.
  * More than one version of code can be supported to facilitate adtech experimentation.
  * Adtech can upload their code to Cloud Storage supported by the Cloud Platform.
  * Code is prefetched by Bidding / Auction services running in [trusted execution environment][29]
    from the Cloud Storage bucket owned by adtech.

Refer to more details [here][86].

### Cloud deployment

Bidding and Auction services are deployed by adtechs to a [public cloud platform][98] so that they are
co-located within a cloud region. 

Servers can be replicated in multiple cloud regions and the availability Service-Level-Objective (SLO)
will be decided by adtechs. 

#### SSP system

There will be a Global Load balancer for managing / routing public traffic to [SellerFrontEnd service][21].
Traffic between SellerFrontEnd and Auction service would be over private VPC network. To save cost,
SellerFrontEnd and Auction server instances will be configured in a [service mesh][87].

#### DSP system

There will be a Global Load balancer for managing / routing public traffic to [BuyerFrontEnd services][22].
Traffic between BuyerFrontEnd and Bidding service would be over private VPC network. To save cost,
BuyerFrontEnd and Bidding server instances will be configured in a [service mesh][87].

### Logging

#### Debug / non-prod build

Bidding and Auction server logs will be available with debug (non-prod) build / mode. The debug binaries 
can be built with higher [level of verbose logging](https://github.com/google/glog#verbose-logging).
For GCP, these logs will be exported to [Cloud Logging][65].

The context logger in Bidding and Auction servers supports logging `generation_id`
passed by the client in encrypted [ProtectedAudienceInput][9] and optional
(per) `buyer_debug_id` and `seller_debug_id` passed in [`SelectAdRequest.AuctionConfig`][35] for 
an ad request. The `adtech_debug_id` (`buyer_debug_id` or `seller_debug_id`) can be an internal 
log / query id used in an adtech's non TEE based systems and if available can help the adtech trace
the ad request log in Bidding and Auction servers and map with the logs in their non TEE based systems.

Logs from adtech's code can be made available in debug mode.

#### Production build

Bidding and Auction servers will support safe logging in production mode and the logs will be
exported to Cloud Logging / Cloud Watch. Refer [Debugging Protected Audience API services][60]
for more details.

Logs from adtech's code can be made available if [adtech / user consented debugging][92] is enabled. 

### Dependencies

Through techniques such as prefetching and caching, the following dependencies are in the non-critical
path of ad serving.

#### Key management systems

The [key management systems][10] are required for Protected Audience service attestation and cryptographic
key generation. Learn more in the [Overview of Protected Audience Services Explainer][6]. The 
key management systems will be deployed to all supported public clouds. Services in the key management
systems will be replicated in multiple cloud regions. 

All services running in TEE prefetch encryption and decryption keys from key management systems at service
startup and periodically in the non critical path. All communication between a service in TEE and another
service in TEE is end-to-end encrypted using Hybrid Public Key Encryption and TLS. Refer [here][11] for more
details.

## Service APIs

Refer to Bidding and Auction services APIs [in open source repo][121].

### Public APIs

#### SellerFrontEnd service and API endpoints

The SellerFrontEnd service exposes an API endpoint (SelectAd). The Seller Ad service would send
a SelectAd RPC or HTTPS request to SellerFrontEnd service. After processing the request,
SellerFrontEnd would return a SelectAdResponse that includes an encrypted AuctionResult. 

The AuctionResult will be encrypted in SellerFrontEnd using [Oblivious HTTP][50] that is based on
bidirectional [HPKE][48].

#### BuyerFrontEnd service and API endpoints

The BuyerFrontEnd service exposes an API endpoint GetBids. The SellerFrontEnd service sends
encrypted GetBidsRequest to the BuyerFrontEnd service that includes BuyerInput and other data.
After processing the request, BuyerFrontEnd returns GetBidsResponse, which includes bid(s) for
each Interest Group. Refer to [AdWithBid][49] for more information.

The communication between the BuyerFrontEnd service and the SellerFrontEnd service is TEE to TEE
communication and is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication will happen
over public network and that can also be cross cloud networks.

### Internal API

Internal APIs refer to the interface for communication between Protected Audience services within a SSP
system or DSP system.

#### Bidding service and API endpoints

The Bidding service exposes an API endpoint GenerateBids. The BuyerFrontEnd service sends
GenerateBidsRequest to the Bidding service, that includes required input for bidding. The code for
bidding is prefetched from Cloud Storage and cached in Bidding service. After processing the request,
i.e. generating bids, the Bidding service returns the GenerateBidsResponse to BuyerFrontEnd service.

The communication between the BuyerFrontEnd service and Bidding service occurs between each service’s TEE
and request-response is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication also happens
over a private VPC network.

#### Auction service and API endpoints

The Auction service exposes an API endpoint ScoreAds. The SellerFrontEnd service sends a
ScoreAdsRequest to the Auction service for running auction; ScoreAdsRequest includes bids from
each buyer and other required signals. The code for auction is prefetched from Cloud Storage and
cached in Auction service. After all ads are scored, the Auction service picks the highest scored
ad candidate and returns the score and other related data for the winning ad in ScoreAdsResponse.

The communication between the SellerFrontEnd service and Auction service occurs within each service’s
TEE and request-response is end-to-end encrypted using [HPKE][48] and TLS/SSL. The communication also
happens over a private VPC network.

[4]: https://privacysandbox.com
[5]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md
[6]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md
[7]: https://developer.android.com/design-for-safety/privacy-sandbox/protected-audience-bidding-and-auction-integration
[8]: https://github.com/WICG/privacy-preserving-ads/tree/main?tab=readme-ov-file#ad-selection-api-proposal
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
[68]: #seller-byos-keyvalue-service
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
[84]: #auctionresult
[85]: #enroll-with-coordinators
[86]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#adtech-code-execution-engine
[87]: https://en.wikipedia.org/wiki/Service_mesh
[88]: #spec-for-ssp
[89]: #spec-for-dsp
[90]: #metadata-forwarding
[91]: #filtering-in-buyers-keyvalue-service
[92]: https://github.com/privacysandbox/fledge-docs/blob/main/debugging_protected_audience_api_services.md#adtech-consented-debugging
[93]: #logging
[94]: #metadata-added-by-client
[95]: #metadata-forwarded-by-sellers-ad-service
[96]: #metadata-forwarded-by-sellerfrontend-service
[97]: #metadata-forwarded-by-buyerfrontend-service
[98]: #supported-public-cloud-platforms
[100]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/buyer/buyer.tf
[101]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/seller/seller.tf
[102]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/buyer/buyer.tf
[103]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/seller/seller.tf
[104]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/packaging/README.md
[105]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/README.md
[106]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/README.md
[107]: https://cloud.google.com/iam/docs/service-account-overview
[121]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/api/bidding_auction_servers.proto
[122]: https://cbor.io/
[123]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/services/seller_frontend_service/schemas/auction_request.json
[124]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/services/seller_frontend_service/schemas/interest_group.json
[125]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/services/seller_frontend_service/schemas/auction_response.json
[126]: #web-platform-schemas
[127]: #timeline-and-roadmap
[128]: #onboarding-and-alpha-testing-guide
[129]: #specifications-for-adtechs
[130]: #high-level-design
[131]: #service-apis
[132]: #data-format
[133]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#device-orchestrated-component-auctions
[134]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md#server-orchestrated-component-auction
[135]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_event_level_reporting.md
[136]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/secure_invoke
[137]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/tools/secure_invoke/README.md
[138]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/public_cloud_tees.md
[139]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/monitoring_protected_audience_api_services.md#list-of-metrics
[140]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/monitoring_protected_audience_api_services.md
[141]: https://github.com/WICG/privacy-preserving-ads/blob/main/Auction%20&%20Infrastructure%20Design.md#infrastructure-design-elements
[142]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals.md
[143]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/protected_audience_auctions_mixed_mode.md
[144]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md
[145]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/monitoring_protected_audience_api_services.md
[146]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/debugging_protected_audience_api_services.md
[147]: https://github.com/privacysandbox/protected-auction-services-docs/tree/main?tab=readme-ov-file#protected-auction-services-documentation
[148]: https://en.wikipedia.org/wiki/Supply-side_platform
[149]: https://en.wikipedia.org/wiki/Demand-side_platform
[150]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding-auction-services-payload-optimization.md#payload-optimization-guide-for-sellers--ssps
[151]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/roma_bring_your_own_binary.md
[152]: https://developers.google.com/privacy-sandbox/private-advertising/protected-audience/android/protected-app-signals
[153]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_system_design.md#adtech-code-execution-engine
[154]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_cost.md
[155]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/protected_app_signals_cost.md
[156]: https://github.com/WICG/protected-auction-services-discussion
[157]: https://github.com/privacysandbox/bidding-auction-servers/releases
[158]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#1-browsers-record-interest-groups
[159]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding-auction-services-payload-optimization.md#payload-optimization-guide-for-buyers--dsps
[160]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/api/udf/generate_bid.proto
[161]: https://github.com/privacysandbox/data-plane-shared-libraries/blob/main/docs/roma/byob/sdk/docs/udf/Communication%20Interface.md#standard-output-stdout
[162]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/roma_bring_your_own_binary.md
[163]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#71-fordebuggingonly-fdo-apis
[164]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#711-post-auction-signals
[165]: https://github.com/privacysandbox/bidding-auction-servers/blob/722e1542c262dddc3aaf41be7b6c159a38cefd0a/api/udf/generate_bid.proto#L261
[166]: https://github.com/privacysandbox/protected-auction-key-value-service
[167]: https://azure.microsoft.com/
[168]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding-auction-services-kanon-integration.md
[169]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_chaffing.md
