> _FLEDGE has been renamed to Protected Audience API. To learn more about the
name change, read the_ [_blog post_][1].

# Protected Audience event level reporting with Bidding and Auction services

**Author:** <br>
[Rashmi Rao][0], Google Privacy Sandbox

[Bidding and Auction services][2] outlines a way to allow Protected
Audience auctions to take place in a [trusted execution
environment][3] (TEE) hosted on a [supported cloud
platform][4].

This explainer describes the system design for event level reporting using the
Bidding and Auction services. For [high level design][5], [ad tech
specifications][6], [API][7], refer to the [Bidding and
Auction services explainer][5].

## Overview

The seller and the winning buyer each have an opportunity to generate URLs for
event-level reporting based on an auction outcome.

Reporting with Bidding and Auction services is planned to be supported in
multiple phases.

<table>
  <tr>
    <td><strong>Reporting support with Bidding and Auction services</strong></td>
    <td><strong>Timeline</strong></td>
  </tr>
  <tr>
    <td>Win reporting for Single Seller Auctions</td>
    <td>LAUNCHED</td>
  </tr>
  <tr>
    <td>Win reporting for Device Orchestrated Multi Seller Auctions.</td>
    <td>Dec 2023</td>
  </tr>
  <tr>
    <td>Win reporting for Server Orchestrated Multi Seller Auctions.</td>
    <td>Jan 2024</td>
  </tr>
  <tr>
    <td>Private Aggregation API</td>
    <td>June 2024</td>
  </tr>
</table>

This document details the design to generate and ping win reporting URLs with
the Bidding and Auction services for single-seller auctions. The reporting URLs
are generated from JavaScript using the `reportResult()` function provided by the
seller and `reportWin()` function provided by the buyer on the Auction service.
Final pings to the URLs happen on the client.

This design supports win reporting on Android and Chrome. The design also
supports _fenced frame reporting_, also known as _interaction reporting_ or
_beacon reporting_.

## Background

A Bidding and Auction service enables processing Protected Auction requests on
the server. Sellers provide the code required for scoring ads and reporting URL
generation using the `scoreAd()` and `reportResult()` functions. Buyers provide the
code for generating the bids and reporting using the `generateBid()` and
`reportWin()` functions. The flow of Bidding and Auction services is as follows:

- The seller sends a [`SelectAd`][8] request to the SellerFrontEnd
  service.
- The service orchestrates requests in parallel to the buyers' BuyerFrontEnd
  services for Protected Auction bidding.
- The buyers generate bids and return the bids to the seller.
- The SellerFrontEnd service calls the Auction service with the bids and scoring
  signals obtained from the seller's Key-Value service.
- The Auction service scores the bids and determines the winner.
- The Auction service returns the winning ad to the SellerFrontEnd service.
- The SellerFrontEnd service encrypts the auction result and returns a
  [`SelectAdResponse`][9] to the seller's ad service.
- Seller's ad service sends the encrypted auction result back to the client.

After scoring ads and determining a winner, the reportResult() and reportWin()
functions generate the reporting URLs. The client must ping these URLs after an
ad is rendered.

## The reportResult specification

The seller-provided endpoint for `scoreAd()` contains the `reportResult()` function.

```
reportResult(auctionConfig, sellerReportingMetadata){
…
  return signalsForWinner
}
```

The arguments for `reportResult()` are described in the following table:

