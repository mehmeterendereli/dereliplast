# Sorun Giderme

## Yaygın Problemler ve Çözümleri

### ESP32 Bağlantı Problemleri

#### 1. Wi-Fi Bağlantısı Kurulamıyor

**Belirtiler:**
- Seri monitörde "WiFi connecting..." mesajı sürekli çıkıyor
- Dashboard'da makine offline görünüyor

**Çözüm:**
```bash
# 1. WiFi credentials kontrolü
pio device monitor --baud 115200

# 2. Ağ bilgilerini sıfırla
# Reset butonuna basılı tutup 10 saniye bekleyin

# 3. WiFiManager ile manuel konfigürasyon
# ESP32'ye bağlanıp http://192.168.4.1 adresine gidin
```

**Debug Komutları:**
```cpp
// WiFi debug için serial output
Serial.println("SSID: " + WiFi.SSID());
Serial.println("Signal Strength: " + String(WiFi.RSSI()));
Serial.println("Gateway: " + WiFi.gatewayIP().toString());
```

#### 2. MQTT Bağlantısı Başarısız

**Belirtiler:**
- "MQTT connection failed" hata mesajı
- Sensör verileri dashboard'a gelmiyor

**Çözüm:**
```bash
# 1. MQTT broker status kontrolü
docker ps | grep mosquitto

# 2. MQTT broker restart
docker restart mosquitto

# 3. Credentials kontrolü
mosquitto_pub -h localhost -t test/topic -m "test message"
```

**Debug:**
```cpp
// MQTT debug
if (!mqttClient.connected()) {
    Serial.println("MQTT disconnected, reason: " + String(mqttClient.state()));
}
```

### Bridge Service Problemleri

#### 1. Firebase Bağlantı Hatası

**Belirtiler:**
- Console'da "Firebase authentication failed"
- Bridge service crash oluyor

**Çözüm:**
```bash
# 1. Service account key kontrolü
cat services/bridge/.env | grep FIREBASE

# 2. Firebase emulator kontrolü  
firebase emulators:start --only database

# 3. Service restart
cd services/bridge && npm run dev
```

#### 2. Memory Leak

**Belirtiler:**
- Servis yavaş çalışıyor
- Node.js process'i çok memory kullanıyor

**Debug:**
```bash
# Memory usage monitoring
node --inspect services/bridge/dist/index.js

# Chrome DevTools'da heap snapshot alın
# chrome://inspect adresine gidin
```

### Dashboard Problemleri

#### 1. Real-time Updates Gelmiyor

**Belirtiler:**
- Dashboard veriler gösteriyor ama update olmuyor
- Chart'lar donuk kalıyor

**Çözüm:**
```bash
# 1. Firebase connection kontrolü
# Browser DevTools Console:
firebase.database().ref('.info/connected').on('value', (snap) => {
    console.log('Connected:', snap.val());
});

# 2. Hard refresh
# Ctrl+Shift+R (Chrome)
# Cache temizle

# 3. Service worker problemleri
# Application tab > Storage > Clear storage
```

#### 2. Authentication Problemleri

**Belirtiler:**
- Login yapılamıyor
- "Invalid token" hatası

**Çözüm:**
```bash
# 1. Token expiry kontrolü
# Browser DevTools > Application > Local Storage

# 2. Firebase auth rules kontrolü
firebase database:get /users --project your-project-id

# 3. Custom token generation test
node tools/generate-test-token.js
```

### Sensör Problemleri

#### 1. DS18B20 Sıcaklık Okunamıyor

**Belirtiler:**
- Temperature: -999 değeri
- "Temperature sensor error" log'u

**Çözüm:**
```cpp
// Hardware kontrolü
void debugDS18B20() {
    if (!ds18b20.begin()) {
        Serial.println("DS18B20 not found");
        return;
    }
    
    int deviceCount = ds18b20.getDeviceCount();
    Serial.println("Found " + String(deviceCount) + " devices");
}
```

**Donanım Kontrolü:**
- Pull-up direnci (4.7kΩ) kontrol edin
- Kablo bağlantılarını kontrol edin
- Multimetre ile sensör pinlerini ölçün

#### 2. MPU-6050 I2C Hatası

**Belirtiler:**
- Accelerometer verisi 0,0,0
- I2C timeout hataları

**Debug:**
```cpp
// I2C scanner
void scanI2C() {
    for (byte i = 8; i < 120; i++) {
        Wire.beginTransmission(i);
        if (Wire.endTransmission() == 0) {
            Serial.println("I2C device found at: 0x" + String(i, HEX));
        }
    }
}
```

### Performance Problemleri

#### 1. Yavaş Response Time

**Belirtiler:**
- Dashboard 5+ saniye gecikmeli
- MQTT mesajları kaybolıyor

