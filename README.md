# Two-Layer CDN Architecture

A demonstration of a multi-tier Content Delivery Network (CDN) architecture using Docker and Nginx. This project implements a three-layer caching system with origin server, shield layer, and edge layer for optimal content delivery and performance.

## 🏗️ Architecture Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │───▶│    Edge     │───▶│   Shield    │───▶│   Origin    │
│             │    │   (Port 80) │    │   (Port 80) │    │   (Port 80) │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

### Components

1. **Origin Server** (`origin-site/`)
   - Serves the original static content (HTML, CSS, JS, images)
   - Contains the main website files
   - No caching layer

2. **Shield Layer** (`origin-shield/`)
   - First caching layer that sits between edge and origin
   - Implements intelligent caching strategies
   - Reduces load on the origin server

3. **Edge Layer** (`edge/`)
   - Front-facing layer that clients connect to
   - Implements rate limiting and edge caching
   - Exposed on port 8082

## 🚀 Quick Start

### Prerequisites
- Docker
- Docker Compose

### Running the Application

1. **Clone and navigate to the project:**
   ```bash
   cd two-layer-cdn
   ```

2. **Start all services:**
   ```bash
   docker-compose up --build
   ```

3. **Access the application:**
   - Open your browser and navigate to `http://localhost:8082`
   - The edge layer will serve content through the CDN architecture

4. **Stop the services:**
   ```bash
   docker-compose down
   ```

## 📁 Project Structure

```
two-layer-cdn/
├── docker-compose.yaml          # Main orchestration file
├── origin-site/                 # Origin server
│   ├── Dockerfile              # Origin container configuration
│   ├── nginx.conf              # Origin Nginx configuration
│   ├── index.html              # Main website file
│   └── assets/                 # Static assets
│       ├── css/
│       ├── js/
│       └── images/
├── origin-shield/              # Shield layer
│   ├── Dockerfile              # Shield container configuration
│   └── nginx.conf              # Shield Nginx configuration
└── edge/                       # Edge layer
    ├── Dockerfile              # Edge container configuration
    └── nginx.conf              # Edge Nginx configuration
```

## ⚙️ Configuration Details

### Origin Server
- **Port:** 80 (internal)
- **Purpose:** Serves original content
- **Features:**
  - Health check endpoint at `/_status/healthz`
  - Static file serving
  - Basic logging

### Shield Layer
- **Port:** 80 (internal)
- **Purpose:** First-level caching
- **Caching Strategy:**
  - CSS files: No caching (always fresh)
  - PNG images: 3-minute cache
  - Other files: 1-minute cache
- **Features:**
  - Proxy caching with `origin_cache` zone
  - Cache status headers (`X-Origin-Cache`)

### Edge Layer
- **Port:** 8082 (external)
- **Purpose:** Front-facing with rate limiting
- **Caching Strategy:**
  - CSS files: No caching
  - PNG images: 1-minute cache (after 3 requests)
  - Other files: 2-minute cache
- **Features:**
  - Rate limiting (200 req/s with burst of 20)
  - Edge caching with `edge_cache` zone
  - Cache status headers (`X-Edge-Cache`)

## 🔧 Customization

### Modifying Cache Durations
Edit the respective `nginx.conf` files:

- **Shield layer:** Modify `proxy_cache_valid` values in `origin-shield/nginx.conf`
- **Edge layer:** Modify `proxy_cache_valid` values in `edge/nginx.conf`

### Changing Rate Limits
Edit `edge/nginx.conf`:
```nginx
limit_req_zone $binary_remote_addr zone=normal_limit:10m rate=200r/s;
```

### Adding New File Types
Add new location blocks in the respective nginx configurations:
```nginx
location ~* \.extension$ {
    proxy_pass http://upstream:80;
    proxy_cache cache_zone;
    proxy_cache_valid 200 5m;
}
```

## 📊 Monitoring

### Health Checks
- Origin server health endpoint: `http://localhost:8082/_status/healthz`

### Cache Headers
Monitor cache effectiveness using response headers:
- `X-Origin-Cache`: Shield layer cache status
- `X-Edge-Cache`: Edge layer cache status

### Logs
View logs for each service:
```bash
# Edge layer logs
docker-compose logs edge

# Shield layer logs
docker-compose logs shield

# Origin server logs
docker-compose logs origin
```

## 🧪 Testing

### Cache Testing
1. **First request:** Should show cache miss
2. **Subsequent requests:** Should show cache hit
3. **After cache expiry:** Should show cache miss again

### Rate Limiting Test
Use tools like `ab` (Apache Bench) to test rate limiting:
```bash
ab -n 1000 -c 10 http://localhost:8082/
```

## 🔍 Troubleshooting

### Common Issues

1. **Port already in use:**
   ```bash
   # Check what's using port 8082
   lsof -i :8082
   # Kill the process or change the port in docker-compose.yaml
   ```

2. **Container won't start:**
   ```bash
   # Check container logs
   docker-compose logs [service-name]
   ```

3. **Cache not working:**
   - Verify nginx configurations
   - Check cache headers in browser dev tools
   - Ensure cache directories are writable

### Challenges Faced

1. Limited experience with Nginx configuration and rate limiting mechanisms.

### Debug Mode
Run with verbose logging:
```bash
docker-compose up --build --verbose
```

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

**Note:** This is a demonstration project for learning CDN architectures. For production use, consider additional security measures, monitoring, and scaling strategies. 