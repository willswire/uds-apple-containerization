# UDS Apple Containerization Environment

> [!IMPORTANT]
> This package should only be used for development and testing purposes. It is not intended for production use and all data is overwritten when the package is re-deployed.

This zarf package serves as a development and test environment for testing [UDS Core](https://github.com/defenseunicorns/uds-core), individual UDS Capabilities, and UDS capabilities aggregated via the [UDS CLI](https://github.com/defenseunicorns/uds-cli). It uses Apple's [Containerization](https://github.com/apple/containerization) framework to run a Kubernetes cluster in a lightweight Linux virtual machine on macOS.

The Kubernetes API is forwarded to `localhost:6443`. HTTP/HTTPS (ports 80/443) are served on the container's IP address, which is printed after deployment.

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

## Routing `*.uds.dev` to the Cluster

`*.uds.dev` resolves to `127.0.0.1` via public DNS. To route this traffic to the cluster for Istio SNI-based routing, use `socat` to forward ports 80 and 443 from localhost to the container's IP.

After deployment, the container IP is printed. Use it to start the forwarders:

```bash
# Replace NODE_IP with the container IP shown after deployment (e.g. 192.168.64.18)
sudo true;
sudo socat TCP-LISTEN:443,bind=127.0.0.1,reuseaddr,fork TCP:NODE_IP:443 &
sudo socat TCP-LISTEN:80,bind=127.0.0.1,reuseaddr,fork TCP:NODE_IP:80 &
```

To stop the forwarders:

```bash
sudo killall socat
```

> [!NOTE]
> Install socat with `brew install socat` if not already available. The forwarders must be restarted after `container rm` and re-deploy (since the container IP may change).

## Custom Kernel

This package uses a custom Linux kernel built from the [Apple Containerization](https://github.com/apple/containerization) framework's kernel configuration (included as a git submodule). This is required because the kernel shipped with the current Containerization framework release lacks `ip6tables` support needed by Istio CNI ambient mode.

After cloning (with `--recurse-submodules`), build the kernel before deploying:

```bash
uds run build-kernel
```

This produces `containerization/kernel/vmlinux`, which is automatically used when creating the cluster.

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