**Optimizasyon:**
```javascript
// MQTT QoS ayarları
const mqttOptions = {
    qos: 1,  // En az bir kez iletim garantisi
    retain: false,  // Retain flag'i kapatın
    dup: false
};

// Firebase batching
const batch = database.ref().update({
    [`machines/${machineId}/temp`]: temperature,
    [`machines/${machineId}/pressure`]: pressure,
    [`machines/${machineId}/timestamp`]: Date.now()
});
```

#### 2. ESP32 Memory Overflow

**Belirtiler:**
- Rastgele reboot'lar
- Watchdog timer reset'leri

**Debug:**
```cpp
void monitorHeap() {
    size_t freeHeap = ESP.getFreeHeap();
    size_t minFreeHeap = ESP.getMinFreeHeap();
    
    Serial.printf("Free heap: %d, Min free: %d\n", freeHeap, minFreeHeap);
    
    if (freeHeap < 10000) {
        Serial.println("WARNING: Low memory!");
    }
}
```

## Log Analizi

### ESP32 Serial Logs

```bash
# PlatformIO serial monitor
pio device monitor --baud 115200 --filter esp32_exception_decoder

# Log dosyasına kaydet
pio device monitor > esp32.log 2>&1
```

### Bridge Service Logs

```bash
# PM2 ile log takibi
pm2 logs bridge-service

# JSON formatında log
npm run logs | jq '.level == "error"'
```

### Dashboard Browser Logs

```javascript
// Performance metrics
window.addEventListener('load', function() {
    const perfData = performance.getEntriesByType('navigation')[0];
    console.log('Page load time:', perfData.loadEventEnd - perfData.fetchStart);
});

// Error tracking
window.addEventListener('error', function(e) {
    console.error('Global error:', e.error);
});
```

## Network Diagnostics

### MQTT Connection Test

```bash
# MQTT subscriber test
mosquitto_sub -h localhost -t "plant1/+/+" -v

# MQTT publisher test
mosquitto_pub -h localhost -t "plant1/machine001/temp" -m '{"value": 25.5}'

# SSL/TLS test
mosquitto_pub -h production-broker.com -p 8883 --cafile ca.crt -t test -m "test"
```

### Firebase Connection Test

```bash
# REST API test
curl -X GET \
  'https://dereliplast-default-rtdb.firebaseio.com/machines.json?auth=YOUR_TOKEN'

# WebSocket test (browser console)
const ws = new WebSocket('wss://s-usc1c-nss-205.firebaseio.com/.ws?v=5');
ws.onopen = () => console.log('Connected');
ws.onerror = (e) => console.error('Error:', e);
```

## Recovery Procedures

### ESP32 Factory Reset

```cpp
// SPIFFS/LittleFS formatla
#include "LittleFS.h"

void factoryReset() {
    LittleFS.format();
    ESP.eraseConfig();  // WiFi credentials sil
    ESP.restart();
}
```

### Database Recovery

```bash
# Firebase backup'tan restore
firebase firestore:import gs://dereliplast-backups/20240315/

# Specific collection restore
firebase firestore:import gs://backup-location/ --collection-ids machines,plants
```

### Service Recovery

```bash
# Docker containers restart
docker-compose down && docker-compose up -d

# Service dependency check
systemctl status mosquitto
systemctl status docker
systemctl status networking
```

## Monitoring Setup

### Health Check Scripts

```bash
#!/bin/bash
# health_check.sh

# MQTT broker check
if ! mosquitto_pub -h localhost -t health/check -m ping -q; then
    echo "MQTT broker DOWN" | mail -s "Alert" admin@dereliplast.com
fi

# Bridge service check
if ! curl -f http://localhost:3000/health; then
    echo "Bridge service DOWN" | mail -s "Alert" admin@dereliplast.com
fi

# ESP32 devices check
for ip in 192.168.1.100 192.168.1.101; do
    if ! ping -c 1 $ip; then
        echo "ESP32 $ip DOWN" | mail -s "Alert" admin@dereliplast.com
    fi
done
```

### Automated Diagnostics

```python
# tools/auto_diagnose.py
import json
import requests
import subprocess

def diagnose_system():
    report = {
        "timestamp": datetime.now().isoformat(),
        "issues": []
    }
    
    # ESP32 connectivity
    for device in devices:
        if not ping_device(device['ip']):
            report['issues'].append(f"ESP32 {device['id']} unreachable")
    
    # Firebase status
    try:
        firebase_status = requests.get(firebase_url + '/.info/connected.json')
        if not firebase_status.json():
            report['issues'].append("Firebase disconnected")
    except:
        report['issues'].append("Firebase API error")
    
    return report
``` 