# Port Configuration

By default, this package exposes ports `80` and `443` with a redirect from `80` to `443` within the Nginx configuration. This works for most packages however some may require additional TCP ports to be opened in order to provide / test all of their functionality.

## Exposing Extra Ports

Set `NGINX_EXTRA_PORTS` to include all of the additional ports you would like to expose:

```bash
--set NGINX_EXTRA_PORTS="[<port>,9999]"
```

The package automatically handles both the container port forwarding and the Nginx configuration — no separate arguments are needed.

> [!IMPORTANT]
> This configuration only supports forwarding traffic exposed over the `tenant` gateway in `uds-core` - if you need to expose traffic over another gateway this configuration will not work.
