# Overview

**Note:** _This is a draft API proposal, and not necessarily the final version. It has not been
decided if and when this API will be integrated with the Trusted Servers._

Privacy Sandbox's trusted servers read data from untrusted data sources outside of the user request
path as part of their regular operations (e.g. Bidding servers load the
[buyer's bidding user-defined function](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_api.md#generatebid)
during server initialization). Parc is an infrastructure-agnostic API definition to facilitate
receiving data from untrusted sources. In a production environment, a server implementing the Parc
API (a "Parc server") would sit between trusted servers running in
[TEEs](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/public_cloud_tees.md)
and Adtech-controlled platform services to provide data needed by the trusted servers.

In the current design of Trusted Servers, functionality to read untrusted data sources is
implemented within the TEE, specifically for each
[supported cloud service provider](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/public_cloud_tees.md).
The Parc service interface abstracts away the specific untrusted third-party components for
interacting with platform services (blob storage, parameter storage etc).These trusted servers can
instead rely on the Parc API and Parc untrusted servers for retrieving data from platform services
at runtime. This decoupling has the added benefit of minimizing the threat surface of the TEE
trusted servers by reducing the presence of third-party, untrusted, infrastructure-specific code in
the trusted executable. Trusted servers would still need to ensure that untrusted data is used in
accordance with the relevant security and privacy considerations of the server.[^1]

![Parc Overview](images/parc-api-overview.png 'Parc Overview')

[^1]:
    Already the Trusted Servers place technical protections when executing such code (e.g. Bidding
    and Auction servers use
    [Roma](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_api.md#generatebid)).
