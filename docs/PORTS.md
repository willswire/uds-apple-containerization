# Port Configuration

By default, this package exposes ports `80` (HTTP) and `443` (HTTPS) on the container's IP address, with a redirect from `80` to `443` within the Nginx configuration. The Kubernetes API server is forwarded to `localhost:6443`.

The `container` CLI does not support binding to privileged ports (< 1024) on localhost. To route `*.uds.dev` (which resolves to `127.0.0.1`) to the cluster for Istio SNI-based routing, macOS `pf` redirect rules are needed. See the [README](../README.md#routing-udsdev-to-the-cluster) for setup instructions.

## Exposing Extra Ports

Some packages may require additional TCP ports. Set `NGINX_EXTRA_PORTS` to include the additional ports you would like to expose through Nginx:

```bash
--set NGINX_EXTRA_PORTS="[<port>,9999]"
```

These ports are available on the container's IP address (shown during deployment).

> [!IMPORTANT]
> This configuration only supports forwarding traffic exposed over the `tenant` gateway in `uds-core` - if you need to expose traffic over another gateway this configuration will not work.
