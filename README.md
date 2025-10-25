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

| Layer | Tool | Container Name (Docker) | Accessibility | Data Storage |
| :--- | :--- | :--- | :--- | :--- |
| **Ingestion/Trigger** | **Formspree** | N/A | Internet (POST) | External |
| **Interface/Security** | **Cloudflare Tunnel** | `cloudflared` (macOS Service) | `https://workflows.yourdomain.com` | N/A |
| **Automation (Engine)**| **n8n** | `n8n` | `http://localhost:5678` (Internal) | PostgreSQL (`postgres` DB) |
| **Data Store** | **PostgreSQL** | `postgres` | `localhost:5432` (Internal) | Persistent Docker Volume (`pgdata`) |
| **Reporting** | **Looker Studio** | N/A | `data-source: Google Sheets` | External (Google Cloud) |

---

## ✅ Prerequisites

1.  **Hardware:** A local machine (e.g., MacBook) that can run 24/7.
2.  **Software:** **Docker Desktop** installed and running.
3.  **Services:**
    *   A registered domain (e.g., at **Namecheap**).
    *   A **Cloudflare** Account (Free Plan).
    *   A **Google Cloud Platform** Account with a Service Account created.
    *   A **Formspree** Account.

> 🚀 **Ready to deploy?** Follow the detailed **[SETUP.md](SETUP.md)** file for a guided, step-by-step installation.

---

## 🚀 Setup in 5 Stages

This setup is designed to be completed sequentially.

### Stage 1: Local Infrastructure with Docker Compose

This stage creates the core services (Database and Automation Engine) on your machine.

1.  **Clone the Repository:**
    ```bash
    git clone [YOUR_REPO_URL] lead-capture-hub
    cd lead-capture-hub
    ```
2.  **Create and Configure the Environment File:**
    ```bash
    cp .env.example .env
    ```
    Open the `.env` file and set a strong `POSTGRES_PASSWORD` and the correct `N8N_HOST` (e.g., `workflows.yourdomain.com`).

3.  **Launch the System:**
    ```bash
    docker-compose up -d
    ```
    This command will download the images and start the `postgres` and `n8n` containers. Verify they are running with `docker ps`.

### Stage 2: Database Initialization

Connect to your newly created PostgreSQL database and create the necessary table structure.

1.  **Connect to the Database:** Use a tool like DBeaver, TablePlus, or PgAdmin.
    *   **Host:** `localhost`
    *   **Port:** `5432`
    *   **Database:** `postgres`
    *   **User:** `postgres`
    *   **Password:** The `POSTGRES_PASSWORD` from your `.env` file.
2.  **Execute the Schema SQL:**
    Open the `sql-schema/schema.sql` file, copy its entire content, paste it into your database tool's query editor, and run it. This will create the `contacts` table.

### Stage 3: Secure Public Access with Cloudflare

This stage exposes your local n8n instance to the internet securely via HTTPS.

1.  **Delegate DNS to Cloudflare:** In your domain registrar (Namecheap), change the nameservers to the ones provided by Cloudflare.
2.  **Create and Configure Cloudflare Tunnel:**
    *   In the Cloudflare Zero Trust dashboard, navigate to `Access -> Tunnels`.
    *   Create a new tunnel, choose `cloudflared` as the connector, and give it a name.
    *   Follow the on-screen instructions to install `cloudflared` on your machine (e.g., `brew install cloudflared`) and run the command to link it to your account (`sudo cloudflared service install ...`).
3.  **Route Traffic to n8n:**
    *   In the tunnel's "Public Hostnames" tab, add a new route.
    *   **Subdomain/Domain:** `workflows.yourdomain.com`
    *   **Service Type:** `HTTP`
    *   **Service URL:** `localhost:5678`
    *   Save the route. Your n8n instance is now live at the specified URL.

### Stage 4: n8n Configuration (Workflows & Credentials)

Now, configure n8n to connect to your services and automate the data flow.

