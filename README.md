# Hazelwood Minecraft server

## First start

Create the two Docker secrets once, then start the stack:

    docker network inspect hazelwood-private >/dev/null 2>&1 || docker network create hazelwood-private
    mkdir -p secrets
    for name in rcon_password minecraft_auth_secret; do
      if [ ! -f "secrets/$name" ]; then
        (umask 077; openssl rand -hex 32 > "secrets/$name")
      fi
    done
    docker compose up -d

The website compose file reads these same files from this repository. Keep the existing minecraft_auth_secret unchanged after the website is deployed; both containers must share it.

The container downloads Paper 26.3 and current compatible versions of AutoTreeChop, BlueMap, CraftEngine, Emotecraft, FlectonePulse, LightPlus, SkinsRestorer, and Simple Voice Chat from Modrinth. TPA, TradeCycle, and HazelwoodAuth are supplied from bootstrap-plugins/. HazelwoodAuth replaces the old SimpleLogin flow with website authentication and polls the portal for the seasonal server icon. Its source and build are in the hazelwood-card/minecraft-auth-plugin module.

On startup, the container removes only top-level plugin JARs, then installs the curated plugins again. Plugin configuration and databases remain in data/plugins. The worlds are never part of this cleanup.

## Production world

Compose binds ./data to /data and deliberately does not set LEVEL; the server keeps the existing level-name in data/server.properties. Preserve the production data directory during deploys. Do not replace it with a fresh checkout or remove it when updating the container.

## Seasonal server icon

HazelwoodAuth polls the portal's /api/seasonal-icon endpoint every five minutes and serves the current image on Minecraft server-list pings. The portal container and Minecraft server share the hazelwood-private network, and HAZELWOOD_PORTAL_ORIGIN points to the portal service. The portal deployment must include that endpoint and the seasonal icon assets. data/server-icon.png is only a local fallback/runtime file and is not tracked.

## Resource pack

CraftEngine does not automatically send the resource pack on every join, preventing a repeated pack reload screen. An admin can send it when needed with `/ce feature send-pack <player>`. The self-hosted pack shares the Minecraft connection port; no extra Docker port is needed.
