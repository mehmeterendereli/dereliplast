# Extruder 80/156 Firmware

ESP32 tabanlı extruder makinesi için firmware kodu.

## Özellikler

- Wi-Fi bağlantı yönetimi
- MQTT iletişimi 
- Sensör okuma (DS18B20, MPU-6050, ACS712, LoadCell)
- PLC ile Modbus haberleşmesi
- OTA güncellemeler
- Offline veri tamponu (LittleFS)

## Geliştirme

```bash
# PlatformIO CLI ile derleme
pio run

# Upload
pio run --target upload

# Monitor
pio device monitor
```

## Yapılandırma

- `platformio.ini`: Proje ayarları
- `src/config.h`: Sensör pinleri ve MQTT ayarları 