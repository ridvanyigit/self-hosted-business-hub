<div align="center">
  <img src="https://raw.githubusercontent.com/n8n-io/n8n-docs/main/archive/static/images/n8n-logo.png" alt="n8n Logo" width="100"/>
  <h1 style="border-bottom: 2px solid #555; padding-bottom: 10px;">Kurulum Rehberi: Self-Hosted Veri Otomasyon Merkezi</h1>
  <p>Bu rehber, tüm veri toplama ve otomasyon altyapısını Docker Compose ile tek komutta kendi makinenize kurmak için adım adım talimatlar içerir.</p>
</div>

---

### **✅ Ön Gereksinimler**

Kuruluma başlamadan önce aşağıdaki hesaplara ve araçlara sahip olduğunuzdan emin olun:

| Servis / Araç | Neden Gerekli? |
| :--- | :--- |
| **[Docker Desktop](https://www.docker.com/products/docker-desktop/)** | n8n ve PostgreSQL servislerini konteyner olarak çalıştırmak için. |
| **[Cloudflare Hesabı](https://cloudflare.com/)** | Webhook'unuzu güvenli bir şekilde internete açmak için. |
| **Cloudflare Tarafından Yönetilen Bir Domain** | `workflows.sizin-domaininiz.com` gibi bir adres oluşturmak için. |
| **[Google Cloud Platform Projesi](https://console.cloud.google.com/)** | Google Sheets & Drive entegrasyonu için Servis Hesabı (Service Account) oluşturmak. |
| **[Pushover Hesabı](https://pushover.net/)** | Anlık başarı ve hata bildirimleri almak için. |
| **[Git](https://git-scm.com/downloads)** | Proje dosyalarını klonlamak için. |

---

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    ⚙️ 1. Aşama: Projeyi Klonlama ve Yerel Yapılandırma
  </h3>
  <div style="padding: 16px;">
    <ol>
      <li>
        <strong>Projeyi Klonlayın:</strong><br>
        Terminali açın ve projeyi bilgisayarınıza klonlayın.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">git clone [PROJE_URL] self-hosted-business-hub</code></pre>
      </li>
      <br>
      <li>
        <strong>Proje Dizinine Girin:</strong>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cd self-hosted-business-hub</code></pre>
      </li>
      <br>
      <li>
        <strong>Ortam Dosyasını (<code>.env</code>) Oluşturun:</strong><br>
        Güvenli bilgilerinizi saklamak için şablon dosyayı kopyalayın.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cp .env.example .env</code></pre>
      </li>
      <br>
      <li>
        <strong>Yapılandırmayı Düzenleyin:</strong><br>
        <code>.env</code> dosyasını bir metin düzenleyiciyle açın ve değerleri doldurun.
        <blockquote style="border-left: 4px solid #d73a49; padding-left: 1rem; color: #cb2431;">
          <p>❗ <strong>Önemli:</strong> Güçlü bir veritabanı şifresi belirleyin ve <code>N8N_HOST</code> alanını Cloudflare'de kullanacağınız kendi alan adınızla güncelleyin (örn: <code>workflows.sirketim.com</code>).</p>
        </blockquote>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    🌐 2. Aşama: Cloudflare Tüneli ile Sistemi İnternete Açma
  </h3>
  <div style="padding: 16px;">
    <p>Bu aşama, internetten gelen isteklerin (form gönderimleri) yerel makinenizdeki n8n'e güvenli bir şekilde ulaşmasını sağlar.</p>
    <ol>
      <li>
        <strong>Cloudflare CLI'yi (<code>cloudflared</code>) Yükleyin:</strong><br>
        Eğer Homebrew kullanıyorsanız, komut basittir:
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">brew install cloudflare/cloudflare/cloudflared</code></pre>
      </li>
      <br>
      <li>
        <strong>Cloudflare Hesabınızda Oturum Açın:</strong><br>
        Bu komut bir tarayıcı penceresi açacaktır. Alan adınızın olduğu hesabı seçin ve yetkilendirin.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel login</code></pre>
      </li>
      <br>
      <li>
        <strong>Tünel Oluşturun:</strong><br>
        n8n için kalıcı bir tünel oluşturun. `n8n-tunnel` yerine istediğiniz bir ismi verebilirsiniz.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel create n8n-tunnel</code></pre>
        <p>Bu komut size bir Tünel ID'si ve bir <code>.json</code> kimlik bilgisi dosyasının konumunu verecektir. Bu ID'yi bir yere not edin.</p>
      </li>
      <br>
      <li>
        <strong>Tüneli Yapılandırın:</strong><br>
        <code>~/.cloudflared/</code> dizini içinde <code>config.yml</code> adında bir dosya oluşturun ve içeriğini aşağıdaki gibi doldurun. <code>YOUR_TUNNEL_ID</code> kısmını önceki adımda aldığınız ID ile değiştirin.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">tunnel: YOUR_TUNNEL_ID
credentials-file: /Users/KULLANICI_ADINIZ/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: workflows.sizin-domaininiz.com # .env dosyasındaki N8N_HOST ile aynı olmalı
    service: http://localhost:5678
  - service: http_status:404</code></pre>
      </li>
      <br>
      <li>
        <strong>DNS Kaydını Oluşturun:</strong><br>
        Tünelinizi alan adınıza bağlayın.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel route dns n8n-tunnel workflows.sizin-domaininiz.com</code></pre>
      </li>
      <br>
      <li>
        <strong>Tüneli Servis Olarak Başlatın:</strong><br>
        Bu komut, tünelin bilgisayarınız her açıldığında otomatik olarak başlamasını sağlar.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">sudo cloudflared service install</code></pre>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    🚀 3. Aşama: Tüm Sistemi Başlatma
  </h3>
  <div style="padding: 16px;">
    <ol>
      <li>
        <strong>Docker Compose'u Çalıştırın:</strong><br>
        Terminalde projenin ana dizininde olduğunuzdan emin olun ve şunu çalıştırın:
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker-compose up -d</code></pre>
      </li>
      <br>
      <li>
        <strong>Çalıştığını Doğrulayın:</strong><br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker ps</code></pre>
        <p>Listede <code>postgres-db</code> ve <code>n8n</code>'i "Up" durumuyla görmelisiniz.</p>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    💡 4. Aşama: Başlatma Sonrası Yapılandırma
  </h3>
  <div style="padding: 16px;">

<details style="margin-bottom: 10px; border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.1: Veritabanı Şemasını Oluşturma</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>PostgreSQL veritabanına bağlanın (PgAdmin, DBeaver veya TablePlus gibi bir araçla). Bağlantı bilgileri:
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

<details style="margin-bottom: 10px; border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.2: n8n Otomasyonlarını Yapılandırma</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>Tarayıcınızda <code>http://localhost:5678</code> adresine gidin ve n8n yönetici kullanıcınızı oluşturun.</li>
      <li>Ana ekranda, <strong><code>Workflows -> Import -> Import from file...</code></strong> seçeneğine gidin.</li>
      <li><code>n8n-workflows</code> dizinindeki <code>.json</code> dosyalarını tek tek içe aktarın.</li>
      <li>
        <strong>Kritik Adım: Kimlik Bilgilerini (Credentials) Oluşturma</strong><br>
        Sol menüden <strong>Credentials</strong>'a gidin ve aşağıdaki kimlik bilgilerini oluşturun:
        <ul>
          <li><strong>PostgreSQL:</strong> Veritabanı bilgilerinizle (`host: postgres`, `password: .env`'deki şifre) yeni bir kimlik bilgisi oluşturun.</li>
          <li><strong>Google:</strong> Kendi Google Cloud Servis Hesabı JSON anahtarınızı kullanarak "Google API" kimlik bilgisi oluşturun.</li>
          <li><strong>Pushover:</strong> Pushover User Key ve API Token'ınızla yeni bir Pushover kimlik bilgisi oluşturun.</li>
        </ul>
      </li>
      <li><strong>İş Akışlarını Güncelleyin:</strong> İçe aktardığınız iş akışını açın. PostgreSQL, Google Sheets/Drive ve Pushover düğümlerine tıklayıp, "Credential" bölümünden az önce oluşturduğunuz doğru kimlik bilgilerini seçin.</li>
      <li>Google Sheets/Drive düğümlerindeki <strong>Spreadsheet ID</strong> ve <strong>Folder ID</strong> alanlarını kendi ID'lerinizle güncelleyin.</li>
      <li>Sağ üst köşedeki <strong>"Active"</strong> düğmesini kullanarak iş akışını etkinleştirin.</li>
    </ol>
  </div>
</details>

<details style="border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.3: Vercel ve Web Sitesi Ayarları</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>
        <strong>Webhook URL'sini Alın:</strong><br>
        n8n'deki iş akışınızda Webhook düğümüne tıklayın. "Production URL" olarak görünen <code>https://workflows.sizin-domaininiz.com/...</code> adresini kopyalayın.
      </li>
      <li>
        <strong>Web Sitenizin Formunu Güncelleyin:</strong><br>
        Vercel'de deploy ettiğiniz web sitenizin form kodunda, formun `action` (eylem) URL'sini bu kopyaladığınız webhook adresine ayarlayın.
      </li>
      <li>
        <strong>CORS Ayarları (Gerekirse):</strong><br>
        Eğer Vercel'deki sitenizden n8n'e istek gönderirken tarayıcıda CORS hatası alırsanız, Vercel projenize bir <code>vercel.json</code> dosyası ekleyerek veya sunucu tarafı kodunuzda (Next.js API route gibi) gerekli `Access-Control-Allow-Origin` başlıklarını (header) ayarlamanız gerekebilir.
      </li>
    </ol>
  </div>
</details>
  </div>
</div>

---

<div align="center">
  <h2>🎉 Tebrikler! 🎉</h2>
  <p>Veri toplama ve otomasyon altyapınız artık tamamen çalışır durumda.</p>
</div>