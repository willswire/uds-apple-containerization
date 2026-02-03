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

`*.uds.dev` resolves to `127.0.0.1` via public DNS. Since the `container` CLI cannot bind to privileged ports (< 1024) on localhost, macOS `pf` redirect rules are needed to route traffic from `127.0.0.1:80`/`443` to the container's IP. This enables seamless browser access for Istio SNI-based routing.

After deployment, the container IP is printed. Use it to set up the redirect:

```bash
# Replace NODE_IP with the container IP shown after deployment (e.g. 192.168.64.10)
echo "rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> NODE_IP port 80
rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> NODE_IP port 443" | sudo pfctl -a com.apple/uds -f - -E
```

To remove the redirect rules:

```bash
sudo pfctl -a com.apple/uds -F all
```

> [!NOTE]
> The redirect rules persist until reboot or manual removal. They do not need to be re-applied after `container stop`/`start`, only after `container rm` and re-deploy (since the container IP may change).

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

## SSH Access

If working with a remote cluster over SSH, you can forward the necessary ports:

```console
ssh -N -L 80:localhost:80 -L 443:localhost:443 -L 6443:localhost:6443 <your-remote-host>
```

### Additional Details and Documentation

- [UDS Dev Stack](docs/DEV-STACK.md)
- [Configuring Minio](docs/MINIO.md)
- [DNS Assumptions](docs/DNS.md)
