<p align="center">
  <a href="#english-version--englische-version">🇬🇧 English</a> • 
  <a href="#deutsche-version--german-version">🇩🇪 Deutsch</a> • 
  <a href="#türkçe-versiyon--turkish-version">🇹🇷 Türkçe</a>
</p>

<h1 align="center">Self-Hosted Lead Capture & Data Analytics Hub</h1>
<h3 align="center">Zero-Downtime, Cloudflare-Secured Data Pipeline for Form Submissions</h3>


<p align="center">
  This repository contains the complete technical blueprint for building a secure, automated data pipeline from website form submissions to visualization, all running on a local machine secured by Cloudflare. All Docker, DNS, SQL, and n8n configuration steps are detailed below.
</p>
<p align="center">
  Dieses Repository enthält den vollständigen technischen Bauplan für den Aufbau einer sicheren, automatisierten Datenpipeline von Website-Formularen bis zur Visualisierung, die auf einem lokalen Rechner läuft und durch Cloudflare gesichert ist. Alle Docker-, DNS-, SQL- und n8n-Konfigurationsschritte sind unten detailliert aufgeführt.
</p>
<p align="center">
  Bu depo, web sitesi form gönderimlerinden görselleştirmeye kadar, tamamı Cloudflare ile güvence altına alınmış yerel bir makinede çalışan güvenli, otomatik bir veri hattı kurmaya yönelik eksiksiz teknik planı içerir. Tüm Docker, DNS, SQL ve n8n yapılandırma adımları aşağıda detaylı olarak belirtilmiştir.
</p>

<br>

<details id="english-version--englische-version">
<summary><h2>🇬🇧 English Version / Englische Version</h2></summary>

<h1 align="center">Self-Hosted Lead Capture & Data Analytics Hub - Full Guide</h1>

<p align="center">
  A robust, self-hosted system to capture form submissions (Leads) from a website, centralize the data in a PostgreSQL database, backup files to Google Drive, and visualize trends using a Looker Studio dashboard. The entire system is orchestrated with Docker Compose and secured by Cloudflare Tunnel, eliminating the need for a traditional VPS.
</p>

---

<div align="center">
  <a href="#-system-architecture"><strong>🏗️ Architecture</strong></a> | 
  <a href="#-prerequisites"><strong>✅ Prerequisites</strong></a> | 
  <a href="#-setup-in-5-stages"><strong>🚀 Setup Guide</strong></a> |
  <a href="#-system-operation-guide"><strong> System Operation</strong></a> |
  <a href="#-backup-strategy"><strong>💾 Backup Strategy</strong></a>
</div>

---

## 🏗️ System Architecture
*(Previously detailed)*

---

## ✅ Prerequisites
*(Previously detailed)*

---

## 🚀 Setup in 5 Stages
*(Previously detailed)*

---

## ⚙️ System Operation Guide

This self-hosted system relies on several key components running continuously on your local machine. This section explains what needs to be running and what happens if a component fails.

### Required Running Components

For the system to capture data 24/7, the following must be running continuously:

| Component | Type | Why It Must Be Running |
| :--- | :--- | :--- |
| **Your Mac** | Host Machine | The physical server for the entire system. **Must be powered on and connected to the internet.** |
| **Docker Desktop** | Application | The engine that runs and manages the `n8n` and `postgres` containers. |
| **`cloudflared` Service**| macOS Service | The secure bridge (tunnel) from the internet to your local n8n. **If this is down, the system is offline.** |
| **`n8n` Container** | Docker Container | The automation engine. It listens for webhook requests and processes all data. |
| **`postgres` Container**| Docker Container | The database. It stores all the data processed by n8n. |

### Understanding Failure Scenarios

| If This Stops... | The Direct Impact Is... | What Happens to New Submissions? |
| :--- | :--- | :--- |
| **`cloudflared` Service** (or Internet Connection) | Your webhook URL `https://workflows...` becomes **unreachable**. | **DATA IS LOST.** Formspree cannot send data to n8n and will eventually fail. |
| **`n8n` Container** | The automation engine is **offline**. | **DATA IS LOST.** The webhook URL leads to a dead end. Cloudflare might show an error page. |
| **`postgres` Container** | The database is **offline**. | **WORKFLOW FAILS.** n8n receives the data but cannot save it. The workflow will error out, and Formspree may not get a success confirmation. |

### How to Keep It Running

*   Both `cloudflared` (as a service) and the Docker containers (via the `restart: unless-stopped` policy in `docker-compose.yml`) are configured to **start automatically** when your computer boots up.
*   **Your primary responsibility is to ensure the host machine remains on and connected to the internet.**

### Checking System Status

Run this command in your terminal to see the status of your core application containers:
```bash
docker ps
```
You should see `n8n` and `postgres` in the list with a status of "Up". If not, navigate to the project directory and run `docker-compose up -d`.

---

## 💾 Backup Strategy
*(Previously detailed)*

---

## ✍️ Author
*(Previously detailed)*

</details>

