# 🛡️ Caddy con Coraza WAF - Developmi Enterprise Edition

[![Registro GHCR](https://img.shields.io/badge/registro-ghcr.io-blue?style=flat-square)](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)
[![GitHub License](https://img.shields.io/github/license/Miguel-DevOps/caddy-waf?style=flat-square)](LICENSE)
[![OpenSSF Best Practices En Curso](https://img.shields.io/badge/OpenSSF-Best_Practices_En_Curso-orange?style=flat-square)](https://www.bestpractices.dev/en/criteria)

**Servidor web Caddy endurecido para producción con Coraza WAF y OWASP CRS** - Una solución de firewall de aplicaciones web segura, performante y fácil de desplegar para aplicaciones modernas.

> **Developmi Enterprise Edition** • Curado por [Miguel Lozano](https://developmi.com) • [GitHub](https://github.com/Miguel-DevOps) • [Container Registry](https://github.com/Miguel-DevOps/caddy-waf/pkgs/container/caddy-waf)

## ✨ Características

### 🔒 Seguridad Primero
- **Ejecución sin privilegios root**: Se ejecuta como usuario `caddy` (UID 1337) - sin privilegios de root
- **Seguridad de cadena de suministro**: Versiones fijadas, verificación SHA256 de reglas OWASP CRS
- **Builds multi-etapa**: Superficie de ataque mínima, capas optimizadas
- **Monitoreo de salud**: Healthcheck que verifica el proceso activo
- **Logs estructurados**: Logs JSON para integración con SIEM

### 🛡️ Capacidades del WAF
- **Coraza WAF v2.2.0**: Firewall de aplicaciones web moderno y de alto rendimiento
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
docker pull ghcr.io/miguel-devops/caddy-waf:v1.0.0
```

### 2. Crear Archivo de Entorno
```bash
cp .env.example .env
# Edita .env con tus valores de dominio/backend/imagen
```

### 3. Crear Caddyfile de Ejecucion Desde la Plantilla (Modo Estricto)
```bash
cp Caddyfile.example Caddyfile
# Edita Caddyfile para tu dominio y upstreams
```

### 4. Construir Tu Imagen Personalizada (Recomendado para tu distribución)
```bash
docker build -t tu-registry/tu-caddy-waf:custom \
  --build-arg CORAZA_CADDY_REF=v2.2.0 \
  --build-arg CADDY_RATELIMIT_REF=v0.1.0 \
  --build-arg CADDY_DNS_CLOUDFLARE_REF=v0.2.3 \
  .
```

Luego define `CADDY_WAF_IMAGE=tu-registry/tu-caddy-waf:custom` en `.env`.

### 5. Configuración Básica de Caddyfile
```caddyfile
# Caddyfile - Guarda esto como Caddyfile en el mismo directorio que docker-compose.yml
{
    order coraza_waf first
}

yourdomain.com {
    respond "Caddy con Coraza WAF está funcionando" 200
}
```

### 6. Iniciar el Contenedor
```bash
docker compose up -d
```

## 📖 Guía de Configuración

### Modos del WAF
El WAF opera en tres modos (configurados en Caddyfile):

1. **DetectionOnly** (Por defecto): Registra ataques sin bloquear - perfecto para despliegue inicial
2. **On**: Protección activa - bloquea solicitudes maliciosas
3. **Off**: Desactiva el WAF completamente

Despliegue recomendado para producción:
- Mantener `SecRuleEngine DetectionOnly` durante la ventana inicial de observación.
- Revisar logs de auditoría y ajustar exclusiones CRS según tráfico real.
- Cambiar a `SecRuleEngine On` solo cuando exista una línea base estable de falsos positivos (comúnmente 7-14 días, según diversidad de tráfico y frecuencia de cambios).

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

El roadmap del proyecto y futuras integraciones de seguridad están en [ROADMAP.md](ROADMAP.md).

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
| `ACME_EMAIL` | (vacío) | Email para certificados Let's Encrypt |
| `SITE_ADDRESS` | `localhost` | Dirección de sitio/server name usado por Caddy |
| `BACKEND_UPSTREAM` | `example-app:80` | Upstream del reverse proxy |
| `CADDY_WAF_IMAGE` | `ghcr.io/miguel-devops/caddy-waf:v1.0.0` | Referencia de imagen Caddy WAF |
| `EXAMPLE_APP_IMAGE` | `containous/whoami:latest` | Imagen backend de demostración |
| `CADDY_ADAPTER` | `caddyfile` | Adaptador de configuración a usar |

### Plugins Incluidos
- `github.com/corazawaf/coraza-caddy/v2@v2.2.0` - Integración Coraza WAF
- `github.com/mholt/caddy-ratelimit@v0.1.0` - Limitación de tasa
- `github.com/caddy-dns/cloudflare@v0.2.3` - DNS de Cloudflare para ACME

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
docker run --rm aquasec/trivy image ghcr.io/miguel-devops/caddy-waf:v1.0.0

# Escanear con Docker Scout
docker scout quickview ghcr.io/miguel-devops/caddy-waf:v1.0.0
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