<table>
  <tr>
    <td><code>Argument</code></td>
    <td><code>JSON Object subfield</code></td>
    <td><code>Description</code></td>
  </tr>
  <tr>
    <td><code>auctionConfig</code></td>
    <td><code>auctionSignals</code></td>
    <td>Contextual signals passed from seller's ad service to the SellerFrontEnd service in <code>SelectAdRequest.AuctionConfig</code></td>
  </tr>
  <tr>
    <td></td>
    <td><code>sellerSignals</code></td>
    <td>Contextual signals passed from seller's ad service to the SellerFrontEnd service in <code>SelectAdRequest.AuctionConfig</code></td>
  </tr>
  <tr>
    <td><code>sellerReportingMetadata</code></td>
    <td><code>interestGroupOwner</code></td>
    <td>DSP / buyer domain.</td>
  </tr>
  <tr>
    <td></td>
    <td><code>renderURL</code></td>
    <td>Ad render URL of winning bid.</td>
  </tr>
  <tr>
    <td></td>
    <td><code>bid</code></td>
    <td>Value of bid that scored highest.</td>
  </tr>
  <tr>
    <td></td>
    <td><code>desirability</code></td>
    <td>Score of winning bid.<br><br>
    Note: Winning bid refers to the bid with the highest score.</td>
  </tr>
  <tr>
    <td></td>
    <td><code>highestScoringOtherBid</code></td>
    <td>This is the value of a bid with the second highest score in the auction. This value represents eCPM (effective cost per thousand impressions), valued in US dollars.<br><br>
      Note:
      <ul>
        <li>A higher bid value can get a lower score, so it is possible this bid value is higher than "highest scoring bid".</li>
        <li>Rejected bids are not included.</li>
        <li>If more than one bid has the second highest score, then the '<code>highestScoringOtherBid</code>' is randomly selected.</li>
        <li>This value will be 0 if there was only 1 bid.</li>
      </ul>
    </td>
  </tr>
  <tr>
    <td></td>
    <td><code>topWindowHostName</code></td>
    <td>This is the host name of the publisher.</td>
  </tr>
</table>

The `reportResult()` function returns signalsForWinner, which is used as input for
the buyer's `reportWin()` function.

## The reportWin specification

The buyer-provided endpoint for `generateBid()` is expected to contain the
`reportWin()` function.

```
reportWin(auctionSignals, perBuyerSignals, sellerSignals, buyerReportingMetadata) {
  ...
}
```

The arguments for `reportWin() are described in the following table:

<table>
<tr>
<td><code>Argument</code></td>
<td><code>JSON Object Subfield</code></td>
<td><code>Description</code></td>
</tr>
<tr>
<td><code>auctionSignals</code></td>
<td></td>
<td>Contextual signals passed from the seller's ad service to the
SellerFrontEnd service in <code>SelectAdRequest.AuctionConfig</code>.</td>
</tr>
<tr>
    <td><code>signalsForWinner</code></td>
    <td></td>
    <td>Signals returned by seller's <code>ReportResult()</code> function execution.</td>
  </tr>

<tr>
    <td><code>perBuyerSignals</code></td>
    <td></td>
    <td>Contextual signals passed from the seller's ad service to the SellerFrontEnd service in <code>SelectAdRequest.AuctionConfig</code>. This is the buyer signals for the winning buyer.</td>
  </tr>

<tr>
    <td><code>buyerReportingMetadata</code></td>
    <td><code>seller</code></td>
    <td>Seller's origin. This will be passed to the Auction service from the SellerFrontEnd service.</td>
  </tr>

<tr>
    <td></td>
    <td><code>adCost</code></td>
    <td>An optional field returned by <code>generateBid()</code>, rounded to fit into a floating point number with an 8 bit mantissa and 8 bit exponent for reporting.</td>
  </tr>

<tr>
    <td></td>
    <td><code>interestGroupName</code></td>
    <td>The name of the interest group corresponding to the highest scoring bid.</td>
  </tr>

<tr>
    <td></td>
    <td><code>madeHighestScoringOtherBid</code></td>
    <td>This is set to true if the winning buyer was the only buyer that made bids with the second highest score.</td>
  </tr>

<tr>
    <td></td>
    <td><code>recency</code></td>
    <td>Duration of time (in minutes) from when this device joined this interest group until now. This is passed by the client.</td>
  </tr>

<tr>
    <td></td>
    <td><code>modelingSignals</code></td>
    <td>Sent as an output from the <code>generateBid()</code> function. This is expected to be a single integer value.</td>
  </tr>

<tr>
    <td></td>
    <td><code>joinCount</code></td>
    <td>The number of times the given device has joined the same interest group in the last 30 days, while the interest group has been continuously stored.</td>
  </tr>

</table>

Note:

- `buyerReportingMetadata` additionally contains all the metadata present in the
  `sellerReportingMetadata`.
- The 'recency`, `modelingSignals` and `joinCount` fields are noised using a
  [noising scheme][10].
