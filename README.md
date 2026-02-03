# UDS Apple Containerization Environment

> [!IMPORTANT]
> This package should only be used for development and testing purposes. It is not intended for production use and all data is overwritten when the package is re-deployed.

This zarf package serves as a development and test environment for testing [UDS Core](https://github.com/defenseunicorns/uds-core), individual UDS Capabilities, and UDS capabilities aggregated via the [UDS CLI](https://github.com/defenseunicorns/uds-cli). It uses Apple's [Containerization](https://github.com/apple/containerization) framework to run a Kubernetes cluster in a lightweight Linux virtual machine on macOS.

If working with a remote cluster over SSH, you can use SSH port-forwarding to connect:

```console
ssh -N -L 8080:localhost:8080 -L 8443:localhost:8443 -L 6443:localhost:6443 <your-remote-host>
```

> [!NOTE]
> The default host ports are 8080 (HTTP) and 8443 (HTTPS) to avoid requiring root privileges. These map to ports 80 and 443 inside the container.

## Prerequisites

- [UDS CLI](https://uds.defenseunicorns.com/reference/cli/quickstart-and-usage/#install): version 0.27.0 or later
- [Apple Containerization framework](https://github.com/apple/containerization): the `container` CLI
- macOS 26 (Tahoe) or later on Apple Silicon

## Deploy

To deploy the package:

<!-- x-release-please-start-version -->

`uds zarf package deploy oci://ghcr.io/willswire/uds-apple-containerization:0.19.4`

<!-- x-release-please-end -->

## Create

This package is published via CI, but can be created locally with the following command:

`uds run build`

## Remove

To delete your cluster:

`container rm -f uds-control-plane` (where `uds` is the default cluster name)

## Start and Stop

To stop and start an existing cluster gracefully, use the following prior to host hibernation, suspension, restart, or shutoff:

```bash
# to stop the default UDS cluster
container stop uds-control-plane

# to start the default UDS cluster
container start uds-control-plane
```

### Additional Details and Documentation

- [UDS Dev Stack](docs/DEV-STACK.md)
- [Configuring Minio](docs/MINIO.md)
- [DNS Assumptions](docs/DNS.md)
