# Inference API

**Authors:** <br>
[Steve Gan][5], Google Privacy Sandbox <br>

## Introduction

This document provides a comprehensive overview of the Bidding and Auction (B&A) Inference service API with externally supplied models ([B&A inference overview][1]), including supported machine learning (ML) frameworks, model formats, model management processes, and the JavaScript API for executing inference tasks.

## Supported Machine Learning Frameworks and Model Formats

The Tensorflow and PyTorch runtimes are supported, each with a dedicated inference service sidecar binary. Currently, we offer the Tensorflow v2.14.0 sidecar binary and the PyTorch v2.1.1 sidecar binary.

For Tensorflow, the SavedModel format is used while PyTorch uses the TorchScript format. We recommend that ad techs develop and train models in an environment in which the ML runtime version aligns with the intended sidecar.

The maximum size for a single ML model is 2GB.

## Model Management

ML models used by the inference service are fetched periodically from a linked cloud storage (e.g., [Google Cloud Storage buckets][2], [Amazon S3 buckets][3]) and made available for serving. Each model is uniquely identified by its path within the cloud storage.

### Model Loading

To determine the models to load, the inference service looks for a JSON model configuration file stored in the same bucket as the models. Ad techs are responsible for maintaining and updating this model configuration file. The path to the configuration file is exposed as a Terraform parameter. The B&A service periodically checks the specified configuration file (at intervals configurable by ad techs) for any changes and triggers the loading of new models into the inference service sidecar’s memory as needed.

Ad techs can implement model versioning by structuring model storage paths to include version identifiers. For example, storing models under the directories such as “pcvr_v1/” and “pcvr_v2/” distinguishes between the two versions.

For example:

```json
{
  "model_metadata": [
    {
      "model_path": "pcvr_v1/",
      "checksum": "dd94b3b08ea19b4240aa5e5f68ff0447b40e37ebdd06cc76fa1a2cf61143a10c",
      "warm_up_batch_request_json": "..."
    },
    {
      "model_path": "pcvr_v2/",
      "checksum": "...",
      "warm_up_batch_request_json": "..."
    }
  ]
}
```

This model configuration file features a top-level array with each entry containing the metadata of the fetched models. The mandatory `model_path` field specifies the path to the model in the cloud storage. If the specified model path points to a directory, it needs to end with a "/" suffix. This suffix indicates that the entire directory will be used for model registration. Without the "/" suffix, an exact path match is expected, and only the specified file will be registered as a model. In the example above, both model paths refer to directories.

In TensorFlow, models are typically stored as directories containing multiple files. For example, a "pcvr/" model directory might have the following structure:

```
pcvr/saved_model.pb
pcvr/variables/variables.data-00000-of-00001
pcvr/variables/variables.index
```
To register the entire model including all three files, the `model_path` field needs to be set to "pcvr/".

For PyTorch, models are stored as single files, such as "pcvr/model.pt". For registration, you can specify either the directory path ("pcvr/") or the file path ("pcvr/model.pt") as the `model_path`.

The optional `checksum` field is the SHA256 checksum of the model represented as a hexadecimal string. Model checksums are computed using the following steps:

1. Compute the SHA256 checksum for each individual model file.
2. Arrange the file checksums in ascending order based on their file paths.
3. Concatenate all the ordered file checksums into a single string.
4. Compute the final SHA256 checksum on the concatenated checksum string.

The listed steps are equivalent to the following bash command. This command works for both a directory model path and a single-file model path:

```
find <model_path> -type f -exec sha256sum {} \; | sort -k 2 | awk '{print $1}' | tr -d '\n' | sha256sum | awk '{print $1}'
```

The optional `warm_up_batch_request_json` is a JSON batch request used to warm up the model before any traffic is served. This request is parsed and sent to the model to trigger initialization during the loading phase, reducing the latency impact of model lazy initialization during the first request. The format of this warm-up request uses the same format as the input to the runInference JavaScript function, which is described later in this document.

Refer to the [B&A Inference Onboarding Guide][4] for more details about using the model configuration file with the Terraform configuration.

Note that after a model is loaded, its original metadata entry must be retained to keep the model. Removing the metadata will be treated as a model deletion, as explained in the next section.

### Model Deletion

Ad techs may want to delete stale models to free up memory for the inference sidecar. This can be accomplished by removing stale model entries from the model configuration file. During the periodic fetching process, the inference service checks whether any loaded model is absent from the configuration file. If such models are found, they are deleted from the inference sidecar.

For example, consider the inference service receiving its first model configuration file at time A, which instructs it to load “pcvr_v1” into the inference sidecar. At time B, if “pcvr_v1” is no longer needed and “pcvr_v2” is required instead, Ad techs can upload a new model configuration file to the cloud bucket. This operation will remove “pcvr_v1” from internal storage (since it is no longer listed in the configuration file) and attempt to load “pcvr_v2” from the cloud bucket. In other words, the inference sidecar synchronizes its internal model storage with the updated configuration file.

Time A

```json
{
  "model_metadata": [
    {
      "model_path": "pcvr_v1/",
      "checksum": "..."
    }
  ]
}
```

Time B

```json
{
  "model_metadata": [
    {
      "model_path": "pcvr_v2/",
      "checksum": "..."
    }
  ]
}
```