1.  **Access n8n:** Navigate to `https://workflows.yourdomain.com` (or `http://localhost:5678` for initial setup) and create your admin account.
2.  **Create Credentials:**
    *   Go to the "Credentials" section and create two new credentials:
        1.  **PostgreSQL:** Use the connection details from Stage 2.
        2.  **Google Service Account:** Paste the `client_email` and `private_key` from your GCP JSON file.
3.  **Import Workflows:**
    *   Go to `File -> Import from file...` and import `1_formspree_pipeline.json` and `2_weekly_backup.json` from the `n8n-workflows` directory.
4.  **Connect Credentials and Activate:**
    *   Open each imported workflow.
    *   For each node (PostgreSQL, Google Sheets, Google Drive), select the credential you just created from the dropdown menu.
    *   Fill in any required IDs (like your Google Sheet ID or Drive Folder ID).
    *   Activate each workflow using the toggle in the top-right.

### Stage 5: Final Integrations (Formspree & Looker Studio)

1.  **Connect Formspree to n8n:**
    *   In your Formspree form's "Integrations" section, add a new Webhook.
    *   Paste the n8n production webhook URL: `https://workflows.yourdomain.com/webhook/formspree-webhook`
2.  **Connect Looker Studio to Google Sheets:**
    *   In Looker Studio, create a new report and add a new data source.
    *   Select the "Google Sheets" connector.
    *   Choose the spreadsheet and worksheet (`FormSubmissions`) where n8n is appending rows.
    *   Build your dashboard visualizations.

---

## ⚙️ System Operation Guide

*   **Continuous Operation:** The system runs as long as your machine is on and Docker Desktop is running. The `cloudflared` service and Docker containers are configured to restart automatically.
*   **Checking Status:** Use `docker ps` to check the status of the `n8n` and `postgres` containers.
*   **Restarting:** If a service is unresponsive, use `docker-compose restart` in the project directory to restart both containers.

---

## 💾 Backup Strategy

This project employs a multi-layered backup strategy:

1.  **Real-time Raw Backup:** Every form submission is instantly saved as a raw `.json` file to Google Drive by the primary n8n workflow.
2.  **Weekly Structured Backup:** A secondary n8n workflow, triggered by a `cron` job, exports the entire `contacts` table from PostgreSQL as a `.csv` file and saves it to Google Drive.
3.  **Manual Full Snapshot:** For disaster recovery, you can create a full `.sql` backup of the database at any time:
    ```bash
    docker-compose exec -T postgres pg_dumpall -U postgres > backup_$(date +%Y-%m-%d).sql
    ```

---

