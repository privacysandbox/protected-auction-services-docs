# Bidding and Auction Cost Estimation Tool

## Overview

The B&A Cost Estimation Tool offers a convenient way for ad techs to estimate the cost of running
Bidding and Auction services. By analyzing utilization metrics from a short test run (about an
hour), it can estimate usage and costs, for B&A development or production environments. This
complements the traditional method of analyzing actual bills after running a setup. While the tool
estimates costs, it is not a substitute for ad techs verifying their actual costs. Factors that
affect the accuracy of cost estimates are set out
[below](#factors-that-affect-accuracy-of-cost-estimates). The
[B&A Cost Explainer](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_cost.md)
provides detailed information on the costs associated with B&A.

## How it works

The tool leverages a configurable cost model that extrapolates observed system metrics - such as CPU
usage, amount of bytes transferred etc. - from ad tech's test environment into estimated usage and
costs. These metrics are gathered from a simulated test run of ad tech's B&A system deployment.
Here's a breakdown of the process:

### Deploying a Cost test environment

A test environment for B&A services is deployed by each ad tech using available guides for
[B&A deployments](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_onboarding_self_serve_guide.md)
for more details.

### Running a Simulation Test

Ad tech generates anticipated workload against an ad tech's deployed B&A services system. Tests may
be performed with any load test utility that supports gRPC (e.g. [ghz](https://ghz.sh/).

### Running the cost estimation tool

Next, ad techs will set up and use the cost estimation tool. Here's a general overview.

#### Cost Estimation Tool Configuration:

The configuration process involves specifying test environment details (e.g. cloud provider, SKUs,
and region) and test duration. Custom SKU pricing can also be included. Sample configuration cost
model (cost.yaml) and SKU pricing (sku.json) files are provided for customization.

#### Metric Acquisition:

The tool then gathers system usage metrics from the simulated test run of an ad tech's deployed B&A
system. These metrics, collected for various services (SFE, BFE, bidding, auction), can be
downloaded from B&A supported cloud providers or ingested from a CSV file.

#### Cost Model Application:

The cost model is the core component of the B&A Cost Estimation Tool, which translates system usage
metrics into cost estimations. It uses a set of formulas to estimate resource usage and, optionally,
monetary costs. This analysis is done for each SKU, which is how cloud platforms usually charge for
their services. The cost model comprises two components:

-   **Cost Model Formulas**: The cost model first uses formulas to link SKUs with corresponding
    system usage metrics. These formulas convert resource consumption (e.g. CPU time, memory usage
    ect) into estimated resource usage for each SKU. The formulas are defined in the `cost.yaml`
    configuration file.
-   **SKU Pricing (optional)**: Once resource usage is estimated by the Cost Model Formulas, the
    tool can use SKU pricing information to translate this usage into a monetary cost. Final cost
    estimates depend on SKU pricing being provided by ad techs. The tool allows for custom SKU
    pricing data (stored in a separate file sku.json, with details such as unit cost per SKU per
    region) to generate cost estimates when combined with usage estimates. For clarity, this data
    will not be accessible to Google.

To start using the tool, see the [Cost Estimation Tool
README](https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/cost_estimation/README.md).

### Example:

Let's take a look at an example of how the tool estimates Seller-Compute costs, specifically for a
single SKU 'N2D AMD Instance Core running in Virginia' on GCP. It is crucial to note that
Seller-Compute represents only one component of the total cost; additional factors, including but
not limited to Confidential Compute and Network costs, must be considered for a comprehensive cost
assessment.

**Example Metrics**:

Assume the cost model requires these metrics:

-   `sfe:request.count`: Represents the count of requests processed by the SFE service. This metric
    is
    [collected](https://github.com/privacysandbox/protected-auction-services-docs/blob/main/monitoring_protected_audience_api_services.md#list-of-metrics)
    as a part of the B&A deployments and will be used here for extrapolating costs per million
    requests.
-   `sfe:total_cores`: Number of CPU cores used by the SFE service. This metric is collected as
    system.cpu.percent with the label "Total cpu cores" and can be used with a shortened name as
    [explained here](https://github.com/privacysandbox/bidding-auction-servers/tree/main/tools/cost_estimation/README.md#download-metrics-section)
-   `auction:total_cores`: Number of CPU cores used by the Auction service.
-   `test.duration`: Duration of the test in hours. This is calculated based on the start and end
    time given to the tool.

Based on a simulated test run, these metrics have been collected.

-   `sfe:total_cores`: 4 cores per instance
-   `auction:total_cores`: 4 cores per instance
-   `test.duration`: 1 hour
-   `sfe:request.count`: 1,500,000 requests

**Cost Model Configuration sample**:

_Sample entries in the cost.yaml file_

-   `**cost_model_metadata**`: Specifies the vendor (`gcp`), region (`us-east4`), and the metric for
    calculating estimated cost per million queries (`sfe:request.count`).
-   `**defined_values**`: Defines intermediate values like `sfe_cpu_hours` and `auction_cpu_hours`
    used in the formula.
-   `**usage_estimations**`: Defines the formula (`sfe_cpu_hours + auction_cpu_hours`) to estimate
    the usage of the "N2D AMD Instance Core running in Virginia" SKU. This SKU is billed per hour,
    so we compute the total number of vCPU hours used to estimate the usage here.

**SKU File Configuration sample**:

_Sample entries in the sku.json file_

This excerpt shows the "N2D AMD Instance Core" SKU in the `us-east4` region has a unit cost of
$x.xx\* per hour (HR). This is a placeholder value.

**Calculation of Usage and Cost**:

The tool performs these calculations:

**Output**:

The tool outputs the calculated information in CSV format:

**Seller-Compute**

| SKU                                       | SKU ID         | Unit Cost | Cost Basis | Estimated Units | Estimated Cost | Estimated Cost per million queries |
| ----------------------------------------- | -------------- | --------- | ---------- | --------------- | -------------- | ---------------------------------- |
| N2D AMD Instance Core running in Virginia | 809C-1E3B-306E | $x.xx\*   | HR         | 24              | $x.xx\*        | $x.xx\*                            |

_\*The **Estimated Cost per million queries** is shown as
$$x.xx* because the unit_cost in the SKU
file is set to $$x.xx\*. If an actual unit cost were
provided, this value would be non-zero, reflecting the cost of running the specified SKU for the
given workload_.

## Interpreting results

The cost estimation tool produces a CSV file containing estimated usage and cost data. The output
varies depending on whether a unit cost is provided in the SKU file.

### Output File

The output file is organized into the following sections:

-   **Summary Section**: Presents the **estimated cost per million queries** aggregated by cost
    entity (e.g., Buyer/Seller Compute, Confidential-Compute, Network). If a unit cost isn't in the
    SKU file, this section will show 0.
-   **Details Section**: Provides a detailed breakdown of estimated costs at the SKU level. This
    includes:
    -   Unit costs (reflects unit cost provided in the SKU file)
    -   Estimated units per million queries
    -   Estimated cost per million queries (if unit cost provided in the SKU file)
-   **Observed Metrics**: Shows metrics collected from the B&A system. For example for a Seller
    (Requests (e.g., `sfe:request.count`), Bid counts (e.g.,
    `auction:auction.business_logic.bids_count`), Errors, and CPU utilization (e.g.,
    `auction:system.cpu.percent:total_cpu_cores` etc.)
-   **Configuration**: Displays information about the test setup: For Example (Start time, Duration,
    Number of instances, Other relevant configuration details ect)

**Note**: The specific metrics and cost categories in the report depend on the test setup and
configuration. The report can focus on the Seller, the Buyer, or provide a combined view of B&A
metrics.

### Usage and Cost Estimates

The tool generates two types of estimates:

##### Usage Estimates (SKU File Omits Unit Cost)

If the SKU file _does not_ include unit costs, the tool provides usage estimates calculated using
the cost model's formulas and metrics from the B&A system. These estimates include:

-   **Estimated Units**: Units of measurement depend on the SKU. For example, compute resources like
    CPU cores are measured in hours, while network resources are measured in gigabytes.
-   **Per-SKU Usage**: The tool provides a usage estimate for each SKU defined in the cost model.
    For example, the number of hours of usage for "N2D AMD Instance Core running in Virginia" is
    shown as "Estimated Units per million queries."

##### Cost Estimates (SKU File Includes Unit Cost)

If the SKU file _includes_ unit costs, the tool estimates both usage and costs. Cost estimates are
calculated by multiplying estimated usage by the unit cost from the SKU file. These estimates
include:

-   **Per-SKU Estimated Cost**: The total cost for each SKU, calculated as the product of estimated
    usage and unit cost.
-   **Total Estimated Cost per million queries**: The total cost for each group of related SKUs.

### Factors that affect accuracy of cost estimates

This cost estimation tool provides estimations based on usage metrics generated during a test run of
B&A and an ad tech defined cost model. It is not a substitute for ad techs verifying their actual
costs. It's crucial to understand factors that can affect cost estimation:

-   **Ad tech-specific Workload Characteristics**: Cost estimates are heavily influenced by workload
    characteristics such as business logic (such as UDFs, Inference models), requests (such as
    number of IGs, number of ads within and IG), lookup size from Key Value Service, QPS of traffic
    and other business-specific workload characteristics.
-   **Ad tech-specific Environment Configuration**: Factors such as hardware configurations (such as
    CPU, memory, storage, instance types), Cloud Regions where the ad tech operates, Network
    configuration, server topology setup (BYOS or Trusted Key Value Service) and utilization of
    servers have a large influence on the cost estimates.
-   **Cost Model Accuracy and Applicability**: The cost model uses predefined formulas to translate
    resource usage into estimated costs. This provides a reasonable starting point for understanding
    potential costs associated with running Bidding and Auction services. However, it's important to
    note that several factors can influence the final cost. The model's accuracy is closely tied to
    the specific deployment structure being used. Deviations from this structure, or changes in
    deployment without corresponding adjustments to the model, can lead to inaccurate cost
    predictions. Additionally, the actual cost will depend on the specific SKU pricing provided by
    the ad tech provider.
-   **External Cost Factors**: The tool focuses on estimating costs directly related to resource
    utilization within the system. It does not account for external factors such as third-party
    service fees, or potential costs associated with development, maintenance, and operations, which
    are independent of the tool itself.

Deviations in any of these areas can cause discrepancies between estimated and actual costs.