When switching model versions, any remaining traffic for the previous model version may need to be processed before fully transitioning to the new version. To avoid traffic disruptions during this process, the inference service can postpone the removal of the older model version. This is achieved by configuring the `eviction_grace_period_in_ms` parameter in the configuration file, which specifies the delay (in milliseconds) before the old model is evicted. For instance, assume that at time A, the following configuration is supplied, upon deletion, the “pcvr_v1” model will be retained for an additional 60 seconds even though “pcvr_v2” is loaded into the inference service. During this grace period, traffic directed to “pcvr_v1” will still be processed.

```json
{
  "model_metadata": [
    {
      "model_path": "pcvr_v1/",
      "checksum": "...",
      "eviction_grace_period_in_ms": 60000,
    },
  ]
}
```

#### In-Place Update

It is possible to update a model in place by overwriting it at the same model path in the model metadata. An in-place update is treated the same as a model deletion followed by a model addition with the previous `eviction_grace_period_in_ms` applied. This blocks the loading of new model content until the grace period expires, after which the new model content is loaded during the next polling cycle. The inference service detects this update by recognizing changes to the model checksum. When a model is updated in this way, all associated metadata, including warm_up_batch_request_json and `eviction_grace_period_in_ms`, is overwritten with the new values from the model configuration file after the previous grace period has expired and that version of the model is deleted. If the model's checksum remains unchanged between pollings, no update will be  triggered. The model along with its original metadata remains unaffected.

## JavaScript API

We expose the Inference service capabilities as JavaScript functions in ad tech code modules. Currently, the Inference service can be executed as a part of the proprietary bidding logic. Ad techs can query available models and issue batched inference service requests to a selected model set.

### JavaScript functions

#### getModelPaths

The `getModelPaths` function doesn’t accept arguments and returns the available models in the inference service sidecar. The returned result is represented in a serialized JSON array of model paths. For example, [“pcvr_v1/”, “pcvr_v2/”].

#### runInference

The `runInference` function accepts batch requests and returns batch responses.

#### Batch request

The batch inference service request is represented as a JSON object containing a top-level array. Each element in the array represents an individual request and is represented as a JSON object.

#### Individual request mandatory fields

- `model_path`: Specifies the target model for the request
- `tensors`: An array of JSON objects representing the input tensors to the model.

#### Tensor mandatory fields

- `data_type`: Specifies the data type of the tensor, chosen from DOUBLE, FLOAT, INT8, INT16, INT32, INT64.
- `tensor_shape`: An array specifying the shape of the tensor.
- `tensor_content`: A flattened row-major representation of the tensor values.
- `tensor_name`: Specifies the tensor's name to match the model's signature. This value is ignored for PyTorch models.

`tensor_content` values are represented as strings, with plans to eventually enable storage as numeric types, as numeric value support is currently under development.

The following illustrates a batch request involving two models, “pcvr_v1/” and “pcvr_v2/”:

```json
{
  "request": [
    {
      "model_path": "pcvr_v1/",
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
      "model_path": "pcvr_v2/",
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

- For a success inference, each tensor JSON object follows the same format as in the request case except that `tensor_content` directly contains numericals instead of strings.

- In the failure case, the `model_path` field is optional. It is populated when the inference directed towards a specific model fails and is left empty when an entire batch request fails. The response has a mandatory `error` field with each error composed of a JSON object with `error_type` and `description` string fields.
  We currently report the following error types:

  - UNKNOWN
  - INPUT_PARSING
  - MODEL_NOT_FOUND
  - MODEL_EXECUTION
  - OUTPUT_PARSING
  - GRPC

The `description` field propagates back the original Abseil error message from the inference sidecar C++ program.

The following are some illustrative scenarios for reference:

- **Batch with all successful outputs:** The top-level array contains multiple response objects, each with the `model_path` and `tensor` fields populated which indicates successful inference for each input in the batch:

```json
{
  "response": [
    {
      "model_path": "pcvr_1/",
      "tensors": [
        {
          "tensor_name": "PartitionedCall:0",
          "data_type": "FLOAT",
          "tensor_shape": [2, 1],
          "tensor_content": [0.1167, 0.1782]
        }
      ]
    },
    {
      "model_path": "pcvr_2/",
      "tensors": [
        {
          "tensor_name": "PartitionedCall:0",
          "data_type": "FLOAT",
          "tensor_shape": [2, 1],
          "tensor_content": [0.3422, 0.2346]
        }
      ]
    }
  ]
}
```

- **Batch with mixed successes and failures:** The top-level array contains a mix of response objects. Some have `model_path` and `tensors`, while others have populated the `error` field, indicating that some inferences succeeded while others failed.

```json
{
  "response": [
    {
      "model_path": "pcvr_1/",
      "tensors": [
        {
          "tensor_name": "PartitionedCall:0",
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

- **Complete batch failure:** The top-level array contains a single response object with a populated `error` field. The `model_path` field is absent in this scenario, as the failure happens at the batch level rather than at the model specific level.

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
  request: [{ model_path: modelPath, tensors: [input1, input2 /*...*/] }],
});

const inference_result = JSON.parse(runInference(batchRequest));

for (const response of inference_result.response) {
  if (response.error) {
    handleError(response.error.error_type);
  } else {
    console.log("Use inference output:", response.tensors);
  }
}
```

[1]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference_overview.md#inference-with-externally-provided-models
[2]: https://cloud.google.com/storage/docs/buckets
[3]: https://aws.amazon.com/s3
[4]: https://github.com/privacysandbox/protected-auction-services-docs/blob/main/inference-explainers/inference_onboarding_guide.md
[5]: https://github.com/steveganzy
