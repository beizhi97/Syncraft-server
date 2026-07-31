# Syncraft Server Delivery

Generated server delivery for v0.3.0-rc.3 from beizhi97/Syncraft commit 77890bd5d3b0878444fd573c0f84eb7c63f136e5.

This repository is a generated deployment view. Do not edit application code
here; make changes in the source repository, validate them, tag the source
version, then republish.

## Deploy

Copy .env.example to .env and set at least SFSC_PUBLIC_HOST, SFSC_ADMIN_TOKEN
and MINIO_ROOT_PASSWORD, then:

    docker compose up --build -d

MINIO_ROOT_PASSWORD is mandatory: compose refuses to start without it rather
than falling back to a placeholder. Management APIs also refuse access unless
SFSC_ADMIN_TOKEN is set.

Runtime files (.env, data/, sing-box/config.json, private keys) are excluded by
design. See docs/DEPLOYMENT_AND_MIGRATION_GUIDE.md for backup, upgrade and
rollback procedures.
