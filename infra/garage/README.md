# Garage (S3-compatible object storage), local setup

[Garage](https://garagehq.deuxfleurs.fr/) stores citizen-uploaded images. Locally it runs as a
single node via `docker-compose.yml`, configured by `garage.toml` in this folder.
Secrets (`GARAGE_RPC_SECRET`, `GARAGE_ADMIN_TOKEN`) come from `.env` and are never committed.

## One-time setup (after the first `docker compose up -d`)

A fresh Garage node has no storage layout, bucket or access key. Run these once per machine
(the data survives restarts in the `garage-meta`/`garage-data` volumes):

```bash
# Shortcut
alias garage="docker compose exec garage /garage"

# 1. Find the node ID (first column under HEALTHY NODES)
garage status

# 2. Give the node a storage role and apply the layout
garage layout assign -z dc1 -c 1G <NODE_ID>
garage layout apply --version 1

# 3. Create the image bucket
garage bucket create kommunfix-images

# 4. Create an access key for the backend (prints Key ID + Secret key ONCE)
garage key create kommunfix-backend

# 5. Allow the backend key to use the bucket
garage bucket allow --read --write --owner kommunfix-images --key kommunfix-backend
```

Copy the printed **Key ID** and **Secret key** into your local `.env` as
`S3_ACCESS_KEY_ID` / `S3_SECRET_ACCESS_KEY`. Never paste them anywhere else.

## Rules

- **Images are private.** Do **not** enable website access (`garage bucket website --allow`) and do
  not create anonymous/public keys. The backend serves images through access-controlled endpoints
  or short-lived presigned URLs.
- Only the backend holds S3 credentials. The frontend never talks to Garage with keys.
- `garage.toml` here is for local development only (single node, `replication_factor = 1`).

## Reset

```bash
docker compose down -v   # removes ALL local data (db + images), then redo the setup above
```
