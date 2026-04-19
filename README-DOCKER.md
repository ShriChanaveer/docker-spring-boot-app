# Docker Setup for Spring Boot Application

This document provides comprehensive instructions for building and running the Spring Boot application using Docker.

## Prerequisites

- Docker Engine 20.10+ installed
- Docker Compose 2.0+ installed
- At least 2GB of free disk space

## Quick Start

### 1. Build the Docker Image

```bash
docker build -t spring-boot-app:latest .
```

### 2. Run the Container

#### Option A: Using Docker Command
```bash
docker run -d \
  --name spring-boot-app \
  -p 8080:8080 \
  -e DATABASE_URL=jdbc:postgresql://host.docker.internal:5432/appdb \
  -e DATABASE_USERNAME=appuser \
  -e DATABASE_PASSWORD=apppassword \
  spring-boot-app:latest
```

#### Option B: Using Docker Compose (Recommended)
```bash
# Copy environment variables example file
cp .env.example .env

# Edit .env file with your configuration
nano .env  # or use your preferred editor

# Start the application
docker-compose up -d

# View logs
docker-compose logs -f app

# Stop the application
docker-compose down
```

## Build Commands

### Standard Build
```bash
docker build -t spring-boot-app:latest .
```

### Build with Custom Tag
```bash
docker build -t myorg/spring-boot-app:1.0.0 .
```

### Build with No Cache (Fresh Build)
```bash
docker build --no-cache -t spring-boot-app:latest .
```

### Build for Specific Platform
```bash
# For ARM64 (Apple M1/M2)
docker build --platform linux/arm64 -t spring-boot-app:latest .

# For AMD64 (Intel/AMD)
docker build --platform linux/amd64 -t spring-boot-app:latest .
```

## Run Commands

### Basic Run
```bash
docker run -d -p 8080:8080 --name spring-boot-app spring-boot-app:latest
```

### Run with Environment Variables
```bash
docker run -d \
  --name spring-boot-app \
  -p 8080:8080 \
  -e SERVER_PORT=8080 \
  -e DATABASE_URL=jdbc:postgresql://postgres:5432/appdb \
  -e DATABASE_USERNAME=appuser \
  -e DATABASE_PASSWORD=securepassword \
  -e SPRING_PROFILES_ACTIVE=prod \
  spring-boot-app:latest
```

### Run with Custom Memory Limits
```bash
docker run -d \
  --name spring-boot-app \
  -p 8080:8080 \
  --memory="1g" \
  --memory-reservation="512m" \
  --cpus="2" \
  spring-boot-app:latest
```

### Run with Volume Mount (for external configuration)
```bash
docker run -d \
  --name spring-boot-app \
  -p 8080:8080 \
  -v $(pwd)/config:/app/config \
  spring-boot-app:latest
```

## Docker Compose Commands

### Start Services
```bash
# Start in foreground (see logs)
docker-compose up

# Start in background (detached mode)
docker-compose up -d

# Build and start
docker-compose up --build -d
```

### Stop Services
```bash
# Stop services (containers remain)
docker-compose stop

# Stop and remove containers, networks
docker-compose down

# Stop, remove containers, networks, and volumes
docker-compose down -v
```

### View Logs
```bash
# View all logs
docker-compose logs

# Follow logs in real-time
docker-compose logs -f

# View logs for specific service
docker-compose logs -f app

# View last 100 lines
docker-compose logs --tail=100 app
```

### Restart Services
```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart app
```

### Scale Services
```bash
# Run multiple instances
docker-compose up -d --scale app=3
```

## Useful Docker Commands

### Container Management
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop container
docker stop spring-boot-app

# Start container
docker start spring-boot-app

# Restart container
docker restart spring-boot-app

# Remove container
docker rm spring-boot-app

# Remove container forcefully
docker rm -f spring-boot-app
```

### Logs and Debugging
```bash
# View container logs
docker logs spring-boot-app

# Follow logs in real-time
docker logs -f spring-boot-app

# View last 100 lines
docker logs --tail=100 spring-boot-app

# Execute bash in running container
docker exec -it spring-boot-app sh

# View container stats
docker stats spring-boot-app

# Inspect container
docker inspect spring-boot-app
```

### Image Management
```bash
# List images
docker images

# Remove image
docker rmi spring-boot-app:latest

# Remove unused images
docker image prune

