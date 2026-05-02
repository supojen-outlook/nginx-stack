# nginx_stack

生產級 Nginx 技術棧，整合 GoAccess 即時日誌分析、GeoIP 地理位置、速率限制和 SSL 優化。

## 功能特性

- **Nginx 網頁伺服器**：最新版本，配置優化
- **GoAccess 分析**：即時網頁日誌分析器，支援 WebSocket 即時更新
- **GeoIP 整合**：MaxMind GeoLite2-City 資料庫，分析訪客地理位置
- **速率限制**：內建防護，抵禦濫用和 DDoS 攻擊
- **SSL 優化**：現代 TLS 配置，支援會話快取
- **Let's Encrypt 就緒**：預設配置 ACME 憑證驗證
- **跨平台支援**：支援 Ubuntu 24.04/22.04、Debian 12/11、RHEL 8/9

## 系統需求

- Ansible 2.9+
- 目標作業系統：Ubuntu 24.04/22.04、Debian 12/11 或 RHEL 8/9
- MaxMind GeoIP 帳號（免費）用於 GeoLite2 資料庫

## Role 變數

### 必要變數

| 變數 | 說明 | 範例 |
|----------|-------------|---------|
| `goaccess_admin_password` | GoAccess 統計頁面密碼 | `secret123` |
| `maxmind_account_id` | 您的 MaxMind 帳號 ID | `123456` |
| `maxmind_license_key` | 您的 MaxMind 授權金鑰 | `abc123xyz` |
| `domain` | 您的網域，用於 WebSocket URL | `example.com` |

### 選填變數

| 變數 | 預設值 | 說明 |
|----------|---------|-------------|
| `nginx_worker_processes` | `auto` | Worker 程序數量 |
| `nginx_worker_connections` | `1024` | 每個 Worker 的最大連線數 |
| `nginx_client_max_body_size` | `100M` | 最大上傳大小 |
| `goaccess_admin_user` | `admin` | GoAccess 使用者名稱 |
| `goaccess_stats_path` | `/var/www/html/stats.html` | GoAccess 輸出路徑 |
| `goaccess_ws_port` | `7890` | WebSocket 連接埠 |

## 安裝方式

### 方法 1：透過 GitHub 安裝（推薦）

在你的 Ansible 專案中建立 `requirements.yml`：

```yaml
---
- src: https://github.com/supojen-outlook/nginx-stack
  name: nginx_stack
```

然後執行安裝：

```bash
ansible-galaxy install -r requirements.yml -p ./roles
```

### 方法 2：直接 Clone

```bash
cd your-project/roles
git clone https://github.com/supojen-outlook/nginx-stack.git nginx_stack
```

## 使用範例

```yaml
- hosts: webservers
  become: true
  roles:
    - role: supojen.nginx_stack
      vars:
        domain: example.com
        goaccess_admin_user: admin
        goaccess_admin_password: "{{ vault_goaccess_password }}"
        maxmind_account_id: "123456"
        maxmind_license_key: "{{ vault_maxmind_key }}"
```

### Nginx 虛擬主機範例

```nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    
    # GoAccess 統計頁面（受 htpasswd 保護）
    location = /stats.html {
        alias /var/www/html/stats.html;
        auth_basic "GoAccess Stats";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
    
    # GoAccess WebSocket
    location /goaccess-ws {
        proxy_pass http://127.0.0.1:7890;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
    
    # 帶速率限制的 API
    location /api/ {
        limit_req zone=api_limit burst=10 nodelay;
        proxy_pass http://backend;
    }
    
    # 靜態檔案
    location / {
        root /var/www/html;
        try_files $uri /index.html;
    }
}
```

## 服務

| 服務 | 說明 | 連接埠 |
|---------|-------------|------|
| nginx | 網頁伺服器 | 80, 443 |
| goaccess | 即時日誌分析器 | 7890 (WebSocket) |
| geoipupdate | 每週 GeoIP 資料庫更新 | - |

## 存取分析報表

造訪 `https://your-domain.com/stats.html` 並使用配置的帳號密碼登入。

## 授權條款

BSD

## 作者

蘇柏仁
