> FLEDGE has been renamed to Protected Audience API. To learn more about the name change, see the [blog post](https://privacysandbox.com/intl/en_us/news/protected-audience-api-our-new-name-for-fledge)

**Authors:** <br>
[Shruti Agarwal](https://github.com/a-shruti), Google Privacy Sandbox<br>
[Peter Meric](https://github.com/pmeric), Google Privacy Sandbox<br>
[Brian Schneider](https://github.com/bjschnei), Google Privacy Sandbox<br>
[Edward Gathuru](https://github.com/eggathuru), Google Privacy Sandbox<br>

# Roma Bring-Your-Own-Binary

Roma is a C++ library used for executing untrusted code in a secure, isolated environment.

Currently, Roma supports ad tech-defined functions implemented in Javascript and Web Assembly. In this document, we propose an extension to Roma to allow binary executables to be provided as UDFs.

## Overview

In Privacy Sandbox, Roma is a C++ library used by Protected Audience's trusted servers to securely execute ad tech-developed functions—referred to as user-defined functions (UDFs) within a secure, isolated environment. Central to this sandboxing is the requirement that the UDF execution handles requests and associated data without any discernible side effects.

Roma's current design uses [Sandbox2](https://developers.google.com/code-sandboxing/sandbox2) and V8 as the execution engine, imposing certain limitations on ad techs that restrict them to the use of JavaScript and WebAssembly (WASM).

This explainer presents an expansion in Roma's functionality to execute binaries compiled from languages such as C/C++ and Go.

## Why Bring-Your-Own-Binary?

Bring-Your-Own-Binary can reduce costs to ad tech by doing the following:

* **Reducing the barrier-to-entry** \- less code to change and time to migrate.
* **Improving latency and reducing cost** 💰💸- our internal benchmarks have shown compute intensive UDFs implemented in C++ and Go have lower latency than equivalent ones written in WASM or JS.

Roma Bring-Your-Own-Binary uses a single instance of a double-sandboxed Virtual Machine Monitor (VMM) called [gVisor](https://gvisor.dev/).  Inside this sandbox, privacy requirements are met through per-process isolation. This involves calling `clone` with appropriate flags, creating `pivot_root` for file system isolation, executing the supplied UDF, waiting for execution to complete and then cleaning up the UDF process and `pivot_root`.

## Usage

This section describes how Roma Bring-Your-Own-Binary might be used.

BYOB integration will be available  through Trusted Execution Environment (TEE) based servers : [Key/Value service](https://github.com/privacysandbox/protected-auction-services-docs/blob/69cef5a4ebb0b4f3077d93e94a9cd9bb56686c54/key_value_service_user_defined_functions.md), [Bidding & Auction services](https://github.com/privacysandbox/protected-auction-services-docs/blob/ef04c4c6f8d788534130938750ae5573691a66dc/bidding_auction_services_api.md). Browsers will continue to  support Javascript or WASM-based UDF execution based on [V8](https://v8.dev/) engine, while TEE based servers depending on Roma for adtech's UDF execution will continue to [support v8](https://github.com/privacysandbox/data-plane-shared-libraries/tree/619fc5d4b6383422e54a3624d49a574e56313bc8/src/roma/sandbox/js_engine/v8_engine).

To simplify UDF debugging and testing, the team plans to release an SDK tool to help the UDF developer determine if their binary meets the UDF requirements regardless of their development environment.

A REPL CLI tool will allow the developer to run their UDF binary in a sandboxed setting in production or debug modes, making it easier to test or debug. A microbenchmark CLI tool will also be offered for benchmarking UDF binaries.

The BYOB SDK for a given UDF contains the Protocol Buffer (protobuf) spec defining the messages pertinent to the UDF input and output. This protobuf spec defines the expected input and output for the binary.

Protobuf messages are used for requests and responses. Communications with the UDF are facilitated by protobuf messages transmitted using a file descriptor.

The specification of the UDF communication protocol can be found in [doc](https://github.com/privacysandbox/data-plane-shared-libraries/blob/619fc5d4b6383422e54a3624d49a574e56313bc8/docs/roma/byob/sdk/docs/udf/Communication%20Interface.md).
 The following section offers an illustrative example.

### Example

The example below demonstrates how BYOB can be used.
Assume you have been supplied with the following proto as the specification for your UDF.

```
// Echo UDF: An API which reads the request and echoes the response.
// The Echo UDF request.
// Request: EchoRequest
// Response: EchoResponse

// Request message for the Echo function.
message EchoRequest {
  // Message to be echoed.
  bytes message = 1;
}

// The Echo UDF response.
// Response message for the Echo function.
message EchoResponse {
  // Response which would be the a copy of the message sent in EchoRequest.
  bytes message = 1;
}
```

A UDF written in C++ might look like the following. For additional examples in C++ and other languages, refer to [our code repository](https://github.com/privacysandbox/data-plane-shared-libraries/tree/619fc5d4b6383422e54a3624d49a574e56313bc8/src/roma/byob/example).

```c
#include <iostream>

#include "google/protobuf/any.pb.h"
#include "google/protobuf/util/delimited_message_util.h"
#include "src/roma/byob/example/example.pb.h"

using ::privacy_sandbox::server_common::byob::example::EchoRequest;
using ::privacy_sandbox::server_common::byob::example::EchoResponse;

EchoRequest ReadRequestFromFd(int fd) {
  google::protobuf::Any any;
  google::protobuf::io::FileInputStream stream(fd);
  google::protobuf::util::ParseDelimitedFromZeroCopyStream(&any, &stream,
                                                           nullptr);
  EchoRequest req;
  any.UnpackTo(&req);
  return req;
}

void WriteResponseToFd(int fd, EchoResponse resp) {
  google::protobuf::Any any;
  any.PackFrom(std::move(resp));
  google::protobuf::util::SerializeDelimitedToFileDescriptor(any, fd);
}

int main(int argc, char* argv[]) {
  if (argc != 2) {
    std::cerr << "Expecting exactly one argument";
    return -1;
  }
  int fd = std::stoi(argv[1]);
  // Any initialization work can be done before this point.
  // The following line will result in a blocking read being performed by the
  // binary i.e. waiting for input before execution.
  // The EchoRequest proto is defined by the Trusted Server team. The UDF reads
  // request from the provided file descriptor.
  EchoRequest request = ReadRequestFromFd(fd);

  EchoResponse response;
  response.set_message(request.message());

  // Once the UDF is done executing, it should write the response (EchoResponse
  // in this case) to the provided file descriptor.
  WriteResponseToFd(fd, std::move(response));
  return 0;
}
```

This C++ code should be compiled to a binary and provided to the server.

## BYOB availability

Roma BYOB is open-sourced and the code can be found on [GitHub](https://github.com/privacysandbox/data-plane-shared-libraries/tree/619fc5d4b6383422e54a3624d49a574e56313bc8/src/roma/byob).

For details about the execution environment and communication protocol, check out [documentation](https://github.com/privacysandbox/data-plane-shared-libraries/tree/619fc5d4b6383422e54a3624d49a574e56313bc8/docs/roma/byob/sdk/docs/udf).

## Github issues

For questions and bug reports, file an [issue](https://github.com/WICG/protected-auction-services-discussion) on GitHub.