- The `bid`, `score`, and `adCost` fields are stochastically rounded.

## Reporting function

In the short-term, `reportResult()` and r`eportWin()` reporting are run by calling a
`sendReportTo()` API. The `sendReportTo()` function can be called at most once
during a function's execution. Eventually, reporting will also support the
Private Aggregation API.

The executable code in [Roma][11] contains a `sendReportTo()`
function which takes a single string argument representing a URL:

```
sendReportTo = function(url){
  // Stores the url in a global variable
}
```

## Fenced Frame reporting

Ad techs can register URLs that correspond to events or beacons when
`reportResult()` and r`eportWin()` are run. These beacons can be used to trigger
pings using the `reportEvent` API on the client.

The beacons can be registered by calling the `registerAdBeacon()` function,
which will be a part of the executable code in [Roma][11]:

```
registerAdBeacon = function(interactionUrlMap){
  // Stores the map to a global variable.
}
```

The input to `registerAdBeacon` is expected to be a JSON object with a mapping
of the event string to a corresponding URL.

## Design

The high level flow for reporting is as follows:

- The client and buyer send the required signals for reporting to the The
  SellerFrontEnd service.
- The SellerFrontEnd service sends the required metadata to run `reportResult()`
  and `reportWin()` to the Auction service
- Auction service executes the `reportResult()` and `reportWin()` functions in
  [Roma][11] and returns the reporting URLs to The SellerFrontEnd
  service.
- The SellerFrontEnd service encodes the reporting URLs using CBOR encoding for
  both buyer and seller and adds it to the [`SelectAdResponse`][9].
  The `reportResult()` and `reportWin()` functions are executed serially in a
  single dispatch call after the winner has been determined.
- Clients use these URLs to ping the buyer and seller reporting endpoints after
  the ad is rendered.

<img src="images/win-reporting.svg" width="90%">

### Rationale for the design choices

#### Why is the reporting URL generation done on the server?

The inputs to the reporting functions include large signals such as:

- Auction config
- Per-buyer signals

The response payload will significantly increase (potentially by 30kb or more)
if reporting URL generation is performed on the client, leading to an increase
in the cost and latency. Server-generated reporting URLs are lightweight and
don't affect the response payload size.

#### Why is the buyer's reportWin function executed on the seller's auction
server?

The reporting function execution can happen only after the ads are scored and
the winner is determined. The `reportWin()` function call depends on an output of
the `reportResult()` function called `signalsForWinner()`. If `reportWin()` needs to
be executed on the buyer's Bidding service, an RPC from the SellerFrontEnd
service to BuyerFrontEnd service is needed to fetch the URLs. This adds
significant latency on the critical response path (>50ms) and additional network
costs to both buyer and seller.

### API changes

The reporting URLs and beacon URL map for both the seller and the buyer are sent
to the client in a [`SelectAdResponse`][9].

These URLs are [CBOR-encoded][12] and encrypted before sending the
response from the SellerFrontEnd service.

```
Message AdScore {
…
// The reporting URLs registered during the execution of reportResult() and
// reportWin().
WinReportingUrls win_reporting_urls = 10;

}

message WinReportingUrls {

message ReportingUrls {
// The url to be pinged for reporting win to the buyer or seller.
string reporting_url = 1;
// The map of (interactionKey, URI).
map<string, string> interaction_reporting_urls = 2;
}

// The reporting URLs registered during the execution of
// reportWin(). These URLs will be pinged from the client.
ReportingUrls buyer_reporting_urls = 1;

// The reporting URLs registered during the execution of reportResult() of the
// seller in case of single seller auction and component seller in case of
// multi seller auctions. These URLs will be pinged from the client.
ReportingUrls component_seller_reporting_urls = 2;

// The reporting URLs registered during the execution of reportResult() of the
// top level seller in case of multi seller auction. These URLs will be pinged
// from the client. This will not be set for single seller auctions.
ReportingUrls top_level_seller_reporting_urls = 3;
}

```

