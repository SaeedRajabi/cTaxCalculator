# CTaxCalculator - Technical Documentation

## System Architecture

### Overview

CTaxCalculator implements a clean architecture with domain-driven design principles. The system is organized into two main directories:

1. **00.Framework**: Shared infrastructure and cross-cutting concerns
2. **01.SRC**: Business logic and domain-specific implementations

### Layered Architecture

```
Presentation Layer (API Controllers)
        ↓
Application Layer (Command/Query Handlers)
        ↓
Domain Layer (Entities, Value Objects, Domain Events)
        ↓
Infrastructure Layer (Data Access, External Services)
```

### Framework Structure (00.Framework)

The framework provides reusable components across multiple domains:

#### Utilities
- **DateTimes**: Persian calendar support, date conversion utilities
- **Extensions**: Extension methods for common types (string, enum, GUID, etc.)
- **Generator**: Snowflake ID generator for distributed unique IDs
- **Services**: External service integration abstractions

#### Core
- **ApplicationServices**: Command and query dispatcher interfaces
- **Contracts**: Shared interfaces for repositories, services
- **Domain**: Base entities, value objects, exceptions
- **RequestResponse**: Command and query base classes, common response models

#### Infrastructure
- **Data.SQLCommands**: Write-side data access implementations
- **Data.SQLQueries**: Read-side data access implementations

#### Endpoints
- **Controllers**: Base controller with common functionality
- **Filters**: Action filters for validation and performance tracking
- **Middlewares**: Exception handling, request processing middleware

### SRC Structure (01.SRC)

Contains the specific business implementation for the tax calculation domain:

#### Core
- **ApplicationServices**: Command and query handlers for tax operations
- **Contracts**: Repository interfaces for tax entities
- **Domain**: Tax domain entities, value objects, enumerations
- **RequestResponse**: Command and query DTOs for tax operations

#### Infrastructure
- **MSSQLData.SQLCommands**: SQL Server implementation for write operations
- **MSSQLData.SQLQueries**: SQL Server implementation for read operations

#### Endpoints
- **API**: REST API controllers, program startup, dependency injection setup

## Domain Model Deep Dive

### Entities

#### City
Represents a city with configurable tax rules. Contains:
- City name (as a value object)
- Collection of tax rules
- Collection of vehicles registered in the city

#### Vehicle
Represents a vehicle subject to congestion tax. Contains:
- Vehicle type (enumeration indicating tax exemption status)
- Plate number (value object with validation)
- Collection of passages
- Total tax amount accumulated
- Last tax passage price

#### Passage
Records when a vehicle passes through a tax zone. Contains:
- Vehicle ID reference
- Date and time of passage

#### TaxRule
Defines tax amounts for specific time periods. Contains:
- Time period (start and end times)
- Tax amount for that period

#### FreeTaxDate
Represents dates when no tax is applied. Contains:
- Specific date
- City reference

### Value Objects

#### PlateNumber
Encapsulates vehicle license plate validation:
- Format validation (3 letters followed by 3 digits)
- Immutability
- Equality comparison based on value

#### Price
Represents monetary values:
- Non-negative decimal amount
- Arithmetic operations (addition, subtraction)
- Immutability
- Equality comparison based on amount

#### TaxRuleDateTime
Represents a time period for tax rules:
- Start time (TimeOnly)
- End time (TimeOnly)
- Immutability
- Validation that end time is after start time

#### CityName
Encapsulates city name validation:
- Non-empty string validation
- Immutability
- Equality comparison based on value

### Domain Events

The system implements domain events for important business occurrences:
- Vehicle added to city
- Passage recorded
- Tax calculated
- Vehicle type changed
- Plate number updated

## Application Services

### Command Handlers

Command handlers implement write operations and business logic:

#### AddVehicleCommandHandler
Creates a new vehicle and associates it with a city:
1. Validates city exists
2. Creates new vehicle with specified type and plate number
3. Associates vehicle with city
4. Persists changes

