# Protected Auction Services documentation

This GitHub repository contains proposals for the Privacy Sandbox's
Protected Auction server-side infrastructure that supports the Privacy
Sandbox's client-side APIs.

Protected Auction includes [Protected Audience](https://github.com/WICG/turtledove/blob/main/FLEDGE.md)
and [Protected App Signals](https://developer.android.com/design-for-safety/privacy-sandbox/protected-app-signals)
ad targeting products.

> [!IMPORTANT]
> This repository is for documentation only. For discussions, visit the [WICG/protected-auction-services-discussion](https://github.com/WICG/protected-auction-services-discussion) repository.

## Protected Auction services trust model

* [Overview of trust model / Coordinators](trusted_services_overview.md)
* [Public cloud TEE requirements](public_cloud_tees.md)
* [Cross Cloud Coordinators](cross_cloud_coordinators.md)

## Protected Audience auctions mixed mode

* [Protected audience auctions mixed mode](protected_audience_auctions_mixed_mode.md)

## Bidding and Auction services (B&A)

* [Bidding and Auction services](bidding_auction_services_api.md)
* Self-serve guide
   * [Bidding and Auction services onboarding and self-serve guide](bidding_auction_services_onboarding_self_serve_guide.md)
* Payload optimization
   * [Bidding and Auction services payload optimization](https://github.com/privacysandbox/fledge-docs/blob/main/bidding-auction-services-payload-optimization.md)
* System design
  * [Bidding and Auction services system design](bidding_auction_services_system_design.md)
* [Multi-Currency Support in Bidding and Auction services](bidding_auction_services_bid_currency.md)
* Multi seller auctions
   * [Bidding and Auction services multi seller auctions](https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_multi_seller_auctions.md)
* Reporting
   * [Protected Audience event level reporting with Bidding and Auction services](bidding_auction_event_level_reporting.md)
   * [Protected Audience event level reporting for multi seller auctions](bidding_auction_multiseller_event_level_reporting.md)
   * [Protected Audience forDebuggingOnly API with Bidding & Auction Services](bidding_auction_for_debugging_only_design.md)
* Chaffing
   * [Bidding and Auction Services: Chaffing Design](bidding_auction_services_chaffing.md)
* Protected App Signals
   * [Bidding and Auction services: Protected App Signals](bidding_auction_services_protected_app_signals.md)
   * [Protected App Signals egress format](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_protected_app_signals_egress.md)
* B&A and K-Anonymity integration
  * [Bidding-Auction services and K-Anonymity integration](bidding-auction-services-kanon-integration.md)
* B&A experiment support
   * [Bidding and Auction services experiment](bidding_auction_experiment.md)
* Cloud support
    * AWS: [Bidding and Auction services AWS cloud support and deployment guide](bidding_auction_services_aws_guide.md)
    * GCP: [Bidding and Auction services GCP cloud support and deployment guide](bidding_auction_services_gcp_guide.md)

## Key/Value service

* [Key/Value service trust model](key_value_service_trust_model.md)
* [Key/Value service use cases](key_value_service_use_cases.md)
* [Key/Value service user-defined functions (UDFs)](key_value_service_user_defined_functions.md)
* [Key/Value sharding](key_value_service_sharding.md)
* [K/V Server Instance Sizing Guide](key_value_service_instance_sizing_guide.md)
* [Key/Value service Cost](key_value_service_cost.md)
* [Key/Value service hybrid mode](key_value_service_hybrid_mode.md)
* [Passing buyer signals from B&A to TEE KV for PA flow](key_value_service_contextual_signals_propagation.md)

## Ad Retrieval service

* [Protected App Signals: Ad Retrieval service](https://github.com/privacysandbox/protected-auction-key-value-service/blob/main/docs/protected_app_signals/ad_retrieval_overview.md)

## B&A and TEE KV / Retrieval version compatibility

* [B&A and TEE KV / Retrieval version compatibility](release_compatibility_bidding_auction_key_value_services.md)

## User Defined Function Execution
* [Roma Bring-Your-Own-Binary](roma_bring_your_own_binary.md)

## Inference

* [Bidding and Auction services: Inference Overview](inference_overview.md)

## Server productionization

* [Production operation](production_operation.md)
* [Debugging protected audience api services](debugging_protected_audience_api_services.md)
* [Monitoring protected audience api services](monitoring_protected_audience_api_services.md)

## Server cost

* [Bidding and Auction services: Cost](bidding_auction_cost.md)
* [Protected App Signals cost](protected_app_signals_cost.md)
* [Bidding and Auction Cost Estimation Tool](bidding_auction_cost_estimation_tool.md)

## Protected Auction services launch phases

* [Protected Auction services launch and testing phases](protected_auction_services_launch_testing_phases.md)

## Client and server integration

### Protected Audience
* [Browser - Bidding and Auction services integration](https://github.com/WICG/turtledove/blob/main/FLEDGE_browser_bidding_and_auction_API.md)
* [Android - Bidding and Auction services integration high level document](https://developer.android.com/design-for-safety/privacy-sandbox/protected-audience-bidding-and-auction-services)

### Protected App Signals
* [Android - Protected App Signals](https://developer.android.com/design-for-safety/privacy-sandbox/protected-app-signals)

## Related explainers

Additional explainers for Protected Audience API.

* Chrome [FLEDGE](https://github.com/WICG/turtledove/blob/main/FLEDGE.md) proposal
