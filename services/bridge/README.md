# Bridge Service

MQTT ve Firebase arasında köprü görevi yapan TypeScript servisi.

## Özellikler

- MQTT subscriber (makine verilerini dinler)
- Firebase Realtime Database writer
- Komut iletimi (Firebase → MQTT)
- Offline buffer management
- Custom token validation

## Kurulum

```bash
cd services/bridge
npm install
npm run dev
```

## Yapılandırma

- `.env`: Firebase credentials, MQTT broker bilgileri
- `src/config.ts`: Servis ayarları

## Docker

```bash
docker build -t dereliplast-bridge .
docker run -d --env-file .env dereliplast-bridge
``` 