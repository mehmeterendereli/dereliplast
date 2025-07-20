# Akıllı Extruder Platformu Dokümantasyonu

Hoş geldiniz! Bu dokümantasyon, Dereliplast Akıllı Extruder Platformu'nun tüm teknik detaylarını içermektedir.

## Platform Genel Bakış

Akıllı Extruder Platformu, endüstriyel plastik extruder makinelerini IoT teknolojisiyle donatarak:

- **Kesintisiz Üretim**: Wi-Fi bağlantısı olmasa bile PLC üzerinden üretim devam eder
- **Gerçek Zamanlı İzleme**: Sensör verilerinin anlık takibi
- **Uzaktan Kontrol**: Dashboard üzerinden makine kontrolü
- **Veri Analizi**: Üretim verilerinin analizi ve raporlanması
- **Arıza Tahmini**: Sensör verilerinden otomatik alarm sistemi

## Sistem Mimarisi

```
[ESP32] ←→ [PLC] ←→ [Extruder Makinesi]
   ↓
[MQTT Broker]
   ↓  
[Bridge Service] ←→ [Firebase]
   ↓
[React Dashboard]
```

## Dokümantasyon Bölümleri

- [Donanım Kurulumu](hardware.md)
- [Yazılım Geliştirme](software.md) 
- [Deployment Rehberi](deployment.md)
- [Sorun Giderme](troubleshooting.md)

## Hızlı Başlangıç

1. [Gerekli yazılımları kurun](software.md#gereksinimler)
2. [Repo'yu klonlayın](software.md#kurulum)
3. [Simülasyonu çalıştırın](software.md#simulasyon) 