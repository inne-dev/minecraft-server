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

The container downloads Paper 26.3 and compatible versions of AutoTreeChop, AxCalendar, BlueMap, Christmas Season, CraftEngine, Emotecraft, FlectonePulse, LightPlus, LuckPerms, MyFurniture, PlayerPresents, SkinsRestorer, Simple Voice Chat, and CDisc Music from Modrinth. It downloads SCore and ExecutableItems as required MyFurniture dependencies. CDisc Music provides custom jukebox discs with repeat, queue, seek, and volume controls; clients need the Simple Voice Chat mod. Operators alone can use CDisc's URL-based `/cdisc create` command. HazelwoodAuth creates requested discs from portal-uploaded OGG files through the CDisc API. CDisc auto-updates are disabled so the container controls plugin versions.

MyFurniture is available year-round. LuckPerms grants `mf.furniture.*` to the default group at startup. For 2026, AxCalendar uses Europe/Moscow during December, with configured daily rewards for December 1–24. Christmas Season's UTC schedule has `12-14` start and `01-07` end boundaries, which fall on December 14, 2026 and January 7, 2027 for this season. Its `xmas.admin` commands remain operator-only. Its own advent calendar is disabled because AxCalendar provides the December calendar. PlayerPresents has no creation-date setting. Startup grants `playerpresents.use`, `playerpresents.create`, and `playerpresents.open` to the default group, so players can create gifts year-round. Its scheduled lock prevents non-creators from opening gifts until January 1, 2027 at 00:00 Europe/Moscow; creators can open their own gifts earlier. Operators retain the `playerpresents.admin` and `playerpresents.open.locked` bypass permissions. TPA, TradeCycle, and HazelwoodAuth are supplied from bootstrap-plugins/. HazelwoodAuth replaces the old SimpleLogin flow with website authentication and polls the portal for the seasonal server icon. Its source and build are in the hazelwood-card/minecraft-auth-plugin module.

On startup, the container removes only top-level plugin JARs, then installs the curated plugins again. Plugin configuration and databases remain in data/plugins. The worlds are never part of this cleanup.

## Production world

Compose binds ./data to /data and deliberately does not set LEVEL; the server keeps the existing level-name in data/server.properties. Preserve the production data directory during deploys. Do not replace it with a fresh checkout or remove it when updating the container.

## Seasonal server icon

HazelwoodAuth polls the portal's /api/seasonal-icon endpoint every five minutes and serves the current image on Minecraft server-list pings. The portal container and Minecraft server share the hazelwood-private network, and HAZELWOOD_PORTAL_ORIGIN points to the portal service. The portal deployment must include that endpoint and the seasonal icon assets. data/server-icon.png is only a local fallback/runtime file and is not tracked.

## Resource pack

The server requests CraftEngine’s required pack through the `RESOURCE_PACK` server property during connection setup, before the login dialog or world appears. It uses the stable CraftEngine self-host URL on port 25565 with one-time tokens disabled so the server property can point to it. `delivery.send-on-join` stays disabled to prevent a second, late request after the player enters the world. Minecraft can reuse the cached pack when it has not changed.
