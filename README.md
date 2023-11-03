# Docker NordVPN

[![GitHubPackage][GitHubPackageBadge]][GitHubPackageLink]
[![DockerPublishing][DockerPublishingBadge]][DockerLink]
[![DockerSize][DockerSizeBadge]][DockerLink]
[![DockerPulls][DockerPullsBadge]][DockerLink]

### The NordVPN client for Docker

Leveraging the latest native NordVPN client, iptables and the Nord API to create the fastest, most stable connection possible.

## The Essentials

Build based on:

- NordVPN `3.16.7`
- Ubuntu `22.04`
  - Updated nighty

Examples of use:

- [nordvpn_proxy.yml](examples/)

Docker Hub repository:

- <https://hub.docker.com/r/tmknight88/nordvpn>

Optimized for NordLynx:

- NordLynx is NordVPN's fast/stable implementation of Wireguard; it is the recommended and default [TECHNOLOGY](#env-technology)

## Requirements

Wireguard on the host

- You <ins>must</ins> install Wireguard on your <ins>host</ins> in order to leverage NordLynx

Capabilities

- [NET_ADMIN](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)

Environment

- [TOKEN](#env-token)

  - **ONLY TOKENS ARE VIABLE IN A CONTAINER**
  - The use of USERNAME and PASSWORD has been deprecated wherein only TOKEN or login via browser are accepted with the Linux client

- [NET_LOCAL](#env-netlocal)

  - Technically not required for the container to work, but it should be set if local traffic is to be routed through NordVPN

## Recommendations

IPv6

- IPv6 support is limited and generally [not supported](https://nordvpn.com/blog/ipv4-vs-ipv6/#:~:text=You%20might%20be%20wondering%20what,tunnel%20with%20the%20IPv4%20protocol.) by most VPN providers at this time
- Therefore, it is recommended to disable IPv6 support in your container via [sysctl](https://docs.docker.com/engine/reference/commandline/run/#configure-namespaced-kernel-parameters-sysctls-at-runtime):

  - `net.ipv6.conf.all.disable_ipv6=1`

DNS

- Prior to establishing the tunnel, the host DNS settings will be used
- If you are concerned with DNS leakage (which will only be nordvpn.com), you should set [docker DNS](https://docs.docker.com/config/containers/container-networking/#dns-services)

  - Note, this is not the same as the [DNS environment variable](#env-dns)

## Environment Variables

Generally, the default settings will provide a great experience, however, several environment variables are available to provide flexibility:

| Variable                        | Default                  | Description                                                                                                                                                                                                                               |
|:-------------------------------:|:------------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| **BYPASS_LIST** |   | Comma-separated list of domain names that should bypass VPN (i.e. these connections should not be secured); if set, `FIREWALL` will default to FALSE                                                                                      |
| **CHECK_CONNECTION_INTERVAL**   | 60                       | Time in seconds to check connection state and remediate as required                                                                                                                                                                       |
| **CHECK_CONNECTION_URL**        | <https://www.google.com> | URL used by `CHECK_CONNECTION_INTERVAL`                                                                                                                                                                                                   |
| **CONNECTION_FILTERS**<span id="env-filters"></span> |                          | To connect to the fastest, lowest load server of special interest. Use the [NordVPN API](#api) to help craft your filters; largely for OpenVPN, though useful with NordLynx when wanting to set a specific country/city (e.g `filters[country_city_id]=8980922`)                                        |
| **CONNECT** ||Provide a [country] (`Australia`), [server] (`jp35`), [country_code] (`us`), [city] (`Hungary Budapest`) or [group] (`Onion_Over_VPN`) (note CONNECT overrides CONNECTION_FILTERS; use one or the other)|
| **CYBER_SEC**                   | FALSE                    | Learn more at [NordVPN](https://nordvpn.com/features/cybersec/) (TRUE/FALSE)                                                                                                                                                              |
| **DNS**<span id="env-dns"></span> |                          | A comma-separated list of IPv4/IPv6 addresses to be set as the VPN tunnel DNS servers, or non-IP hostnames to be set as the tunnel's DNS search domains (leave unset to use NordVPN servers)                                          |
| **FIREWALL**                    | TRUE                     | Use the NordVPN firewall over iptables (TRUE/FALSE; will default to FALSE when `BYPASS_LIST` in use)                                                                                                                                      |
| **KILLSWITCH**                  | TRUE                     | Use the NordVPN kill switch; `FIREWALL` must also be TRUE (TRUE/FALSE)                                                                                                                                                                    |
| **NET_LOCAL**<span id="env-netlocal"></span> |                          | Add a route to local IPv4 network once the VPN is up; the Docker network is automatically added; must be CIDR IPv4 format (e.g. `192.168.1.0/24`)                                                                                         |
| **NET6_LOCAL**                  |                          | Add a route to local IPv4 network once the VPN is up; the Docker network is automatically added; must be CIDR IPv6 format (e.g. `fe00:d34d:b33f::/64`)                                                                                    |
| **OBFUSCATE**                   | FALSE                    | Only valid when using TECHNOLOGY OpenVPN; learn more at [NordVPN](https://nordvpn.com/features/obfuscated-servers/) (TRUE/FALSE)                                                                                                          |
| **PORT_RANGE**                  |                          | Port range to whitelist for both UDP and TCP; (e.g. `PORT_RANGE=9091 9095`)                                                                                                                                                               |
| **PORTS**                       |                          | Semicolon delimited list of ports to whitelist for both UDP and TCP; (e.g `PORTS=9091;9095`)                                                                                                                                              |
| **POST_CONNECT**                |                          | Command to execute after successful connection                                                                                                                                                                                            |
| **PRE_CONNECT**                 |                          | Command to execute before attempt to connect                                                                                                                                                                                              |
| **PROTOCOL**                    | UDP                      | Only valid when using TECHNOLOGY OpenVPN (TCP/UDP)                                                                                                                                                                                        |
| **REFRESH_CONNECTION_INTERVAL** | 120                      | Time in minutes to trigger VPN reconnection to help ensure best connection available (0 = disable)                                                                                                                                                      |
| **TECHNOLOGY**<span id="env-technology"></span> | NordLynx                 | Specify the VPN Technology to use (NordLynx/OpenVPN)                                                                           |
| **TOKEN**<span id="env-token"></span> |                          | Generated from your [NordVPN account web portal](https://my.nordaccount.com/dashboard/nordvpn/)                                                                                                                    |

### Using a callback URL for authentication

To use a callback URL for authentication, start the docker container without providing token. When running in interactive mode, the authentication process gets started automatically. If you're running the container in non-interactive mode you need to run the following command:

```sh
docker exec -i <container_id> nord_login_callback
```

The full command will be provided in the logs of the container when starting up.

NordVPN will show an URL in the console that can be used to retrieve a callback URL. Open this URL in your browser and login with your NordVPN account. After logging in you need to copy the link of the "Continue" text (Your browser might try to open the NordVPN application if you have that installed) and paste it into the prompt. After this the setup will continue.

Keep in mind that each callback link can only be used once. The link you need to paste will look like this:

```
nordvpn://login?action=login&exchange_token=MGFlY2E1NmE4YjM2NDM4NjUzN2VjOWIzYWM3ZTU3ZDliNDdiNzRjZTMwMjE5YjkzZTNhNTI3ZWZlOTIwMGJlOQ%3D%3D&status=done
```

To reauthenticate you should use the `docker exec` command. Alternatively, you can recreate the container and delete its volume or delete the file that keeps track of the callback URL (if you have access to the volume of the container). The callback URL gets stored in the following file within the container: `/var/lib/nordvpn/previous_callback.txt`

## Troubleshooting

- Ensure you have read all of the above information
- Ensure you have pulled the latest available image
  - Use `--force-recreate` to be sure
- Check and double-check all of your values
- Perform the following:
  - Start a basic container:
    - docker run -it --rm --name=nordvpn-tmp tmknight88/nordvpn:latest bash
  - Perform the following in the container:
    - nordvpnd &
    - nordvpn login --token [your token]
    - nordvpn connect
  - If basic container connectes without issue, then slowly/one-at-a-time, start applying any cusomizations and go through the previous steps with each change (yes, laborious, but that's what it takes)
    - docker run -it --rm --name=nordvpn-tmp -v [something] tmknight88/nordvpn:latest bash
    - docker run -it --rm --name=nordvpn-tmp -v [something] -e [something else] tmknight88/nordvpn:latest bash
    - docker run -it --rm --name=nordvpn-tmp -v [something] -e [something else] -e [and so on] tmknight88/nordvpn:latest bash
- If you've performed all of the above without determining the issue, feel free to open an issue
  - Be sure to include your log entries and be as descriptive as possible

## Additional Information

Using the NordVPN API<span id="api"></span>

- <https://sleeplessbeastie.eu/2019/02/18/how-to-use-public-nordvpn-api>

## Credits

- [bubuntux](https://github.com/bubuntux)
- [kubernetes-sigs](https://github.com/kubernetes-sigs)

## Disclaimers

This project is independently developed for personal use; there is no affiliation with NordVPN or Nord Security companies.  Nord Security companies are not responsible for, nor have control over, the nature, content and availability of this project.

[GitHubPackageBadge]: https://github.com/tmknight/docker-nordvpn/actions/workflows/github-package.yml/badge.svg
[GitHubPackageLink]: https://github.com/tmknight/docker-nordvpn/pkgs/container/nordvpn
[DockerPublishingBadge]: https://github.com/tmknight/docker-nordvpn/actions/workflows/docker-publish.yml/badge.svg
[DockerPullsBadge]: https://badgen.net/docker/pulls/tmknight88/nordvpn?icon=docker&label=Docker+Pulls&labelColor=black&color=green
[DockerSizeBadge]: https://badgen.net/docker/size/tmknight88/nordvpn?icon=docker&label=Docker+Size&labelColor=black&color=green
[DockerLink]: https://hub.docker.com/r/tmknight88/nordvpn
