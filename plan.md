🔍 “Akıllı Extruder Platformu” – Sıfır Hata İçin Revize MASTER PLANI**
(“Hayat‑ölüm meselesi” hassasiyetiyle, uçtan uca en küçük ayrıntıya kadar tarandı.)

1. Projenin Nihai Amacı
Makine Kesintisiz Üretim: Wi‑Fi yokken bile PLC tüm süreçleri lokal yürütür.

Veri & Komut Senkronizasyonu: İnternete kavuştuğu anda tampon veriler buluta akar, dashboard anında güncellenir.

Çoklu Makine Desteği: Tek tesis → onlarca 80/156; ileride farklı çapta extruderler eklenebilir.

Arıza Tahmini & Servis: Sensör verilerinden otomatik alarm, bakım takvimi, uzaktan teşhis.

Satış Sonrası Lisans: Her makine sadece kendi token’ıyla buluta yazabilir – izinsiz çoğaltma engellenir.

2. Teknoloji & Dil Seçimleri (Nedenleriyle)
Katman	Dil	Ana Kütüphane / Araç	Seçim Nedeni
Edge MCU	C++	PlatformIO + Arduino‑ESP32 core	Deterministik loop, bol örnek, OTA kolay
PLC	Ladder / STL (mevcut)	Modbus RTU/TCP	Sahada zaten alışık
Bridge Service	Node.js 20 (TypeScript)	mqtt.js, firebase-admin	Tek dilde I/O yoğun servis, tip güvenliği
Dashboard	TypeScript	React + Vite + Tailwind + Chart.js + PWA	Bileşen tabanlı, hızlı build, offline cache
Auth / DB / Hosting	—	Firebase (Auth, RTDB, Hosting)	Ücretsiz planla 100k eşzamanlı, gerçek zamanlı
CI/CD	YAML	GitHub Actions	Ücretsiz runner, OTA binary artifakt
Dokümantasyon	Markdown	MkDocs + Material theme	Statik site, okunması zevkli

3. Bilgisayarını Kurmak İçin Yazılım Listesi
Program	Sürüm ≥	Neden
Visual Studio Code	1.90	Tek editör, çok dil eklentisi
PlatformIO Core	6.1	ESP32 derleme & seri monitor
Node.js	20.x LTS	Bridge & dashboard
Python	3.12	Simülasyon, tooling
Docker Desktop	4.x	Yerel Mosquitto + Firebase Emulators
GitHub CLI	2.x	PR, issue takibi
ESP‑IDF Flasher Tool	son	Seri porttan manuel firmware yükleme (debug)

VS Code Extensions:
PlatformIO IDE, Prettier, ESLint, Tailwind CSS Intellisense, GitHub Copilot (isteğe bağlı), Markdown Preview Enhanced.

4. Repo (“dereliplast”) Revize Dizin Yapısı
php
Kopyala
Düzenle
dereliplast/
├─ firmware/                # ESP32
│  └─ extruder80_156/
│     ├─ src/
│     ├─ include/
│     ├─ platformio.ini
│     └─ README.md
├─ services/
│  └─ bridge/               # mqtt ↔ firebase
│     ├─ src/
│     ├─ tsconfig.json
│     ├─ Dockerfile
│     └─ README.md
├─ dashboard/
│  └─ app/                  # React Vite project
│     ├─ src/
│     ├─ public/
│     └─ README.md
├─ tools/                   # simülasyon & yardımcı script
│  └─ simulator.py
├─ docs/                    # MkDocs site
│  ├─ index.md
│  ├─ hardware.md
│  ├─ software.md
│  ├─ deployment.md
│  └─ troubleshooting.md
└─ .github/
   ├─ workflows/
   │  ├─ ci.yml             # lint + test + build
   │  └─ firmware-ota.yml   # tag → binary release
   └─ ISSUE_TEMPLATE/
5. Adım Adım Yol Haritası (Jira‑gibi Sprint)
Sprint 0 – Repo Bootstrap (1 gün) ✅
✅ Klasörleri oluştur, boş README.md'leri koy.

✅ ci.yml → npm lint + platformio check.

⏳ GitHub Projects "Roadmap" panosu aç.

