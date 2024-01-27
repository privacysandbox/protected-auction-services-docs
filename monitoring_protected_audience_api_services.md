## Monitoring Protected Audience API Services

**Authors:** <br> 
[Akshay Pundle][27], Google Privacy Sandbox <br> 
[Brian Schneider][6], Google Privacy Sandbox <br>
[Xing Gao][24], Google Privacy Sandbox <br>
[Roopal Nahar][25], Google Privacy Sandbox <br>
[Chau Huynh][26], Google Privacy Sandbox <br>

Protected Audience API ([Android][1], [Chrome][2]) proposes multiple real time
services ([Bidding and Auction][3] and [Key/Value][4] services) running in a
[trusted execution environment][5] (TEE). These are isolated environments for
securely processing sensitive data with very limited data egress. Due to the
privacy guarantees of such a system, traditional ways of monitoring are not
viable as they may leak sensitive information. Nevertheless, monitoring is a
critical activity for operating such services.

We have implemented mechanisms to provide telemetry data using privacy
preserving technologies. Services running inside a [Trusted Execution
Environment][7] (TEE) are now able to export common system and business metrics
using these technologies. We use methods including [differential privacy][8] and
data aggregation to maintain the privacy preserving nature of Protected Audience
API services while providing critical metrics that will help ad techs monitor
the systems effectively.

## High level Implementation

- The servers have integrated with [OpenTelemetry][9] so data can be exported to
  monitoring systems for creating dashboard, defining alerts and other uses.
- System metrics , such as CPU resource utilization, and server specific
  business metrics are supported. These metrics are predefined in the system.
- Metrics are either non-noised or have noise added.
- There are a subset of metrics which are sensitive and may reveal information
  about user activity or which might give the ad tech running this server an
  unfair advantage. These metrics are noised using Differential Privacy.
- Other metrics do not have noise added, but are aggregated by Open telemetry
  for performance reasons before being published.

## Code references

The metrics implementation is split into two parts. Common metrics are
implemented in [definition.h][10] and bidding and auction server specific
metrics are implemented in [server_definition.h][11].

## Properties of a metric

This section describes the properties for the [list of metrics][29] below.

### Metric name

A name that uniquely identifies the metric among any privacy sandbox servers.

### Instrument

The [Open Telemetry instrument][12] we use to measure the metric. Currently supported
instruments are:

1. [UpDownCounter][13]: Counts the number, e.g. The number of requests. These values can be aggregated.
1. [Histogram][14]: Records the distribution, e.g. Request latency.
1. [Gauge][15]: Records a non-aggregatable value, e.g. Memory usage.

### Noising

Metrics can either have noise added or be non-noised. The below table calls out
the metrics that will be exported without noise, and the metrics that will have
noise added.

### Attribute

Metrics can have associated attributes. Attributes common to all metrics are
listed in [Common Attributes][28] section.  In addition, metrics can have extra
attributes that are per metric. These are recorded in the Attributes column in
the below table.


## List of metrics

### Common Metrics

These are common metrics tracked across all B&A servers.


| Metric Name                                         | Description                                                                     | Instrument    | Noising        | Attributes                                                                     |
|-----------------------------------------------------|---------------------------------------------------------------------------------|---------------|----------------|--------------------------------------------------------------------------------|
| request.count                                       | Total number of requests received by the server                                 | UpDownCounter | Not Noised     |                                                                                |
| request.duration_ms                                 | Total time taken by the server to execute the request                           | Histogram     | Not Noised     |                                                                                |
| request.failed_count_by_status                      | Total number of requests that resulted in failure partitioned by Error Code     | UpDownCounter | Not Noised     | Absl error code                                                                |
| request.size_bytes                                  | Request size in bytes                                                           | Histogram     | Not Noised     |                                                                                |
| response.size_bytes                                 | Response size in bytes                                                          | Histogram     | Not Noised     |                                                                                |
| system.cpu.percent                                  | CPU usage                                                                       | Gauge         | Not Noised     | Total_utilization, main process utilization                                    |
| system.cpu.total_cores                              | CPU total cores                                                                 | Gauge         | Not Noised     | Total CPU cores                                                                |
| system.memory.usage_kb for main process             | Memory usage                                                                    | Gauge         | Not Noised     | Main Process                                                                   |
| system.memory.usage_kb for MemAvailable             | Memory usage                                                                    | Gauge         | Not Noised     | MemAvailable                                                                   |
| system.thread.count                                 | Thread count                                                                    | Gauge         | Not Noised     |                                                                                |
| initiated_request.count_by_server                   | Total number of requests initiated by the server partitioned by outgoing server | UpDownCounter | Noised with DP | Server name                                                                    |
| system.key_fetch.failure_count                      | Failure counts for fetching keys with the coordinator                           | Gauge         | Not Noised     | public key dispatch, public key async, private key dispatch, private key async |
| system.key_fetch.num_keys_parsed_on_recent_fetch    | Number of keys parsed on the most recent key fetch                              | Gauge         | Not Noised     | public key GCP, public key AWS, private key                                    |
| system.key_fetch.num_keys_cahced_after_recent_fetch | Number of keys currently cached in memory after the most recent key fetch       | Gauge         | Not Noised     | public key GCP, public key AWS, private key                                    |


