# UDS Apple Containerization Environment

> [!IMPORTANT]
> This package should only be used for development and testing purposes. It is not intended for production use and all data is overwritten when the package is re-deployed.

This zarf package serves as a development and test environment for testing [UDS Core](https://github.com/defenseunicorns/uds-core), individual UDS Capabilities, and UDS capabilities aggregated via the [UDS CLI](https://github.com/defenseunicorns/uds-cli). It uses the `cluster` CLI (backed by Apple's [Containerization](https://github.com/apple/containerization) framework) to run a Kubernetes cluster on macOS.

The Kubernetes API is forwarded to `localhost:6443`. HTTP/HTTPS (ports 80/443) are served on the container's IP address, which is printed after deployment.

## Quick Start

```sh
# Install Apple's container CLI and start the container system
brew install container
container system start --enable-kernel-install

# Ensure the `cluster` CLI is installed and on your PATH

# Run the default UDS task which will:
# - Build the dev package
# - Deploy the dev package
# - Validate the cluster by performing a Zarf init
uds run

# Build and Deploy UDS Core Slim Dev
cd uds/
uds create --confirm
uds deploy --confirm uds-bundle-core-slim-dev-arm64-0.60.1.tar.zst
```

## Routing `*.uds.dev` to the Cluster

`*.uds.dev` resolves to `127.0.0.1` via public DNS. To route this traffic to the cluster for Istio SNI-based routing, use `socat` to forward ports 80 and 443 from localhost to the container's IP.

After deployment, you can dynamically fetch the node IP and start the forwarders like this:

```bash
NODE_IP="$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type==\"InternalIP\")].address}')"

sudo true;
sudo socat TCP-LISTEN:443,bind=127.0.0.1,reuseaddr,fork TCP:${NODE_IP}:443 &
sudo socat TCP-LISTEN:80,bind=127.0.0.1,reuseaddr,fork TCP:${NODE_IP}:80 &
```

To stop the forwarders:

```bash
sudo killall socat
```

> [!NOTE]
> Install socat with `brew install socat` if not already available. The forwarders must be restarted after `cluster delete` and re-deploy (since the node IP may change).

## CoreDNS Overrides

This package patches CoreDNS to import a custom override file and mounts the `coredns-custom` ConfigMap into CoreDNS so in-cluster `*.uds.dev` names resolve to the Istio gateways instead of `127.0.0.1`.

The patches live in:

- `manifests/coredns-corefile-import-patch.yaml` (Corefile import)
- `manifests/coredns-deployment-patch.yaml` (volume + mount)

## Remove

To delete your cluster:

`cluster delete --name uds` (where `uds` is the default cluster name)

## Start and Stop

To stop and start an existing cluster gracefully, use the following prior to host hibernation, suspension, restart, or shutoff:

```bash
# to stop the default UDS cluster
cluster stop --name uds

# to start the default UDS cluster
cluster start --name uds
```

### Additional Details and Documentation

- [UDS Dev Stack](docs/DEV-STACK.md)
- [Configuring Minio](docs/MINIO.md)
- [DNS Assumptions](docs/DNS.md)
