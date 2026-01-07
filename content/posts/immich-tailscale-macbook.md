+++
title = "Breathing New Life into an Old MacBook with Immich and Tailscale"
description = "How I turned an unused MacBook into a personal photo server using Immich and Tailscale for secure remote access."
date = 2026-01-07
categories = ["self-hosting", "docker", "photography"]
tags = ["immich", "tailscale", "docker", "macos", "self-hosting"]
toc = true
+++

## The Beginning: An Unused MacBook

I had an old MacBook sitting around, collecting dust. Instead of letting it go to waste, I decided to turn it into a personal photo server. My goal was to create a secure, self-hosted photo management solution that I could access from anywhere.

## Choosing the Right Tools

After some research, I settled on **Immich** - a powerful open-source photo management tool - and **Tailscale** for secure remote access. This combination would give me a private Google Photos alternative with enterprise-grade security.

## Setting Up Immich

### Initial Setup

First, I created a dedicated directory for the Immich setup:

```bash
mkdir ./immich-app
cd ./immich-app
```

Then I downloaded the latest Docker Compose configuration:

```bash
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### Configuration

The `.env` file contains all the configuration variables that Immich uses when starting up. Here's what each key setting does:

**IMMICH_ADMIN_PASSWORD**: This is the password for the admin account that gets created when you first access Immich. The default is weak, so you should change it to something secure.

**UPLOAD_LOCATION**: This tells Immich where to store your uploaded photos and videos. By default it might use a generic path, but we want to point it to our dedicated `upload` directory.

**DB_PASSWORD/DB_USERNAME/DB_DATABASE_NAME**: These are the PostgreSQL database credentials. The default password is not secure, so you should generate a strong one and set a custom username.

**DB_DATA_LOCATION**: This is where the PostgreSQL database files are stored. We're pointing it to our `postgres` directory to keep everything organized.

**TZ**: This sets the timezone for the Immich server, which affects how photo timestamps are displayed and processed.

Here are the specific changes I made to the `.env` file:

```bash
# Admin password for initial setup
IMMICH_ADMIN_PASSWORD=your-secure-password-here

# Custom storage location for photos
UPLOAD_LOCATION=/Users/your-username/immich-app/upload

# Database settings
DB_PASSWORD=your-database-password
DB_USERNAME=immich
DB_DATABASE_NAME=immich
DB_DATA_LOCATION=/Users/your-username/immich-app/postgres

# Timezone settings
TZ=America/Los_Angeles
```

Key changes made:
- Set a strong admin password for the Immich web interface
- Configured upload location to use the dedicated directory we created
- Updated database credentials for security
- Set the correct timezone for proper photo metadata handling

### macOS Docker Considerations

Since I'm using Colima instead of Docker Desktop on macOS, I had to make some adjustments to avoid permission issues. The default Docker Compose file uses bind mounts, which can be problematic on macOS.

I created the necessary directories first:

```bash
mkdir -p postgres upload library
```

Then modified the `docker-compose.yml` to use Docker volumes instead of bind mounts:

```yaml
# Changed from:
- ${DB_DATA_LOCATION}:/var/lib/postgresql/data

# To:
- pgdata:/var/lib/postgresql/data

# And added the volume definition:
volumes:
  pgdata:
  model-cache:
```

While the documentation warns against editing the database volume line, this change is necessary on macOS with Colima to avoid permission issues with bind mounts.

### Launching Immich

With everything configured, I started the services:

```bash
docker compose up -d
```

Immich was now running on port 2283, accessible on my local network using `ipconfig getifaddr en0:2283`.

## Making It Accessible Anywhere with Tailscale

Local access was great, but I wanted to access my photos from anywhere. This is where Tailscale came in.

### Installing Tailscale

```bash
brew install --cask tailscale-app
```

After installation, I:
- Opened the Tailscale app
- Authenticated and logged in
- Granted the necessary system permissions for network access and VPN functionality

### Serving Immich Over Tailscale

The magic happened with a single command:

```bash
tailscale serve 2283
```

This command exposes my local Immich instance over the Tailscale network, making it accessible from any device connected to my Tailnet.

### Verifying Peer-to-Peer Functionality

I tested the setup to ensure Tailscale was working in true peer-to-peer mode (not master-slave). I had another device already serving a different service on the same Tailnet, and I could access both services simultaneously from a third device. This confirmed that Tailscale's peer-to-peer architecture was working correctly.

## The Result

I now have a fully functional, private photo server running on my old MacBook. I can access all my photos securely from anywhere in the world, and the setup is completely self-hosted and under my control.

![Immich dashboard placeholder](/img/misc/immich-dashboard.png)

*The Immich dashboard running on my MacBook*

![Tailscale status placeholder](/img/misc/tailscale-status.png)

*Tailscale showing connected devices*

## Why This Setup Works

1. **Cost-effective**: Repurposed existing hardware
2. **Secure**: Tailscale provides encrypted, peer-to-peer connections
3. **Private**: All data stays on my own hardware
4. **Accessible**: Available from anywhere with an internet connection
5. **Feature-rich**: Immich offers AI-powered search, facial recognition, and more

If you have old hardware lying around, consider giving it new life with a similar setup. The combination of Docker, self-hosted applications, and Tailscale opens up endless possibilities for personal infrastructure.