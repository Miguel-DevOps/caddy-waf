# 🛡️ Caddy con Coraza WAF - Developmi Enterprise Edition

[![Docker Pulls](https://img.shields.io/docker/pulls/miguel-devops/caddy-waf?style=flat-square)](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)
[![GitHub License](https://img.shields.io/github/license/Miguel-DevOps/caddy+waf?style=flat-square)](LICENSE)
[![OpenSSF Best Practices](https://www.bestpractices.dev/projects/9617/badge)](https://www.bestpractices.dev/projects/9617)

**Servidor web Caddy endurecido para producción con Coraza WAF y OWASP CRS** - Una solución de firewall de aplicaciones web segura, performante y fácil de desplegar para aplicaciones modernas.

> **Developmi Enterprise Edition** • Curado por [Miguel Lozano](https://developmi.com) • [GitHub](https://github.com/Miguel-DevOps) • [Container Registry](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)

## ✨ Características

### 🔒 Seguridad Primero
- **Ejecución sin privilegios root**: Se ejecuta como usuario `caddy` (UID 1000) - sin privilegios de root
- **Seguridad de cadena de suministro**: Versiones fijadas, verificación SHA256 de reglas OWASP CRS
- **Builds multi-etapa**: Superficie de ataque mínima, capas optimizadas
- **Monitoreo de salud**: Healthcheck que verifica el proceso activo
- **Logs estructurados**: Logs JSON para integración con SIEM

### 🛡️ Capacidades del WAF
- **Coraza WAF v2.1.0**: Firewall de aplicaciones web moderno y de alto rendimiento
- **OWASP CRS v4.23.0**: Último conjunto de reglas principales con 290+ reglas de protección
- **DetectionOnly por defecto**: Previene falsos positivos en nuevos despliegues
- **Logs de auditoría**: Logs de auditoría JSON a stdout para monitoreo fácil
- **Rate limiting**: Plugin de limitación de tasa integrado para protección contra DDoS

### 🚀 Listo para Producción
- **Base Alpine optimizada**: Huella pequeña (~45MB comprimido)
- **TLS por defecto**: Integración automática con Let's Encrypt
- **Multi-arquitectura**: Soporta linux/amd64 y linux/arm64
- **Cloud-native**: Perfecto para Kubernetes, Docker Swarm y Docker independiente

## 🚀 Inicio Rápido

### 1. Descargar la Imagen
```bash
docker pull ghcr.io/miguel-devops/caddy-waf:latest
```

### 2. Docker Compose (Recomendado)
```yaml
# docker-compose.yml

services:
  caddy-waf:
    image: ghcr.io/miguel-devops/caddy-waf:latest
    container_name: caddy-waf
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
      - caddy_config:/config
    environment:
      - ACME_EMAIL=admin@yourdomain.com

volumes:
  caddy_data:
  caddy_config:
```

### 3. Configuración Básica de Caddyfile
```caddyfile
# Caddyfile - Guarda esto como Caddyfile en el mismo directorio que docker-compose.yml
{
    email admin@yourdomain.com
    order coraza_waf first
}

yourdomain.com, www.yourdomain.com {
    respond "¡Caddy con Coraza WAF está funcionando! 🚀" 200
}
```

### 4. Iniciar el Contenedor
```bash
docker-compose up -d
```

## 📖 Guía de Configuración

### Modos del WAF
El WAF opera en tres modos (configurados en Caddyfile):

1. **DetectionOnly** (Por defecto): Registra ataques sin bloquear - perfecto para despliegue inicial
2. **On**: Protección activa - bloquea solicitudes maliciosas
3. **Off**: Desactiva el WAF completamente

### Ejemplo de Caddyfile con WAF
```caddyfile
{
    email admin@example.com
    order coraza_waf first
    
    # Logs JSON para observabilidad
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
            
            # Comenzar con DetectionOnly, cambiar a On después de ajustar
            SecRuleEngine DetectionOnly
            
            # Logs de auditoría
            SecAuditEngine RelevantOnly
            SecAuditLog /dev/stdout
            SecAuditLogFormat JSON
        `
    }
}

# Tu configuración de sitio
example.com {
    import waf
    reverse_proxy backend:8080
}
```

### Configuración Avanzada
Para ajuste detallado del WAF, excepciones de reglas y optimización de rendimiento, consulta la [GUÍA DE AJUSTE](TUNING.md) completa.

## 🔧 Personalización

### Usando Reglas OWASP CRS Personalizadas
Monta tu directorio de reglas personalizadas:
```yaml
volumes:
  - ./custom-crs:/etc/caddy/owasp-crs
```

### Variables de Entorno
| Variable | Por defecto | Descripción |
|----------|---------|-------------|
| `ACME_EMAIL` | (ninguno) | Email para certificados Let's Encrypt |
| `CADDY_ADAPTER` | `caddyfile` | Adaptador de configuración a usar |

### Plugins Incluidos
- `github.com/corazawaf/coraza-caddy/v2@v2.1.0` - Integración Coraza WAF
- `github.com/mholt/caddy-ratelimit` - Limitación de tasa
- `github.com/caddy-dns/cloudflare` - DNS de Cloudflare para ACME

## 🧪 Pruebas y Validación

### Verificar Instalación
```bash
# Verificar salud del contenedor
docker ps --filter "name=caddy-waf"

# Ver logs
docker logs caddy-waf

# Probar que el WAF funciona
curl -I https://yourdomain.com
```

### Escaneo de Seguridad
```bash
# Escanear imagen con Trivy
docker run --rm aquasec/trivy image ghcr.io/miguel-devops/caddy-waf:latest

# Escanear con Docker Scout
docker scout quickview ghcr.io/miguel-devops/caddy-waf:latest
```

## 📈 Monitoreo y Observabilidad

### Estructura de Logs
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

### Métricas del WAF a Monitorear
- `coraza_waf_processed_total` - Total de solicitudes procesadas
- `coraza_waf_blocked_total` - Solicitudes bloqueadas por WAF
- `coraza_waf_rules_triggered` - Reglas disparadas (por ID)

## 🤝 Contribuciones y Soporte

### Problemas y Preguntas
- **Issues de GitHub**: [Reportar bugs o solicitar características](https://github.com/Miguel-DevOps/caddy-waf/issues)
- **Documentación**: [TUNING.md](TUNING.md) para configuración avanzada

### Soporte Comercial
Para soporte empresarial, configuraciones personalizadas o consultoría de seguridad:
- **Sitio web**: [developmi.com](https://developmi.com)
- **Email**: miguel@developmi.com
- **GitHub**: [Miguel-DevOps](https://github.com/Miguel-DevOps)

## 📄 Licencia

Licencia MIT - Ver archivo [LICENSE](LICENSE) para detalles.

## 🙏 Agradecimientos

- [Caddy Server](https://caddyserver.com) - Increíble servidor web con HTTPS automático
- [Coraza WAF](https://coraza.io) - Motor WAF de grado empresarial
- [OWASP Core Rule Set](https://coreruleset.org) - Reglas de protección estándar de la industria
- [Developmi](https://developmi.com) - Consultoría DevOps & Seguridad

---

**Mantenido con ❤️ por [Miguel Lozano](https://developmi.com)**