### SFE Metrics

These are metrics tracked on the Seller front end B&A servers.

| Metric                                           | Description                                                                      | Instrument    | Noising        | Attributes      |
|--------------------------------------------------|----------------------------------------------------------------------------------|---------------|----------------|-----------------|
| initiated_request.auction.duration_ms            | Total duration request takes to get response back from Auction server            | Histogram     | Noised with DP |                 |
| initiated_request.auction.errors_count_by_status | Initiated requests by auction that resulted in failure partitioned by Error Code | UpDownCounter | Noised with DP | Absl error code |
| initiated_request.auction.size_bytes             | Size of the initiated Request to Auction server in Bytes                         | Histogram     | Noised with DP |                 |
| initiated_request.bfe.errors_count_by_status     | Initiated requests by KV that resulted in failure partitioned by Error Code      | UpDownCounter | Noised with DP | Absl error code |
| initiated_request.kv.duration_ms                 | Total duration request takes to get response back from KV server                 | Histogram     | Noised with DP |                 |
| initiated_request.kv.errors_count_by_status      | Initiated requests by KV that resulted in failure partitioned by Error Code      | UpDownCounter | Noised with DP | Absl error code |
| initiated_request.kv.size_bytes                  | Size of the Initiated Request to KV server in Bytes                              | Histogram     | Noised with DP |                 |
| initiated_response.auction.size_bytes            | Size of the initiated Response by Auction server in Bytes                        | Histogram     | Noised with DP |                 |
| initiated_response.kv.size_bytes                 | Size of the Initiated Response by KV server in Bytes                             | Histogram     | Noised with DP |                 |
| sfe.error_code                                   | Number of errors in the SFE server by error code                                 | UpDownCounter | Noised with DP | Error code      |
| sfe.initiated_request.count_by_buyer             | Total number of initiated requests per buyer                                     | UpDownCounter | Noised with DP | Buyer           |
| sfe.initiated_request.duration_by_buyer          | Initiated requests duration per buyer                                            | UpDownCounter | Noised with DP | Buyer           |
| sfe.initiated_request.errors_count_by_buyer      | Total number of initiated requests failed per buyer                              | UpDownCounter | Noised with DP | Buyer           |
| sfe.initiated_request.size_by_buyer              | Initiated requests size per buyer                                                | UpDownCounter | Noised with DP | Buyer           |
| sfe.initiated_response.size_by_buyer             | Initiated response size per buyer                                                | UpDownCounter | Noised with DP | Buyer           |



### Buyer Frontend Metrics

These are metrics tracked on the BFE B&A servers.

| Metric                                           | Description                                                                 | Instrument    | Noising        | Attributes      |
|--------------------------------------------------|-----------------------------------------------------------------------------|---------------|----------------|-----------------|
| bfe.error_code                                   | Number of errors in the BFE server by error code                            | UpDownCounter | Noised with DP | Error code      |
| initiated_request.bidding.duration_ms            | Total duration request takes to get response back from bidding server       | Histogram     | Noised with DP |                 |
| initiated_request.bidding.errors_count_by_status | Initiated requests by KV that resulted in failure partitioned by Error Code | UpDownCounter | Noised with DP | Absl error code |
| initiated_request.bidding.size_bytes             | Size of the Initiated Request to Bidding server in Bytes                    | Histogram     | Noised with DP |                 |
| initiated_request.kv.duration_ms                 | Total duration request takes to get response back from KV server            | Histogram     | Noised with DP |                 |
| initiated_request.kv.errors_count_by_status      | Initiated requests by KV that resulted in failure partitioned by Error Code | UpDownCounter | Noised with DP | Absl error code |
| initiated_request.kv.size_bytes                  | Size of the Initiated Request to KV server in Bytes                         | Histogram     | Noised with DP |                 |
| initiated_response.bidding.size_bytes            | Size of the Initiated Response by Bidding server in Bytes                   | Histogram     | Noised with DP |                 |
| initiated_response.kv.size_bytes                 | Size of the Initiated Response by KV server in Bytes                        | Histogram     | Noised with DP |                 |

