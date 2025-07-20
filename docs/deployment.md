# Deployment Rehberi

## Production Ortamına Hazırlık

### 1. Firebase Projesi Kurulumu

```bash
# Firebase CLI kurulum
npm install -g firebase-tools

# Firebase projesine giriş
firebase login
firebase projects:create dereliplast-prod

# Database ve hosting aktifleştir
firebase init
```

### 2. Ortam Değişkenleri

```bash
# Production .env (services/bridge/)
FIREBASE_PROJECT_ID=dereliplast-prod
FIREBASE_PRIVATE_KEY=...
MQTT_BROKER_URL=ssl://production-mqtt-broker:8883
MQTT_USERNAME=production_user
MQTT_PASSWORD=strong_password_here
NODE_ENV=production
```

## ESP32 Firmware Deployment

### 1. OTA (Over-the-Air) Güncelleme

```bash
# GitHub Actions ile otomatik build
git tag v1.0.0
git push origin v1.0.0

# Manual OTA
cd firmware/extruder80_156
pio run --target upload --upload-port [ESP32_IP]
```

### 2. Toplu Güncelleme

```bash
# Birden fazla ESP32 için script
python tools/mass_update.py --version v1.0.0 --machines machine1,machine2,machine3
```

### 3. Rollback Stratejisi

```bash
# Önceki sürüme geri dönüş
python tools/rollback.py --version v0.9.5 --machine machine1
```

## Bridge Service Deployment

### 1. Docker Container

```bash
# Production image build
cd services/bridge
docker build -t dereliplast-bridge:v1.0.0 .

# Docker Hub'a push
docker tag dereliplast-bridge:v1.0.0 yourusername/dereliplast-bridge:v1.0.0
docker push yourusername/dereliplast-bridge:v1.0.0
```

### 2. Kubernetes Deployment

```yaml
# k8s/bridge-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dereliplast-bridge
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dereliplast-bridge
  template:
    metadata:
      labels:
        app: dereliplast-bridge
    spec:
      containers:
      - name: bridge
        image: yourusername/dereliplast-bridge:v1.0.0
        env:
        - name: FIREBASE_PROJECT_ID
          valueFrom:
            secretKeyRef:
              name: firebase-config
              key: project-id
```

### 3. Health Checks

```bash
# Kubernetes health check endpoint
curl http://bridge-service:3000/health

# Response: {"status": "healthy", "mqtt": "connected", "firebase": "connected"}
```

## Dashboard Deployment

### 1. Firebase Hosting

```bash
cd dashboard/app

# Production build
npm run build

# Firebase hosting deploy
firebase deploy --only hosting
```

### 2. CDN Konfigürasyonu

```json
// firebase.json
{
  "hosting": {
    "public": "dist",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [{
      "source": "**",
      "destination": "/index.html"
    }],
    "headers": [{
      "source": "**/*.@(js|css)",
      "headers": [{
        "key": "Cache-Control",
        "value": "max-age=31536000"
      }]
    }]
  }
}
```

## MQTT Broker Deployment

### 1. Mosquitto Cluster

```yaml
# docker-compose.production.yml
version: '3.8'
services:
  mosquitto:
    image: eclipse-mosquitto:2.0
    ports:
      - "1883:1883"
      - "8883:8883"
    volumes:
      - ./mosquitto.conf:/mosquitto/config/mosquitto.conf
      - ./certs:/mosquitto/certs
    restart: unless-stopped
```

### 2. SSL/TLS Konfigürasyonu

```conf
# mosquitto.conf
port 1883
listener 8883
certfile /mosquitto/certs/server.crt
keyfile /mosquitto/certs/server.key
cafile /mosquitto/certs/ca.crt
require_certificate true
use_identity_as_username true
```

## Monitoring ve Logging

### 1. Prometheus Metrics

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'dereliplast-bridge'
    static_configs:
      - targets: ['bridge-service:3000']
    metrics_path: '/metrics'
```

### 2. Grafana Dashboard

Import edilecek JSON dashboard dosyası:
- ESP32 metrics (heap, wifi, uptime)
- MQTT message rates
- Firebase read/write operations
- Error rates ve latency

### 3. Log Aggregation

```bash
# Filebeat configuration
filebeat.inputs:
- type: container
  paths:
    - '/var/lib/docker/containers/*/*.log'
  processors:
  - add_docker_metadata: ~

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
```

## Backup Stratejisi

### 1. Firebase Backup

```bash
# Daily backup script
firebase firestore:export gs://dereliplast-backups/$(date +%Y%m%d)

# Automated backup (Cloud Scheduler)
gcloud scheduler jobs create app-engine backup-firestore \
  --schedule="0 2 * * *" \
  --relative-url="/backup"
```

### 2. Configuration Backup

```bash
# ESP32 konfigürasyonları
python tools/backup_configs.py --output configs-backup-$(date +%Y%m%d).json
```

## Security

### 1. Network Security

```bash
# Firewall rules
ufw allow 1883/tcp  # MQTT
ufw allow 8883/tcp  # MQTT SSL
ufw allow 443/tcp   # HTTPS
ufw deny 22/tcp     # SSH sadece VPN'den
```

### 2. Certificate Management

```bash
# Let's Encrypt sertifikaları
certbot --nginx -d mqtt.dereliplast.com
certbot renew --dry-run
```

### 3. API Rate Limiting

```javascript
// Express rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

app.use('/api/', limiter);
```

## Performance Optimization

### 1. Database Indexing

```javascript
// Firebase Database rules
{
  "rules": {
    "plants": {
      "$plantId": {
        "machines": {
          "$machineId": {
            ".indexOn": ["timestamp", "status"]
          }
        }
      }
    }
  }
}
```

### 2. Caching Strategy

```javascript
// Redis cache
const redis = require('redis');
const client = redis.createClient();

// Cache sensor data for 1 minute
client.setex(`sensor:${machineId}`, 60, JSON.stringify(sensorData));
``` 