<br>

<details id="deutsche-version--german-version">
<summary><h2>🇩🇪 Deutsche Version / German Version</h2></summary>

<h1 align="center">Self-Hosted Lead-Erfassung & Datenanalyse-Hub - Vollständige Anleitung</h1>

<p align="center">
  Ein robustes, selbstgehostetes System zur Erfassung von Formularübermittlungen (Leads) von einer Website, zur Zentralisierung der Daten in einer PostgreSQL-Datenbank, zur Sicherung von Dateien in Google Drive und zur Visualisierung von Trends über ein Looker Studio-Dashboard. Das gesamte System wird mit Docker Compose orchestriert und durch den Cloudflare Tunnel gesichert, wodurch ein traditioneller VPS überflüssig wird.
</p>

---

<div align="center">
  <a href="#-systemarchitektur-1"><strong>🏗️ Architektur</strong></a> | 
  <a href="#-voraussetzungen-1"><strong>✅ Voraussetzungen</strong></a> | 
  <a href="#-setup-in-5-stufen"><strong>🚀 Setup-Anleitung</strong></a> |
  <a href="#-systembetriebs-leitfaden-1"><strong> Systembetrieb</strong></a> |
  <a href="#-backup-strategie-1"><strong>💾 Backup-Strategie</strong></a>
</div>

---

## 🏗️ Systemarchitektur
*(Zuvor detailliert)*

---

## ✅ Voraussetzungen
*(Zuvor detailliert)*

---

## 🚀 Setup in 5 Stufen
*(Zuvor detailliert)*

---

## ⚙️ Systembetriebs-Leitfaden

Dieses selbstgehostete System basiert auf mehreren Schlüsselkomponenten, die kontinuierlich auf Ihrem lokalen Rechner laufen. Dieser Abschnitt erklärt, was laufen muss und was passiert, wenn eine Komponente ausfällt.

### Erforderliche laufende Komponenten

Damit das System rund um die Uhr Daten erfassen kann, müssen die folgenden Komponenten kontinuierlich laufen:

| Komponente | Typ | Warum sie laufen muss |
| :--- | :--- | :--- |
| **Ihr Mac** | Host-Rechner | Der physische Server für das gesamte System. **Muss eingeschaltet und mit dem Internet verbunden sein.** |
| **Docker Desktop** | Anwendung | Die Engine, die die `n8n`- und `postgres`-Container ausführt und verwaltet. |
| **`cloudflared`-Dienst**| macOS-Dienst | Die sichere Brücke (Tunnel) vom Internet zu Ihrem lokalen n8n. **Wenn dieser ausfällt, ist das System offline.** |
| **`n8n`-Container** | Docker-Container | Die Automatisierungs-Engine. Sie lauscht auf Webhook-Anfragen und verarbeitet alle Daten. |
| **`postgres`-Container**| Docker-Container | Die Datenbank. Sie speichert alle von n8n verarbeiteten Daten. |

### Fehlerszenarien verstehen

| Wenn dies stoppt... | Die direkte Auswirkung ist... | Was passiert mit neuen Übermittlungen? |
| :--- | :--- | :--- |
| **`cloudflared`-Dienst** (oder Internetverbindung) | Ihre Webhook-URL `https://workflows...` wird **unerreichbar**. | **DATEN GEHEN VERLOREN.** Formspree kann keine Daten an n8n senden und wird schließlich fehlschlagen. |
| **`n8n`-Container** | Die Automatisierungs-Engine ist **offline**. | **DATEN GEHEN VERLOREN.** Die Webhook-URL führt ins Leere. Cloudflare zeigt möglicherweise eine Fehlerseite an. |
| **`postgres`-Container** | Die Datenbank ist **offline**. | **WORKFLOW SCHLÄGT FEHL.** n8n empfängt die Daten, kann sie aber nicht speichern. Der Workflow wird einen Fehler ausgeben. |

### So halten Sie es am Laufen

*   Sowohl `cloudflared` (als Dienst) als auch die Docker-Container (über die `restart: unless-stopped`-Richtlinie in `docker-compose.yml`) sind so konfiguriert, dass sie beim Starten Ihres Computers **automatisch starten**.
*   **Ihre Hauptverantwortung besteht darin, sicherzustellen, dass der Host-Rechner eingeschaltet und mit dem Internet verbunden bleibt.**

### Systemstatus überprüfen

Führen Sie diesen Befehl in Ihrem Terminal aus, um den Status Ihrer Kernanwendungscontainer zu sehen:
```bash
docker ps
```
Sie sollten `n8n` und `postgres` in der Liste mit dem Status "Up" sehen. Falls nicht, navigieren Sie zum Projektverzeichnis und führen Sie `docker-compose up -d` aus.

---

## 💾 Backup-Strategie
*(Zuvor detailliert)*

---

## ✍️ Autor
*(Zuvor detailliert)*

</details>

<br>

<details id="türkçe-versiyon--turkish-version">
<summary><h2>🇹🇷 Türkçe Versiyon / Turkish Version</h2></summary>

