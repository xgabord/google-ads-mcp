# LAAVA Google Ads MCP on xCloud (Docker + Nginx)

## Scope
This branch adds xCloud deployment configuration to the official Google Ads MCP fork.
The upstream Google Ads MCP's current tools are read-only; this deployment does not
implement or activate campaign modifications.

## Prerequisites
- Google Cloud project `laava-marketing-mcp` with Google Ads API enabled and
  sufficient access to production Google Ads accounts (Explorer or higher).
- Google Auth Platform OAuth client of type **Web application** with the
  approved redirect URI `https://ads-mcp.laava.hu/auth/callback`.
- The Google account used to sign in must have access to LAAVA Google Ads.
- xCloud **Docker + Nginx** server (the existing LAAVA AI v2 Nginx managed
  server does not support xCloud-managed Python/Docker Git deployments).
- Domain `ads-mcp.laava.hu` mapped to the selected xCloud server, with HTTPS.

## xCloud deployment
1. Add a Docker + Nginx server in xCloud (requires separate server approval
   if no such server currently exists).
2. **New Site > Deploy via Git**.
3. Repository: `https://github.com/xgabord/google-ads-mcp`, branch
   `deploy/xcloud-docker`.
4. Deploy method: Docker Compose, filename `docker-compose.yml`.
5. Public domain: `ads-mcp.laava.hu`. Primary web port: `8080`.
6. In xCloud's private Environment File settings, enter the variables from
   `.env.example` and actual OAuth client values. Never commit any
   production values to Git, and never send secrets in chat.
7. Generate two *different*, high-entropy secrets on a trusted machine for
   `GOOGLE_ADS_MCP_JWT_SIGNING_KEY` and
   `GOOGLE_ADS_MCP_STORAGE_ENCRYPTION_KEY`. Keep them stable across redeploys.
8. Deploy. Confirm the HTTPS address and connect an OAuth-capable MCP client
   to `https://ads-mcp.laava.hu/mcp`.
9. Log in with the authorized Google Ads user; test
   `customers_list_accessible_customers` and then a read-only GAQL query
   for LAAVA (`7424587400`).
10. Restrict access, set monitoring, and complete an authenticated MCP security
    audit before adding any write-capable tools.

## OAuth persistence
The named Docker volume `google_ads_mcp_oauth` persists the encrypted
FastMCP OAuth state across container replacements. Keep the encryption and
JWT-signing keys stable and protected. The service is bound to
`127.0.0.1:8080` on the Docker host; the public endpoint uses xCloud Nginx
with TLS.

## OAuth testing limitation
External Google OAuth consent apps in *Testing* may issue refresh tokens that
expire after seven days for requested scopes beyond basic identity. Review
the consent screen's production / verification requirements separately from
Google Ads API access before relying on a long-lived connection.