### Bidding Metrics

These are metrics tracked on the Bidding B&A servers.

| Metric                                           | Description                                              | Instrument    | Noising        | Attributes |
|--------------------------------------------------|----------------------------------------------------------|---------------|----------------|------------|
| js_execution.duration_ms                         | Time taken to execute the JS dispatcher                  | Histogram     | Noised with DP |            |
| js_execution.error.count                         | No. of times js execution returned status != OK          | UpDownCounter | Noised with DP |            |
| business_logic.bidding.bids.count                | Total number of bids generated by bidding service        | UpDownCounter | Noised with DP |            |
| business_logic.bidding.zero_bid.count            | Total number of times bidding service returns a zero bid | UpDownCounter | Noised with DP |            |
| business_logic.bidding.zero_bid.percent          | Percentage of times bidding service returns a zero bid   | UpDownCounter | Noised with DP |            |
| bidding.error_code                               | Number of errors in the bidding server by error code     | UpDownCounter | Noised with DP | Error code |

### Auction Metrics

These are metrics tracked on the Auction B&A servers.

| Metric                                           | Description                                                                                    | Instrument    | Noising        | Attributes              |
|--------------------------------------------------|------------------------------------------------------------------------------------------------|---------------|----------------|-------------------------|
| js_execution.duration_ms                         | Time taken to execute the JS dispatcher                                                        | Histogram     | Noised with DP |                         |
| js_execution.error.count                         | No. of times js execution returned status != OK                                                | UpDownCounter | Noised with DP |                         |
| business_logic.auction.bids.count                | Total number of bids used to score in auction service                                          | UpDownCounter | Noised with DP |                         |
| business_logic.auction.bid_rejected.count        | Total number of times auction service rejects a bid partitioned by the seller rejection reason | UpDownCounter | Noised with DP | Seller_rejection_reason |
| business_logic.auction.bid_rejected.percent      | Percentage of times auction service rejects a bid                                              | Histogram     | Noised with DP |                         |
| auction.error_code                               | Number of errors in the auction server by error code                                           | UpDownCounter | Noised with DP | Error code              |


## Common attributes

These attributes are consistent across all metrics tracked on the B&A servers
and will not be subjected to noise addition. This is because they are either
constant throughout the server's lifetime (e.g., service version) or externally
available (e.g., timestamp). Once differential privacy is implemented, these
attributes will be appended to the data post-noising, just prior to their
release from the TEE.

|      Attribute name     |                             Description                             |
|:-----------------------:|:-------------------------------------------------------------------:|
| Time                    | Time the metric was released.                                       |
| Service name            | Name of the service that the metric was measured on                 |
| Server-id               | The id of the machine the metric was measured on.                   |
| Task-id                 | Unique id of the replica index identifying the task within the job. |
| Deployment Environment  | The environment in which the server is deployed on.                 |
| Server Release Version  | Specifies the current version number of the server software in use  |

## Integration with OpenTelemetry and monitoring systems

[OpenTelemetry][9] provides a cloud-agnostic API for recording metrics, traces,
and logging. The open source code running in the secure enclaves will be
instrumented using this API. The code will also be responsible for defining how
these metrics are exported from within the TEEs to the untrusted outside world.
This includes the steps defined above, like aggregation and differential
privacy. Once this telemetry leaves the secure enclave, it is available to the
ad tech to do with it as they please.

We expect that most ad techs will choose to use an off-the-shelf [OpenTelemetry
collector][16] to collect the data, filter or sample as desired, and then
transport the data to the monitoring system(s) of their choice. The collector
will not run in the trusted enclave, meaning the ad tech can freely change the
configuration, shaping and sending the data to systems like [AWS CloudWatch][17],
[Google Cloud Monitoring][18] or any system that integrates with OpenTelemetry
(see [list][19]).

Data made available to monitoring systems can then be used to determine and
alert on system health. They can also provide valuable dashboards and debugging
information. The trusted server code guarantees that exported telemetry will not
compromise user privacy, and therefore any sort of querying of this data can be
done to benefit the ad tech that operates the system.

## GCP Cloud Monitoring Integration

[Deploying Bidding and Auction Servers on GCP][30] includes functionality to
export metrics to [GCP cloud Monitoring][18].  Dashboards with the metrics are
included for both buyer and seller deployments. To locate the dashboards,
navigate to the Cloud Monitoring section in your GCP console. Upon opening a
dashboard, you will find various widgets displaying the instrumented metrics.

The following figures show the location of the dashboards and a sample dashboard.

