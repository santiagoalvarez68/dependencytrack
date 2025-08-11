# Dependency-Track PoC Setup

This repository contains a Proof of Concept (PoC) setup for [Dependency-Track](https://dependencytrack.org/), an intelligent Component Analysis platform that allows organizations to identify and reduce risk in the software supply chain.

## Overview

Dependency-Track helps you:
- **Continuous SBOM Analysis**: Analyze Software Bill of Materials (SBOM) for security vulnerabilities
- **Vulnerability Management**: Identify known vulnerabilities using multiple intelligence sources (NVD, GitHub Advisories, Snyk, OSV, etc.)
- **Policy Compliance**: Enforce security, operational, and license policies
- **Impact Analysis**: Quickly respond to vulnerabilities across your portfolio
- **Full-Stack Inventory**: Track components across applications, containers, operating systems, and more

## Architecture

This PoC includes:
- **Dependency-Track API Server**: Core analysis engine
- **Dependency-Track Frontend**: Web interface for visualization and management
- **PostgreSQL Database**: Persistent storage for analysis results and metadata
- **Volume Mounts**: For SBOM uploads and persistent data storage

## Prerequisites

- Docker Desktop or Docker Engine with Docker Compose
- At least 8GB RAM available for containers
- 10GB free disk space for data volumes

## Quick Start

1. **Clone this repository**:
   ```bash
   git clone <repository-url>
   cd dependencytrack
   ```

2. **Start the services**:
   ```bash
   docker compose up -d
   ```

3. **Access the application**:
   - Frontend: http://localhost:8080
   - API Server: http://localhost:8081
   
4. **Default credentials**:
   - Username: `admin`
   - Password: `admin`
   
   ⚠️ **Important**: Change the default password immediately after first login!

## Configuration

### Environment Variables

Key configuration options in `docker-compose.yml`:

- **Database Configuration**: PostgreSQL connection settings
- **Memory Allocation**: API server configured with 8GB RAM
- **Port Mapping**: Frontend (8080), API (8081), Database (5432)
- **Volume Mounts**: Persistent storage for data and SBOM uploads

### Volume Structure

```
dependency-track-data/     # Main data volume
├── uploads/              # SBOM file uploads
├── reports/              # Generated reports
└── config/               # Configuration overrides
```

## SBOM Analysis Use Cases

This PoC is configured to handle various types of SBOM analysis:

### 1. Docker Image Analysis
- Upload SBOMs generated from Docker images using tools like:
  - [Syft](https://github.com/anchore/syft)
  - [Docker SBOM](https://github.com/docker/sbom-cli-plugin)
  - [Trivy](https://github.com/aquasecurity/trivy)

### 2. Ubuntu Machine Analysis
- Analyze system packages and installed components
- Track OS-level vulnerabilities and compliance

### 3. Python Project Analysis
- Scan Python dependencies from:
  - `requirements.txt`
  - `Pipfile`
  - `pyproject.toml`
- Tools: [CycloneDX Python](https://github.com/CycloneDX/cyclonedx-python)

### 4. Go Project Analysis
- Analyze Go modules and dependencies
- Scan `go.mod` and `go.sum` files
- Tools: [CycloneDX Go](https://github.com/CycloneDX/cyclonedx-go)

## SBOM Upload Methods

### 1. Web Interface
1. Navigate to http://localhost:8080
2. Create a new project
3. Upload SBOM files directly through the UI

### 2. REST API
```bash
# Upload SBOM via API
curl -X POST "http://localhost:8081/api/v1/bom" \
  -H "X-API-Key: YOUR_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F "project=PROJECT_UUID" \
  -F "bom=@path/to/sbom.json"
```

### 3. CLI Tools
Install and use the [Dependency-Track CLI](https://github.com/DependencyTrack/client-cli):
```bash
# Upload using CLI
dtrack upload --server-url http://localhost:8081 \
  --api-key YOUR_API_KEY \
  --project-name "My Project" \
  --bom-file sbom.json
```

## Example SBOM Generation

### Docker Images
```bash
# Using Syft
syft packages docker:image:tag -o cyclonedx-json > image-sbom.json

# Using Trivy
trivy image --format cyclonedx --output image-sbom.json image:tag
```

### Python Projects
```bash
# Install CycloneDX Python
pip install cyclonedx-bom

# Generate SBOM
cyclonedx-py -o sbom.json
```

### Go Projects
```bash
# Install CycloneDX Go
go install github.com/CycloneDX/cyclonedx-go/cmd/cyclonedx-go@latest

# Generate SBOM
cyclonedx-go -output sbom.json
```

## Monitoring and Maintenance

### Health Checks
```bash
# Check service status
docker compose ps

# View logs
docker compose logs -f dtrack-apiserver
docker compose logs -f dtrack-frontend
docker compose logs -f postgres
```

### Backup
```bash
# Backup database
docker compose exec postgres pg_dump -U dtrack dtrack > backup.sql

# Backup data volume
docker run --rm -v dependency-track-data:/data -v $(pwd):/backup alpine tar czf /backup/data-backup.tar.gz /data
```

### Updates
```bash
# Pull latest images
docker compose pull

# Restart with new images
docker compose up -d
```

## Security Considerations

1. **Change Default Credentials**: Update admin password immediately
2. **API Keys**: Generate and use API keys for programmatic access
3. **Network Security**: Consider placing behind reverse proxy in production
4. **Database Security**: Use strong database passwords
5. **Volume Permissions**: Ensure proper file permissions on mounted volumes

## Troubleshooting

### Common Issues

1. **Memory Issues**: Ensure Docker has at least 8GB RAM allocated
2. **Port Conflicts**: Check if ports 8080, 8081, or 5432 are already in use
3. **Volume Permissions**: Ensure Docker can write to mounted volumes
4. **Database Connection**: Check PostgreSQL container logs if API server fails to start

### Useful Commands
```bash
# Reset everything (⚠️ Will delete all data!)
docker compose down -v
docker volume rm dependency-track-data dependency-track-postgres

# View detailed logs
docker compose logs --tail=100 -f

# Access database directly
docker compose exec postgres psql -U dtrack -d dtrack
```

## Resources

- [Dependency-Track Documentation](https://docs.dependencytrack.org/)
- [CycloneDX SBOM Standard](https://cyclonedx.org/)
- [OWASP Dependency-Track](https://owasp.org/www-project-dependency-track/)
- [GitHub Repository](https://github.com/DependencyTrack/dependency-track)

## License

This PoC setup is provided as-is for educational and evaluation purposes. Dependency-Track is licensed under Apache 2.0.