#### AddPassageCommandHandler
Records a vehicle passage:
1. Validates vehicle exists
2. Creates new passage record
3. Associates passage with vehicle
4. Calculates and updates tax amount
5. Persists changes

#### AddTaxRuleCommandHandler
Defines a new tax rule for a city:
1. Validates city exists
2. Creates new tax rule with time period and amount
3. Associates rule with city
4. Persists changes

### Query Handlers

Query handlers implement read operations:
- Get vehicle by ID
- Search vehicles with pagination
- Get city by name
- List tax rules for a city

## Data Access Layer

### Entity Framework Core Implementation

The system uses Entity Framework Core for data persistence with:

#### Database Schema
- **Cities** table with city names
- **Vehicles** table with plate numbers and tax amounts
- **Passages** table with vehicle references and timestamps
- **TaxRules** table with time periods and amounts
- **FreeTaxDates** table with tax-exempt dates

#### Entity Configurations
Each entity has a dedicated configuration class that defines:
- Table names and schemas
- Column types and constraints
- Indexes for performance optimization
- Value object conversions

#### Repository Pattern
Repositories abstract data access operations:
- **ITaxCityCommandRepository**: Write operations for cities
- **ITaxCityQueryRepository**: Read operations for cities
- Similar interfaces for other entities

## API Layer

### RESTful Design

API endpoints follow REST conventions:
- Resource-based URLs
- Standard HTTP methods (GET, POST, PUT, DELETE)
- Proper HTTP status codes
- JSON request/response bodies

### Controllers

Each aggregate has a dedicated controller:
- **VehicleController**: Vehicle management operations
- **CityController**: City management operations
- **PassageController**: Passage recording operations
- **TaxRulesController**: Tax rule management operations
- **FreeTaxDateTimeController**: Free tax date management

### Base Controller Functionality

The BaseController provides common functionality:
- Command and query dispatching
- Standardized response handling
- Excel export capabilities
- Error handling and logging

## Cross-Cutting Concerns

### Logging

Serilog is used for comprehensive logging:
- Console output for development
- File output for persistent logs
- Structured logging with contextual information

### Exception Handling

Global exception handling middleware:
- Catches unhandled exceptions
- Logs error details
- Returns standardized error responses
- Prevents information leakage

### Validation

Model validation through:
- Data annotations on command/query objects
- Custom validation attributes
- Fluent validation for complex rules

### Security

Authentication and authorization:
- JWT token-based authentication
- Role-based access control
- Secure configuration management

## Persian Localization Features

### Date Handling

Comprehensive support for Persian calendar:
- Conversion between Gregorian and Persian dates
- Validation of Persian date formats
- Cultural formatting for Iranian users

### Number Formatting

Persian number support:
- Conversion between English and Persian digits
- Proper formatting for Persian locale

## Performance Considerations

### Caching

In-memory caching for frequently accessed data:
- City configurations
- Tax rules
- Free tax dates

### Database Optimization

- Proper indexing strategies
- Efficient query design
- Connection pooling
- Asynchronous database operations

### Memory Management

- Proper disposal of resources
- Efficient object creation
- Garbage collection optimization

## Deployment Considerations

### Configuration

Environment-specific configuration through:
- appsettings.json files
- Environment variables
- Azure Key Vault integration

### Scalability

- Stateless API design
- Database connection management
- Caching strategies
- Load balancing support

### Monitoring

- Health check endpoints
- Performance metrics
- Logging aggregation
- Alerting mechanisms

## Testing Strategy

### Unit Testing

- Domain entity behavior testing
- Value object validation testing
- Command handler logic testing
- Query handler data retrieval testing

### Integration Testing

- Database integration tests
- API endpoint testing
- External service integration testing

### Test Data Management

- Seeded test databases
- Mock data generation
- Test environment isolation

