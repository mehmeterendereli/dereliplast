# Yazılım Geliştirme

## Gereksinimler

### Gerekli Yazılımlar

| Program | Minimum Sürüm | Kullanım Amacı |
|---------|----------------|-----------------|
| Visual Studio Code | 1.90+ | Ana geliştirme editörü |
| PlatformIO Core | 6.1+ | ESP32 firmware derleme |
| Node.js | 20.x LTS | Bridge service ve dashboard |
| Python | 3.12+ | Simülasyon ve tooling |
| Docker Desktop | 4.x | Mosquitto ve Firebase emulators |
| GitHub CLI | 2.x+ | PR ve issue yönetimi |

### VS Code Extensions

Gerekli eklentiler:
- PlatformIO IDE
- Prettier - Code formatter  
- ESLint
- Tailwind CSS IntelliSense
- GitHub Copilot (opsiyonel)
- Markdown Preview Enhanced

## Kurulum

### 1. Repo Klonlama

```bash
git clone https://github.com/mehmeterendereli/dereliplast.git
cd dereliplast
```

### 2. Geliştirme Ortamını Hazırlama

```bash
# Node.js bağımlılıklarını yükle
cd services/bridge && npm install
cd ../../dashboard/app && npm install
cd ../..

# Python sanal ortam
python -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

### 3. Docker Services

```bash
# Mosquitto MQTT Broker
docker run -d --name mosquitto -p 1883:1883 eclipse-mosquitto

# Firebase Emulators
cd services/bridge
npm run firebase:emulators
```

## Geliştirme Workflow'u

### Firmware (ESP32)

```bash
# PlatformIO ile derleme
cd firmware/extruder80_156
pio run

# Upload (ESP32 bağlı)
pio run --target upload

# Serial monitor
pio device monitor --baud 115200
```

### Bridge Service

```bash
cd services/bridge

# Development mode
npm run dev

# Type checking
npm run type-check

# Build
npm run build
```

### Dashboard

```bash
cd dashboard/app

# Development server
npm run dev

# Build production
npm run build

# Preview build
npm run preview
```

## Simülasyon

ESP32 donanımı olmadan test etmek için simülatör kullanın:

```bash
# MQTT simülatörü başlat
python tools/simulator.py

# Farklı senaryolar
python tools/simulator.py --scenario high_temp
python tools/simulator.py --scenario vibration_alarm
```

## Test Stratejisi

### Unit Tests

```bash
# Bridge service tests
cd services/bridge
npm test

# Dashboard tests  
cd dashboard/app
npm test
```

### Integration Tests

```bash
# End-to-end test
npm run test:e2e
```

### Hardware-in-the-Loop (HIL)

```bash
# Gerçek ESP32 ile test
pio test --environment esp32dev
```

## Kod Standartları

### TypeScript/JavaScript

- ESLint + Prettier kullanımı zorunlu
- Strict TypeScript mode
- Function naming: camelCase
- Constants: UPPER_SNAKE_CASE

### C++ (ESP32)

- PlatformIO linting kuralları
- Header guards: `#pragma once`
- Variable naming: snake_case
- Class naming: PascalCase

### Commit Mesajları

Conventional Commits formatı:

```
feat: yeni özellik ekleme
fix: hata düzeltme  
docs: dokümantasyon güncelleme
style: kod formatı düzenleme
refactor: kod refactoring
test: test ekleme/düzenleme
chore: build/config değişiklikleri
```

## Debugging

### ESP32 Debug

```bash
# Serial output
pio device monitor

# GDB debugging (JTAG gerekli)
pio debug
```

### Bridge Service Debug

```bash
# Node.js inspector
npm run debug

# VS Code ile attach
# F5 tuşu ile debug configuration başlat
```

### Dashboard Debug

```bash
# Browser dev tools
npm run dev

# Redux DevTools Extension kullanın
```

## Performance Monitoring

### Metrics

- ESP32: Heap usage, WiFi RSSI, loop timing
- Bridge: Memory usage, MQTT queue depth  
- Dashboard: Bundle size, render time

### Profiling

```bash
# Bundle analyzer
cd dashboard/app
npm run analyze

# Memory profiling
npm run profile
``` 