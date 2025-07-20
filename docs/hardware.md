# Donanım Kurulumu

## Gerekli Donanım Bileşenleri

### ESP32 Modülü
- **Model**: ESP32 DevKitC
- **Özellikler**: Wi-Fi, Bluetooth, 32-bit dual-core processor
- **Flash**: Minimum 4MB (LittleFS için)

### Sensörler

#### Sıcaklık Sensörü
- **Model**: DS18B20 (waterproof metal probe)
- **Adet**: 2 (yedekli ölçüm için)
- **Bağlantı**: OneWire protokolü, pull-up direnci gerekli

#### Titreşim Sensörü  
- **Model**: MPU-6050 (6-axis gyroscope + accelerometer)
- **Protokol**: I2C
- **Kullanım**: Vibrasyon analizi, arıza tespiti

#### Akım Sensörü
- **Model**: ACS712-20A
- **Ölçüm Aralığı**: ±20A
- **Çıkış**: Analog voltage (0-5V)

#### Ağırlık Sensörü
- **Model**: Load cell 20kg + HX711 amplifier
- **Protokol**: Digital
- **Kalibrasyon**: Gerekli

### Aktüatörler

#### Röle Modülü
- **Model**: 2-Channel SSR (Solid State Relay)
- **Marka**: Omron G3MB serisi
- **Kontrol**: Pump ve auger kontrolü

### Güç Kaynağı
- **Model**: Mean Well DC-DC converter
- **Giriş**: 24V DC (PLC'den)
- **Çıkış**: 5V/3A (ESP32 ve sensörler için)

### Ağ Bağlantısı
- **Model**: Teltonika RUT240 (opsiyonel)
- **Özellik**: 4G fallback, VPN desteği
- **Kullanım**: Zayıf Wi-Fi bölgeleri için

## Kablo Şeması

```
ESP32 Pin Atamaları:
- GPIO4:  DS18B20 Data (OneWire)
- GPIO21: SDA (I2C - MPU6050)
- GPIO22: SCL (I2C - MPU6050)  
- GPIO36: ACS712 Analog Out
- GPIO32: HX711 DT
- GPIO33: HX711 SCK
- GPIO25: Relay Channel 1 (Pump)
- GPIO26: Relay Channel 2 (Auger)
```

## Montaj Talimatları

### 1. ESP32 Yerleştirme
- DIN ray montaj kutusu kullanın
- EMI koruması için metal kasa tercih edin
- Soğutma için havalandırma açıklıkları bırakın

### 2. Sensör Montajı  

#### DS18B20 Sıcaklık
- Metal probe'u extruder barrel'ine monte edin
- Tork değeri: 15-20 Nm
- Termal paste kullanın

#### MPU-6050 Titreşim
- Makine gövdesine sıkıca monte edin
- Anti-vibration pad kullanmayın (titreşim ölçmek için)
- Eksenleri doğru yönlendirin

#### ACS712 Akım
- Ana güç hattına seri bağlayın
- İzolasyon için uygun klemens kullanın

### 3. Güvenlik Önlemleri
- Tüm bağlantıları çift kontrol edin
- Topraklama bağlantısını unutmayın
- 24V ve 5V bölgelerini ayırın
- Kısa devre koruması için sigorta kullanın

## Test Prosedürü

1. **Güç Testi**: Voltaj seviyelerini ölçün
2. **Sensör Testi**: Her sensörden veri alınabildiğini doğrulayın  
3. **Haberleşme Testi**: MQTT bağlantısını test edin
4. **Röle Testi**: Pump/auger kontrolünü test edin 