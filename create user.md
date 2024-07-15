以下是上述步驟轉換為 Markdown 格式的文檔：

```markdown
# 允許用戶管理彼此的 Podman 容器

## 目標

設置 `sysmon1` 和 `sysmon2` 兩個用戶，允許他們彼此管理和訪問 Podman 容器。

## 步驟

### 1. 創建群組並添加用戶

創建一個新的群組 `podman-users`，並將 `sysmon1` 和 `sysmon2` 添加到該群組：

```bash
sudo groupadd podman-users
sudo usermod -aG podman-users sysmon1
sudo usermod -aG podman-users sysmon2
```

### 2. 設置共享的 Podman 存儲與配置目錄

設置共享的存儲目錄和配置文件：

```bash
sudo mkdir -p /etc/containers/shared-config
sudo chown -R :podman-users /etc/containers/shared-config
sudo chmod -R 770 /etc/containers/shared-config

sudo mkdir -p /var/lib/shared-containers
sudo chown -R :podman-users /var/lib/shared-containers
sudo chmod -R 770 /var/lib/shared-containers
```

### 3. 設置每個用戶的環境變量

在每個用戶的 `.bashrc` 或 `.bash_profile` 中添加以下環境變量：

```bash
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export XDG_CONFIG_HOME=/etc/containers/shared-config' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export CONTAINERS_STORAGE_CONF=/etc/containers/shared-config/containers.conf' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export CONTAINERS_STORAGE=/var/lib/shared-containers' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
```

### 4. 確保 Podman 存儲目錄的群組權限

確保 Podman 存儲目錄的群組權限正確：

```bash
sudo chown -R :podman-users /var/lib/shared-containers
sudo chmod -R 770 /var/lib/shared-containers
```

### 5. 設置訪問控制列表（ACLs）

使用 ACLs 來設置細粒度的權限控制：

```bash
sudo apt-get install acl
sudo setfacl -m g:podman-users:rwx /etc/containers/shared-config
sudo setfacl -m d:g:podman-users:rwx /etc/containers/shared-config
sudo setfacl -m g:podman-users:rwx /var/lib/shared-containers
sudo setfacl -m d:g:podman-users:rwx /var/lib/shared-containers
```

## 完整的實施過程

以下是完整的腳本：

```bash
# 創建群組
sudo groupadd podman-users

# 將用戶添加到群組
sudo usermod -aG podman-users sysmon1
sudo usermod -aG podman-users sysmon2

# 創建並設置共享配置目錄
sudo mkdir -p /etc/containers/shared-config
sudo chown -R :podman-users /etc/containers/shared-config
sudo chmod -R 770 /etc/containers/shared-config

# 創建並設置共享存儲目錄
sudo mkdir -p /var/lib/shared-containers
sudo chown -R :podman-users /var/lib/shared-containers
sudo chmod -R 770 /var/lib/shared-containers

# 為每個用戶創建獨立的運行時目錄
sudo mkdir -p /run/user/$(id -u sysmon1)
sudo mkdir -p /run/user/$(id -u sysmon2)
sudo chown sysmon1:sysmon1 /run/user/$(id -u sysmon1)
sudo chown sysmon2:sysmon2 /run/user/$(id -u sysmon2)

# 配置每個用戶的環境變量
echo 'export XDG_RUNTIME_DIR=/run/user/$(id -u)' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export XDG_CONFIG_HOME=/etc/containers/shared-config' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export CONTAINERS_STORAGE_CONF=/etc/containers/shared-config/containers.conf' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc
echo 'export CONTAINERS_STORAGE=/var/lib/shared-containers' | sudo tee -a /home/sysmon1/.bashrc /home/sysmon2/.bashrc

# 設置存儲目錄權限
sudo chown -R :podman-users /var/lib/shared-containers
sudo chmod -R 770 /var/lib/shared-containers

# 設置 ACLs
sudo apt-get install acl
sudo setfacl -m g:podman-users:rwx /etc/containers/shared-config
sudo setfacl -m d:g:podman-users:rwx /etc/containers/shared-config
sudo setfacl -m g:podman-users:rwx /var/lib/shared-containers
sudo setfacl -m d:g:podman-users:rwx /var/lib/shared-containers
```

這樣設置後，`sysmon1` 和 `sysmon2` 可以互相管理和訪問對方創建的 Podman 容器。如果還有其他問題或需要進一步的幫助，請隨時告訴我。
```