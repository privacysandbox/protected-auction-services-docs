PAS Cost Estimate Explainer

**Authors:** 
<br> [Stefano Galarraga][0], Google Privacy Sandbox 


Overview and Goals
==================

[Protected App Signals][1] is a new design aimed at the app install ads market. The system is based on server-side execution via the [B&A services][2] and [TEE K/V services][16]. Shifting the auction execution on the server side has benefits in terms of latency and computing power but introduces cost to adtechs to run the cloud instances where B&A will reside.

This document is based on the existing documentation about cost analysis and reduction ([B&A][3], [K/V][13]) and aims at providing an analysis of how to apply those concepts to the PAS architecture.

Assumptions
===========

The following assumptions are made in computing the cost:

*   About the DSP K/V server used by the SFE we will consider two cases
  * TEE Ad Retrieval
  * Contextual path ad retrieval
*   SFE is using a BYOS K/V server
*   Buyside and Sellside B&A services can be running in different networks
    *   This means that there could be network cost for communications among the trusted components

Cost Analysis
=============

The following diagram shows the cost items for the case of BYOS K/V Server

![Diagram for PAS Servers Cost Analysis](images/pas-cost-elements-diagram.png "Diagram for PAS Server Cost Analysis")

With reference to the diagram above there are 9 factors contributing to the cost of running PAS:

*   _A,B_: Cost related to increase in payloads received from and sent back to the device
*   _C_: Incremental cost of running the _Seller Ad Server_: this is assumed to be negligible
*   _D, E_: Incremental cost caused of extra traffic from/to Buyer RTB Server
*   _F_: Extra cost of running the Buyer RTB Server: this is assumed to be negligible
*   _G_: Cost associated to the traffic from the _Seller Ad Server_ to the SFE
    *   For this one we will provide two estimates for the cases of TEE ad retrieval and contextual path ad retrieval
*   _H_: Cost of running the SFE
*   _I_: Cost incurred by the buyer for the SFE to BFE communication. This cost will vary depending on the seller and buyer hosting its services in the same or a different cloud providers
*   _S_: Cost incurred by the SFE for the SFE to BFE communication. This cost will vary depending on the seller and buyer hosting its services in the same or a different cloud providers
*   _Z_: Cost of running the auction server
*   _L_: Cost of running the BFE
*   _N_: Cost of running the Bidding Server
*   _P_: Cost of running the Retrieval Server. This should include the cost of updating the storage
*   _Q,R_: Cost of updating the ads and metadata in the Ad Retrieval Server
*   _T_: Cost associated to the traffic from SFE to the K/V Server
*   _U_: Cost of running the KV server
*   _M,V_: Cost for intra B&A communication

Seller Cost
-----------

The seller cost for B&A is analyzed [here][4]. This section will describe the cost differences introduced by PAS.

### A,B: Cost related to increase in payload sent to and received from the device

There is no cost expected for the incoming traffic A.

The exchange will incur additional cost in case of extra data included for PAS in the encrypted payload with the response from SFE to be sent to the device. As of today PAS is requiring no extra data to be sent so the cost is expected to be the same as estimated for [C9 in the B&A cost explainer][4].

### D,E: Cost associated to the extra traffic from the Seller Ad Server to the RTBs

The extra traffic is caused by the data that the RTB will decide to send to their BFE via the Ad Server. The data will likely include, contextual embeddings to use in generateBid and, depending on the solution adopted for ad retrieval, the amount of data sent is expected to be higher since it will contain an additional list of ad identifiers and possibly some per-ad embeddings to be used during TopK selection (in case of hybrid ad retrieval) more details about network cost are discussed in the [B&A cost explainer][5].

### G: Cost associated to the traffic from the Seller Ad Server to the SFE

There are three elements contributing to this cost:

*   The cost for the exchange of sending the [SelectAdRequest][6] including the PAS signals from the device for all the sellers to SFE (refer to C1 in the [B&A cost explainer][7])
*   The cost for the SFE of receiving the SelectAd request from Ad Server: (refer to [C2 in the B&A cost explainer][8])
*   The cost for the SFE of sending the SelectAdResponse response to the Ad Server (see [C8 in the B&A cost explainer][9])

In the case of PAS the costs C1 and C2 of the communication from AdServer to SFE will depend on the solution adopted for ad retrieval. In case of TEE ad retrieval the RTB server will send to the Ad Server information to help during retrieval and bidding such as contextual embeddings while in case of contextual the amount of data sent is expected to be higher since it will contain an additional list of ad identifiers and possibly some per-ad embeddings to be used during TopK selection (in case of hybrid ad retrieval) or bidding.

### H, V: Cost of running the SFE and Auction Server

The CPU cost of running both the SFE and Auction Server are expected to be similar to that of [serving the PA traffic][10].

### T: Cost associated to the traffic from SFE to the K/V Server

This cost too is expected to be the [same as per the PA traffic][5].

Buyer Cost
----------

We will refer to [this breakdown][11] from the B&A cost explainer.

### D,E: Cost associated to the extra traffic from the Seller Ad Server to the RTBs

This is the cost sustained by the RTB for the contextual path traffic. Please refer to the seller side discussion for details.

### F: Extra cost of running the Buyer RTB Server

There will be changes both in the RTB processing of the PAS request and in the size of the data to be sent to the RTB server. We assume this difference in cost to be negligible.

