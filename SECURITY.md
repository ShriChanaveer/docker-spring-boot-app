# Security Considerations for Production Deployment

## Overview
This document outlines important security considerations when deploying the Spring Boot application to production.

## Spring Boot Actuator Security

### Current Configuration
The application exposes Spring Boot Actuator endpoints for monitoring and health checks. By default, only `health` and `info` endpoints are exposed for security reasons.

**Current Endpoints:**
- `/actuator/health` - Health check status (needed for Docker health checks)
- `/actuator/info` - Application information

### Production Recommendations

#### 1. Secure Sensitive Endpoints
If you need to expose additional actuator endpoints (like `metrics`, `env`, `beans`, etc.), you **must** add Spring Security:

**Add to pom.xml:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**Update application.properties:**
```properties
# Expose additional endpoints
management.endpoints.web.exposure.include=health,info,metrics,env

# Require authentication for actuator endpoints
management.endpoints.web.base-path=/actuator
spring.security.user.name=${ACTUATOR_USERNAME:admin}
spring.security.user.password=${ACTUATOR_PASSWORD:change-this-password}

# Show health details only when authenticated
management.endpoint.health.show-details=when-authorized
```

**Set environment variables:**
```bash
docker run -d \
  -e ACTUATOR_USERNAME=secure-admin \
  -e ACTUATOR_PASSWORD=strong-random-password \
  spring-boot-app:latest
```

#### 2. Network-Level Security
Consider restricting actuator access at the network level:

**Option A: Use a separate management port**
```properties
# Main application port
server.port=8080

# Management endpoints on different port
management.server.port=9090
```

Then only expose port 8080 externally, and keep 9090 internal.

**Option B: Use path-based restrictions**
Configure your reverse proxy (nginx, AWS ALB, etc.) to block `/actuator/*` from external access.

#### 3. Production Checklist

- [ ] Add Spring Security if exposing sensitive actuator endpoints
- [ ] Use strong passwords stored as environment variables
- [ ] Never expose `/actuator/env` without authentication (shows all env vars)
- [ ] Consider using separate management port (9090) for internal monitoring
- [ ] Configure firewall rules to restrict actuator access
- [ ] Enable HTTPS/TLS for all endpoints
- [ ] Regularly rotate actuator credentials
- [ ] Monitor actuator access logs

## Database Security

### Connection Security
```properties
# Use SSL/TLS for database connections
spring.datasource.url=jdbc:postgresql://db-host:5432/dbname?ssl=true&sslmode=require

# Use connection pooling limits
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.connection-timeout=30000
```

### Secrets Management
**Never hardcode secrets!** Use one of these approaches:

#### Option 1: Environment Variables (Current Approach)
```bash
docker run -d \
  -e DATABASE_PASSWORD=${DB_PASS} \
  spring-boot-app:latest
```

#### Option 2: Docker Secrets (Docker Swarm)
```yaml
services:
  app:
    secrets:
      - db_password
    environment:
      - DATABASE_PASSWORD_FILE=/run/secrets/db_password

secrets:
  db_password:
    external: true
```

#### Option 3: External Secret Management
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager

## Container Security

### Image Security
1. **Scan for vulnerabilities:**
```bash
docker scan spring-boot-app:latest
```

2. **Keep base images updated:**
```bash
# Rebuild regularly to get security patches
docker build --pull -t spring-boot-app:latest .
```

3. **Use specific image versions:**
```dockerfile
# Instead of: FROM eclipse-temurin:17-jre-alpine
FROM eclipse-temurin:17.0.10_7-jre-alpine
```

### Runtime Security
1. **Run as non-root (already configured)**
   - Container runs as user `spring:spring`

2. **Read-only filesystem:**
```bash
docker run -d \
  --read-only \
  --tmpfs /tmp \
  spring-boot-app:latest
```

3. **Drop capabilities:**
```bash
docker run -d \
  --cap-drop=ALL \
  --security-opt=no-new-privileges:true \
  spring-boot-app:latest
```

