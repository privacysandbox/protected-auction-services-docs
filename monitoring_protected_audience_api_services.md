## Monitoring Protected Audience API Services

**Authors:** <br> [Akshay Pundle][13], Google Privacy Sandbox <br> Brian
Schneider, Google Privacy Sandbox

Protected Audience API ([Android][1], [Chrome][2]) proposes multiple real time
services ([Bidding and Auction][3] and [Key/Value][4] services) running in a
[trusted execution environment][5] (TEE). These are isolated environments for
securely processing sensitive data with very limited data egress. Due to the
privacy guarantees of such a system, traditional ways of monitoring are not
viable as they may leak sensitive information. Nevertheless, monitoring is a
critical activity for operating such services.

We propose mechanisms to provide telemetry data in a privacy preserving way.
Services running inside a TEE will be able to export common system and business
metrics while preserving user privacy. We will use methods including
[differential privacy][6] and data aggregation to maintain the privacy
preserving nature of Protected Audience API services while providing critical
metrics that will help ad techs monitor the systems effectively.

### High level proposal

- The servers will integrate with [OpenTelemetry][7] so data can be exported to
  monitoring systems for creating dashboard, defining alerts and other uses.
- System metrics , such as CPU resource utilization, and server specific
  business metrics will be supported. These metrics will be predefined in the
  system.
- Metrics will be classified according to their Privacy impact. Privacy
  sensitive metrics will be aggregated and have noise added to preserve privacy
  through differential privacy ([privacy safe telemetry][14]).
- Metrics not impacting privacy will not have noise added, but will be
  aggregated by Open telemetry for performance reasons before being published.

## Proposed Metrics

This is the initial list of metrics. We will be adding more details (e.g. which
metrics are privacy impacting) and may amend the list with more metrics in
future revisions of this document.

| Metric                                          | Description                                                                                                |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Replicas                                        | Number of replicas successfully started                                                                    |
| CPU utilization                                 | Amount of CPU resource in use                                                                              |
| CPU available                                   | Amount of CPU resource available                                                                           |
| Memory utilization                              | Amount of memory in use                                                                                    |
| Memory available                                | Amount of memory available                                                                                 |
| Requests responded                              | Total number of requests responded to                                                                      |
| Requests failed                                 | Number of requests that resulted in an error                                                               |
| Request duration                                | Total time to respond to request                                                                           |
| Request duration histogram                      | Histogram of total time to respond to request                                                              |
| Request size                                    | Size of the request in bytes                                                                               |
| Response size                                   | Size of the response in bytes                                                                              |
| Initiated requests                              | Number of requests initiated by this server                                                                |
| Initiated requests per destination              | Number of requests initiated by this server segmented by the destination                                   |
| Initiated request duration                      | Total time taken to receive a response for the request                                                     |
| Initiated request duration per destination      | Total time taken to receive a response for the request segmented by the destination                        |
| Initiated request size                          | Size of the outgoing request, in bytes                                                                     |
| Initiated request size per destination          | Size of the outgoing request, in bytes, segmented by the destination                                       |
| Initiated request response size                 | Size of the response received, in bytes                                                                    |
| Initiated request response size per destination | Size of the response received, in bytes segmented by the destination                                       |
| Initiated requests failed                       | Total number of requests initiated by this server that got an error response                               |
| Initiated requests failed per destination       | Total number of requests initiated by this server that got an error response, segmented by the destination |

### Attributes

These attributes will accompany all metrics.

| Attribute name   | Description                                                |
| ---------------- | ---------------------------------------------------------- |
| Time             | Time the metric was released.                              |
| Service name     | Name of the service that the metric was measured on        |
| Server-id        | The id of the machine the metric was measured on.          |
| Software version | The version of software running that measured the metric.  |
| Platform         | The platform the software ran on that measured the metric. |

## Integration with OpenTelemetry and monitoring systems

[OpenTelemetry][7] provides a cloud-agnostic API for recording metrics, traces,
and logging. The open source code running in the secure enclaves will be
instrumented using this API. The code will also be responsible for defining how
these metrics are exported from within the TEEs to the untrusted outside world.
This includes the steps defined above, like aggregation and differential
privacy. Once this telemetry leaves the secure enclave, it is available to the
ad tech to do with it as they please.

We expect that most ad techs will choose to use an off-the-shelf [OpenTelemetry
collector][8] to collect the data, filter or sample as desired, and then
transport the data to the monitoring system(s) of their choice. The collector
will not run in the trusted enclave, meaning the ad tech can freely change the
configuration, shaping and sending the data to systems like [AWS CloudWatch][9],
[Google Cloud Monitoring][10] or any system that integrates with OpenTelemetry
(see [list][11]).

Data made available to monitoring systems can then be used to determine and
alert on system health. They can also provide valuable dashboards and debugging
information. The trusted server code guarantees that exported telemetry will not
compromise user privacy, and therefore any sort of querying of this data can be
done to benefit the ad tech that operates the system.

## Privacy safe telemetry

For maintaining the privacy guarantees of the Protected Audience API, we will
use the Google's [differential privacy libraries][12] to add noise to privacy
impacting metrics before they leave the TEE.

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
[6]: https://en.wikipedia.org/wiki/Differential_privacy
[7]: https://opentelemetry.io/
[8]: https://opentelemetry.io/docs/collector/
[9]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
[10]: https://cloud.google.com/monitoring/docs
[11]: https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/exporter
[12]: https://github.com/google/differential-privacy
[13]: https://github.com/akshaypundle
[14]: #Privacy-safe-telemetry
