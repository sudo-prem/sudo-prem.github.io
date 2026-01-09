+++
title = "Resurrecting a MacBook: Building a Private Cloud with Immich & Tailscale"
description = "Don't let that old Mac collect dust. Here's how I turned mine into a secure, self-hosted Google Photos alternative in one afternoon."
date = 2026-01-09
categories = ["self-hosting", "docker", "DIY"]
tags = ["immich", "tailscale", "colima", "macos", "homelab"]
toc = true
+++

## From Paperweight to Private Cloud

We all have one: that aging laptop sitting in a drawer, too slow for daily work but too valuable to toss. I decided to give my old MacBook a second life. The goal? A completely private, self-hosted photo server accessible from anywhere in the world—without paying a cent in subscription fees.

The stack is simple but powerful: **Immich** (the best open-source alternative to Google Photos) running on **Docker**, exposed securely via **Tailscale**.

![Photo of the old MacBook setup on a desk](/img/posts/macbook-setup-shot.png)
*The hardware: An old MacBook ready for its new life as a server.*

## Part 1: The Engine (Immich Setup)

Immich is heavy-duty software, but setting it up via Docker is surprisingly straightforward.

### 1. Prepping the Environment
I created a home for the app and grabbed the official Docker configuration files:

```bash
mkdir ./immich-app && cd ./immich-app
wget -O docker-compose.yml [https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml](https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml)
wget -O .env [https://github.com/immich-app/immich/releases/latest/download/example.env](https://github.com/immich-app/immich/releases/latest/download/example.env)
```

### 2. Configuration & Secrets
The `.env` file controls the show. I tweaked the defaults to secure the database and point the storage to a dedicated local folder.

> Update IMMICH_ADMIN_PASSWORD and DB_PASSWORD with strong passwords, set the UPLOAD_LOCATION and DB_DATA_LOCATION paths to your preferred local directories, and configure your timezone in TZ.

![Screenshot of diff showing the .env configuration](/img/posts/env-config-editor.png)
*Configuring the environment variables for security and storage.*

### 3. The macOS "Gotcha" (Colima Users Read This)
Since I run **Colima** instead of Docker Desktop, standard bind mounts can cause permission headaches. I modified `docker-compose.yml` to use named volumes for the database, ensuring stability.

> **The Fix:** Change the postgres volume from a direct path to a named volume.

```yaml
services:
  database:
    volumes:
      - pgdata:/var/lib/postgresql/data # Changed from direct path

volumes:
  pgdata: # Define the volume at the bottom
  model-cache:
```

With the config patched, I fired it up:
```bash
docker compose up -d
```

## Part 2: The Teleporter (Tailscale Access)

Getting Immich running locally is easy; securely accessing it from a coffee shop is usually hard. **Tailscale** makes it trivial.

I installed the Tailscale macOS client, logged in, and ran a single "magic" command in the terminal:

```bash
tailscale serve 2283
```

![Screenshot of the terminal running the tailscale serve command](/img/posts/tailscale-terminal.png)
*One command to expose the local server to my private network.*

Just like that, my local Immich instance (running on port 2283) was instantly accessible to every device on my Tailnet. No port forwarding, no firewall headaches, just an encrypted peer-to-peer tunnel.

## The Result

I now have a robust, AI-powered photo management system running on hardware I already owned.

* **Speed:** Local network uploads are blazing fast.
* **Privacy:** My data never leaves my hardware.
* **Access:** Using the Immich mobile app, I can back up photos from anywhere as if I were sitting next to the server.

![Screenshot of the Immich Dashboard populated with photos](/img/posts/immich-final-dashboard.png)
*Success: The Immich dashboard running smoothly on the resurrected Mac.*

If you have an old laptop gathering dust, this is the sign you've been waiting for. Wipe it, Dockerize it, and reclaim your digital memories.
