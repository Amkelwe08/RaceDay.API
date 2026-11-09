# RaceDay - Event Management System

## 1. Project Overview

RaceDay is a web-based event management system developed using ASP.NET Core MVC and ASP.NET Core Web API.

The system is designed to manage race-style sporting events such as marathons, trail runs and cycling events.

RaceDay provides two main user roles:

- **Organisers:** Can create and manage events, manage categories, view participant enrolments and capture race results.
- **Participants:** Can browse events, register for categories, view their enrolments and access their race results.

The system aims to make race event management easier by providing a central platform for organisers and participants.

## 2. Project Objectives

The main objectives of RaceDay are to:

- Provide an online platform for managing race events.
- Allow organisers to create, update and manage events.
- Allow participants to browse and enrol in events.
- Manage event categories and participant enrolments.
- Record and display race results.
- Provide secure authentication and role-based access.
- Use an API-based architecture to separate the user interface from backend operations.

## 3. Main Features

### Organiser Features
- Organiser registration and login.
- Organiser dashboard.
- Create, view, update and delete events.
- Manage event categories.
- View participant enrolments.
- Update enrolment statuses.
- Capture participant results.

### Participant Features
- Participant registration and login.
- Browse available events.
- View event details.
- Enrol in event categories.
- View personal enrolments.
- View race results.
- Manage profile information.

### General Features
- Role-based navigation and access control.
- Form validation.
- Session-based authentication.
- API communication between MVC and Web API.
- SQL Server database integration.

## 4. Technologies Used

| Technology | Purpose |
|---|---|
| C# | Main programming language |
| .NET 8 | Application framework |
| ASP.NET Core MVC | Website and user interface |
| ASP.NET Core Web API | Backend services |
| SQL Server | Database management |
| Entity Framework Core | Database access |
| Swagger | API documentation and testing |
| Visual Studio 2022 | Development environment |
| Git and GitHub | Version control |
| Docker | Application containerisation |
| GitHub Actions | Automated build and testing |

## 5. System Architecture

RaceDay uses a separated MVC and API architecture.

The MVC application handles the user interface and sends HTTP requests to the Web API.

The API processes requests, applies application logic and communicates with the SQL Server database.

**Application flow:**

Browser → RaceDay.Mvc → RaceDay.Api → SQL Server

The MVC project does not connect directly to the database.

## 6. Database Structure

The original RaceDay database design consists of six main tables:

| Table | Description |
|---|---|
| Roles | Stores user roles |
| Users | Stores user account information |
| Events | Stores race event information |
| Categories | Stores event categories |
| Enrolments | Stores participant registrations |
| Results | Stores race results |

The database uses primary keys and foreign keys to maintain relationships between records.

## 7. Solution Structure

The solution contains four main projects:

```text
RaceDay.API.slnx
│
├── RaceDay.Api
│   ├── Controllers
│   ├── Data
│   ├── Models
│   ├── Services
│   └── Program.cs
│
├── RaceDay.Contracts
│   └── DTOs
│
├── RaceDay.Mvc
│   ├── Controllers
│   ├── Views
│   ├── ViewModels
│   ├── Services
│   ├── wwwroot
│   └── Program.cs
│
└── RaceDay.Api.Tests
    └── Integration tests
```

## 8. Requirements

To run the project locally, the following software is required:

- Visual Studio 2022.
- .NET 8 SDK.
- SQL Server LocalDB.
- SQL Server Management Studio (SSMS), recommended for database management.
- Git, if cloning the repository.

## 9. Database Configuration

The application uses SQL Server.

For local development, the connection string should point to the LocalDB instance.

Example:

```json
{
  "ConnectionStrings": {
    "RaceDayDb": "Server=(localdb)\\MSSQLLocalDB;Database=RaceDayDB;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
  }
}
```

Ensure that the database exists and that the connection string matches the configuration expected by the API.

Do not commit real passwords, secrets or production connection details to GitHub.

## 10. Running the Application

1. Clone or download the repository.
2. Open `RaceDay.API.slnx` in Visual Studio 2022.
3. Restore NuGet packages.
4. Check the SQL Server connection string.
5. Ensure the database is available.
6. Configure Visual Studio to start both `RaceDay.Api` and `RaceDay.Mvc`.
7. Run the solution.

The API Swagger page can be used to inspect and test available API endpoints.

The MVC application can be accessed through the browser using its configured launch URL.

## 11. Running Automated Tests

The solution includes a test project named `RaceDay.Api.Tests`.

To run tests in Visual Studio:

1. Open Test Explorer.
2. Build the solution.
3. Select Run All Tests.
4. Review the passed and failed tests.

Alternatively, use:

```bash
dotnet test
```

## 12. Docker

RaceDay includes Docker configuration for containerising the MVC and API applications.

The Docker Compose configuration is used to manage the application services.

Before running Docker, ensure that:

- Docker Desktop is installed and running.
- The Dockerfiles are configured correctly.
- The API URL used by MVC is correct for the Docker network.
- The API can connect to the configured SQL Server instance.

Run the application using:

```bash
docker compose up --build
```

To stop the containers:

```bash
docker compose down
```

Examples may include:

- Azure Blob Storage integration, if not yet completed.
- Docker deployment, if not yet verified.
- Features that still require browser testing.

Update this section before final submission to accurately reflect the final application.

