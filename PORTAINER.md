# Deploy from GitHub with Portainer

This repository can be deployed directly as a Portainer Git stack. Portainer
clones the complete repository and builds the application images from the
Dockerfiles in this repository.

## Stack settings

In Portainer, open **Stacks**, select **Add stack**, and choose **Repository**.
Use these settings:

```text
Repository URL:        https://github.com/marttrach/THSR-Sniper.git
Repository reference: refs/heads/main
Compose path:          docker-compose.yml
```

The application services use `pull_policy: build`, so every stack update builds
the current Git checkout instead of silently using the upstream Docker Hub
images.

## Required environment variables

Generate keys locally first:

```bash
python3 generate_keys.py
```

Do not commit the generated `.env` file. Copy its `SECRET_KEY` and
`ENCRYPTION_KEY` values into the stack's **Environment variables** section,
then add the database settings below with strong, unique passwords:

```env
SECRET_KEY=replace-with-the-generated-secret-key
ENCRYPTION_KEY=replace-with-the-generated-fernet-key
MYSQL_ROOT_PASSWORD=replace-with-a-strong-root-password
MYSQL_DATABASE=thsr_sniper
MYSQL_USER=thsr
MYSQL_PASSWORD=replace-with-a-strong-password
```

`SECRET_KEY`, `ENCRYPTION_KEY`, `MYSQL_ROOT_PASSWORD`, and `MYSQL_PASSWORD` are
required. Portainer will stop with a clear error if any of them are missing.

The repository includes an intentionally empty `stack.env` placeholder and the
services reference it with `env_file`. Keep credentials in Portainer; do not add
secret values to the committed placeholder.

## GitOps updates

Enable **GitOps updates** for the stack. Either configure polling (for example,
every five minutes) or use the webhook URL provided by Portainer in a GitHub
push webhook. On a new commit, Portainer fetches the repository, rebuilds the
images, and recreates the changed containers.

## Network exposure

The default host ports are:

- `3000`: frontend (the recommended application entry point)
- `48000`: booking API (container port `8000`)
- `48001`: authentication API (container port `8001`)
- `8080`: phpMyAdmin

MySQL is intentionally not published on the host. Other services access it as
`mysql:3306` through the internal Compose network.

Application state is persisted with TrueNAS host bind mounts:

```text
/mnt/NEW_PA9A1_2T/Truenas_App/THSR/data  -> /app/data
/mnt/NEW_PA9A1_2T/Truenas_App/THSR/mySQL -> /var/lib/mysql
```

Create both host directories before deploying and ensure the container users
can write to them.
