# Salestock protocol buffer compiler plugin for [golang](https://golang.org)

This is a fork of [golang/protobuf](https://github.com/golang/protobuf)'s `protoc-gen-go`. This fork allow to use `import_prefix` option when an import path in proto file matched prefix defined by `prefix_path` option.

## Background

We are separating our protobuf files according to our domains under `shared-libs/proto`. The current library ([golang/protobuf](https://github.com/golang/protobuf)), works totally fine when working with standalone file. But then some domain, have to use other domain proto files to work correctly. In this scenario,  [golang/protobuf](https://github.com/golang/protobuf) library doesn't work very well in a monorepo environment.

Take a look at this example:

Let say we have `payment.proto` like this:

```proto
// proto/domain/payment/payment.proto
syntax="proto3";

package net.salestock.proto.domain.payment;

import "google/protobuf/wrappers.proto";

message Payment {
    string id = 1;
}
```

And an `order.proto` which need payment data was defined like below

```proto
// proto/domain/order/order.proto
syntax="proto3";

package net.salestock.proto.domain.order;

import "google/protobuf/wrappers.proto";
import "proto/domain/payment/payment.proto";

message Order {
    string id = 1;
    net.salestock.proto.domain.payment.Payment payment = 2;
}
```

The `order.pb.go`generated golang source from `order.proto` are like below:

```go
// ssource/<service>/proto/domain/order/order.pb.go
// ...
import google_protobuf "github.com/golang/protobuf/proto";
// This doesn't work in a monorepo environment,
// since we need to use the full repository path to import correctly 
import net_salestock_proto_domain_payment "proto/domain/payment";
```

When using `import_prefix=github.com/salestock/ssource/` provided by  [golang/protobuf](https://github.com/golang/protobuf) library, the generated code looks like below:

```go
// ssource/<service>/proto/domain/order/order.pb.go
// ...
// The import prefix applied to google wrappers too
// This generate compile error since the path does not exists
import google_protobuf "github.com/salestock/ssource/github.com/golang/protobuf/wrappers";
// This import works correctly
import net_salestock_proto_domain_payment "github.com/salestock/ssource/proto/domain/payment";
```

We need to be able to make the `import_prefix` options applied ONLY to specific import path. The idea is to add a new option to the plugin to match import path prefix. The expected generated code with this new option should looks like below:

```go
// github.com/salestock/ssource/<service>/proto/domain/order/order.pb.go
// ...
// The import prefix applied to google wrappers too
// This generate compile error since the path does not exists
import google_protobuf "github.com/golang/protobuf/wrappers";
// This import works correctly
import net_salestock_proto_domain_payment "github.com/salestock/ssource/<service_name>/proto/domain/payment";
```

## Installation

- Run `ssi go get github.com/golang/protobuf/proto`
- Run `ssi go get github.com/salestock/ssource/shared-libs/go/protoc-gen-go`
- Then use `ssi init` or `ssi compile-protobuf` to generate go source code from
  protobuf files defined in `service.yaml`

## Example use

Please see `ssource/shared-libs/proto/domain/order` and `ssource/order/proto`

## TODO

When there are official fix from original [golang/protobuf](https://github.com/golang/protobuf) issue [#333](https://github.com/golang/protobuf/issues/333) this plugin will be deprecated
