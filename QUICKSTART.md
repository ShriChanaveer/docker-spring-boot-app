# Quick Start Guide

## Build and Run in 3 Steps

### Step 1: Build the Docker Image
```bash
docker build -t spring-boot-app:latest .
```

### Step 2: Run the Container
```bash
docker run -d -p 8080:8080 --name spring-boot-app spring-boot-app:latest
```

### Step 3: Test the Application
```bash
# Check health
curl http://localhost:8080/actuator/health

# Test API
curl http://localhost:8080/api/hello
curl http://localhost:8080/api/info
```

## Using Docker Compose (Recommended)

### Step 1: Create Environment File
```bash
cp .env.example .env
# Edit .env if needed
```

### Step 2: Start the Application
```bash
docker-compose up -d
```

### Step 3: View Logs
```bash
docker-compose logs -f app
```

### Step 4: Test
```bash
curl http://localhost:8080/actuator/health
```

### Stop the Application
```bash
docker-compose down
```

## Common Commands

### View Container Logs
```bash
docker logs -f spring-boot-app
# or
docker-compose logs -f app
```

### Restart Container
```bash
docker restart spring-boot-app
# or
docker-compose restart app
```

### Stop Container
```bash
docker stop spring-boot-app
# or
docker-compose stop
```

### Remove Container
```bash
docker rm -f spring-boot-app
# or
docker-compose down
```

## Environment Variables

Set these when running the container:

```bash
docker run -d -p 8080:8080 \
  -e DATABASE_URL=jdbc:postgresql://db-host:5432/mydb \
  -e DATABASE_USERNAME=myuser \
  -e DATABASE_PASSWORD=mypassword \
  -e SERVER_PORT=8080 \
  -e LOG_LEVEL=INFO \
  spring-boot-app:latest
```

Or use `.env` file with docker-compose (recommended).

## Available Endpoints

- `http://localhost:8080/api/hello` - Welcome message
- `http://localhost:8080/api/info` - Application info
- `http://localhost:8080/actuator/health` - Health check

## Documentation

For detailed documentation, see:
- **README-DOCKER.md** - Complete Docker guide with all commands
- **DOCKER-SETUP-SUMMARY.md** - Architecture and design decisions
- **SECURITY.md** - Security best practices and guidelines
- **HELP.md** - Spring Boot reference documentation

## Production Deployment

For production deployment with database:

1. Uncomment PostgreSQL service in `docker-compose.yml`
2. Update `.env` with production credentials
3. Enable database dependencies in `pom.xml`
4. Uncomment database configuration in `application.properties`
5. Run: `docker-compose up -d`

See **SECURITY.md** for production security checklist.

## Troubleshooting

**Container won't start?**
```bash
docker logs spring-boot-app
```

**Port already in use?**
```bash
docker run -d -p 8081:8080 spring-boot-app:latest
```

**Need to rebuild?**
```bash
docker-compose up --build -d
```

For more troubleshooting, see **README-DOCKER.md**.
