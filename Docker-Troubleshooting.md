# RaceDay Docker troubleshooting

There are two supported local Docker arrangements. Choose one and use its matching API URL:

| Arrangement | MVC API URL | MVC URL |
| --- | --- | --- |
| Standalone MVC container with API on Windows host | `http://host.docker.internal:5000/` | `http://localhost:8080` |
| Compose API and MVC containers | `http://api:8080/` | `http://localhost:5000` |

The standalone workflow is documented in [README.md](README.md). The Compose workflow is described below.

The Compose setup contains two services:

- MVC: `http://localhost:5000`
- API: `http://localhost:5200`

The API container does not include SQL Server. Set `RACEDAY_DB_CONNECTION` to a SQL Server connection string reachable from Docker before starting the services. The default LocalDB connection is for Windows-hosted development and is not reachable from a Linux container.

## Run these commands first

```powershell
docker ps
docker ps -a
docker logs raceday-mvc
docker inspect raceday-mvc
```

Also confirm Docker Desktop is running:

```powershell
docker --version
docker info
```

For WSL-related failures:

```powershell
wsl --version
wsl --update
```

## Start the Compose local HTTP demo

From the solution root, provide a reachable SQL Server connection and start Compose:

```powershell
$env:RACEDAY_DB_CONNECTION = "Server=host.docker.internal,1433;Database=RaceDayDb;User Id=sa;Password=YourStrong!Passw0rd;TrustServerCertificate=True;"
docker compose up --build
```

Use `docker ps` and wait for both containers to report as running/healthy. Browse to `http://localhost:5000`. The API is available at `http://localhost:5200`.

To stop and remove the containers:

```powershell
docker compose down
```

## Browser says connection refused

1. Run `docker ps` and confirm `raceday-mvc` is running.
2. Check the `PORTS` column for `0.0.0.0:5000->8080/tcp`.
3. Run `docker logs raceday-mvc` and check whether ASP.NET Core started.
4. If host port 5000 is already allocated, change the Compose mapping to `5001:8080` and browse to `http://localhost:5001`.

## MVC cannot reach the API

Inside Compose, MVC must use `http://api:8080/`, not `localhost` or `host.docker.internal`. This is configured by `ApiSettings__BaseUrl` in `docker-compose.yml`.

From the Windows host, the API is exposed at `http://localhost:5200`. `host.docker.internal` is only needed when a container must reach a service running on the Windows host, such as the external SQL Server connection.

Inspect the effective MVC configuration:

```powershell
docker inspect raceday-mvc
```

Look for `ApiSettings__BaseUrl=http://api:8080/`.

For the standalone MVC container, the expected override is instead:

```powershell
docker run -d --name raceday-mvc -p 8080:8080 -e ApiSettings__BaseUrl=http://host.docker.internal:5000/ raceday-mvc:part3
```

In that arrangement, start the API on the Windows host with `dotnet run --project RaceDay.Api --urls http://0.0.0.0:5000`.

## API or MVC container exits immediately

```powershell
docker ps -a
docker logs raceday-api
docker logs raceday-mvc
```

A source compilation failure occurs during `docker compose up --build` and must be fixed in the project before the image can start. A database connection failure normally indicates that `RACEDAY_DB_CONNECTION` is missing, invalid, or points to a SQL Server that is not reachable from Docker.

## Port or container-name conflicts

The Compose file uses the stable names `raceday-api` and `raceday-mvc`. If an old container remains:

```powershell
docker ps -a
docker rm -f raceday-mvc raceday-api
```

If a host port is occupied, use a different host-side port while keeping the container port `8080`, for example `5001:8080` for MVC or `5201:8080` for the API.

## HTTPS certificate errors

The Compose profile uses HTTP intentionally and disables HTTPS redirection through `RACEDAY_USE_HTTPS_REDIRECTION=false`. Use the documented HTTP URLs for local assessment testing. Do not expect the container to trust a Visual Studio development certificate automatically.

## Images do not display

The container configuration does not make private blob storage public. Verify that the API persists the uploaded image URL, that the MVC view uses the returned URL/property, and that the storage access policy permits the browser to retrieve it.

## Changes are not visible

Rebuild the images and recreate the containers:

```powershell
docker compose down
docker compose up --build --force-recreate
```
