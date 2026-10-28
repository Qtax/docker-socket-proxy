# Docker Socket Proxy

Allows you to control which features of docker to be exposed and available to others.

So that you do not have to expose the docker socket directly (that is equivalent to giving the user root access).

Based on https://github.com/Tecnativa/docker-socket-proxy


## Changes compared to `Tecnativa/docker-socket-proxy`

Notable changes:

- Simplified implementation to only two files
- Upgraded HAProxy to 3.4 LTS
- Fixed warnings
- Proxy config changes:
	- Allow `POST`s to start/stop/restart depending on corresponding setting, without the need to enable the `POST` setting
	- Set a 24h idle timeout for events (previously unlimited) and follow-log streams
		- Unlimited timeout may be needed in some cases/tools: https://github.com/Tecnativa/docker-socket-proxy/pull/98
	- Set a 24h tunnel idle timeout
	- Removed `option redispatch` (didn't do anything as it's only a single backend server)
	- Set `retries 0` instead of 3
	- Bound API route matching
		- So that for example `AUTH` allows `/auth`, and no other paths starting with `/auth`
	- Match API route names case-sensitively


## Example usage

Docker compose usage example:

```
services:
  docker-proxy:
    build: https://github.com/Qtax/docker-socket-proxy.git
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - DISABLE_IPV6=1
      - CONTAINERS=1
      - INFO=1
    cap_drop:
      - ALL
    security_opt:
      - no-new-privileges=true
```

Then other containers (on the same network) can use it by setting (or equivalent):

```
DOCKER_HOST=tcp://docker-proxy:2375
```

To update and rebuild you can run: `docker compose build --pull`
