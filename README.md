# 🛡️ Caddy with Coraza WAF - Developmi Enterprise Edition

[![GHCR Registry](https://img.shields.io/badge/registry-ghcr.io-blue?style=flat-square)](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)
[![GitHub License](https://img.shields.io/github/license/Miguel-DevOps/caddy-waf?style=flat-square)](LICENSE)
[![OpenSSF Best Practices In Progress](https://img.shields.io/badge/OpenSSF-Best_Practices_In_Progress-orange?style=flat-square)](https://www.bestpractices.dev/en/criteria)

**Production-hardened Caddy web server with Coraza WAF and OWASP CRS** - A secure, performant, and easy-to-deploy web application firewall solution for modern applications.

> **Developmi Enterprise Edition** • Curated by [Miguel Lozano](https://developmi.com) • [GitHub](https://github.com/Miguel-DevOps) • [Container Registry](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)

## ✨ Features

### 🔒 Security First
- **Non-root execution**: Runs as `caddy` user (UID 1337) - no root privileges
- **Supply chain security**: Pinned versions, SHA256 verification of OWASP CRS rules
- **Multi-stage builds**: Minimal attack surface, optimized layers
- **Health monitoring**: Process verification healthcheck
- **Structured logging**: JSON logs for SIEM integration

### 🛡️ WAF Capabilities
- **Coraza WAF v2.2.0**: Modern, high-performance web application firewall
- **OWASP CRS v4.23.0**: Latest Core Rule Set with 290+ protection rules
- **DetectionOnly by default**: Prevents false positives in new deployments
- **Audit logging**: JSON audit logs to stdout for easy monitoring
- **Rate limiting**: Built-in rate limiting plugin for DDoS protection

### 🚀 Production Ready
- **Optimized Alpine base**: Small footprint (~45MB compressed)
- **TLS by default**: Automatic Let's Encrypt integration
- **Multi-architecture**: Supports linux/amd64 and linux/arm64
- **Cloud-native**: Perfect for Kubernetes, Docker Swarm, and standalone Docker

## 🚀 Quick Start

### 1. Pull the Image
```bash
docker pull ghcr.io/miguel-devops/caddy-waf:v1.0.0
```

### 2. Create Environment File
```bash
cp .env.example .env
# Edit .env with your domain/backend/image values
```

### 3. Create Runtime Caddyfile From Template (Strict Mode)
```bash
cp Caddyfile.example Caddyfile
# Edit Caddyfile for your domain and upstreams
```

### 4. Build Your Custom Image (Recommended for your own distribution)
```bash
docker build -t your-registry/your-caddy-waf:custom \
  --build-arg CORAZA_CADDY_REF=v2.2.0 \
  --build-arg CADDY_RATELIMIT_REF=v0.1.0 \
  --build-arg CADDY_DNS_CLOUDFLARE_REF=v0.2.3 \
  .
```

Then set `CADDY_WAF_IMAGE=your-registry/your-caddy-waf:custom` in `.env`.

### 5. Basic Caddyfile Configuration
```caddyfile
# Caddyfile - Save this as Caddyfile in the same directory as docker-compose.yml
{
    order coraza_waf first
}

yourdomain.com {
    respond "Caddy with Coraza WAF is running" 200
}
```

### 6. Start the Container
```bash
docker compose up -d
```

## 📖 Configuration Guide

### WAF Modes
The WAF operates in three modes (configured in Caddyfile):

1. **DetectionOnly** (Default): Logs attacks without blocking - perfect for initial deployment
2. **On**: Active protection - blocks malicious requests
3. **Off**: Disables WAF completely

Recommended rollout for production:
- Keep `SecRuleEngine DetectionOnly` during the initial observation window.
- Review audit logs and tune CRS exclusions based on real traffic.
- Switch to `SecRuleEngine On` only after the application has run long enough to establish a stable false-positive baseline (commonly 7-14 days, depending on traffic diversity and release cadence).

### Example Caddyfile with WAF
```caddyfile
{
    email admin@example.com
    order coraza_waf first
    
    # JSON logging for observability
    log {
        output stdout
        format json
    }
}

(waf) {
    coraza_waf {
        directives `
            Include /etc/caddy/coraza.conf
            Include /etc/caddy/owasp-crs/crs-setup.conf
            Include /etc/caddy/owasp-crs/rules/*.conf
            
            # Start with DetectionOnly, change to On after tuning
            SecRuleEngine DetectionOnly
            
            # Audit logging
            SecAuditEngine RelevantOnly
            SecAuditLog /dev/stdout
            SecAuditLogFormat JSON
        `
    }
}

# Your site configuration
example.com {
    import waf
    reverse_proxy backend:8080
}
```

### Advanced Configuration
For detailed WAF tuning, rule exceptions, and performance optimization, see the complete [TUNING GUIDE](TUNING.md).

Project roadmap and planned security integrations are tracked in [ROADMAP.md](ROADMAP.md).

## 🔧 Customization

### Using Custom OWASP CRS Rules
Mount your custom rules directory:
```yaml
volumes:
  - ./custom-crs:/etc/caddy/owasp-crs
```

### Environment Variables
| Variable | Default | Description |
|----------|---------|-------------|
| `ACME_EMAIL` | (empty) | Email for Let's Encrypt certificates |
| `SITE_ADDRESS` | `localhost` | Site address/server name used by Caddy |
| `BACKEND_UPSTREAM` | `example-app:80` | Reverse proxy backend upstream |
| `CADDY_WAF_IMAGE` | `ghcr.io/miguel-devops/caddy-waf:v1.0.0` | Caddy WAF image reference |
| `EXAMPLE_APP_IMAGE` | `containous/whoami:latest` | Demo backend image |
| `CADDY_ADAPTER` | `caddyfile` | Configuration adapter to use |

### Plugins Included
- `github.com/corazawaf/coraza-caddy/v2@v2.2.0` - Coraza WAF integration
- `github.com/mholt/caddy-ratelimit@v0.1.0` - Rate limiting
- `github.com/caddy-dns/cloudflare@v0.2.3` - Cloudflare DNS for ACME

## 🧪 Testing & Validation

### Verify Installation
```bash
# Check container health
docker ps --filter "name=caddy-waf"

# View logs
docker logs caddy-waf

# Test WAF is working
curl -I https://yourdomain.com
```

### Security Scanning
```bash
# Scan image with Trivy
docker run --rm aquasec/trivy image ghcr.io/miguel-devops/caddy-waf:v1.0.0

# Scan with Docker Scout
docker scout quickview ghcr.io/miguel-devops/caddy-waf:v1.0.0
```

## 📈 Monitoring & Observability

### Log Structure
```json
{
  "level": "info",
  "ts": 1678901234.567,
  "logger": "http.log.access",
  "msg": "handled request",
  "request": {
    "method": "GET",
    "uri": "/test",
    "proto": "HTTP/2",
    "remote_ip": "192.168.1.100"
  },
  "waf_action": "detected",
  "waf_rule_id": "941100"
}
```

### WAF Metrics to Monitor
- `coraza_waf_processed_total` - Total requests processed
- `coraza_waf_blocked_total` - Requests blocked by WAF
- `coraza_waf_rules_triggered` - Rules triggered (by ID)

## 🤝 Contributing & Support

### Issues & Questions
- **GitHub Issues**: [Report bugs or request features](https://github.com/Miguel-DevOps/caddy-waf/issues)
- **Documentation**: [TUNING.md](TUNING.md) for advanced configuration

### Commercial Support
For enterprise support, custom configurations, or security consulting:
- **Website**: [developmi.com](https://developmi.com)
- **Email**: miguel@developmi.com
- **GitHub**: [Miguel-DevOps](https://github.com/Miguel-DevOps)

## 📄 License

MIT License - See [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Caddy Server](https://caddyserver.com) - Amazing web server with automatic HTTPS
- [Coraza WAF](https://coraza.io) - Enterprise-grade WAF engine
- [OWASP Core Rule Set](https://coreruleset.org) - Industry-standard protection rules
- [Developmi](https://developmi.com) - DevOps & Security consulting

---

**Maintained with ❤️ by [Miguel Lozano](https://developmi.com)**