4. **Resource limits:**
```yaml
deploy:
  resources:
    limits:
      cpus: '2'
      memory: 1G
    reservations:
      cpus: '0.5'
      memory: 512M
```

## Network Security

### 1. Use Private Networks
```yaml
networks:
  app-network:
    driver: bridge
    internal: true  # No external access
```

### 2. Limit Port Exposure
```yaml
# Only expose what's necessary
ports:
  - "127.0.0.1:8080:8080"  # Only localhost
```

### 3. Use TLS/HTTPS
Configure Spring Boot with SSL:
```properties
server.ssl.enabled=true
server.ssl.key-store=/app/keystore.p12
server.ssl.key-store-password=${SSL_KEYSTORE_PASSWORD}
server.ssl.key-store-type=PKCS12
```

## Logging Security

### Don't Log Sensitive Data
```properties
# Configure logging to exclude sensitive headers
logging.level.org.springframework.web=INFO
logging.level.org.hibernate.SQL=WARN

# Never log passwords, tokens, or PII
```

### Centralized Logging
Send logs to external systems instead of storing in containers:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Splunk
- CloudWatch Logs
- Datadog

## Environment Variables Security

### Sensitive Variables
Treat these as secrets:
- `DATABASE_PASSWORD`
- `ACTUATOR_PASSWORD`
- `JWT_SECRET`
- `API_KEYS`

### Best Practices
1. Never commit secrets to git
2. Use `.env` file only for local development
3. Use secret management systems in production
4. Rotate secrets regularly
5. Audit secret access

## Compliance Considerations

### GDPR / Data Protection
- Ensure proper data encryption at rest and in transit
- Implement data retention policies
- Log data access for auditing
- Implement right to deletion

### PCI DSS (if handling payment data)
- Never log credit card numbers
- Implement proper access controls
- Regular security scanning
- Network segmentation

## Security Monitoring

### 1. Container Security Scanning
```bash
# Scan with Trivy
trivy image spring-boot-app:latest

# Scan with Snyk
snyk container test spring-boot-app:latest
```

### 2. Application Monitoring
- Enable Spring Boot Actuator metrics
- Monitor for unusual patterns
- Set up alerts for errors
- Track authentication failures

### 3. Log Monitoring
- Monitor for SQL injection attempts
- Track failed authentication
- Alert on unusual activity patterns
- Regular security audits

## Incident Response

### Preparation
1. Maintain updated container images
2. Have rollback procedures ready
3. Document incident response plan
4. Regular security drills

### Detection
1. Monitor logs for anomalies
2. Set up intrusion detection
3. Regular vulnerability scans
4. Performance monitoring

### Response
1. Isolate affected containers
2. Collect logs and evidence
3. Roll back to known good state
4. Patch and redeploy
5. Post-incident review

## Security Checklist

### Before Deployment
- [ ] All secrets are in environment variables or secret management
- [ ] Actuator endpoints are secured with authentication
- [ ] Database connections use SSL/TLS
- [ ] Container runs as non-root user
- [ ] Resource limits are configured
- [ ] HTTPS/TLS is enabled
- [ ] Security headers are configured
- [ ] Input validation is implemented
- [ ] Dependencies are up to date
- [ ] Security scanning is performed

### Production Runtime
- [ ] Regular security scans scheduled
- [ ] Log monitoring is active
- [ ] Alerts are configured
- [ ] Backup procedures tested
- [ ] Incident response plan documented
- [ ] Access controls reviewed
- [ ] Secrets rotated regularly
- [ ] Compliance requirements met

## Additional Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Spring Security Documentation](https://spring.io/projects/spring-security)
- [Docker Security Best Practices](https://docs.docker.com/develop/security-best-practices/)
- [CIS Docker Benchmark](https://www.cisecurity.org/benchmark/docker)
- [Spring Boot Security Guide](https://spring.io/guides/topicals/spring-security-architecture/)