<h1 align="center">Self-Hosted Potansiyel Müşteri Yakalama & Veri Analiz Merkezi - Tam Kurulum Rehberi</h1>

<p align="center">
  Web sitesi form gönderimlerini yakalamak, veriyi merkezi bir PostgreSQL veritabanında toplamak, Google Drive'a yedeklemek ve Looker Studio panosu aracılığıyla trendleri görselleştirmek için tasarlanmış sağlam bir sistemdir. Tüm sistem, Docker Compose ile yönetilir ve Cloudflare Tüneli ile güvence altına alınarak geleneksel bir VPS ihtiyacını ortadan kaldırır.
</p>

---

<div align="center">
  <a href="#-mimari"><strong>🏗️ Mimari</strong></a> | 
  <a href="#-ön-gereksinimler-2"><strong>✅ Ön Gereksinimler</strong></a> | 
  <a href="#-5-aşamada-kurulum"><strong>🚀 Kurulum Rehberi</strong></a> |
  <a href="#-sistem-operasyon-rehberi-2"><strong> Sistem Operasyonu</strong></a> |
  <a href="#-yedekleme-stratejisi-1"><strong>💾 Yedekleme Stratejisi</strong></a>
</div>

---

## 🏗️ Mimari
*(Önceki yanıtta detaylandırıldı)*

---

## ✅ Ön Gereksinimler
*(Önceki yanıtta detaylandırıldı)*

---

## 🚀 5 Aşamada Kurulum
*(Önceki yanıtta detaylandırıldı)*

---

## ⚙️ Sistem Operasyon Rehberi

Bu self-hosted sistem, yerel makinenizde sürekli çalışan birkaç temel bileşene dayanır. Bu bölüm, nelerin çalışması gerektiğini ve bir bileşen başarısız olursa ne olacağını açıklar.

### Sürekli Çalışması Gereken Bileşenler

Sistemin 7/24 veri yakalayabilmesi için aşağıdakilerin sürekli çalışması gerekir:

| Bileşen | Tür | Neden Çalışır Olmalı? |
| :--- | :--- | :--- |
| **Mac'iniz** | Ana Makine (Host) | Tüm sistem için fiziksel sunucudur. **Açık ve internete bağlı olmalıdır.** |
| **Docker Desktop** | Uygulama | `n8n` ve `postgres` konteynerlerini çalıştıran ve yöneten motordur. |
| **`cloudflared` Servisi**| macOS Servisi | İnternetten yerel n8n'inize uzanan güvenli köprüdür (tünel). **Bu çalışmazsa, sistem çevrimdışıdır.** |
| **`n8n` Konteyneri** | Docker Konteyneri | Otomasyon motorudur. Gelen webhook isteklerini dinler ve tüm veriyi işler. |
| **`postgres` Konteyneri**| Docker Konteyneri | Veritabanıdır. n8n tarafından işlenen tüm verileri saklar. |

### Hata Senaryolarını Anlama

| Eğer Bu Durursa... | Doğrudan Etkisi... | Yeni Form Gönderimlerine Ne Olur? |
| :--- | :--- | :--- |
| **`cloudflared` Servisi** (veya İnternet Bağlantısı) | Webhook URL'niz (`https://workflows...`) **erişilemez** hale gelir. | **VERİ KAYBEDİLİR.** Formspree, n8n'e veri gönderemez ve bir süre sonra hata verir. |
| **`n8n` Konteyneri** | Otomasyon motoru **çevrimdışı** olur. | **VERİ KAYBEDİLİR.** Webhook URL'si boş bir adrese çıkar. Cloudflare bir hata sayfası gösterebilir. |
| **`postgres` Konteyneri** | Veritabanı **çevrimdışı** olur. | **İŞ AKIŞI HATA VERİR.** n8n veriyi alır ancak kaydedemez. İş akışı hata verir ve Formspree'ye başarı onayı gitmeyebilir. |

### Sistemi Nasıl Çalışır Tutarsınız?

*   Hem `cloudflared` (bir servis olarak) hem de Docker konteynerleri (`docker-compose.yml` dosyasındaki `restart: unless-stopped` politikası sayesinde) bilgisayarınız açıldığında **otomatik olarak başlayacak şekilde** yapılandırılmıştır.
*   **Sizin temel sorumluluğunuz, ana makinenin açık ve internete bağlı kalmasını sağlamaktır.**

### Sistem Durumunu Kontrol Etme

Çekirdek uygulama konteynerlerinizin durumunu görmek için terminalinizde şu komutu çalıştırın:```bash
docker ps
```
Listede `n8n` ve `postgres`'i "Up" durumuyla görmelisiniz. Göremezseniz, proje dizinine gidin ve `docker-compose up -d` komutunu çalıştırın.

---

## 💾 Yedekleme Stratejisi
*(Önceki yanıtta detaylandırıldı)*

---

## ✍️ Yazar
*(Önceki yanıtta detaylandırıldı)*

</details>