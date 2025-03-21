# Passing `buyer signals` from B&A to TKV for PA flow


# Overview

For a Protected Audience (PA) auction, we are passing an existing input signal [Buyer signals](https://github.com/privacysandbox/bidding-auction-servers/blob/09e0c1fcd1aa4acd4fbd3ae8285eda882b197314/api/bidding_auction_servers.proto#L675) to TEE based [Key Value service (TKV)](https://github.com/privacysandbox/protected-auction-services-docs/blob/81eaa0b471c1ea0d8299816feea78318b004e57e/key_value_service_use_cases.md) with [Bidding and Auction (B&A)](https://github.com/privacysandbox/protected-auction-services-docs/blob/81eaa0b471c1ea0d8299816feea78318b004e57e/bidding_auction_services_api.md) version 4.5.0.  Starting with that version, `Buyer signals` can be optionally propagated to the buyer’s TKV in the encrypted payload, i.e. never through cleartext.

The purpose of this document is to explain how this works and how to enable it.


# Background

During a PA auction performed using B&A servers, the Buyer Front End (BFE) calls out to the buyer’s TKV.  Signals sent to TKV are [limited](https://github.com/WICG/turtledove/blob/fab196ac40402a1c537fd328aa4549c492ad314b/FLEDGE_Key_Value_Server_API.md#schema-of-the-request) because historically the TKV implementation matched Bring-Your-Own-Server (BYOS) functionality.  The signals sent to BYOS are limited to reduce the impact if the service operator uses them to create a user’s cross-site profile.

Now that the TKV implementation can be trusted to protect the user’s data, we are considering what additional signals can be made available in TKV.

BFE has had access to [buyer signals](https://github.com/privacysandbox/bidding-auction-servers/blob/09e0c1fcd1aa4acd4fbd3ae8285eda882b197314/api/bidding_auction_servers.proto#L675), but previously TKV did not. More context on what buyer signals are is available [here](#what-are-buyer-signals). See this [section](#propagation-details) for more context on where the buyer signals are coming from. See this section talking about [motivation](#motivation).

# Motivation

To select the best ad to show to a user, adtechs can utilize all available data.  They also want to make their ad selection decision in the environment that not only has compute, e.g. B&A, but has access to data, i.e. TKV.

To that end we can consider sending more contextual data to the TKV, where adtechs can use that data in their user defined functions (UDFs) along with data from advertiser sites to make improved ad retrieval  decisions in an environment that has access to the AdTech's loaded data.


# What are `buyer signals`

![Architecture diagram.](images/unified-contextual-remarketing-bidding-auction-services.png)

The `buyer signals` field is a free form string. That string has serialized _contextual_ signals corresponding to each buyer in an auction that could help in generating bids. Each buyer is aware of the format in which that string is serialized and can take advantage of it in respective UDFs.

Buyer signals are [set](https://github.com/privacysandbox/bidding-auction-servers/blob/8b1a9808c279ebc66dc360d14b5ad9d35e34965c/api/bidding_auction_servers.proto#L724) by an untrusted <span style="text-decoration:underline;">Seller Ad Service</span> based on the contextual information available at the time of the B&A `SelectAdReqest`request creation. \
`buyer signals` have a parameter name of `perBuyerSignals` in [generateBid](https://github.com/privacysandbox/protected-auction-services-docs/blob/81eaa0b471c1ea0d8299816feea78318b004e57e/bidding_auction_services_api.md#generatebid-javascriptwasm-spec).


# Propagation details


## Seller's ad server to B&A

As mentioned above, buyer signals are [set](https://github.com/privacysandbox/bidding-auction-servers/blob/8b1a9808c279ebc66dc360d14b5ad9d35e34965c/api/bidding_auction_servers.proto#L724) by an untrusted Seller Ad Service based on the contextual information available at the time of the B&A `SelectAdReqest`that is sent to [SellerFrontEnd service (SFE).](https://github.com/privacysandbox/protected-auction-services-docs/blob/81eaa0b471c1ea0d8299816feea78318b004e57e/bidding_auction_services_api.md#sellerfrontend-service)

That means that the  perBuyerSignals (contextual signals) need to be available before SFE is called.

SFE then [sends](https://github.com/privacysandbox/bidding-auction-servers/blob/722e1542c262dddc3aaf41be7b6c159a38cefd0a/api/bidding_auction_servers.proto#L854) that info to BFE.


## B&A to buyer's TEE Key/value service

BFE sends that info in the [metadata](https://github.com/privacysandbox/protected-auction-key-value-service/blob/5d586e0046e7b482e70c1b97bf322a923340bfab/public/query/v2/get_values_v2.proto#L109) field `buyer_signals` request. That field is later available in each UDF execution. See the UDF API section in this [doc](https://github.com/privacysandbox/protected-auction-key-value-service/blob/6f702e02833a06c1bfd6a31f7d9c8f0ded98536d/docs/APIs.md) and this [proto](https://github.com/privacysandbox/protected-auction-key-value-service/blob/6f702e02833a06c1bfd6a31f7d9c8f0ded98536d/public/api_schema.proto#L30).


### Enabling signals propagation

These signals won’t be propagated by default. By default there is no extra bandwidth or change in behavior. \
An Adtech can choose to propagate by enabling a feature flag `propagate_buyer_signals_to_tkv` in BFE.

Adding buyer signals to the payload is extra bandwidth.


# On device

We will add buyer signals propagation support for on device shortly after. We’re planning to update this document with additional details once they are finalized and are ready to be shared.


# Feedback

Feedback on this feature is welcome. Please provide it by opening an issue on github.
