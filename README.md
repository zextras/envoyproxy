# Envoyproxy

Envoyproxy is a packaging repository for the Envoy Proxy as part of the Carbonio platform. Envoy is a high-performance, open-source edge and service proxy designed for cloud-native applications. This repository provides pre-built binary packages for multiple Linux distributions, making it easier to deploy Envoy without the complexity of building from source.

**Note:** For Rocky Linux 8, the build process still compiles from source due to specific compatibility requirements.

## Quick Start

### Prerequisites

- Docker or Podman installed
- Make

### Building Packages

```bash
# Build packages for Ubuntu 22.04
make build TARGET=ubuntu-jammy

# Build packages for Rocky Linux 9
make build TARGET=rocky-9

# Build packages for Ubuntu 24.04
make build TARGET=ubuntu-noble
```

### Supported Targets

- `ubuntu-jammy` - Ubuntu 22.04 LTS
- `ubuntu-noble` - Ubuntu 24.04 LTS
- `rocky-8` - Rocky Linux 8
- `rocky-9` - Rocky Linux 9

### Configuration

You can customize the build by setting environment variables:

```bash
# Use a specific container runtime
make build TARGET=ubuntu-jammy CONTAINER_RUNTIME=docker

# Use a different output directory
make build TARGET=rocky-9 OUTPUT_DIR=./my-packages
```

## Installation

This package is distributed as part of the [Carbonio platform](https://zextras.com/carbonio).

### Ubuntu (Jammy/Noble)

```bash
apt-get install envoyproxy
```

### Rocky Linux (8/9)

```bash
yum install envoyproxy
```

## Envoy and Consul Compatibility

When using Envoy with Consul service mesh, ensure version compatibility. Refer to the [Consul Envoy compatibility table](https://www.consul.io/docs/connect/proxies/envoy) for supported version combinations.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for information on how to contribute to this project.

## License

The build scripts, patches, and configuration files in this repository are licensed under the GNU Affero General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details.

This repository does not contain the source code of the third-party project it packages. The build scripts download upstream sources at build time from its original location. The upstream project retains its own licenses, and the resulting built package/s are distributed under those original licenses. Please refer to the component's upstream documentation for specific licensing information.

Envoy Proxy is licensed under Apache-2.0 by the Envoy Project Authors. Please refer to the [upstream Envoy documentation](https://www.envoyproxy.io/) for specific licensing information about the Envoy software itself.
