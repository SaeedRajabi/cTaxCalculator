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