## Extensibility Points

### Plugin Architecture

- Modular design allows for easy extension
- Interface-based contracts for new features
- Dependency injection for service replacement

### Customization

- Configurable tax calculation rules
- Extendable vehicle types
- Custom date/time handling
- Pluggable external services

## Future Enhancements

### Planned Features

1. **Multi-city Support**: Enhanced support for managing multiple cities
2. **Advanced Reporting**: Detailed tax calculation reports
3. **Mobile Integration**: Mobile app APIs
4. **Real-time Processing**: WebSocket support for real-time updates
5. **Machine Learning**: Predictive tax analysis

### Architecture Improvements

1. **Event Sourcing**: Full event sourcing implementation
2. **Microservices**: Decomposition into domain-specific services
3. **CQRS Enhancement**: Complete separation of read/write models
4. **Cloud Native**: Full cloud-native deployment model

## Best Practices Implemented

### Code Quality

- SOLID principles adherence
- Clean code practices
- Comprehensive documentation
- Consistent naming conventions

### Security

- Input validation
- Output encoding
- Secure configuration
- Authentication/authorization

### Performance

- Efficient algorithms
- Database optimization
- Caching strategies
- Resource management

### Maintainability

- Modular design
- Clear separation of concerns
- Comprehensive logging
- Error handling strategies

## Conclusion

CTaxCalculator demonstrates a well-architected, domain-driven .NET application that follows industry best practices. Its modular design, comprehensive documentation, and adherence to clean architecture principles make it maintainable and extensible for future enhancements.


--
# CTaxCalculator - Setup Guide

## Prerequisites

Before you begin, ensure you have the following installed on your system:

