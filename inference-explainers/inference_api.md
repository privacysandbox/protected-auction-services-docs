**Authors:** <br>
[Steve Gan][5], Google Privacy Sandbox <br>

# Inference API

This document provides a comprehensive overview of the Bidding and Auction (B&A) Inference service API with externally supplied models ([B&A inference overview][1]), including supported machine learning (ML) frameworks, model formats, model management processes, and the JavaScript API for executing inference tasks.

## Supported Machine Learning Frameworks and Model Formats

The Tensorflow and PyTorch runtimes are supported, each with a dedicated inference service sidecar binary. Currently, we offer the Tensorflow v2.14.0 sidecar binary and the PyTorch v2.1.1 sidecar binary.

For Tensorflow, the SavedModel format is used while PyTorch uses the TorchScript format. We recommend that ad techs develop and train models in an environment in which the ML runtime version aligns with the intended sidecar.

The maximum size for a single ML model is 2GB.

## Model management

ML models used by the inference service are fetched periodically from a linked cloud storage (e.g.,  [Google Cloud Storage buckets][2], [Amazon S3 buckets][3]) and made available for serving. Each model is uniquely identified by its path within the cloud storage. Ad techs can implement model versioning by structuring model storage paths to include version identifiers. For example, storing models at paths such as “pcvr_v1” and “pcvr_v2” distinguishes between the two versions.

Ad techs can prepare a model configuration file in JSON format within the same cloud storage as the models. The B&A service periodically checks the configuration file (according to period configurable by ad techs) for any changes and triggers the loading of new models into the inference service sidecar’s memory as needed.

For example:

```json
{
  "model_metadata": [
      {"model_path": "pcvr_v1","checksum":...,"warm_up_batch_request_json":...},
      {"model_path": "pcvr_v2","checksum":...,"warm_up_batch_request_json":...}
    ]
}
```

This model configuration file features a top-level array with each entry containing the metadata of the fetched models. The mandatory model_path field specifies the path to the model in the cloud storage. The optional checksum field is the SHA256 checksum of the model represented as a hexadecimal string.
Model checksums are computed using the following steps:

Model checksums are computed using the following steps:
1. Compute the SHA256 checksum for each individual model file.
2. Arrange the file checksums in ascending order based on their file paths.
3. Concatenate all the ordered file checksums into a single string.
4. Compute the final SHA256 checksum on the concatenated checksum string.

The listed steps are equivalent to the following bash command:
```
find <model_path> -type f -exec sha256sum {} \; | sort -k 2 | awk '{print $1}' | tr -d '\n' | sha256sum | awk '{print $1}'
```

The optional **warm_up_batch_request_json** is a JSON batch request used to warm up the model before any traffic is served. The format of this warm-up request uses the same format as the input to the runInference JavaScript function, which is described later in this document.

Refer to the [B&A Inference Onboarding Guide][4] for more details about using the model configuration file with the Terraform configuration.

## JavaScript API

We expose the Inference service capabilities as JavaScript functions in ad tech code modules. Currently, the Inference service can be executed as a part of the proprietary bidding logic. Ad techs can query available models and issue batched inference service requests to a selected model set.

### JavaScript functions

#### getModelPaths
The `getModelPaths` function doesn’t accept arguments and returns the available models in the inference service sidecar. The returned result is represented in a serialized JSON array of model paths. For example, [“pcvr_v1”, “pcvr_v2”].

#### runInference
The `runInference` function accepts batch requests and returns batch responses.

##### Batch request
The batch inference service request is represented as a JSON object containing a top-level array. Each element in the array represents an individual request and is represented as a JSON object.

##### Individual request mandatory fields
* **model_path:** Specifies the target model for the request
* **tensors:** An array of JSON objects representing the input tensors to the model.

#### Tensor mandatory fields
* **data_type:** Specifies the data type of the tensor, chosen from DOUBLE, FLOAT, INT8, INT16, INT32, INT64.
* **tensor_shape:** An array specifying the shape of the tensor.
* **tensor_content:** A flattened representation of the tensor values.
* **tensor_name:** Specifies the tensor's name to match the model's signature. This value is ignored for PyTorch models.

**tensor_content** values are represented as strings, with plans to eventually enable storage as numeric types, as numeric value support is currently under development.

The following illustrates a batch request involving two models,  “pcvr_v1” and “pcvr_v2”:

```json
{
    "request": [
        {
            "model_path": "pcvr_v1",
            "tensors": [
                {
                    "tensor_name": "feature1",
                    "data_type": "DOUBLE",
                    "tensor_shape": [2, 1],
                    "tensor_content": ["0.454920", "-0.25752"]
                }
            ]
        },
        {
            "model_path": "pcvr_v2",
            "tensors": [
                {
                    "tensor_name": "feature1",
                    "data_type": "INT32",
                    "tensor_shape": [2, 1],
                    "tensor_content": ["5", "6"]
                },
                {
                    "tensor_name": "feature2",
                    "data_type": "FLOAT",
                    "tensor_shape": [2, 2],
                    "tensor_content": ["0.5", "0.6", "0.7", "0.8"]
                }
            ]
        }
    ]
}
```

#### Batch response
The batch inference request is structured as a JSON object containing a top-level array. Each element within this array represents an individual response represented as a JSON object. This array consists of at least one element. If a batch inference fails, this one element represents the batch error.

Each individual response object signifies either successful inference output or an error that was encountered during inference.

* For a success inference, each tensor JSON object follows the same format as in the request case except that **tensor_content** directly contains numericals instead of strings.

* In the failure case, the **model_path** is optional. It is populated when the inference directed towards a specific model fails and is left empty when an entire batch request fails. The **model_path** has a mandatory error field with each error composed of a JSON object with **error_type** and description string fields.
We currently report the following error types:

    * UNKNOWN
    * INPUT_PARSING
    * MODEL_NOT_FOUND
    * MODEL_EXECUTION
    * OUTPUT_PARSING
    * GRPC

The description field propagates back the original Abseil error message from the inference sidecar C++ program.

The following are some illustrative scenarios for reference:

* **Batch with all successful outputs:** The top-level array contains multiple response objects, each with the **model_path** and tensor fields populated which indicates successful inference for each input in the batch:

```json
{
    "response": [
        {
            "model_path": "pcvr_1",
            "tensors": [
                {
                    "tensor_name": "StatefulPartitionedCall:0",
                    "data_type": "FLOAT",
                    "tensor_shape": [2, 1],
                    "tensor_content": [0.1167, 0.1782]
                }
            ]
        },
        {
            "model_path": "pcvr_2",
            "tensors": [
                {
                    "tensor_name": "StatefulPartitionedCall:0",
                    "data_type": "FLOAT",
                    "tensor_shape": [2, 1],
                    "tensor_content": [0.3422, 0.2346]
                }
            ]
        }
    ]
}
```

* **Batch with mixed successes and failures:** The top-level array contains a mix of response objects. Some have **model_path** and tensors, while others have populated error fields, indicating that some inferences succeeded while others failed.

```json
{
    "response": [
        {
            "model_path": "pcvr_1",
            "tensors": [
                {
                    "tensor_name": "StatefulPartitionedCall:0",
                    "data_type": "FLOAT",
                    "tensor_shape": [2, 1],
                    "tensor_content": [0.1167, 0.1782]
                }
            ]
        },
        {
            "model_path": "pcvr_2",
            "error": {
                "error_type": "MODEL_NOT_FOUND",
                "description": "NOT_FOUND: Requested model pcvr_2 not found"
            }
        }
    ]
}
```

* **Complete batch failure:** The top-level array contains a single response object with a  populated error field. The **model_path** field is absent in this scenario, as the failure happens at the batch level rather than at the model specific level.

```json
{
    "response": [
        {
            "error": {
                "error_type": "GRPC",
                "description": "transport closed"
            }
        }
    ]
}
```

#### Inference JavaScript API code example

The following is an example of how ad techs can use `getModelPaths` with `runInference` functions together to execute an inference task in JavaScript:

```javascript
const modelPath = getLatestModel(getModelPaths());

const batchRequest = JSON.stringify({
    request: [{ model_path: modelPath, tensors: [input1, input2, /*...*/] }]
});

const inference_result = JSON.parse(runInference(batchRequest));

for (const response of inference_result.response) {
    if (response.error) {
        handleError(response.error.error_type);
    } else {
        console.log('Use inference output:', response.tensors);
    }
}
```


[1]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference_overview.md#inference-with-externally-provided-models
[2]: https://cloud.google.com/storage/docs/buckets
[3]: https://aws.amazon.com/s3
[4]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference-explainers/inference_onboarding_guide.md
[5]: https://github.com/steveganzy
