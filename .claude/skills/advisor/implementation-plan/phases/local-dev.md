---
name: local-dev-phase
description: Detailed template for Phase 4 Local Development Environment. Referenced by implementation-plan skill.
---

# Phase 4: Local Development Environment

**Goals**:
- Enable developers to run the full solution locally
- Minimize time-to-first-run
- Ensure parity with production where possible

**Assumptions**:
- Docker Desktop is available
- Local Kubernetes is available

## Step Template

```yaml
phase_4_local_dev:
  goals:
    - "Enable full local development environment"
    - "Minimize time-to-first-run"
    - "Document any prod parity gaps"

  orchestration_mode: "<docker-compose|local-k8s|hybrid|aspire>"

  steps:
    - id: "4.1"
      name: "Local Orchestration Setup"
      description: "Configure local orchestration"

      docker_compose:
        files:
          - "docker-compose.yml"
          - "docker-compose.override.yml"
        services: []

      local_k8s:
        manifests: ["k8s/local/"]
        tools: ["skaffold.yaml", "tilt"]

      hybrid:
        compose_for: ["database", "cache", "queue"]
        k8s_for: ["application services"]

      aspire:
        apphost_project: "<path>"
        services_defined: []
        dashboard_enabled: true

    - id: "4.2"
      name: "Dependencies Runbook"
      description: "How to run all dependencies locally"

      database:
        container_image: "<image>"
        ports: ["5432:5432"]
        volumes: ["db-data:/var/lib/postgresql/data"]
        health_check: "pg_isready -U postgres"
        seed_data: "<seeding approach>"

      cache:
        container_image: "<redis:alpine if needed>"
        ports: ["6379:6379"]

      queue:
        container_image: "<rabbitmq if needed>"
        ports: []

      object_storage:
        solution: "minio"
        configuration: "<details>"

    - id: "4.3"
      name: "Seeding & Migrations"
      description: "Database setup workflow"
      commands:
        run_migrations: "<command>"
        seed_data: "<command>"
        reset_database: "<command>"

    - id: "4.4"
      name: "Local Secrets & Config"
      description: "Secret/config management"
      approach: "<.env files|local secrets manager>"
      files:
        - ".env.example (template, committed)"
        - ".env.local (gitignored)"

    - id: "4.5"
      name: "Local Observability"
      description: "Dev-time logging/tracing"

      standard:
        logging: "stdout with structured format"
        tracing: "local Jaeger (optional)"

      aspire:
        dashboard: "Aspire Dashboard (OTLP)"
        setup: "Automatic via AppHost"

    - id: "4.6"
      name: "Prod Parity Notes"
      description: "Document differences"
      parity_gaps:
        - category: "Auth"
          local: "<approach>"
          prod: "<approach>"
          impact: "<impact>"
        - category: "Data"
          local: "<approach>"
          prod: "<approach>"
          impact: "<impact>"

    - id: "4.7"
      name: "Aspire Configuration"
      condition: "when orchestration_mode == aspire"
      steps:
        - "Define services in AppHost"
        - "Configure service discovery"
        - "Enable Aspire Dashboard"
        - "Run: aspire mcp init"

  quick_start:
    prerequisites:
      - "Docker Desktop running"
      - "Local K8s enabled (if using)"
      - "<runtime> installed"

    commands:
      - step: "Clone repository"
        command: "git clone <repo>"
      - step: "Copy env template"
        command: "cp .env.example .env.local"
      - step: "Start dependencies"
        command: "docker-compose up -d"
      - step: "Run migrations"
        command: "<migration command>"
      - step: "Start application"
        command: "<start command>"
      - step: "Verify"
        command: "curl http://localhost:<port>/health"

    estimated_time: "<X minutes>"
```

## Docker Compose Example

```yaml
# docker-compose.yml
version: '3.8'
services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "dev"]

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  db-data:
```

## Aspire AppHost Example

```csharp
var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres")
    .AddDatabase("appdb");

var redis = builder.AddRedis("cache");

builder.AddProject<Projects.MyApp_Api>("api")
    .WithReference(postgres)
    .WithReference(redis);

builder.Build().Run();
```
