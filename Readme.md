How to run a cloudflare tunnel via docker-compose

In Cloudflare's Tunnels page, when you select docker environment, you are given a similar docker command:

```
docker run cloudflare/cloudflared:2022.4.1 tunnel --no-autoupdate run --token 1234567890abcdefghijklmnopqrstuvwxyz
```

The important part here is the token `1234567890abcdefghijklmnopqrstuvwxyz`. This is secret, so you cannot just paste it in the docker-compose file/push it to github.

Instead, you can set the environment variable `TUNNEL_TOKEN` to this value. You can do that via an `.env` file, or any other way you wish. 
`cloudflared` reads this env value and uses it as if it were provided via CLI.

# Example

- `docker-compose.yml`

```yaml
version: "2.4"
services:
  cloudflared-laptop:
    image: cloudflare/cloudflared:2022.4.1
    command: tunnel --no-autoupdate run
    restart: unless-stopped
    environment:
      TUNNEL_TOKEN:
```

- `.env`

```properties
TUNNEL_TOKEN=1234567890abcdefghijklmnopqrstuvwxyz
```

Now you can start and destroy your docker-compose project anytime and the tunnel will be active only when compose is running.

You can see the 2 files in this repo as well. Replace the contents inside `.env` then run `docker-compose up`.

# Networking

One thing  you still need to solve is how will your running application be accessed from the docker container running `cloudflared`. This can be configured in cloudflare under `tunnel -> public hostname -> service` field.
I present 2 options: 
- run all services attached to the same docker network. In this case set in `service` the hostname `localhost` and whatever port you need
- access a service from the docker host (ie. for development). In this case set in `service` the hostname `host.docker.internal` and whatever port you need, as documented [here](https://docs.docker.com/desktop/windows/networking/#use-cases-and-workarounds)

For me the final `service` value is `http://host.docker.internal:13337`.