## ✍️ Author
This system was designed, built, and documented by **[Your Name Here]**.
*   **GitHub:** [ridvanyigit](https://github.com/ridvanyigit)
*   **LinkedIn:** [Ridvan Yigit](https://linkedin.com/in/profiliniz)
*   **Website:** [www.ridvanyigit.com](https://www.ridvanyigit.com/)

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

| Schicht | Tool | Container-Name (Docker) | Erreichbarkeit | Datenspeicherung |
| :--- | :--- | :--- | :--- | :--- |
| **Erfassung/Auslöser** | **Formspree** | N/A | Internet (POST) | Extern |
| **Schnittstelle/Sicherheit** | **Cloudflare Tunnel** | `cloudflared` (macOS-Dienst) | `https://workflows.deinedomain.com` | N/A |
| **Automatisierung**| **n8n** | `n8n` | `http://localhost:5678` (Intern) | PostgreSQL (`postgres` DB) |
| **Datenspeicher** | **PostgreSQL** | `postgres` | `localhost:5432` (Intern) | Persistentes Docker Volume (`pgdata`) |
| **Reporting** | **Looker Studio** | N/A | `Datenquelle: Google Sheets` | Extern (Google Cloud) |

---

## ✅ Voraussetzungen

1.  **Hardware:** Ein lokaler Rechner (z.B. MacBook), der rund um die Uhr laufen kann.
2.  **Software:** **Docker Desktop** installiert und aktiv.
3.  **Dienste:**
    *   Eine registrierte Domain (z.B. bei **Namecheap**).
    *   Ein **Cloudflare**-Konto (Kostenloser Plan).
    *   Ein **Google Cloud Platform**-Konto mit einem erstellten Service Account.
    *   Ein **Formspree**-Konto.

> 🚀 **Bereit zur Bereitstellung?** Folge der detaillierten **[SETUP.md](SETUP.md)**-Datei für eine schrittweise geführte Installation.

---

## 🚀 Setup in 5 Stufen

### Stufe 1: Lokale Infrastruktur mit Docker Compose

1.  **Repository klonen:**
    ```bash
    git clone [DEINE_REPO_URL] lead-capture-hub && cd lead-capture-hub
    ```
2.  **Umgebungsdatei erstellen und konfigurieren:**
    ```bash
    cp .env.example .env
    ```
    Öffne die `.env`-Datei und setze ein starkes `POSTGRES_PASSWORD` und den korrekten `N8N_HOST`.

3.  **System starten:**
    ```bash
    docker-compose up -d
    ```

### Stufe 2: Datenbankinitialisierung

1.  **Mit der Datenbank verbinden:** Verwende ein Tool wie DBeaver.
    *   **Host:** `localhost`, **Port:** `5432`, **Datenbank/Benutzer:** `postgres`, **Passwort:** aus der `.env`-Datei.
2.  **Schema-SQL ausführen:**
    Öffne `sql-schema/schema.sql`, kopiere den Inhalt und führe ihn im Query-Editor deines DB-Tools aus, um die `contacts`-Tabelle zu erstellen.

### Stufe 3: Sicherer öffentlicher Zugriff mit Cloudflare

1.  **DNS an Cloudflare delegieren:** Ändere die Nameserver bei deinem Domain-Registrar auf die von Cloudflare.
2.  **Cloudflare Tunnel erstellen und konfigurieren:**
    *   Im Cloudflare Zero Trust Dashboard, erstelle einen neuen Tunnel.
    *   Installiere `cloudflared` auf deinem Rechner und starte den Service mit dem bereitgestellten Befehl.
3.  **Traffic an n8n weiterleiten:**
    *   Füge einen "Public Hostname" hinzu:
    *   **Subdomain/Domain:** `workflows.deinedomain.com`
    *   **Service:** `HTTP` an `localhost:5678`

### Stufe 4: n8n-Konfiguration (Workflows & Zugangsdaten)

1.  **Auf n8n zugreifen:** Gehe zu `https://workflows.deinedomain.com` und erstelle deinen Admin-Account.
2.  **Zugangsdaten erstellen:**
    1.  **PostgreSQL:** Mit den Details aus Stufe 2.
    2.  **Google Service Account:** Mit `client_email` und `private_key` aus deiner GCP JSON-Datei.
3.  **Workflows importieren:**
    *   Importiere die beiden `.json`-Dateien aus dem `n8n-workflows`-Verzeichnis.
4.  **Zugangsdaten verbinden und aktivieren:**
    *   Öffne jeden Workflow, weise den Nodes die erstellten Zugangsdaten zu und fülle benötigte IDs (z.B. Google Sheet ID) aus.
    *   Aktiviere jeden Workflow.

### Stufe 5: Finale Integrationen (Formspree & Looker Studio)

1.  **Formspree mit n8n verbinden:**
    *   Füge in den Formspree-Integrationen einen neuen Webhook mit der URL `https://workflows.deinedomain.com/webhook/formspree-webhook` hinzu.
2.  **Looker Studio mit Google Sheets verbinden:**
    *   Erstelle in Looker Studio eine neue Datenquelle mit dem "Google Sheets"-Connector und wähle dein Tabellenblatt aus.

---

## ⚙️ Systembetriebs-Leitfaden

*   **Dauerbetrieb:** Das System läuft, solange dein Rechner und Docker Desktop aktiv sind.
*   **Statusprüfung:** `docker ps` zeigt den Status der Container.
*   **Neustart:** `docker-compose restart` im Projektverzeichnis startet beide Container neu.

---

## 💾 Backup-Strategie

1.  **Echtzeit-Rohdaten-Backup:** Jede Übermittlung wird sofort als `.json`-Datei in Google Drive gespeichert.
2.  **Wöchentliches strukturiertes Backup:** Ein Cronjob löst einen n8n-Workflow aus, der die `contacts`-Tabelle als `.csv` in Google Drive exportiert.
3.  **Manuelles vollständiges Snapshot:**
    ```bash
    docker-compose exec -T postgres pg_dumpall -U postgres > backup_$(date +%Y-%m-%d).sql
    ```

---

## ✍️ Autor
Dieses System wurde von **[Dein Name Hier]** entworfen, erstellt und dokumentiert.
*   **GitHub:** [ridvanyigit](https://github.com/ridvanyigit)
*   **LinkedIn:** [Ridvan Yigit](https://linkedin.com/in/profiliniz)
*   **Website:** [www.ridvanyigit.com](https://www.ridvanyigit.com/)

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
  <a href="#-sistem-mimarisi-2"><strong>🏗️ Mimari</strong></a> | 
  <a href="#-ön-gereksinimler-2"><strong>✅ Ön Gereksinimler</strong></a> | 
  <a href="#-5-aşamada-kurulum"><strong>🚀 Kurulum Rehberi</strong></a> |
  <a href="#-sistem-operasyon-rehberi-2"><strong> Sistem Operasyonu</strong></a> |
  <a href="#-yedekleme-stratejisi-1"><strong>💾 Yedekleme Stratejisi</strong></a>
</div>

---

## 🏗️ Sistem Mimarisi

| Katman | Araç | Konteyner Adı (Docker) | Erişilebilirlik | Veri Depolama |
| :--- | :--- | :--- | :--- | :--- |
| **Giriş/Tetikleyici** | **Formspree** | Yok | İnternet (POST) | Harici |
| **Arayüz/Güvenlik** | **Cloudflare Tunnel** | `cloudflared` (macOS Servisi) | `https://workflows.alanadiniz.com` | Yok |
| **Otomasyon (Motor)**| **n8n** | `n8n` | `http://localhost:5678` (İç Ağ) | PostgreSQL (`postgres` DB) |
| **Veri Deposu** | **PostgreSQL** | `postgres` | `localhost:5432` (İç Ağ) | Kalıcı Docker Volume (`pgdata`) |
| **Raporlama** | **Looker Studio** | Yok | `Veri Kaynağı: Google Sheets` | Harici (Google Cloud) |

---

## ✅ Ön Gereksinimler

1.  **Donanım:** 7/24 çalışabilen yerel bir makine (Örn: MacBook).
2.  **Yazılım:** **Docker Desktop** kurulu ve çalışır durumda.
3.  **Hizmetler:**
    *   Tescilli bir alan adı (Örn: **Namecheap** üzerinde).
    *   Bir **Cloudflare** Hesabı (Ücretsiz Plan).
    *   Bir Servis Hesabı oluşturulmuş **Google Cloud Platform** Hesabı.
    *   Bir **Formspree** Hesabı.

> 🚀 **Kuruluma hazır mısınız?** Adım adım yönlendirmeli kurulum için detaylı **[SETUP.md](SETUP.md)** dosyasını takip edin.

---

## 🚀 5 Aşamada Kurulum

### Aşama 1: Docker Compose ile Yerel Altyapı

1.  **Depoyu Klonlayın:**
    ```bash
    git clone [DEPO_URL] lead-capture-hub && cd lead-capture-hub
    ```
2.  **Ortam Dosyasını Oluşturun ve Yapılandırın:**
    ```bash
    cp .env.example .env
    ```
    `.env` dosyasını açın, güçlü bir `POSTGRES_PASSWORD` ve doğru `N8N_HOST` belirleyin.

3.  **Sistemi Başlatın:**
    ```bash
    docker-compose up -d
    ```

### Aşama 2: Veritabanı Başlatma

1.  **Veritabanına Bağlanın:** DBeaver gibi bir araç kullanın.
    *   **Host:** `localhost`, **Port:** `5432`, **Veritabanı/Kullanıcı:** `postgres`, **Şifre:** `.env` dosyanızdaki şifre.
2.  **Şema SQL'ini Çalıştırın:**
    `sql-schema/schema.sql` dosyasının içeriğini kopyalayıp veritabanı aracınızın sorgu düzenleyicisinde çalıştırarak `contacts` tablosunu oluşturun.

### Aşama 3: Cloudflare ile Güvenli Genel Erişim

1.  **DNS'i Cloudflare'e Devredin:** Alan adı kayıt firmanızda (Namecheap) nameserver'ları Cloudflare'ınkilerle değiştirin.
2.  **Cloudflare Tüneli Oluşturun ve Yapılandırın:**
    *   Cloudflare Zero Trust panelinde yeni bir tünel oluşturun.
    *   `cloudflared`'ı makinenize kurun ve verilen komutla servisi başlatın.
3.  **Trafiği n8n'e Yönlendirin:**
    *   Tünelin "Public Hostnames" sekmesinde yeni bir rota ekleyin:
    *   **Alan Adı:** `workflows.alanadiniz.com`
    *   **Servis:** `HTTP` - `localhost:5678`

### Aşama 4: n8n Yapılandırması (İş Akışları & Kimlik Bilgileri)

1.  **n8n'e Erişin:** `https://workflows.alanadiniz.com` adresine gidin ve yönetici hesabınızı oluşturun.
2.  **Kimlik Bilgileri (Credentials) Oluşturun:**
    1.  **PostgreSQL:** Aşama 2'deki bağlantı detaylarını kullanarak.
    2.  **Google Service Account:** GCP JSON dosyanızdan `client_email` ve `private_key`'i yapıştırarak.
3.  **İş Akışlarını İçe Aktarın:**
    *   `n8n-workflows` dizinindeki iki `.json` dosyasını `File -> Import from file...` ile içe aktarın.
4.  **Kimlik Bilgilerini Bağlayın ve Aktive Edin:**
    *   Her iş akışını açın, ilgili düğümlerde oluşturduğunuz kimlik bilgilerini seçin ve gerekli ID'leri (Google Sheet ID vb.) doldurun.
    *   Her iş akışını sağ üstteki düğme ile aktive edin.

### Aşama 5: Son Entegrasyonlar (Formspree & Looker Studio)

1.  **Formspree'yi n8n'e Bağlayın:**
    *   Formspree formunuzun "Integrations" bölümüne `https://workflows.alanadiniz.com/webhook/formspree-webhook` URL'si ile yeni bir Webhook ekleyin.
2.  **Looker Studio'yu Google Sheets'e Bağlayın:**
    *   Looker Studio'da "Google Sheets" bağlayıcısını kullanarak n8n'in veri yazdığı e-tabloyu veri kaynağı olarak ekleyin.

---

## ⚙️ Sistem Operasyon Rehberi

*   **Sürekli Çalışma:** Makineniz ve Docker Desktop çalıştığı sürece sistem aktiftir.
*   **Durum Kontrolü:** Konteyner durumlarını kontrol etmek için `docker ps` komutunu kullanın.
*   **Yeniden Başlatma:** Bir servis yanıt vermiyorsa, proje dizininde `docker-compose restart` komutunu çalıştırın.

---

## 💾 Yedekleme Stratejisi

1.  **Anlık Ham Veri Yedeği:** Her form gönderimi, ana n8n iş akışı tarafından anında bir `.json` dosyası olarak Google Drive'a kaydedilir.
2.  **Haftalık Yapılandırılmış Yedek:** Bir `cron` işi tarafından tetiklenen ikincil bir n8n iş akışı, tüm `contacts` tablosunu PostgreSQL'den bir `.csv` dosyası olarak dışa aktarır ve Google Drive'a kaydeder.
3.  **Manuel Tam Anlık Görüntü:**
    ```bash
    docker-compose exec -T postgres pg_dumpall -U postgres > backup_$(date +%Y-%m-%d).sql
    ```

---

## ✍️ Yazar
Bu sistem **[Adınız Soyadınız]** tarafından tasarlanmış, inşa edilmiş ve belgelenmiştir.
*   **GitHub:** [ridvanyigit](https://github.com/ridvanyigit)
*   **LinkedIn:** [Ridvan Yigit](https://linkedin.com/in/profiliniz)
*   **Website:** [www.ridvanyigit.com](https://www.ridvanyigit.com/)

</details>