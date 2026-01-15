---
name: aspire-evaluator
description: Evaluate whether .NET Aspire fits a project. Use when user asks about Aspire, .NET orchestration, distributed .NET apps, or local development for .NET microservices.
---

# Aspire Fit & Enablement Evaluator

Assess whether .NET Aspire is appropriate for a project and provide implementation guidance.

## Standalone Usage

Can be invoked directly:
- "Should I use .NET Aspire for my project?"
- "What is Aspire and when should I use it?"
- "How do I set up Aspire for local development?"

## What is .NET Aspire?

.NET Aspire is an opinionated stack for building distributed applications with:

1. **AppHost** - Code-first declaration of services and dependencies
2. **Dashboard** - Dev-time visibility (logs, traces, config)
3. **Components** - Pre-configured integrations (Redis, PostgreSQL, etc.)
4. **MCP Support** - AI-assisted development via `aspire mcp init`

## When Aspire Fits (Strong Signals)

| Signal | Why It Matters |
|--------|----------------|
| Multi-service architecture | Aspire excels at orchestration |
| .NET backend services | Native .NET integration |
| Complex local orchestration | Replaces docker-compose complexity |
| Need unified observability | Dashboard provides logs/traces/config |
| Service discovery needed | Automatic wiring between services |
| Team knows .NET | Leverages existing skills |

## When Aspire Doesn't Fit

| Signal | Why |
|--------|-----|
| Single service/monolith | Overkill, adds complexity |
| Non-.NET stack | Limited benefit |
| Simple deps only | docker-compose is simpler |
| Team unfamiliar with .NET | Learning curve |
| Polyglot microservices | Better with K8s/docker-compose |

## What Aspire Provides

### AppHost (Orchestration)

```csharp
var builder = DistributedApplication.CreateBuilder(args);

// Add resources
var postgres = builder.AddPostgres("postgres")
    .AddDatabase("catalog");

var redis = builder.AddRedis("cache");

var rabbitmq = builder.AddRabbitMQ("messaging");

// Add services with references
builder.AddProject<Projects.CatalogApi>("catalog-api")
    .WithReference(postgres)
    .WithReference(redis);

builder.AddProject<Projects.OrdersApi>("orders-api")
    .WithReference(rabbitmq)
    .WithReference(redis);

builder.Build().Run();
```

### Dashboard

- **Logs**: Structured logs from all services
- **Traces**: Distributed tracing (OTLP)
- **Metrics**: Service metrics
- **Resources**: Health and status of all components
- **Config**: Environment variables and settings

### Built-in Components

| Component | What It Does |
|-----------|--------------|
| PostgreSQL | Database with connection management |
| Redis | Caching with automatic configuration |
| RabbitMQ | Messaging with connection strings |
| Azure Storage | Blob, Queue, Table storage |
| SQL Server | Database integration |
| MongoDB | Document database |
| Kafka | Event streaming |

## Output Contract

```yaml
aspire_assessment:
  project_description: "<what's being evaluated>"

  fit_score: "<strong|moderate|weak|not_applicable>"

  signals:
    multi_service: true|false
    dotnet_backend: true|false
    complex_orchestration: true|false
    need_observability: true|false
    service_count: <N>

  recommendation: "<use_aspire|consider_aspire|skip_aspire>"

  benefits:
    - "<benefit 1>"
    - "<benefit 2>"

  considerations:
    - "<consideration 1>"

  alternative: "<docker-compose|local-k8s|native>"
  alternative_reason: "<why alternative might be better>"

  implementation:
    steps:
      - "<step 1>"
      - "<step 2>"

    apphost_example: |
      <code example>

    commands:
      create: "dotnet new aspire-starter -n MyApp"
      run: "dotnet run --project MyApp.AppHost"
      mcp: "aspire mcp init"
```

## Fit Score Criteria

| Score | Criteria |
|-------|----------|
| **strong** | 3+ .NET services, complex deps, need observability |
| **moderate** | 2 .NET services, would benefit from dashboard |
| **weak** | Single service, simple deps |
| **not_applicable** | No .NET components |

## Quick Start

### Create New Aspire Project

```bash
# Create starter template
dotnet new aspire-starter -n MyApp

# Structure created:
# MyApp/
# ├── MyApp.AppHost/        # Orchestration
# ├── MyApp.ServiceDefaults/ # Shared config
# ├── MyApp.ApiService/     # Example API
# └── MyApp.Web/            # Example frontend
```

### Add to Existing Project

```bash
# Add AppHost project
dotnet new aspire-apphost -n MyApp.AppHost

# Add reference to existing service
cd MyApp.AppHost
dotnet add reference ../MyApp.Api/MyApp.Api.csproj

# Add ServiceDefaults to services
dotnet add ../MyApp.Api package Aspire.ServiceDefaults
```

### Enable MCP

```bash
# Initialize MCP support
aspire mcp init

# This enables AI assistants to interact with Aspire
```

## Example Assessment

```yaml
aspire_assessment:
  project_description: "E-commerce platform with 4 .NET services"

  fit_score: "strong"

  signals:
    multi_service: true
    dotnet_backend: true
    complex_orchestration: true
    need_observability: true
    service_count: 4

  recommendation: "use_aspire"

  benefits:
    - "Unified orchestration of all services"
    - "Built-in observability dashboard"
    - "Automatic service discovery"
    - "Simplified local development"
    - "Code-first infrastructure"

  considerations:
    - "Team needs basic Aspire knowledge"
    - "Adds dependency on Aspire packages"

  implementation:
    commands:
      create: "dotnet new aspire-apphost -n ECommerce.AppHost"
      run: "dotnet run --project ECommerce.AppHost"
```
