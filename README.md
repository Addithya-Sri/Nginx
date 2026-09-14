# Local Nginx Learning Environment

This project runs a small Nginx web server inside Docker. Nginx serves the
static files in `html/`, records requests in `logs/`, and exposes a simple
health check at `/health`.

## Project structure

```text
.
├── docker-compose.yml  # Defines the Nginx container and port/volume mappings
├── nginx.conf          # Nginx server configuration
├── html/
│   └── index.html      # Website served at http://localhost:8080
├── logs/
│   ├── access.log      # Requests (created by Nginx after startup)
│   └── error.log       # Nginx errors (created by Nginx after startup)
└── README.md
```

## How Docker and Nginx fit together

Docker runs the official lightweight `nginx:alpine` image as a single
container. The container listens on port 80. The mapping `8080:80` means that
port 8080 on your computer forwards to port 80 inside the container, so open
<http://localhost:8080> in a browser.

The Compose volume mounts keep the learning files on your computer:

| Host path | Container path | Purpose |
| --- | --- | --- |
| `./nginx.conf` | `/etc/nginx/nginx.conf` | Read-only Nginx configuration |
| `./html` | `/usr/share/nginx/html` | Read-only website files |
| `./logs` | `/var/log/nginx` | Writable access and error logs |

Edit `html/index.html` to change the page. Edit `nginx.conf` to practice
Nginx configuration; restart the container after configuration changes.

## Start and manage the environment

Run these commands from this project directory:

```bash
docker compose up -d
docker compose ps
docker compose logs nginx
docker compose exec nginx nginx -t
docker compose restart nginx
docker compose down
```

`docker compose up -d` starts the server in the background. `ps` shows its
status, `nginx -t` tests the configuration inside the running container,
`restart` applies configuration changes, and `down` stops and removes the
container (your local files remain).

To validate the Compose file without starting the container, use:

```bash
docker compose config
```

## Test the server

Open these URLs:

- <http://localhost:8080> — the static homepage
- <http://localhost:8080/health> — returns plain text `OK` with HTTP 200

The `/health` location is defined explicitly in `nginx.conf` and returns
`OK`. All other paths use the normal static-file location and are served from
`/usr/share/nginx/html` inside the container.

## View logs

Container logs can be viewed with:

```bash
docker compose logs nginx
docker compose logs --follow nginx
docker compose logs --since 2h nginx
```

The last command is supported by current Docker Compose versions. Because the
host `logs/` directory is mounted into the container, the same files can also
be inspected directly:

```text
logs/access.log
logs/error.log
```

After visiting the homepage or `/health`, the requests should appear in
`logs/access.log`. `logs/error.log` is available for Nginx errors and should
remain quiet during normal operation.

## Learning ideas

This clean starting point can later be extended to explore location blocks,
reverse proxying, upstreams, load balancing, headers, TLS/HTTPS, and Nginx
troubleshooting.