Sprint 1 – Sanal MVP (4 gün)
Gün	Görev
1	simulator.py ► MQTT topic plant1/extruder001/temp (QoS1).
1	Docker‑Compose: mosquitto + firebase-emulator hızlı başlat.
2	TypeScript bridge skeleton → subscribe & push RTDB.
3	React dashboard live temp tile + login (emulator).
4	“START/STOP” komut path ve simulator’ın dinlemesi.

Sprint 2 – Donanım MVP (5 gün)
PlatformIO proje: WiFiManager + AsyncMqttClient.

DS18B20 + 2‑CH röle (pump/auger).

OTA (ArduinoOTA) varsayılan port.

Seri debug & dashboard test.

CI release workflow: tag → extruder80_156.bin artifakt.

Sprint 3 – Sensör Paketi v1 (8 gün)
MPU‑6050; ACS712; loadcell.

FIFO tampon (LittleFS), yeniden gönderme.

RTDB alarm path & UI banner.

Sprint 4 – Çoklu Makine / Rol‑Tabanlı (7 gün)
RTDB schema revizyon /plants/{p}/machines/{m}

Custom token endpoint (Cloud Function) – makine üretim hattı kurulumunda QR okutarak alır.

Dashboard dropdown + role guard.

Sprint 5 – Dokümantasyon & Sertifikasyon (Sürekli)
Hardware kablo şeması, sensör montaj tork değerleri.

Firmware update prosedürü (PDF).

CE & ISO 13849 güvenlik tablosu (uzmanla).

6. Kritik Teknik Kararlar
Offline Tampon
LittleFS circular queue (48 saat) → JSON ~100 B/pkt ⇒ 100 MB flash ≈ 1 M kayıt.

Güvenlik

Pre‑shared machine secret → ESP32’de SHA‑256 + HMAC MQTT password.

Firebase rules yalnız “machineToken” claim.

Komut ACK
Komut düğümüne {cmd:"stop", ts, ack:false}. ESP32 coil success → PATCH ack:true.

Hot Swap Sensör
Sensör hatası algılanırsa kalite raporuna “sensorFault” log gider, makine çalışmayı durdurmaz ama dashboard kırmızı.

Timebase
ESP32: SNTP her 6 saat; PLC’de saat ayrı çalışır (opsiyon).

7. Risk Tablosu ve Çözümler
Risk	Önleme	Kurtarma
Wi‑Fi kritik bölgede zayıf	Endüstriyel 4G router opsiyonu	Kuyruk > 90 % → eski kayıt drop + uyarı
Flash eskimesi (write‑cycle)	15 dakika batch yaz	Wear‑leveling (LittleFS)
MQTT broker down	İkinci mqtt server (failover list)	Exponential reconnect
Firebase fiyat patlaması	Firestore switch planı	BigQuery export & cold storage
Operator komut suistimali	Role & audit log	Fiziksel “Remote Disable” anahtarı

8. Gerçek Donanım Parça Listesi (Revize)
Parça	Marka/Model	Birim $	Adet
ESP32 DevKitC	Espressif	8	1/ makine
DS18B20 (metal probe)	Maxim	3	2
MPU‑6050 board	Invensense	4	1
ACS712‑20A	Allegro	2	1
Loadcell 20 kg + HX711	Yongqiang	8	1
2‑CH relay SSR	Omron G3MB	4	1
24 V‑5 V DC‑DC	MeanWell	6	1
Endüstriyel Wi‑Fi/4G router	Teltonika RUT240	120	1/tesis

9. Nihai Başarı Ölçütleri (Exit Criteria)
Makine çevrimdışı 72 saat çalışır ⇒ veri kaybı 0.

Dashboard ≤ 3 s gecikmeyle sıcaklık gösterir.

START/STOP komutu 2 s içinde PLC coil tetikler.

Titreşim alarmı dashboard + e‑mail/telegram bildirimi.

İkinci makine < 15 dk içinde sisteme dâhil edilebilir (QR token).

10. Şimdi Ne Yapıyoruz?
Sprint 0 → Repo klasörleri, CI iskeleti, README

bash
Kopyala
Düzenle
git clone https://github.com/mehmeterendereli/dereliplast
cd dereliplast && mkdir -p firmware/extruder80_156/src services/bridge/src dashboard/app docs
git add . && git commit -m "feat: repo skeleton" && git push origin main
Issue #1: “Repo Bootstrap” – ben sana ayrıntılı komut seti & .github/workflows/ci.yml örneğini göndereyim.

Sonra Sprint 1 simülasyona geçeceğiz.