1. **.NET 8 SDK** - Download from [Microsoft's official website](https://dotnet.microsoft.com/download/dotnet/8.0)
2. **SQL Server** - Either SQL Server Express or full version
3. **Visual Studio 2022** (recommended) or **Visual Studio Code**
4. **Git** - For version control (optional but recommended)

## Project Structure Overview

The CTaxCalculator solution is organized into two main directories:

```
CTaxCalculator/
├── 00.Framework/          # Shared framework components
│   ├── 01.Utilities/      # Utility classes and extensions
│   ├── 02.Core/           # Core domain abstractions
│   ├── 03.Infrastructure/ # Data access abstractions
│   └── 04.Endpoints/      # Base controllers and middleware
└── 01.SRC/                # Business logic implementation
    ├── 01.Core/           # Domain models and application services
    ├── 02.Infrastructure/ # Data persistence implementation
    └── 03.Endpoints/      # API controllers and startup
```

## Getting Started

### 1. Clone the Repository

```bash
git clone <repository-url>
cd CTaxCalculator
```

### 2. Restore NuGet Packages

Navigate to the solution root directory and run:

```bash
dotnet restore
```

### 3. Database Setup

#### Configure Connection Strings

Open `01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API/appsettings.Development.json` and update the connection strings:

```json
"ConnectionStrings": {
  "QueryLocalDB_ConnectionString": "Data source=.;initial catalog=CongestionTaxCalculator_Database_V1.0;Integrated security=true;TrustServerCertificate=True;MultipleActiveResultSets=true",
  "CommandLocalDB_ConnectionString": "Data source=.;initial catalog=CongestionTaxCalculator_Database_V1.0;Integrated security=true;TrustServerCertificate=True;MultipleActiveResultSets=true"
}
```

#### Create the Database

You can create the database in two ways:

**Option A: Using Entity Framework Migrations**

1. Open a terminal in the solution root directory
2. Navigate to the SQL Commands project:
   ```bash
   cd 01.SRC/02.Infrastructure/MSSQLData/CTaxCalculator.Src.Infra.MSSQLData.SQLCommands
   ```
3. Run the migrations:
   ```bash
   dotnet ef database update
   ```

**Option B: Manual Database Creation**

1. Connect to your SQL Server instance
2. Create a new database named `CongestionTaxCalculator_Database_V1.0`
3. The application will automatically create tables on first run

### 4. Build the Solution

From the solution root directory:

```bash
dotnet build
```

### 5. Run the Application

#### Using Command Line

1. Navigate to the API project directory:
   ```bash
   cd 01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API
   ```
2. Run the application:
   ```bash
   dotnet run
   ```

#### Using Visual Studio

1. Open `CTaxCalculator.sln` in Visual Studio
2. Set `CTaxCalculator.Src.Endpoints.API` as the startup project
3. Press `F5` to build and run

#### Using Visual Studio Code

1. Open the project folder in VS Code
2. Open the integrated terminal
3. Navigate to the API project:
   ```bash
   cd 01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API
   ```
4. Run the application:
   ```bash
   dotnet run
   ```

## Initial Configuration

### Swagger UI

Once the application is running, you can access the Swagger UI at:
```
https://localhost:{port}/swagger
```

This provides:
- Interactive API documentation
- Ability to test endpoints directly
- Schema definitions for all requests/responses

### Authentication Setup

The application uses JWT authentication. To configure:

1. In `appsettings.Development.json`, review the Jwt section:
   ```json
   "Jwt": {
     "JwtIssuer": "http://localhost:5000",
     "JwtAudience": "https://localhost:5001",
     "JwtSignKey": "CongestionTexCalculator_ThisCalculateGotenbergTask_WithCustomizeCityAndTime_V1.0",
     "JwtLifetimeMinutes": 30,
     "RefreshTokenLifetimeDays": 3
   }
   ```

2. The application includes default authentication endpoints that can be customized as needed.

### Logging Configuration

The application uses Serilog for logging. Configuration can be found in `appsettings.Development.json`:

```json
"Serilog": {
  "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
  "MinimumLevel": {
    "Default": "Information",
    "Override": {
      "Microsoft": "Warning",
      "System": "Warning",
      "ArkaSoftware": "Verbose"
    }
  },
  "WriteTo": [
    { "Name": "Console" },
    {
      "Name": "File",
      "Args": { "path": "%TEMP%\\Logs\\CTexCalculator.Log.txt" }
    }
  ]
}
```

## Sample Data Setup

To quickly get started with sample data:

### 1. Create a City

Using Swagger or a REST client, call the POST `/api/TaxCalculator/Cities/Add` endpoint:

```json
{
  "cityName": "Gothenburg"
}
```

### 2. Add Tax Rules

Add several tax rules for different time periods:

```json
{
  "cityId": 1,
  "startTime": "06:00:00",
  "endTime": "06:29:59",
  "amount": "8.00"
}
```

### 3. Add a Vehicle

```json
{
  "cityId": 1,
  "vehicleType": "Other",
  "plateNumber": "KJD258"
}
```

### 4. Record a Passage

```json
{
  "vehicleId": 1,
  "passageDateTime": "2025-10-19T08:30:00"
}
```

## Development Workflow

### Adding New Features

1. **Create a new branch** for your feature:
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Implement domain logic** in the appropriate files in `01.SRC/01.Core/`

3. **Add data persistence** in `01.SRC/02.Infrastructure/`

4. **Create API endpoints** in `01.SRC/03.Endpoints/`

5. **Test your changes** using Swagger or integration tests

6. **Commit and push** your changes:
   ```bash
   git add .
   git commit -m "Add your feature description"
   git push origin feature/your-feature-name
   ```

### Running Tests

The solution includes test projects in the `02.TEST/` directory:

```bash
cd 02.TEST/CTaxCalculator.Tests
dotnet test
```

### Code Style and Conventions

The project follows these conventions:

1. **Naming**: PascalCase for public members, camelCase for private members
2. **Structure**: Clean architecture with separation of concerns
3. **Documentation**: XML comments for public APIs
4. **Error Handling**: Consistent use of result patterns
5. **Logging**: Structured logging with appropriate levels

## Troubleshooting

### Common Issues

#### 1. Database Connection Issues

**Problem**: "A network-related or instance-specific error occurred"

**Solution**:
- Verify SQL Server is running
- Check connection string in appsettings
- Ensure SQL Server allows TCP/IP connections
- Verify firewall settings

#### 2. Port Already in Use

**Problem**: "Failed to bind to address https://localhost:5001"

**Solution**:
- Change the port in `launchSettings.json`
- Kill processes using the port:
  ```bash
  netstat -ano | findstr :5001
  taskkill /PID <process-id> /F
  ```

#### 3. Missing Dependencies

**Problem**: "Could not load file or assembly"

**Solution**:
- Run `dotnet restore` in the solution root
- Verify all NuGet packages are restored
- Clean and rebuild the solution

#### 4. Migration Errors

**Problem**: "The Entity Framework tools version 'x.x.x' is older than that of the runtime"

**Solution**:
- Update EF Core tools:
  ```bash
  dotnet tool update --global dotnet-ef
  ```

### Useful Commands

```bash
# Restore packages
dotnet restore

# Build solution
dotnet build

# Run application
dotnet run --project 01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API

# Run tests
dotnet test

# Apply migrations
dotnet ef database update --project 01.SRC/02.Infrastructure/MSSQLData/CTaxCalculator.Src.Infra.MSSQLData.SQLCommands

# Create new migration
dotnet ef migrations add MigrationName --project 01.SRC/02.Infrastructure/MSSQLData/CTaxCalculator.Src.Infra.MSSQLData.SQLCommands

# Remove last migration
dotnet ef migrations remove --project 01.SRC/02.Infrastructure/MSSQLData/CTaxCalculator.Src.Infra.MSSQLData.SQLCommands
```

## Deployment

### Production Configuration

For production deployment:

1. **Update appsettings.json** with production values
2. **Configure HTTPS** with valid certificates
3. **Set up reverse proxy** (nginx, IIS, etc.)
4. **Configure logging** for production environments
5. **Set up monitoring** and health checks

### Docker Deployment (Optional)

To containerize the application:

1. Create a Dockerfile in the API project directory:
   ```dockerfile
   FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
   WORKDIR /app
   EXPOSE 80
   EXPOSE 443
   
   FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
   WORKDIR /src
   COPY ["01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API/CTaxCalculator.Src.Endpoints.API.csproj", "01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API/"]
   RUN dotnet restore "01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API/CTaxCalculator.Src.Endpoints.API.csproj"
   COPY . .
   WORKDIR "/src/01.SRC/03.Endpoints/CTaxCalculator.Src.Endpoints.API"
   RUN dotnet build "CTaxCalculator.Src.Endpoints.API.csproj" -c Release -o /app/build
   
   FROM build AS publish
   RUN dotnet publish "CTaxCalculator.Src.Endpoints.API.csproj" -c Release -o /app/publish
   
   FROM base AS final
   WORKDIR /app
   COPY --from=publish /app/publish .
   ENTRYPOINT ["dotnet", "CTaxCalculator.Src.Endpoints.API.dll"]
   ```

2. Build and run the container:
   ```bash
   docker build -t ctaxcalculator .
   docker run -p 8080:80 ctaxcalculator
   ```

## Support

For issues and feature requests, please:
1. Check the existing documentation
2. Review the codebase for similar implementations
3. Contact the development team for complex issues
4. Submit pull requests for improvements

## Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a pull request

Please ensure your code follows the established patterns and includes appropriate tests.

--

# CTaxCalculator - API Documentation

## Overview

This document provides detailed information about the RESTful API endpoints available in the CTaxCalculator system. All endpoints follow standard REST conventions and return JSON responses.

## Base URL

```
https://localhost:{port}/api/TaxCalculator
```

## Authentication

Most endpoints require authentication using JWT tokens. Include the token in the Authorization header:

```
Authorization: Bearer {token}
```

## Common Response Formats

### Success Response

```json
{
  "status": "Ok",
  "data": {},
  "messages": []
}
```

### Error Response

```json
{
  "status": "BadRequest",
  "data": null,
  "messages": ["Error description"]
}
```

## Vehicle Management

### Add Vehicle

**POST** `/Vehicle/Add`

Creates a new vehicle in the system.

#### Request Body

```json
{
  "cityId": 1,
  "vehicleType": "Other",
  "plateNumber": "KJD258"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| cityId | long | Yes | ID of the city where the vehicle is registered |
| vehicleType | string | Yes | Type of vehicle (Motorcycle, Tractor, Emergency, Diplomat, Foreign, Military, Other) |
| plateNumber | string | Yes | License plate number (format: 3 letters followed by 3 digits) |

#### Response

```json
{
  "status": "Created",
  "data": "Vehicle Added Is Success.",
  "messages": []
}
```

### Get Vehicle By ID

**GET** `/Vehicle/GetById?Id=1`

Retrieves vehicle details by ID.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Id | long | Yes | ID of the vehicle to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "plateNumber": "KJD258",
    "vehicleType": "Other",
    "totalTaxAmount": "0.00",
    "lastTaxPassagePrice": "0.00"
  },
  "messages": []
}
```

### Search Vehicles

**GET** `/Vehicle/SearchBy`

Searches for vehicles based on optional criteria.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Page | integer | No | Page number (default: 1) |
| PageSize | integer | No | Number of items per page (default: 10) |
| PlateNumber | string | No | Filter by plate number |
| VehicleType | string | No | Filter by vehicle type |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "data": [
      {
        "id": 1,
        "plateNumber": "KJD258",
        "vehicleType": "Other",
        "totalTaxAmount": "25.00"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 1,
    "totalCount": 1
  },
  "messages": []
}
```

### Update Vehicle Plate Number

**PUT** `/Vehicle/UpdateVehiclePlateNumber`

Updates a vehicle's plate number.

#### Request Body

```json
{
  "id": 1,
  "plateNumber": "ABC123"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the vehicle to update |
| plateNumber | string | Yes | New license plate number |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Update Vehicle Type

**PUT** `/Vehicle/UpdateVehicleType`

Updates a vehicle's type.

#### Request Body

```json
{
  "id": 1,
  "vehicleType": "Motorcycle"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the vehicle to update |
| vehicleType | string | Yes | New vehicle type |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Delete Vehicle

**DELETE** `/Vehicle/DeleteById`

Removes a vehicle from the system.

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the vehicle to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

## City Management

### Add City

**POST** `/Cities/Add`

Creates a new city in the system.

#### Request Body

```json
{
  "cityName": "Gothenburg"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| cityName | string | Yes | Name of the city |

#### Response

```json
{
  "status": "Created",
  "data": "City Added Is Success.",
  "messages": []
}
```

### Get City By ID

**GET** `/Cities/GetById?Id=1`

Retrieves city details by ID.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Id | long | Yes | ID of the city to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "cityName": "Gothenburg"
  },
  "messages": []
}
```

### Get City By Name

**GET** `/Cities/GetByName?Name=Gothenburg`

Retrieves city details by name.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Name | string | Yes | Name of the city to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "cityName": "Gothenburg"
  },
  "messages": []
}
```

### Search Cities

**GET** `/Cities/SearchBy`

Searches for cities based on optional criteria.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Page | integer | No | Page number (default: 1) |
| PageSize | integer | No | Number of items per page (default: 10) |
| CityName | string | No | Filter by city name |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "data": [
      {
        "id": 1,
        "cityName": "Gothenburg"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 1,
    "totalCount": 1
  },
  "messages": []
}
```

### Update City Name

**PUT** `/Cities/UpdateCityName`

Updates a city's name.

#### Request Body

```json
{
  "id": 1,
  "cityName": "Stockholm"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the city to update |
| cityName | string | Yes | New city name |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Delete City

**DELETE** `/Cities/DeleteById`

Removes a city from the system.

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the city to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Delete City By All References

**DELETE** `/Cities/DeleteByAllReference`

Removes a city and all related entities (vehicles, passages, tax rules).

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the city to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

## Tax Rule Management

### Add Tax Rule

**POST** `/TaxRules/Add`

Creates a new tax rule for a city.

#### Request Body

```json
{
  "cityId": 1,
  "startTime": "06:00:00",
  "endTime": "06:29:59",
  "amount": "8.00"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| cityId | long | Yes | ID of the city for this tax rule |
| startTime | string | Yes | Start time (HH:mm:ss format) |
| endTime | string | Yes | End time (HH:mm:ss format) |
| amount | string | Yes | Tax amount for this time period |

#### Response

```json
{
  "status": "Created",
  "data": "TaxRule Added Is Success.",
  "messages": []
}
```

### Get Tax Rule By ID

**GET** `/TaxRules/GetById?Id=1`

Retrieves tax rule details by ID.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Id | long | Yes | ID of the tax rule to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "startTime": "06:00:00",
    "endTime": "06:29:59",
    "amount": "8.00"
  },
  "messages": []
}
```

### Search Tax Rules

**GET** `/TaxRules/SearchBy`

Searches for tax rules based on optional criteria.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Page | integer | No | Page number (default: 1) |
| PageSize | integer | No | Number of items per page (default: 10) |
| CityId | long | No | Filter by city ID |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "data": [
      {
        "id": 1,
        "startTime": "06:00:00",
        "endTime": "06:29:59",
        "amount": "8.00"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 1,
    "totalCount": 1
  },
  "messages": []
}
```

### Update Tax Rule Amount

**PUT** `/TaxRules/UpdateTaxRuleAmount`

Updates a tax rule's amount.

#### Request Body

```json
{
  "id": 1,
  "amount": "10.00"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the tax rule to update |
| amount | string | Yes | New tax amount |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Update Tax Rule Period

**PUT** `/TaxRules/UpdateTaxRulePeriod`

Updates a tax rule's time period.

#### Request Body

```json
{
  "id": 1,
  "startTime": "06:00:00",
  "endTime": "06:59:59"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the tax rule to update |
| startTime | string | Yes | New start time |
| endTime | string | Yes | New end time |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Delete Tax Rule

**DELETE** `/TaxRules/DeleteById`

Removes a tax rule from the system.

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the tax rule to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

## Passage Management

### Add Passage

**POST** `/Passage/Add`

Records a vehicle passage through a tax zone.

#### Request Body

```json
{
  "vehicleId": 1,
  "passageDateTime": "2025-10-19T08:30:00"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| vehicleId | long | Yes | ID of the vehicle |
| passageDateTime | string | Yes | Date and time of passage (ISO 8601 format) |

#### Response

```json
{
  "status": "Created",
  "data": "Passage Added Is Success.",
  "messages": []
}
```

### Get Passage By ID

**GET** `/Passage/GetById?Id=1`

Retrieves passage details by ID.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Id | long | Yes | ID of the passage to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "vehicleId": 1,
    "passageDateTime": "2025-10-19T08:30:00"
  },
  "messages": []
}
```

### Search Passages

**GET** `/Passage/SearchBy`

Searches for passages based on optional criteria.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Page | integer | No | Page number (default: 1) |
| PageSize | integer | No | Number of items per page (default: 10) |
| VehicleId | long | No | Filter by vehicle ID |
| FromDate | string | No | Filter by date range start |
| ToDate | string | No | Filter by date range end |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "data": [
      {
        "id": 1,
        "vehicleId": 1,
        "passageDateTime": "2025-10-19T08:30:00"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 1,
    "totalCount": 1
  },
  "messages": []
}
```

### Update Passage Date Time

**PUT** `/Passage/UpdatePassageDateTime`

Updates a passage's date and time.

#### Request Body

```json
{
  "id": 1,
  "passageDateTime": "2025-10-19T09:00:00"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the passage to update |
| passageDateTime | string | Yes | New date and time of passage |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

### Delete Passage

**DELETE** `/Passage/DeleteById`

Removes a passage record from the system.

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the passage to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

## Free Tax Date Management

### Add Free Tax Date

**POST** `/FreeTaxDateTime/Add`

Adds a date when no tax is applied.

#### Request Body

```json
{
  "cityId": 1,
  "freeTaxDate": "2025-12-25"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| cityId | long | Yes | ID of the city |
| freeTaxDate | string | Yes | Date when no tax is applied (YYYY-MM-DD format) |

#### Response

```json
{
  "status": "Created",
  "data": "FreeTaxDateTime Added Is Success.",
  "messages": []
}
```

### Get Free Tax Date By ID

**GET** `/FreeTaxDateTime/GetById?Id=1`

Retrieves free tax date details by ID.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Id | long | Yes | ID of the free tax date to retrieve |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "id": 1,
    "cityId": 1,
    "freeTaxDate": "2025-12-25"
  },
  "messages": []
}
```

### Search Free Tax Dates

**GET** `/FreeTaxDateTime/SearchBy`

Searches for free tax dates based on optional criteria.

#### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Page | integer | No | Page number (default: 1) |
| PageSize | integer | No | Number of items per page (default: 10) |
| CityId | long | No | Filter by city ID |
| FromDate | string | No | Filter by date range start |
| ToDate | string | No | Filter by date range end |

#### Response

```json
{
  "status": "Ok",
  "data": {
    "data": [
      {
        "id": 1,
        "cityId": 1,
        "freeTaxDate": "2025-12-25"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 1,
    "totalCount": 1
  },
  "messages": []
}
```

### Delete Free Tax Date

**DELETE** `/FreeTaxDateTime/DeleteById`

Removes a free tax date from the system.

#### Request Body

```json
{
  "id": 1
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | long | Yes | ID of the free tax date to delete |

#### Response

```json
{
  "status": "Ok",
  "data": null,
  "messages": []
}
```

## File Management

### Upload City Configuration (JSON)

**POST** `/FileManagement/CityFileUploader/UploadJsonFile`

Uploads a JSON file containing city and tax rule configurations.

#### Form Data

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | file | Yes | JSON file with city configurations |

#### Response

```json
{
  "status": "Created",
  "data": "File Upload Is Success.",
  "messages": []
}
```

### Upload City Configuration (CSV)

**POST** `/FileManagement/CityFileUploader/UploadCsvFile`

Uploads a CSV file containing city and tax rule configurations.

#### Form Data

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| file | file | Yes | CSV file with city configurations |

#### Response

```json
{
  "status": "Created",
  "data": "File Upload Is Success.",
  "messages": []
}
```

## Utility Endpoints

### Clear Memory

**GET** `/Vehicle/Clear`

Triggers garbage collection to clear unused memory.

#### Response

```json
{
  "status": "Ok",
  "data": true,
  "messages": []
}
```

## Error Handling

The API uses standard HTTP status codes:

| Code | Description |
|------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Internal Server Error |

## Rate Limiting

The API implements rate limiting to prevent abuse:
- 100 requests per minute per IP address
- 1000 requests per hour per authenticated user

## Versioning

The API is versioned through the URL path:
- v1: Current version

## Changelog

### v1.0.0 (Initial Release)
- Vehicle management endpoints
- City management endpoints
- Tax rule management endpoints
- Passage tracking endpoints
- Free tax date management endpoints
- File upload capabilities
- Comprehensive search and filtering
