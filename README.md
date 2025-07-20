# 🏭 Akıllı Extruder Platformu

Endüstriyel plastik extruder makineleri için IoT tabanlı izleme ve kontrol sistemi.

[![CI Pipeline](https://github.com/mehmeterendereli/dereliplast/actions/workflows/ci.yml/badge.svg)](https://github.com/mehmeterendereli/dereliplast/actions/workflows/ci.yml)
[![Firmware OTA](https://github.com/mehmeterendereli/dereliplast/actions/workflows/firmware-ota.yml/badge.svg)](https://github.com/mehmeterendereli/dereliplast/actions/workflows/firmware-ota.yml)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## 🎯 Özellikler

- **🔄 Kesintisiz Üretim**: Wi-Fi bağlantısı kesilse bile PLC üzerinden üretim devam eder
- **📊 Gerçek Zamanlı İzleme**: Sensör verilerinin anlık takibi ve görselleştirme
- **🎮 Uzaktan Kontrol**: Web dashboard üzerinden makine kontrolü
- **📈 Veri Analizi**: Üretim verilerinin analizi ve raporlanması
- **⚠️ Arıza Tahmini**: Sensör verilerinden otomatik alarm sistemi
- **🔐 Güvenlik**: Token tabanlı makine kimlik doğrulama

## 🏗️ Sistem Mimarisi

```
[ESP32] ←→ [PLC] ←→ [Extruder Makinesi]
   ↓
[MQTT Broker]
   ↓  
[Bridge Service] ←→ [Firebase RTDB]
   ↓
[React Dashboard]
```

## 🚀 Hızlı Başlangıç

### Gereksinimler

- **Node.js** 20.x LTS
- **Python** 3.12+
- **PlatformIO Core** 6.1+
- **Docker Desktop** 4.x
- **Visual Studio Code** 1.90+

### Kurulum

```bash
# 1. Repo'yu klonlayın
git clone https://github.com/mehmeterendereli/dereliplast.git
cd dereliplast

# 2. Bridge service bağımlılıklarını yükleyin
cd services/bridge
npm install

# 3. Dashboard bağımlılıklarını yükleyin  
cd ../../dashboard/app
npm install

# 4. Docker servislerini başlatın
docker run -d --name mosquitto -p 1883:1883 eclipse-mosquitto

# 5. Simülatörü çalıştırın
cd ../../tools
python simulator.py
```

## 📁 Proje Yapısı

```
dereliplast/
├── 🔧 firmware/           # ESP32 firmware (C++)
│   └── extruder80_156/    # Extruder 80/156 için özel kod
├── 🌐 services/           # Backend servisler
│   └── bridge/            # MQTT ↔ Firebase köprüsü (TypeScript)
├── 🎨 dashboard/          # Frontend dashboard
│   └── app/               # React + Vite + Tailwind
├── 🛠️ tools/              # Yardımcı araçlar
│   └── simulator.py      # MQTT simülatörü
├── 📖 docs/               # MkDocs dokümantasyonu
└── ⚙️ .github/            # CI/CD workflows
```

## 🔧 Geliştirme

### Firmware (ESP32)

```bash
cd firmware/extruder80_156

# Derleme
pio run

# Upload (ESP32 bağlı)
pio run --target upload

# Seri monitör
pio device monitor
```

### Bridge Service

```bash
cd services/bridge

# Development mode
npm run dev

# Build
npm run build

# Test
npm test
```

### Dashboard

```bash
cd dashboard/app

# Development server
npm run dev

# Build production
npm run build
```

## 📊 Sensör Desteği

| Sensör | Model | Protokol | Kullanım |
|--------|-------|----------|----------|
| 🌡️ Sıcaklık | DS18B20 | OneWire | Barrel sıcaklığı |
| 📳 Titreşim | MPU-6050 | I2C | Vibrasyon analizi |
| ⚡ Akım | ACS712-20A | Analog | Motor akımı |
| ⚖️ Ağırlık | Load Cell + HX711 | Digital | Hammadde seviyesi |

## 🛡️ Güvenlik

- 🔐 **Pre-shared machine secrets** ile ESP32 kimlik doğrulama
- 🔥 **Firebase Security Rules** ile API koruması  
- 🔒 **HTTPS/TLS** ile şifreli iletişim
- 📝 **Audit logging** ile işlem takibi

## 📋 Sprint Planı

- [x] **Sprint 0**: Repo bootstrap, CI/CD
- [ ] **Sprint 1**: MQTT simülatörü, Firebase bridge
- [ ] **Sprint 2**: ESP32 firmware MVP
- [ ] **Sprint 3**: Sensör entegrasyonu
- [ ] **Sprint 4**: Çoklu makine desteği
- [ ] **Sprint 5**: Dokümantasyon ve sertifikasyon

## 🚀 Deployment

### Production

```bash
# Firebase deployment
cd dashboard/app
npm run build
firebase deploy

# Docker deployment
cd services/bridge
docker build -t dereliplast-bridge .
docker run -d dereliplast-bridge
```

### OTA Updates

```bash
# Firmware release
git tag v1.0.0
git push origin v1.0.0

# Otomatik build ve release GitHub Actions ile
```

## 📈 Monitoring

- 📊 **Grafana** dashboards
- 🔍 **Prometheus** metrics
- 📝 **ELK Stack** logging
- ⚠️ **AlertManager** notifications

## 🤝 Katkıda Bulunma

1. Fork yapın
2. Feature branch oluşturun (`git checkout -b feature/amazing-feature`)
3. Commit yapın (`git commit -m 'feat: add amazing feature'`)
4. Push yapın (`git push origin feature/amazing-feature`)
5. Pull Request açın

## 📄 Lisans

Bu proje MIT lisansı altında lisanslanmıştır. Detaylar için [LICENSE](LICENSE) dosyasına bakın.

## 📞 İletişim

- **Proje Sahibi**: Mehmet Eren Dereli
- **Email**: mehmet@dereliplast.com
- **GitHub**: [@mehmeterendereli](https://github.com/mehmeterendereli)

## 🙏 Teşekkürler

- [PlatformIO](https://platformio.org/) - ESP32 geliştirme ortamı
- [Firebase](https://firebase.google.com/) - Gerçek zamanlı veritabanı
- [React](https://reactjs.org/) - Dashboard framework
- [Tailwind CSS](https://tailwindcss.com/) - UI styling 