# View image layers
docker history spring-boot-app:latest
```

### Health Check
```bash
# Check health status
docker inspect --format='{{.State.Health.Status}}' spring-boot-app

# View health check logs
docker inspect --format='{{range .State.Health.Log}}{{.Output}}{{end}}' spring-boot-app
```

## Environment Variables

| Variable | Description | Default Value |
|----------|-------------|---------------|
| `SERVER_PORT` | Application server port | `8080` |
| `DATABASE_URL` | JDBC database connection URL | `jdbc:postgresql://postgres:5432/appdb` |
| `DATABASE_USERNAME` | Database username | `appuser` |
| `DATABASE_PASSWORD` | Database password | `apppassword` |
| `JPA_DDL_AUTO` | Hibernate DDL auto mode | `update` |
| `JPA_SHOW_SQL` | Show SQL queries in logs | `false` |
| `LOG_LEVEL` | Root logging level | `INFO` |
| `JAVA_OPTS` | Additional JVM options | (see Dockerfile) |
| `SPRING_PROFILES_ACTIVE` | Active Spring profiles | - |

## Production Deployment

### Best Practices

1. **Use Environment Variables**: Never hardcode sensitive data
2. **Set Resource Limits**: Prevent container from consuming all host resources
3. **Enable Health Checks**: Ensure container restarts if unhealthy
4. **Use Secrets Management**: For production, use Docker secrets or external secret managers
5. **Monitor Logs**: Use centralized logging (ELK, Splunk, etc.)
6. **Regular Updates**: Keep base images and dependencies updated

### Docker Secrets (Docker Swarm)
```bash
# Create secrets
echo "securepassword" | docker secret create db_password -

# Use in docker-compose.yml (swarm mode)
# services:
#   app:
#     secrets:
#       - db_password
#     environment:
#       - DATABASE_PASSWORD_FILE=/run/secrets/db_password
```

### Production Compose Override
Create a `docker-compose.prod.yml`:
```yaml
version: '3.8'
services:
  app:
    restart: always
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '4'
          memory: 2G
```

Run with:
```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## Troubleshooting

### Application Won't Start
```bash
# Check logs
docker logs spring-boot-app

# Check if port is already in use
netstat -an | grep 8080
# or
lsof -i :8080
```

### Out of Memory Errors
```bash
# Increase memory limit
docker run -d -p 8080:8080 --memory="2g" spring-boot-app:latest

# Or adjust JAVA_OPTS
docker run -d -p 8080:8080 \
  -e JAVA_OPTS="-XX:MaxRAMPercentage=50.0" \
  spring-boot-app:latest
```

### Slow Build Times
```bash
# Use BuildKit for faster builds
DOCKER_BUILDKIT=1 docker build -t spring-boot-app:latest .

# Check layer caching
docker build --progress=plain -t spring-boot-app:latest .
```

### Database Connection Issues
```bash
# Test database connectivity from within container
docker exec -it spring-boot-app sh
wget -O- http://localhost:8080/actuator/health
```

## Advanced Features

### Multi-Stage Build Explanation

The Dockerfile uses a multi-stage build:
1. **Build Stage**: Uses Maven with full JDK to compile and package
2. **Runtime Stage**: Uses slim JRE image, copies only the JAR file

Benefits:
- Smaller final image (~200MB vs ~800MB)
- Faster deployment
- Better security (no build tools in production)
- Optimized layer caching

### JVM Optimizations

The container includes production-ready JVM settings:
- `XX:+UseContainerSupport`: Respects container memory limits
- `XX:MaxRAMPercentage=75.0`: Uses up to 75% of available memory
- `XX:InitialRAMPercentage=50.0`: Starts with 50% of available memory
- Faster startup with non-blocking entropy source

### Non-Root User

The application runs as user `spring` (non-root) for security:
- Reduces attack surface
- Follows least-privilege principle
- Required by many enterprise security policies

## Testing the Deployment

### Health Check Endpoint
```bash
curl http://localhost:8080/actuator/health
```

### Application Logs
```bash
docker logs -f spring-boot-app
```

### Resource Usage
```bash
docker stats spring-boot-app
```

## Cleanup

### Remove Everything
```bash
# Stop and remove containers
docker-compose down

# Remove images
docker rmi spring-boot-app:latest

# Remove volumes (if any)
docker volume prune

# Remove all unused resources
docker system prune -a
```

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
