**Authors:** <br>
[Steve Gan][17], Google Privacy Sandbox <br>

# B&A Inference Basic Setup and Testing Guide

This document describes how to package and deploy Bidding and Auction services ([B&A system design][1]) with machine learning (ML) inference capabilities ([B&A inference overview][2]) in cloud and local environments. This guide assumes you have administrative access to your cloud provider accounts and are familiar with the deployment process for Bidding and Auction servers and using ML models for inference.

## Cloud Deployment

The Bidding and Auction services can be deployed on Google Cloud Platform (GCP) or Amazon Web Services (AWS). We provide GCP and AWS support for using Bidding and Auction services with inference capabilities. Documentation for packaging and deploying Bidding and Auction services is available in the [Bidding and Auction services GCP cloud support and deployment guide][3] (GCP) and the [Bidding and Auction services AWS cloud support and deployment guide][4] (AWS).

The inference capabilities require additional steps to configure cloud storage and Terraform-based services, and are described in this document. While inference for Bidding and Auction services is designed to be generalized to other services, inference support is currently limited to the Bidding service.

To deploy and use the inference capabilities, you will:

- Build and package the Bidding and Auction servers images.
- Upload ML models to a cloud bucket and generate a metadata file summarizing all models you are using.
- Update UDF code modules to invoke the inference JavaScript functions.
- Supply additional inference-specific Terraform configurations to deploy the Bidding and Auction servers with inference enabled.

The following sections provide a step-by-step breakdown of the cloud deployment process for each supported cloud.

### GCP Guide: Package, deploy, and run a service

This section describes how to package and deploy B&A services on GCP.

The process for creating a functioning service with inference on GCP has two major steps:

1. **Packaging:** Create a Docker image containing the services’ code.
2. **Deployment:** Run Terraform to bring up the Bidding and Auction services with inference.

#### Step 0: Prerequisites

1. Refer to the [Bidding and Auction services GCP cloud support and deployment guide][3] to familiarize yourself with the packaging and deployment process for GCP. This document assumes you are able to create a functioning service stack (following Step 0 to Step 2.5 in the Package, Deploy, and Run a Service Guide section of the above guide). It highlights the additional steps required to enable inference capabilities.
2. Running Bidding and Auction services with inference requires functioning machine learning models. Choose Tensorflow or PyTorch as your ML platform, and then familiarize yourself with the [Tensorflow][5] or [PyTorch][6] ML framework. This document assumes you are able to create and save models in your chosen framework.

#### Step 1: Packaging

1. **Build the GCP Confidential Space Docker image:** Follow the instructions in the aforementioned [GCP Guide][3] to build a docker image for all B&A services. The inference artifacts, including both the “tensorflow_v2_14_0” and “pytorch_v2_1_1” (ML runtimes and versions that we currently support) sidecar binaries, will be packaged within the Bidding service image. <br/>
*Note: Other versions of Tensorflow and PyTorch will be supported in the future.*

#### Step 2: Deployment

1. **Upload ML models:** Use a GCS bucket to host proprietary ML models. The Bidding server fetches the uploaded models during runtime and forwards them to the inference sidecar for handling requests. The bucket name is required by the Terraform configuration. Additionally, you must upload a model configuration file that specifies the models being fetched to the cloud bucket and include its path relative to the cloud bucket in the Terraform configuration. The model configuration file has the following format:

```json
{
  "model_metadata": [
    {
      "model_path": "model_1",
      "checksum": "<SHA-256 checksum value of hexadecimal string>"
    },
    {
      "model_path": "model_2",
      "checksum": "<SHA-256 checksum value of hexadecimal string>"
    }
  ]
}
```

<p style="margin-left: 2em;"> Each model is specified by its path relative to the cloud bucket and may be accompanied by an optional SHA-256 checksum of the model's content. The checksum can be computed by using the following bash command:

```
find <model_path> -type f -exec sha256sum {} \; | sort -k 2 | awk '{print $1}' | tr -d '\n' | sha256sum | awk '{print $1}'
```

