# s3-postgres-backup

A small Rust utility that creates a PostgreSQL dump, compresses it, and uploads it to an S3-compatible bucket.

## Project Overview

s3-postgres-backup is a container-oriented PostgreSQL backup tool. It:

1. Reads runtime configuration from environment variables.

2. Executes pg_dump against a PostgreSQL DATABASE_URL.
   
3. Optionally limits backup scope by schema pattern.

4. Compresses the dump into a .tar.gz.

5. Uploads the archive to an S3 bucket.

The project is built written with Rust and packaged with Nix flakes into a Docker image.

## Features

- PostgreSQL backup via pg_dump.

- Optional schema filtering with DATABASE_SCHEMA_PATTERN (pg_dump -n ...).

- Timestamped backup filenames (backup-\<db\>-\<UTC timestamp\>.tar.gz).

- S3 upload via AWS Rust SDK.

- Reproducible build + container packaging via Nix flakes.

- CI pipeline (GitHub Actions) to build and publish image to Docker Hub on pushes to main.

## Configuration

Create a .env file (or provide environment variables another way):

Variables
```
DATABASE_URL (required): PostgreSQL connection string.
DATABASE_SCHEMA_PATTERN (optional): schema pattern passed to pg_dump -n.
AWS_S3_BUCKET (required): target bucket name.
AWS_S3_ENDPOINT (required): intended custom S3 endpoint.
AWS_S3_REGION (required): AWS region used by SDK config.
```
## Build & Deployment

Local build (dev image)

```bash
nix build .#devImage && ./result | docker load
```

Local run (interactive)

```bash
docker compose up
```

Current compose service starts with an interactive shell (command: ["bash"]), so you can run the backup binary manually inside the container.

Example:

```bash
docker exec -it <container_name> bash
s3-postgres-backup
```

## CI build & publish path

On push to `main`, GitHub Actions:

1. Builds `.#dockerImage` via Nix.
2. Loads the image into Docker.
3. Tags and pushes to Docker Hub using repository variables/secrets.

## Usage Examples

### Full database backup

```bash
export DATABASE_URL='postgres://user:pass@db:5432/app'
export AWS_S3_BUCKET='my-db-backups'
export AWS_S3_REGION='us-east-1'
export AWS_S3_ENDPOINT='https://s3.amazonaws.com'
unset DATABASE_SCHEMA_PATTERN

s3-postgres-backup
```

### Single schema backup pattern

```bash
export DATABASE_URL='postgres://user:pass@db:5432/app'
export DATABASE_SCHEMA_PATTERN='public'
export AWS_S3_BUCKET='my-db-backups'
export AWS_S3_REGION='us-east-1'
export AWS_S3_ENDPOINT='https://s3.amazonaws.com'

s3-postgres-backup
```

### Restore workflow

After downloading an archive:

```bash
tar -xvf backup-<db>-<timestamp>.tar.gz
pg_restore -d "<postgres_connection_string>" /path/to/extracted.dump
```