<figure id = "fig-1">
  <img src = "images/monitoring_protected_audience_api_services_fig_1.png"
  width = "100%"
  alt = "INSERT ALT TEXT HERE">
  <figcaption><b>Figure 1.</b> Locating the dashboard</figcaption>
</figure><br><br>


<figure id = "fig-2">
  <img src = "images/monitoring_protected_audience_api_services_fig_2.png"
  width = "100%"
  alt = "INSERT ALT TEXT HERE">
  <figcaption><b>Figure 2.</b> Sample Dashboard</figcaption>
</figure><br><br>


**Terraform Configuration:**
  - The definitions of the dashboards can be found at [Dashboard terraform][22].
  - Customizing Metric Exporter can be done through the [metric exporter
    deployment config][21].


## AWS integration

A section for integration with AWS will be added in a future update.


## Privacy safe telemetry

For maintaining the privacy guarantees of the Protected Audience API, we will
use the [Differential privacy][23] to add noise to sensitive metrics before they
leave the TEE.

The ad tech will be able to configure the metrics they want to track from the
list given above. This config will be per service, and will be an ad tech
defined subset of all metrics that are available to be tracked on the system.
All servers of the same service will track the same metrics.

A total privacy budget will be set cumulatively for all privacy impacting
metrics tracked by the ad tech. This budget will be shared among all privacy
impacting metrics that the ad tech has chosen to track. The metrics that do not
affect privacy will not consume any budget and will not have noise added. These
will also be available to the ad techs, in addition to the chosen subset of
privacy impacting metrics.

Tracking more privacy impacting metrics will mean that a lesser budget will be
available to each metric, and this will cause more noise to be added. If fewer
privacy impacting metrics are tracked, relatively more privacy budget will be
available to each metric resulting in lesser noise.

Privacy impacting metrics will be aggregated for a time period `T` inside the
TEE. Noise will be added at the end of the period just before the aggregate
metrics are pushed out, to make them differentially private. Period `T` will be
configurable by the ad tech subject to a minimum and maximum limit. A smaller
`T` will mean more frequent data, but relatively more noise whereas a larger `T`
will mean more accurate data, released less frequently.

For gathering real world data to tune values, we will add a flag that will
enable turning off the addition of noise to privacy impacting metrics. This flag
will be removed before third-party cookie deprecation. ad techs will be able to
experiment with the flag on and off to compare utility and fine tune the time
period and metrics they want to track.

[1]: https://developer.android.com/design-for-safety/ads/fledge
[2]: https://developer.chrome.com/docs/privacy-sandbox/fledge/
[3]: https://github.com/privacysandbox/fledge-docs/blob/main/bidding_auction_services_api.md
[4]: https://github.com/WICG/turtledove/blob/main/FLEDGE_Key_Value_Server_API.md
[5]: https://en.wikipedia.org/wiki/Trusted_execution_environment
[6]: https://github.com/bjschnei
[7]: https://github.com/privacysandbox/fledge-docs/blob/main/trusted_services_overview.md#trusted-execution-environment
[8]: https://en.wikipedia.org/wiki/Differential_privacy
[9]: https://opentelemetry.io/
[10]: https://github.com/privacysandbox/data-plane-shared-libraries/blob/main/src/cpp/metric/definition.h
[11]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/services/common/metric/server_definition.h
[12]: https://opentelemetry.io/docs/specs/otel/metrics/api/#instrument
[13]: https://opentelemetry.io/docs/specs/otel/metrics/api/#updowncounter-operations
[14]: https://opentelemetry.io/docs/specs/otel/metrics/api/#histogram
[15]: https://opentelemetry.io/docs/specs/otel/metrics/api/#gauge
[16]: https://opentelemetry.io/docs/collector/
[17]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
[18]: https://cloud.google.com/monitoring/docs
[19]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter
[20]: https://github.com/privacysandbox/bidding-auction-servers/tree/b27547a55f20021eb91e1e61b0d2175b4aee02ea/production/deploy/gcp/terraform
[21]: https://github.com/privacysandbox/bidding-auction-servers/blob/b27547a55f20021eb91e1e61b0d2175b4aee02ea/production/deploy/gcp/terraform/services/autoscaling/collector_startup.tftpl#L63
[22]: https://github.com/privacysandbox/bidding-auction-servers/tree/b27547a55f20021eb91e1e61b0d2175b4aee02ea/production/deploy/gcp/terraform/services/dashboards
[23]: https://github.com/google/differential-privacy
[24]: https://github.com/xinggao01
[25]: https://github.com/roopalna
[26]: https://github.com/chau-huynh
[27]: https://github.com/akshaypundle
[28]: #common-attributes
[29]: #list-of-metrics
[30]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_gcp_guide.md