<p style="margin-left: 2em;"> Store the metadata file in the same bucket as the models.
You can update it after server deployment to fetch additional models. Note that you must grant READ permission for your proprietary endpoints to access your models and use them with your service.

2. **Upload Code Modules:** The inference capabilities of your proprietary JavaScript bidding code modules are exposed with the `runInference` and `getModelPaths` inference callbacks. To use inference capabilities, update your existing code modules to invoke these functions. Refer to [this section][7] of the B&A Inference Overview Explainer for more details about these two functions, and to [this section][8] of the GCP Guide for more information about uploading code modules.

3. **Configure Terraform variables:** Configure the inference capabilities using inference Terraform flags. The names of all inference-related Terraform flags have an `INFERENCE` prefix. Include them as part of the `runtime_flags` in the “buyer” module. Use `INFERENCE_SIDECAR_BINARY_PATH` to expose your ML runtime selection. To enable inference and select the ML runtime, set this flag to `/server/bin/inference_sidecar_<runtime_name>`, where `<runtime_name>` is either tensorflow_v2_14_0 for a Tensorflow runtime with the version 2.14.0, or pytorch_v2_1_1 for a PyTorch runtime with the version 2.1.1 . **Note that inference is disabled if this flag is not set.** If inference is disabled, all inference-related Terraform flags are ignored and invocations of inference callbacks in code modules will fail. <br/>
For recommended deployment configurations for Bidding and Auction servers with inference, see [this configuration file][9]. You can find example values for the following flags in the same configuration file. <br/>
Set `INFERENCE_MODEL_BUCKET_NAME` to the name of the GCS bucket used to store models. <br/>
Set `INFERENCE_MODEL_CONFIG_PATH` to the path to the model configuration file relative to the model GCS bucket. This metadata file lists the models to be fetched. <br/>
Set `INFERENCE_MODEL_FETCH_PERIOD_MS` to the period that the bidding server checks updates in the model configuration file and fetches models. <br/>
Set `INFERENCE_SIDECAR_RUNTIME_CONFIG` to a JSON config that sets runtime configurations for the inference sidecar binary. It has the following format:

```
{
    "num_interop_threads": <integer_value>,
    "num_intraop_threads": <integer_value>,
    "cpuset": <an array of integer values>
}
```

<p style="margin-left: 2em;"> These are performance-related flags for tuning the sidecar's performance.	 You can find more details about these flags <a href="https://github.com/privacysandbox/bidding-auction-servers/blob/release-4.3/services/inference_sidecar/common/proto/inference_sidecar.proto#L102">here</a>.</p>

4. **Apply Terraform:** There is no change in applying the Terraform step from [here][10].

5. **Test the Service:** To test your Bidding and Auction services with inference capabilities, you can follow the same steps as described in [this section][11]. The Bidding and Auction flow will use your code modules that invoke inference callbacks. You can verify that inference is being invoked by monitoring the “inference.request.count” metric exported from the bidding server in your GCP monitoring dashboard to ensure it is incrementing as expected.

### AWS Guide: Package, deploy, and run a service

This section describes how to package and deploy B&A services on AWS.

The process for creating a functioning service with inference on AWS has two major steps:

1. **Packaging:** Create an [Amazon Machine Image][12] (AMI) containing the services’ code in a Nitro Enclave.
2. **Deployment:** Run Terraform to bring up the Bidding and Auction services with inference.

#### Step 0: Prerequisites
1. Refer to the [Bidding and Auction services AWS cloud support and deployment guide][4] to familiarize yourself with the packaging and deployment process for running Bidding and Auction services on AWS. This document assumes you are able to create a functioning service stack (following Step 0 to Step 2.5 in the Package, Deploy, and Run a Service Guide section of the above guide). It highlights the additional steps required to enable inference capabilities.

