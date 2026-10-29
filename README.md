# KR2 template

Counter service on Flask + Redis.

Local run: `pip install -r requirements.txt && flask --app app run`

## Variant 2

- Student: Seidov; hostname: `notes.devops.local`; application port: `8002`.
- Application and Redis run without root. Redis uses `redis:7-alpine`, AOF and
  the named volume `kr2_redis_data`; only `127.0.0.1:8002` is published.
- Nginx configuration: `/etc/nginx/sites-available/kr2`; certificate:
  `/etc/ssl/certs/kr2.crt`; private key: `/etc/ssl/private/kr2.key` (root, 600).
- The actual Redis password is generated only on the VM in `/home/devops/kr2/.env`
  (devops, 600). The inherited template password is retired. Do not reuse it.

Delivery from the repository root:

```bash
rsync -avz --delete --exclude=.env --exclude=.git --dry-run ./ devops:kr2/
rsync -avz --delete --exclude=.env --exclude=.git ./ devops:kr2/
```

On the VM, initialize `.env` once with a random password, then run
`docker compose up -d --build --wait` from `~/kr2`. Do not export
`REDIS_PASSWORD` in the shell or overwrite an existing `.env` during delivery.
Wait for `/health` before sending requests to a newly started application.

Checks: `docker compose ps`, `docker compose top`, `docker compose port app 8002`,
and `curl --cacert kr2.crt https://notes.devops.local/whoami` on the client.
An unauthenticated Redis PING returns NOAUTH. The counter continues after
`docker compose down` and `up`; do not use `down -v` for this project.

The extended audit is in `scripts/audit.sh`. The VM copy is also installed as
`/home/devops/devops-course-2026/scripts/audit.sh`. Run after `sudo -v`:
19 checks pass. Publishing `8002:8002` makes the new section fail with exit 1;
the saved configuration restores loopback publication and exit 0.

The original PR6/PR7 resources and UFW rules are preserved.
Pull request `kr2/variant-2` to `main` remains open until the defense.
