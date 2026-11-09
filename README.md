# RaceDay

RaceDay is a .NET 8 solution containing the API, shared contracts, MVC website, and API tests.

## Run locally before Docker

Docker adds a networking and packaging layer. First confirm that the application works normally:

1. Start `RaceDay.Api` from Visual Studio or a terminal.
2. Start `RaceDay.Mvc` normally.
3. Register or log in through the MVC website.
4. Confirm the role-aware navigation.
5. Open an authorised Organiser or Participant page.
6. Confirm that at least one API-backed data operation succeeds.

## Docker: RaceDay.Mvc

Build the MVC image from the solution root, where `RaceDay.Mvc` and `RaceDay.Contracts` are available to the Docker build context:

```powershell
docker build -f RaceDay.Mvc/Dockerfile -t raceday-mvc:part3 .
```

Start the API on the Windows host before starting the MVC container:

```powershell
dotnet run --project RaceDay.Api --urls http://0.0.0.0:5000
```

If Swagger is enabled, check the host API at <http://localhost:5000/swagger> before continuing.

Run the MVC container and override its API URL for Docker networking:

```powershell
docker run -d --name raceday-mvc -p 8080:8080 -e ApiSettings__BaseUrl=http://host.docker.internal:5000/ raceday-mvc:part3
```

Open the MVC website at <http://localhost:8080>.

Inside the container, `localhost` refers to the MVC container itself. `host.docker.internal` is required to reach the API running on the Windows host. Test the same login, role navigation, API data display, and data-changing operation that worked locally. Stop the normal MVC debugging session while leaving the container running to confirm that Docker is serving the website.

## Inspect, stop, and restart

Useful diagnostic commands:

```powershell
docker ps
docker logs raceday-mvc
docker stop raceday-mvc
docker rm raceday-mvc
```

Press `Ctrl+C` to stop following `docker logs -f`; this does not stop the container.

## Rebuild after source changes

An existing image contains the code from the time it was built. Rebuild and recreate the container after changing C# or Razor files:

```powershell
docker stop raceday-mvc
docker rm raceday-mvc
docker build -f RaceDay.Mvc/Dockerfile -t raceday-mvc:part3 .
docker run -d --name raceday-mvc -p 8080:8080 -e ApiSettings__BaseUrl=http://host.docker.internal:5000/ raceday-mvc:part3
```

## Verification status

The standalone MVC workflow has been verified locally: the image `raceday-mvc:part3` built successfully, the `raceday-mvc` container started with port mapping `0.0.0.0:8080->8080/tcp`, and `http://localhost:8080` returned HTTP 200. API-backed login and data operations still require the API to be running on the Windows host at `http://0.0.0.0:5000` and must be tested separately.

The complete solution still has unrelated source compilation blockers in the API test contract alignment and `EnrolmentsController` relationship code. These do not prevent the MVC project from building or the standalone MVC container from serving its home page.

The repository also contains a separate two-service Compose setup. It uses `http://api:8080/` for MVC-to-API communication and is not the same as this standalone workflow, where MVC uses `http://host.docker.internal:5000/`. See [Docker-Troubleshooting.md](Docker-Troubleshooting.md) for diagnostic steps and database connectivity requirements.

## Azure Blob Storage image configuration

Image uploads always go through `RaceDay.Api`; `RaceDay.Mvc` never connects directly to Azure Storage. Supply the API settings through user secrets, environment variables, or an approved secret store. Do not commit a real connection string.

For local development, user secrets are preferred:

```powershell
dotnet user-secrets --project RaceDay.Api init
dotnet user-secrets --project RaceDay.Api set "AzureBlobStorage:ConnectionString" "<storage-connection-string>"
dotnet user-secrets --project RaceDay.Api set "AzureBlobStorage:ContainerName" "raceday-images"
```

For Docker or another hosted environment, use an environment variable instead:

```powershell
$env:AzureBlobStorage__ConnectionString = "<storage-connection-string>"
$env:AzureBlobStorage__ContainerName = "raceday-images"
```

For Azure-hosted deployments, prefer managed identity with the Storage Blob Data Contributor role and a token-based BlobServiceClient rather than a long-lived account key. Raw blob URLs are only directly displayable when storage access allows it; private storage requires an approved API download or signed-access design.