2. Running Bidding and Auction services with inference requires functioning machine learning models. Choose Tensorflow or PyTorch as your ML platform, and then familiarize yourself with the [Tensorflow][5] or [PyTorch][6] ML framework. This document assumes you are able to create and save models in your chosen framework.

#### Step 1: Packaging:
1. **Build the Amazon Machine Image (AMI):** Follow the instructions in the aforementioned [AWS Guide][4] to build a docker image for all B&A services. The inference artifacts, including both the “tensorflow_v2_14_0” and “pytorch_v2_1_1” (ML runtimes and versions that we currently support) sidecar binaries, will be packaged within the Bidding service image. <br/>
*Note: Other versions of Tensorflow and PyTorch will be supported in the future.*

#### Step 2: Deployment:

1. **Upload ML models:** Use an S3 bucket to host your proprietary ML models. The Bidding server fetches the uploaded models during runtime and forwards them to the inference sidecar for production. The bucket name is required by the Terraform configuration. Additionally, you must upload a model configuration file that specifies the models being fetched to the cloud bucket and include its path relative to the cloud bucket in the Terraform configuration. The model configuration file has the following format.

```json
{
  "model_metadata": [
    {
      "model_path": "model_1",
      "checksum": "<SHA-256 checksum value of hexadecimal string>"
    },
    {
      "model_path": "model_2",
      "checksum": "<SHA-256 checksum value of hexadecimal string>"
    }
  ]
}
```

<p style="margin-left: 2em;"> Each model is specified by its path relative to the cloud bucket and may be accompanied by an optional SHA-256 checksum of the model's content. The checksum can be computed by using the following bash command:

```
find <model_path> -type f -exec sha256sum {} \; | sort -k 2 | awk '{print $1}' | tr -d '\n' | sha256sum | awk '{print $1}'
```

<p style="margin-left: 2em;"> Store the metadata file in the same bucket as the models. You can update it after server deployment to fetch additional models. You must grant READ permission for your EC2 host IAM role to access your models and use them with your service.

2. **Upload Code Modules:** The inference capabilities of your proprietary JavaScript bidding code modules are exposed with the `runInference` and `getModelPaths` inference callbacks. To use inference capabilities, update your existing code modules to invoke these functions. Refer to [this section][7] of the B&A Inference Overview Explainer for more details about these two functions, and to [this section][13] of the AWS Guide for more information about uploading code modules.

3. **Configure Terraform variables:** Configure the inference capabilities using inference Terraform flags. The names of all inference-related Terraform flags have an `INFERENCE` prefix. Include them as part of the `runtime_flags` in the “buyer” module. Use `INFERENCE_SIDECAR_BINARY_PATH` to expose your ML runtime selection. To enable inference and select the ML runtime, set this flag to `/server/bin/inference_sidecar_<runtime_name>`, where `<runtime_name>` is either tensorflow_v2_14_0 for a Tensorflow runtime with the version 2.14.0, or pytorch_v2_1_1 for a PyTorch runtime with the version 2.1.1 . **Note that inference is disabled if this flag is not set.** If inference is disabled, all inference-related Terraform flags are ignored and invocations of inference callbacks in code modules will fail. <br/>
For recommended deployment configurations for Bidding and Auction servers with inference, see [this configuration file][14]. You can find example values for the following flags in the same configuration file. <br/>
Set `INFERENCE_MODEL_BUCKET_NAME` to the name of the S3 bucket used to store models. <br/>
Set `INFERENCE_MODEL_CONFIG_PATH` to the path to the model configuration file relative to the model GCS bucket. This metadata file lists the models to be fetched. <br/>
Set `INFERENCE_MODEL_FETCH_PERIOD_MS` to the period that the bidding server checks updates in the model configuration file and fetches models. <br/>
Set `INFERENCE_SIDECAR_RUNTIME_CONFIG` to a JSON config that sets runtime configurations for the inference sidecar binary. It has the following format:

```
{
    "num_interop_threads": <integer_value>,
    "num_intraop_threads": <integer_value>,
    "cpuset": <an array of integer values>
}
```

