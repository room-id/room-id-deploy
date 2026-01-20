# room-id-deploy

## Deployment repository for the Room-ID stack.

This repository contains the docker-compose configuration and supporting files used to run the Room-ID services on a server. Technical details, API specifications, and endpoints for the backend live in the room-id-backend repository and are intentionally not duplicated here.

&nbsp;

# Services / Docker Images

The compose stack runs the following services:

* **nginx**

    Image: nginx:1.27-alpine

    Acts as reverse proxy and TLS termination.

* **certbot**

    Image: certbot/certbot:latest
    
    Used to obtain Let’s Encrypt certificates.

* **certbot-renew**

    Image: certbot/certbot:latest

    Periodically renews existing certificates.

* **backend**

    Image: ghcr.io/room-id/room-id-backend:<tag>

    [Room-ID backend application, pulled from GitHub Container Registry (GHCR).](https://github.com/room-id/room-id-backend)

* **mongo**

    Image: mongo:7

    MongoDB database used by the backend.

* **qdrant**

    Image: qdrant/qdrant:latest

    Vector database used for image embeddings.

────────────────────────────────────────────────────────────

# Prerequisites

Docker and Docker Compose plugin installed on the server

DNS records pointing the relevant domain(s) to the server

A .env file in the repo root with required paths and secrets

If the backend image is private: the server must be logged in to GHCR

────────────────────────────────────────────────────────────

# Environment Variables

The docker-compose configuration expects a .env file (not committed to git).
Example keys:

UPLOADS_PATH=/var/room-id/uploads
QDRANT_PATH=/var/room-id/qdrant
MONGO_PATH=/var/room-id/mongo

LETSENCRYPT_PATH=/var/room-id/letsencrypt
CERTBOT_WWW_PATH=/var/room-id/certbot-www

GEOCODE_API_KEY= ...

────────────────────────────────────────────────────────────

# Backend Image Build & Pull Flow

The backend image is built in the room-id-backend repository using GitHub Actions and published to GHCR.

Image tags used:

ghcr.io/room-id/room-id-backend:latest
Built upon a push to the main branch.

ghcr.io/room-id/room-id-backend:test
Built upon a push to the develop branch (if enabled).

This repository does not build images. It only pulls and runs them.

────────────────────────────────────────────────────────────

# GHCR Authentication (Private Images)

The backend image is private, you will need to log into GHCR on the server.

Run on the server (as the user that runs docker compose):

`docker login ghcr.io`

Using a GitHub PAT (classic) with at least read:packages permission. ***Fine-grained tokens do not work for package/image permissions. You must use a classic token.***

────────────────────────────────────────────────────────────

# Running the stack

From the repository root:

```
docker compose pull
docker compose up -d
```

To view logs:

`docker compose logs -f --tail=200`

To stop all services:

`docker compose down`

────────────────────────────────────────────────────────────

# Test / Staging Backend (OPTIONAL)

Upon push to the develop branch in the backend repository, an image with the tag `:test` will be created. You can override the docker-compose and use this image for testing as follows:

Create docker-compose.test.yml:

```
services:
  backend:
    image: ghcr.io/room-id/room-id-backend:test
```

Run with:

```
docker compose -f docker-compose.yml -f docker-compose.test.yml pull
docker compose -f docker-compose.yml -f docker-compose.test.yml up -d
```