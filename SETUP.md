<div align="center">
  <h1 style="border-bottom: 2px solid #555; padding-bottom: 10px;">Setup Guide: Lead Capture & Analytics Hub</h1>
  <p>Bu rehber, tüm veri toplama ve otomasyon altyapısını Docker Compose ile tek komutta yerel makinenize kurmak için adım adım talimatlar içerir.</p>
</div>

---

### **✅ Ön Gereksinimler**

Başlamadan önce, sisteminizde tek bir şeyin kurulu olması yeterlidir:
- **[Docker Desktop](https://www.docker.com/products/docker-desktop/):** Lütfen devam etmeden önce kurulu ve çalışır durumda olduğundan emin olun.

---

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    ⚙️ Adım 1: Projeyi Alın ve Yapılandırmanızı Oluşturun
  </h3>
  <div style="padding: 16px;">
    <p>İlk olarak, proje dosyalarını alıp şifreleriniz ve ayarlarınız için güvenli bir yapılandırma dosyası oluşturacağız.</p>
    <ol>
      <li>
        <strong>Projeyi Klonlayın veya İndirin</strong><br>
        Terminali açın ve projeyi istediğiniz bir konuma klonlayın.
        <br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">git clone [PROJE_URL] lead-capture-hub</code></pre>
      </li>
      <br>
      <li>
        <strong>Ortam Dosyasını (Environment File) Oluşturun</strong><br>
        Proje dizinine gidin. <code>.env.example</code> adlı bir şablon dosyası göreceksiniz.
        <br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cd lead-capture-hub
cp .env.example .env</code></pre>
        <p>Bu komut, Git tarafından yok sayılan ve sırlarınızı güvende tutan yeni bir <code>.env</code> dosyası oluşturur.</p>
      </li>
      <br>
      <li>
        <strong>Yapılandırmayı Düzenleyin</strong><br>
        Yeni <code>.env</code> dosyasını bir metin düzenleyiciyle açın. Gerekli değerleri doldurun. Bunlar servisler için kişisel kimlik bilgileriniz olacaktır.
        <blockquote style="border-left: 4px solid #d73a49; padding-left: 1rem; color: #cb2431;">
          <p>❗ <strong>Önemli:</strong> Lütfen veritabanı şifresi için güçlü ve benzersiz bir şifre seçin ve N8N_HOST alanını kendi alan adınızla güncelleyin.</p>
        </blockquote>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    🚀 Adım 2: Tek Komutla Tüm Sistemi Başlatın
  </h3>
  <div style="padding: 16px;">
    <p>Yapılandırmanız hazır olduğunda, tüm servisleri tek bir komutla başlatabilirsiniz.</p>
    <ol>
      <li>
        <strong>Docker Compose'u Çalıştırın</strong><br>
        Terminalde projenin ana dizininde olduğunuzdan emin olun ve şunu çalıştırın:
        <br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker-compose up -d</code></pre>
        <p>Bu komut, <code>docker-compose.yml</code> dosyasını okur ve gerekli tüm konteynerleri (PostgreSQL, n8n) ve onları birbirine bağlayan ağı otomatik olarak oluşturur ve başlatır.</p>
      </li>
      <br>
      <li>
        <strong>Her Şeyin Çalıştığını Doğrulayın</strong><br>
        Birkaç dakika sonra, iki konteynerin de çalışır durumda olup olmadığını kontrol etmek için şu komutu çalıştırın:
        <br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker ps</code></pre>
        <p>Listede <code>postgres-db</code> ve <code>n8n</code>'i "Up" durumuyla görmelisiniz.</p>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    💡 Adım 3: Başlatma Sonrası Yapılandırma
  </h3>
  <div style="padding: 16px;">
    <p>Altyapınız artık çalışıyor. Son adım, her servisi verilerinizle ve iş akışlarınızla çalışacak şekilde yapılandırmaktır.</p>

<details style="margin-bottom: 10px; border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">3.1: Veritabanı Şemasını Oluşturma</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>PostgreSQL veritabanına bağlanın (PgAdmin veya DBeaver gibi bir araçla). Bağlantı bilgileri:
        <ul>
          <li><strong>Host:</strong> <code>localhost</code></li>
          <li><strong>Port:</strong> <code>5432</code></li>
          <li><strong>Database:</strong> <code>postgres</code></li>
          <li><strong>Kullanıcı:</strong> <code>postgres</code></li>
          <li><strong>Şifre:</strong> <code>.env</code> dosyanızdaki <code>POSTGRES_PASSWORD</code>.</li>
        </ul>
      </li>
      <li>Bir "Query Tool" (Sorgu Aracı) açın.</li>
      <li><code>sql-schema/schema.sql</code> dosyasının tüm içeriğini kopyalayıp Sorgu Aracına yapıştırın ve tablolarınızı oluşturmak için çalıştırın.</li>
    </ol>
  </div>
</details>

<details style="border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">3.2: Otomasyonları Yapılandırma (n8n)</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>Tarayıcınızda <strong><code>http://localhost:5678</code></strong> adresine gidin ve n8n yönetici kullanıcınızı oluşturun.</li>
      <li>Her iş akışı için, <strong><code>File -> Import from file...</code></strong> seçeneğine gidin ve <code>n8n-workflows</code> dizinindeki <code>.json</code> dosyalarını tek tek içe aktarın.</li>
      <li><strong>Kritik Adım:</strong> İçe aktardıktan sonra, her servis için kimlik bilgilerini yeniden oluşturmanız gerekir.
        <ul>
          <li><strong>Google Drive/Sheets</strong> düğümlerinde, kendi Google Cloud Servis Hesabı JSON anahtarınızı kullanarak yeni kimlik bilgileri oluşturun.</li>
          <li><strong>PostgreSQL</strong> düğümünde, <code>.env</code> dosyanızdaki veritabanı şifresini kullanarak yeni bir kimlik bilgisi oluşturun.</li>
        </ul>
      </li>
      <li>Kimlik bilgileri ayarlandıktan ve düğümler test edildikten sonra, sağ üst köşedeki düğmeyi kullanarak her iş akışını etkinleştirin.</li>
    </ol>
  </div>
</details>
  </div>
</div>

---

<div align="center">
  <h2>🎉 Tebrikler! 🎉</h2>
  <p>Veri toplama ve analiz altyapınız artık tamamen çalışır durumda.</p>
</div>