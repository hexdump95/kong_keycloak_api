# About the project
It's a showcase of Keycloak and Kong, and how they interact with a protected endpoint.

For the purpose of this demo, I'm not showing the full Keycloak configuration. Instead I have added the realm configuration to import it from Keycloak. It contains the realm ``kong_keycloak_api``, with a client ``web_client`` and a user ``zookabazooka`` with password ``password``. The realm also contains the JWKS to generate the PEM file. It's running in development mode (with H2 as database). I'm also using Kong Gateway in DB-less mode (with declarative configuration).

# Stack
- Docker
- Keycloak v26.0
- Kong v3.4

# Run the containers

```bash
docker-compose up
```

It will:
- Run the Kong Gateway service in DB-less mode, importing the ``kong/kong.yml`` configuration.
- Run the Keycloak service, first importing the realm configuration from ``./keycloak/``, then running in development mode.

# How to use

## Test unauthorized endpoint

Example use:
```bash
curl localhost:8000/mock/anything
```
Response:
```json
{"message":"Unauthorized"}
```

## Get the JWT
First, we have to log in via Keycloak's service to get the bearer token:
```bash
curl -X POST http://localhost:8000/login \
-d 'grant_type=password' \
-d 'username=zookabazooka' \
-d 'password=password' \
-d 'client_id=web_client'
```

We'll receive a response like:
```json
{
    "access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIg...",
    "expires_in":300,
    "refresh_expires_in":1800,
    "refresh_token":"eyJhbGciOiJIUzUxMiIsInR5cCIg...",
    "token_type":"Bearer",
    "not-before-policy":0,
    "session_state":"169c1e52-ce89-415c-b7d1-5498715e3583",
    "scope":"email profile"
}
```

## Test the protected endpoint

Copy the "access_token" content and use it here:
```bash
curl localhost:8000/mock/anything \
-H 'Authorization: Bearer {here}'
```