### L: Cost of running the BFE

This is the [VM Cost][12] marked C2 and C5 in the B&A explainer. With PAS, the BFE is running an additional flow to PA, thus incurring extra cost. The cost of the PAS flow could be considered equivalent to the cost of processing an extra CustomAudience (or Interest Group in the Chrome parlance).

### N: Cost of running the Bidding Server

This is where we expect noticeable differences, mostly because of the computation cost. Initial estimates suggest the AppInstall bidding server may require ~70% additional computational power for multiple javascript executions. We should account the extra workload required to run more complex inference

The distribution of the cost for running the Bidding Server based on internal estimates is shown below:

The three factors are:

*   [Inference][15]. This is expected to constitute the vast majority of the cost. Details on how to compute the inference costs will be added later.
*   Cost of preparing the data for ad retrieval: essentially the cost for running the `prepareDataForAdRetrievalUDF`. This is a minor contributor.
*   Base cost for running the bidding process, this is the cost for coordinating the flow, processing the data from the ads retrieval server and sending it to generateBid, handling the response and returning the results to the BFE.

There are a lot of changes in bidding server

1.  prepareDataForAdRetrieval - This is a new function that runs before generateBid when using TEE based retrieval and it is skipped in case of contextual based retrieval. The function is intended to be used to decode the signals received from the device and make them available to the retrieval server UDF and to the generateBid. We are expecting that this function might run some inference too. The main factor to consider when estimating this cost are:
    1.  Complexity of the model used for feature extraction
    2.  Amount of signals received (will likely always be the currently max allowed size of 1.5KB but we are working on enabling the seller to control the payload size alloted for each buyer so this might change).
    3.  Logic and language used to implement the UDF (using JS or WASM)
    4.  Cost of communicating with the inference service (sidecar process at the moment)
2.  Sending request to retrieval server (marked as O in the diagram above) - cost is zero
3.  Getting info back from retrieval and sending it to generateBid. This may involve data copying into v8. We need to figure out how much data this is, and what the copying costs are (response size from retrieval, size of embeddings, number of embeddings, number of ads).
4.  generateBid:
    1.  Cost of the running the script - this is likely similar to the remarketing case. 
    2.  Cost of sending inference requests to inference sidecar + cost per inference. To compute this you need to know how many ads will be batched in a single inference, how many models will be used during bidding and preparation of data for retrieval, the size of each model and how expensive each inference call is.
    3.  Compared to PA, where each Custom Audience will run generateBid with its subset of ads, there is only one generateBid invocation for all the PAS ads
5.  General inference cost - download models, general bookkeeping (memory cost of storing models in sidecars). Model storage cost on cloud storage.

### P: Cost of running the Retrieval Server + Q,R: Cost of updating the ads and metadata in the Ad Retrieval Server

For details about the K/V server cost you can refer to [this doc][13].

#### TEE Ad Retrieval

The cost contributions are:

*   _Compute cost_: this is the cost for running the ad retrieval UDF, we expect this to be the majority of the cost. This is the C2 cost contributor in the high level cost breakdown [here][14].
*   _Bulk storage cost_: the cost for the data to be stored in the Ad Retrieval server. This is expected to be minimal. This is the C4 cost contributor in the high level cost breakdown [here][14].
*   _Cost to transfer data to bulk storage_: This is the cost for periodically updating the data in the server and depends on the amount of data and frequency of updates. This is covering both C3 and C5 cost contributors in the high level cost breakdown [here][14].
*   _Server Monitoring Cost:_ This is the part of the cost for using the monitoring services and tracked as C7 in the high level cost breakdown [here][14].

#### Contextual Ad Retrieval

The cost contributions are:

*   _Compute cost_: In this case the cost is expected to be much lower than in the case of TEE AdRetrieval because lookup is supported by a simple system-provided UDF and soon no UDF at all. 
*   _Bulk storage cost_: See the comments for the TEE Ad Retrieval cost
*   _Cost to transfer data to bulk storage_: See the comments for the TEE Ad Retrieval cost
*   _Server Monitoring Cost:_ This will be the same as for TEE Ad Retrieval

[0]: https://github.com/galarragas
[1]: https://developers.google.com/privacy-sandbox/relevance/protected-audience/android/protected-app-signals
[2]: https://developers.google.com/privacy-sandbox/relevance/protected-audience/android/bidding-and-auction-services
[3]: ./bidding_auction_cost.md
[4]: ./bidding_auction_cost.md#high-level-cost-breakdown
[5]: ./bidding_auction_cost.md#network-costs
[6]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/api/bidding_auction_servers.proto#L404
[7]: ./bidding_auction_cost.md#seller-untrusted-server
[8]: ./bidding_auction_cost.md#load-balancer-and-network-address-translation-nat
[9]: ./bidding_auction_cost.md#seller-untrusted-server
[10]: ./bidding_auction_cost.md#vm-costs
[11]: ./bidding_auction_cost.md#high-level-cost-breakdown-1
[12]: ./bidding_auction_cost.md#vm-costs-1
[13]: ./key_value_service_cost.md
[14]: ./key_value_service_cost.md#high-level-cost-breakdown
[15]: ./inference_overview.md
[16]: https://github.com/privacysandbox/protected-auction-key-value-service