<p style="margin-left: 2em;"> These are performance-related flags for tuning the sidecar's performance.	 You can find more details about these flags <a href="https://github.com/privacysandbox/bidding-auction-servers/blob/release-4.3/services/inference_sidecar/common/proto/inference_sidecar.proto#L102">here</a>.</p>

4. **Apply Terraform:** There is no change in applying the Terraform step from [here][15].

5. **Test the Service:** To test your Bidding and Auction services with inference capabilities, you can follow the same steps as described in [this section][16]. The Bidding and Auction flow will execute your code modules that invoke inference callbacks. You can verify that inference is being invoked by monitoring the “inference.request.count” metric within the AWS CloudWatch Log Group /metrics/bidding to ensure it increments as expected.

## Local Testing Support
You can test the B&A server stack with inference locally, before deploying to the cloud, using the steps outlined below.

1. Run builders/tools/bazel-debian build to build each server.
    * To use debug logging, pass the build_flavor directly to this tool.
    * Build the BuyerFrontEnd, Bidding, SellerFrontEnd, and Auction servers.

```
cd $(git rev-parse --show-toplevel)
# For debug build with logging
builders/tools/bazel-debian build //services/xxx_service:server --config=non_prod

# For prod build without logging
builders/tools/bazel-debian build //services/xxx_service:server --config=prod
```

2. Generate the inference sidecar.
    * For local testing, AdTech can generate either the “tensorflow_v2_14_0” sidecar for a Tensorflow runtime with the version 2.14.0, the “pytorch_v2_1_1” sidecar for a PyTorch runtime with the version 2.1.1 individually, or both, depending on your needs. (For local testing, you can choose to build one or the other. For cloud deployment, running the tool automatically generates both sidecars.)
    * Note that inference sidecars are built within different Bazel workspaces than the servers even though the source code resides in the same Github repository.

```
# To generate the Tensorflow sidecar
cd services/inference_sidecar/modules/tensorflow_v2_14_0
builders/tools/bazel-debian build //:generate_artifacts

# To generate the PyTorch sidecar
cd services/inference_sidecar/modules/pytorch_v2_1_1
builders/tools/bazel-debian build //:generate_artifacts
```

3. Use debug scripts to start the servers.

```
cd $(git rev-parse --show-toplevel)
# To start the buyer frontend server
./tools/debug/start_bfe

# To start the bidding server with inference
./tools/debug/start_bidding --enable_inference

# To start the seller frontend server
./tools/debug/start_sfe

# To start the auction server
./tools/debug/start_auction
```

4. You may adjust the startup arguments in the local debug scripts to experiment with different server startup configurations as you see fit.

5. To test inference is being invoked:
    * Refer to the information outlined [here][11] to send a request to your local Bidding and Auction servers.
    * The Bidding and Auction flow will execute the default local debug code module at services/inference_sidecar/common/tools/debug/generateBidRunInference.js , which prints inference related logs. You may mount a different code module in the tools/debug/strat_bidding script and use console logging to verify inference is being invoked.

[1]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_system_design.md
[2]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference_overview.md
[3]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_gcp_guide.md
[4]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_aws_guide.md
[5]: https://www.tensorflow.org
[6]: https://pytorch.org
[7]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference_overview.md#using-versions-in-ba
[8]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_gcp_guide.md#step-24-upload-code-modules
[9]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/buyer/buyer.tf
[10]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/gcp/terraform/environment/demo/README.md
[11]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_gcp_guide.md#step-25-test-the-service
[12]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html
[13]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_aws_guide.md#step-24-upload-code-modules
[14]: https://github.com/privacysandbox/bidding-auction-servers/blob/main/production/deploy/aws/terraform/environment/demo/buyer/buyer.tf
[15]: https://github.com/privacysandbox/bidding-auction-servers/tree/main/production/deploy/aws/terraform/environment/demo/README.md
[16]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/bidding_auction_services_aws_guide.md#step-25-test-the-service
[17]: https://github.com/steveganzy