### Flags

Reporting function execution is enabled by 2 flags on the server:

- `enableReportResultUrlGeneration`: To enable executing the `reportResult()`
  function
- `enableReportWinUrlGeneration`: To enable executing the `reportWin()` function.

If `enableReportResultUrlGeneration` is set to `false`, neither of the functions
are executed.

### Code fetch and loading

The code executed in the auction service is loaded only once at startup time,
and periodically every few hours (as configured by the developer) in the Auction
service from endpoints where the ad tech's reporting code modules are hosted.

The function name to be called is determined at run time before the dispatch
request. The code is periodically fetched from public endpoints provided by the
seller and buyer. The code loaded into [Roma][11] should to contain
the following functions:

- `scoreAd()`
- `reportResult()`
- `reportWin()` for all buyers. `reportWin()` is called only for the winner.

The `reportWin()` functions are loaded if `enableReportWinUrlGeneration` has
been set to `true`.

Note: Try to keep `reportWin()` in a separate code module from `generateBid()`,
so that a seller can only fetch the partner buyers' `reportWin()` code modules.

### JavaScript wrapper

The reporting functions are called using a [wrapper JavaScript
function][13] such as:

```
function reportingEntryFunction(auctionConfig, sellerReportingSignals, directFromSellerSignals, enable_logging, buyerReportingMetadata) {
      ……
        return {
       reportResultResponse: {
        reportResultUrl : "",
        interactionReportingUrls : "",
        sendReportToInvoked : false,
        registerAdBeaconInvoked : false,
      },
      sellerLogs: "",
      sellerErrors: "",
      sellerWarnings: "",
      reportWinResponse: {
        reportWinResponse: {
        reportWinUrl : "",
        interactionReportingUrls : "",
        sendReportToInvoked : false,
        registerAdBeaconInvoked : false,
      },
      buyerLogs: ""
      buyerErrors: "",
      buyerWarnings: "",
}
```

Inputs to the wrapper function include:

- Inputs required for the `reportResult()` call
- Inputs required for `reportWin()` call
- Flag that enables `reportWin()`
- Flag that enables exporting logs captured during ad tech script execution.

During each function call, the ad tech can log information or error messages
using `console.log` and `console.error` functions. These logs are exported in
the response from the wrapper. Log capture is enabled only if the server side
flag `enableAdTechCodeLogging` is set to `true`. In the future, the logging will
be controlled by an additional user-consented debugging flag.

### Client side

Chrome and Android will extract the reporting URLs and beacon URL map from the
Protected Audience encrypted `AuctionResult` response. When the ad is rendered,
reporting URLs are pinged based on the client controlled flag. Clients are
expected to do an additional check to verify if the domain of the reporting URLs
belong to the seller's or buyer's domain.

[0]: https://github.com/rashmijrao
[1]: https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge
[2]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md
[3]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trusted-execution-environment
[4]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#supported-public-cloud-platforms
[5]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#unified-contextual-and-fledge-auction-flow-with-bidding-and-auction-services
[6]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#specifications-for-adtechs
[7]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#service-apis
[8]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md#sellerfrontend-service-and-api-endpoints
[9]: https://github.com/privacysandbox/bidding-auction-servers/blob/4a7accd09a7dabf891b5953e5cdbb35d038c83c6/api/bidding_auction_servers.proto#L441
[10]: https://github.com/WICG/turtledove/blob/main/FLEDGE.md#521-noised-and-bucketed-signals
[11]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_system_design.md#roma
[12]: https://cbor.io/
[13]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/services/auction_service/code_wrapper/seller_code_wrapper.h
