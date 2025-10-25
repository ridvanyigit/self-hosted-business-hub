<div align="center">
  <h1 style="border-bottom: 2px solid #555; padding-bottom: 10px;">Setup Guide: Self-Hosted Data Automation Hub</h1>
  <p>This guide contains step-by-step instructions to set up the entire data collection and automation infrastructure on your own machine with a single Docker Compose command.</p>
</div>

---

### **✅ Prerequisites**

Before starting the setup, ensure you have the following accounts and tools:

| Service / Tool | Why is it required? |
| :--- | :--- |
| **[Docker Desktop](https://www.docker.com/products/docker-desktop/)** | To run the n8n and PostgreSQL services as containers. |
| **[Cloudflare Account](https://cloudflare.com/)** | To securely expose your webhook to the internet. |
| **A Domain Managed by Cloudflare** | To create an address like `workflows.yourdomain.com`. |
| **[Google Cloud Platform Project](https://console.cloud.google.com/)** | To create a Service Account for Google Sheets & Drive integration. |
| **[Pushover Account](https://pushover.net/)** | To receive instant success and failure notifications. |
| **[Git](https://git-scm.com/downloads)** | To clone the project files. |

---

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    ⚙️ Stage 1: Cloning the Project and Local Configuration
  </h3>
  <div style="padding: 16px;">
    <ol>
      <li>
        <strong>Clone the Project:</strong><br>
        Open your terminal and clone the project to your computer.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">git clone [PROJECT_URL] self-hosted-business-hub</code></pre>
      </li>
      <br>
      <li>
        <strong>Enter the Project Directory:</strong>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cd self-hosted-business-hub</code></pre>
      </li>
      <br>
      <li>
        <strong>Create the Environment File (<code>.env</code>):</strong><br>
        Copy the template file to store your secrets.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cp .env.example .env</code></pre>
      </li>
      <br>
      <li>
        <strong>Edit the Configuration:</strong><br>
        Open the <code>.env</code> file with a text editor and fill in the values.
        <blockquote style="border-left: 4px solid #d73a49; padding-left: 1rem; color: #cb2431;">
          <p>❗ <strong>Important:</strong> Set a strong database password and update the <code>N8N_HOST</code> field with the domain you will be using in Cloudflare (e.g., <code>workflows.mycompany.com</code>).</p>
        </blockquote>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    🌐 Stage 2: Exposing the System to the Internet with Cloudflare Tunnel
  </h3>
  <div style="padding: 16px;">
    <p>This stage ensures that requests from the internet (form submissions) can securely reach n8n on your local machine.</p>
    <ol>
      <li>
        <strong>Install the Cloudflare CLI (<code>cloudflared</code>):</strong><br>
        If you are using Homebrew, the command is simple:
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">brew install cloudflare/cloudflare/cloudflared</code></pre>
      </li>
      <br>
      <li>
        <strong>Log in to Your Cloudflare Account:</strong><br>
        This command will open a browser window. Select the account that holds your domain and authorize it.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel login</code></pre>
      </li>
      <br>
      <li>
        <strong>Create a Tunnel:</strong><br>
        Create a persistent tunnel for n8n. You can replace `n8n-tunnel` with any name you like.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel create n8n-tunnel</code></pre>
        <p>This command will give you a Tunnel ID and the location of a <code>.json</code> credential file. Take note of this ID.</p>
      </li>
      <br>
      <li>
        <strong>Configure the Tunnel:</strong><br>
        Create a file named <code>config.yml</code> inside the <code>~/.cloudflared/</code> directory and fill it with the following content. Replace <code>YOUR_TUNNEL_ID</code> with the ID you obtained in the previous step.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">tunnel: YOUR_TUNNEL_ID
credentials-file: /Users/YOUR_USERNAME/.cloudflared/YOUR_TUNNEL_ID.json

ingress:
  - hostname: workflows.yourdomain.com # Must match N8N_HOST in the .env file
    service: http://localhost:5678
  - service: http_status:404</code></pre>
      </li>
      <br>
      <li>
        <strong>Create the DNS Record:</strong><br>
        Link your tunnel to your domain name.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">cloudflared tunnel route dns n8n-tunnel workflows.yourdomain.com</code></pre>
      </li>
      <br>
      <li>
        <strong>Start the Tunnel as a Service:</strong><br>
        This command ensures the tunnel starts automatically every time your computer boots up.
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">sudo cloudflared service install</code></pre>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    🚀 Stage 3: Launching the Entire System
  </h3>
  <div style="padding: 16px;">
    <ol>
      <li>
        <strong>Run Docker Compose:</strong><br>
        Make sure you are in the project's root directory in your terminal and run the following:
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker-compose up -d</code></pre>
      </li>
      <br>
      <li>
        <strong>Verify It's Running:</strong><br>
        <pre><code style="background-color: #eaf5ff; padding: 5px; border-radius: 4px;">docker ps</code></pre>
        <p>You should see <code>postgres-db</code> and <code>n8n</code> in the list with an "Up" status.</p>
      </li>
    </ol>
  </div>
</div>

<div style="background-color: #f6f8fa; border: 1px solid #d1d5da; border-radius: 8px; margin-bottom: 20px;">
  <h3 style="margin: 0; padding: 12px 16px; background-color: #f6f8fa; border-top-left-radius: 8px; border-top-right-radius: 8px; border-bottom: 1px solid #d1d5da;">
    💡 Stage 4: Post-Launch Configuration
  </h3>
  <div style="padding: 16px;">

<details style="margin-bottom: 10px; border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.1: Creating the Database Schema</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>Connect to the PostgreSQL database (with a tool like PgAdmin, DBeaver, or TablePlus). Connection details:
        <ul>
          <li><strong>Host:</strong> <code>localhost</code></li>
          <li><strong>Port:</strong> <code>5432</code></li>
          <li><strong>Database:</strong> <code>postgres</code></li>
          <li><strong>User:</strong> <code>postgres</code></li>
          <li><strong>Password:</strong> The <code>POSTGRES_PASSWORD</code> from your <code>.env</code> file.</li>
        </ul>
      </li>
      <li>Open a "Query Tool".</li>
      <li>Copy the entire content of the <code>sql-schema/schema.sql</code> file, paste it into the Query Tool, and run it to create your tables.</li>
    </ol>
  </div>
</details>

<details style="margin-bottom: 10px; border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.2: Configuring n8n Automations</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>In your browser, go to <code>http://localhost:5678</code> and create your n8n admin user.</li>
      <li>On the main screen, go to <strong><code>Workflows -> Import -> Import from file...</code></strong>.</li>
      <li>Import the <code>.json</code> files from the <code>n8n-workflows</code> directory one by one.</li>
      <li>
        <strong>Critical Step: Creating Credentials</strong><br>
        Go to <strong>Credentials</strong> from the left menu and create the following credentials:
        <ul>
          <li><strong>PostgreSQL:</strong> Create new credentials with your database information (<code>host: postgres</code>, <code>password:</code> from the <code>.env</code> file).</li>
          <li><strong>Google:</strong> Create a "Google API" credential using your own Google Cloud Service Account JSON key.</li>
          <li><strong>Pushover:</strong> Create a new Pushover credential with your Pushover User Key and API Token.</li>
        </ul>
      </li>
      <li><strong>Update the Workflows:</strong> Open the workflow you imported. Click on the PostgreSQL, Google Sheets/Drive, and Pushover nodes and select the correct credentials you just created from the "Credential" section.</li>
      <li>Update the <strong>Spreadsheet ID</strong> and <strong>Folder ID</strong> fields in the Google Sheets/Drive nodes with your own IDs.</li>
      <li>Activate each workflow using the <strong>"Active"</strong> toggle in the top-right corner.</li>
    </ol>
  </div>
</details>

<details style="border: 1px solid #d1d5da; border-radius: 6px;">
  <summary style="padding: 10px; font-weight: bold; cursor: pointer;">4.3: Vercel and Website Settings</summary>
  <div style="padding: 15px; border-top: 1px solid #d1d5da;">
    <ol>
      <li>
        <strong>Get the Webhook URL:</strong><br>
        In your n8n workflow, click on the Webhook node. Copy the address shown as the "Production URL", which will look like <code>https://workflows.yourdomain.com/...</code>.
      </li>
      <li>
        <strong>Update Your Website's Form:</strong><br>
        In the code for the form on your website deployed on Vercel, set the form's <code>action</code> URL to this webhook address you copied.
      </li>
      <li>
        <strong>CORS Settings (If Needed):</strong><br>
        If you encounter a CORS error in the browser when sending a request from your Vercel site to n8n, you may need to set the necessary <code>Access-Control-Allow-Origin</code> headers by adding a <code>vercel.json</code> file to your Vercel project or in your server-side code (like a Next.js API route).
      </li>
    </ol>
  </div>
</details>
  </div>
</div>

---

<div align="center">
  <h2>🎉 Congratulations! 🎉</h2>
  <p>Your data collection and automation infrastructure is now fully operational.</p>
</div>

<br>
<br>

<div align="center">
  <p>Created by <a href="https://www.ridvanyigit.com" target="_blank">Rıdvan Yiğit</a></p>
</div>