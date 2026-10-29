# Social Metadata & Contact Extractor API

FastAPI service that extracts page metadata, public email addresses, and links to
supported social platforms from a public webpage.

## Run locally

```powershell
python -m pip install -r requirements.txt
$env:REQUIRE_RAPIDAPI_PROXY = "false"
python -m uvicorn app.main:app --reload
```

Proxy authentication is required by default. Disable it only for local
development as shown above; the app refuses to start in protected mode unless
`RAPIDAPI_PROXY_SECRET` is configured.

The API guide is at `http://127.0.0.1:8000/docs`, Swagger UI at
`http://127.0.0.1:8000/swagger`, and the dashboard at `http://127.0.0.1:8000/ui`.

## Endpoint

`POST /v1/extract`

Request:

```json
{
  "url": "https://example.com"
}
```

The response contains `success` and `data` fields. `data` includes the requested
URL, page title and description, a list of found email addresses, and social
profile links grouped by platform. The endpoint does not verify email addresses
or guarantee that extracted contact details are current.

## Deploy with Docker

Build and run:

```powershell
docker build -t socialmeta-api .
docker run --rm -p 8000:8000 `
  -e PORT=8000 `
  -e REQUIRE_RAPIDAPI_PROXY=true `
  -e RAPIDAPI_PROXY_SECRET=replace-with-your-provider-proxy-secret `
  -e CORS_ORIGINS=https://your-frontend.example `
  socialmeta-api
```

Configure the same `RAPIDAPI_PROXY_SECRET` in the RapidAPI provider settings
and in your hosting platform's secret/environment settings. Keep it private.
When `REQUIRE_RAPIDAPI_PROXY=true`, the API rejects extraction requests that do
not include the matching `X-RapidAPI-Proxy-Secret` header. The root health
endpoint and public documentation remain available.

For local development without the RapidAPI gateway, explicitly set
`REQUIRE_RAPIDAPI_PROXY=false`. If a proxy secret is set, requests to the
extraction endpoint must include the matching header.

## Publish on RapidAPI

1. Deploy this Docker image to a host that provides a public HTTPS URL and set
   the environment variables above. Use the host's assigned `PORT`.
2. Verify `/`, `/v1/extract`, and `/openapi.json` on the deployed HTTPS origin.
3. In the RapidAPI provider dashboard, add the deployed origin as the upstream
   base URL and import its OpenAPI document from `https://YOUR-DOMAIN/openapi.json`.
4. Configure RapidAPI's provider proxy secret to the same value as
   `RAPIDAPI_PROXY_SECRET`, then keep `REQUIRE_RAPIDAPI_PROXY=true`.
5. Set subscription plans, quotas, and rate limits in the RapidAPI dashboard.
   RapidAPI subscriber keys are handled by its gateway; do not put provider
   secrets or subscriber keys in this repository.
6. Test the endpoint through RapidAPI's test console and confirm requests sent
   directly to the origin are rejected.

The app also serves a small English guide at `/docs`, Swagger UI at `/swagger`,
and a dashboard at `/ui`. The dashboard uses its current origin by default.
`CORS_ORIGINS` accepts a comma-separated list of frontend origins and defaults
to `*`; set it to the actual frontend origins for a public deployment.

## Limits and production notes

The service rejects non-public target addresses, redirects, nonstandard ports,
and responses larger than 2 MiB. It uses bounded connection/read timeouts.
Configure outbound network rules at the hosting layer as an additional SSRF
control, and use RapidAPI plan quotas/rate limits to manage public traffic.

## Tests

```powershell
python -m unittest discover -s tests
```
