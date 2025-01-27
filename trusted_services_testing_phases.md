
# Testing phases

The supported testing phases for Protected Auction Trusted services are Alpha, Beta, and Scale.
These phases are rolling windows and specific timelines may vary for every ad tech. 

## Alpha testing  

During this phase, ad techs onboard, set up, and run services on **non-production user opt-in traffic**. 

The goal of this phase is to complete end-to-end functional testing from client to server. This would require a seller integrating with clients (web browsers, Android) and one or more partner buyers; and a buyer integrating with at least one partner seller. 

For alpha testing, check the following:
*   [Coordinator integration](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-3-enroll-with-coordinators) is not required.
*   Debug build and production build would be available for testing. Ad techs can run Bidding and Auction services [locally (debug mode)](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#local-testing) or in TEE using [test mode](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#test-mode).
*   As an interim milestone, ad techs can start [testing](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-6-test) their Bidding and Auction setup, even before fully integrating with clients or partner ad techs. 
    *   Buyers or DSPs can independently test their setup, before fully integrating with partner sellers.
    *   Sellers can independently test their setup with a real partner buyer or fake buyer, before fully integrating with the seller's ad server or clients.
*   Clients do not necessarily need to enable a traffic experiment. Testing can be done with test or developer-owned devices.
*   Ad tech testing or experimentation can be enabled on user opt-in traffic (that is, on a small set of users). 


## Beta testing

During this phase, clients (web, Android) would enable APIs on
**limited stable (end user) devices**. Ad techs can run services on
**limited stable production traffic**. 

For beta testing, check the following:
*   [Enrollment with coordinators](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-3-enroll-with-coordinators) is required.
*   Ad techs must run Bidding and Auction services in TEE in production mode.
    *   Sellers must [integrate with clients](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-5-integration-with-clients) and at least one partner buyer for conducting protected auctions. 
    *   The buyer must integrate with at least one seller.
*   Clients would enable and ramp experiments to enable Bidding and Auction APIs on a small percentage of real end user devices.
    *   _Note: Clients would determine the ramp percentage for an experiment._
*   Ad techs can enable and ramp experiments on a limited stable production or live traffic for **initial performance analysis**.
    *   _Note: Seller or SSP would determine the ramp percentage for an experiment and align with partner buyers._


## Scale testing

During this phase, ad techs can run services on **full stable, production scale testing**. 

For scale testing, check the following:
*   [Enrollment with Coordinators](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-3-enroll-with-coordinators) is required.
*   Adtechs must run Bidding and Auction services in TEE, in production mode.
    *   Sellers must [integrate with clients](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md#step-5-integration-with-clients) and at least one partner buyer for conducting protected auctions. 
    *   The buyer must integrate with at least one seller.
*   Clients would ramp experiments to enable Bidding and Auction APIs on a larger percentage of real end user devices.
*   Ad techs can enable and ramp experiments on full stable, production or live traffic. At this point, ad techs can do **advanced performance, utility, and [cost](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_cost.md) analysis**.

