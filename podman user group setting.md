
# 使用群組來管理不同用戶之間對容器的訪問權限

## 步驟 1：創建一個新的群組

創建一個新的群組 `podman-users`：

```bash
sudo groupadd podman-users
```

## 步驟 2：將用戶添加到群組

將用戶 `sysmon1` 和 `sysmon2` 添加到這個新群組中：

```bash
sudo usermod -aG podman-users sysmon1
sudo usermod -aG podman-users sysmon2
```

## 步驟 3：設置共享的 Podman 配置目錄

創建一個共享的 Podman 配置目錄，並將其群組設置為 `podman-users`：

```bash
sudo mkdir /etc/containers/shared-config
sudo chown -R :podman-users /etc/containers/shared-config
sudo chmod -R 770 /etc/containers/shared-config
```

## 步驟 4：配置每個用戶使用共享目錄

在每個用戶的 `.bashrc` 或 `.bash_profile` 中添加以下環境變量：

```bash
export XDG_RUNTIME_DIR=/etc/containers/shared-config/run
export XDG_CONFIG_HOME=/etc/containers/shared-config/config
export CONTAINERS_STORAGE_CONF=/etc/containers/shared-config/containers.conf
```

你可以使用以下命令來編輯每個用戶的配置文件，例如：

```bash
sudo -u sysmon1 bash -c 'echo "export XDG_RUNTIME_DIR=/etc/containers/shared-config/run" >> ~/.bashrc'
sudo -u sysmon1 bash -c 'echo "export XDG_CONFIG_HOME=/etc/containers/shared-config/config" >> ~/.bashrc'
sudo -u sysmon1 bash -c 'echo "export CONTAINERS_STORAGE_CONF=/etc/containers/shared-config/containers.conf" >> ~/.bashrc'

sudo -u sysmon2 bash -c 'echo "export XDG_RUNTIME_DIR=/etc/containers/shared-config/run" >> ~/.bashrc'
sudo -u sysmon2 bash -c 'echo "export XDG_CONFIG_HOME=/etc/containers/shared-config/config" >> ~/.bashrc'
sudo -u sysmon2 bash -c 'echo "export CONTAINERS_STORAGE_CONF=/etc/containers/shared-config/containers.conf" >> ~/.bashrc'
```

## 步驟 5：確保共享目錄的權限正確

確保共享目錄及其子目錄和文件對 `podman-users` 群組有正確的讀寫權限：

```bash
sudo chown -R :podman-users /etc/containers/shared-config
sudo chmod -R 770 /etc/containers/shared-config
```

## 步驟 6：設置 Podman 容器存儲的群組權限

配置 Podman 容器存儲目錄的權限，使其對 `podman-users` 群組可讀寫：

```bash
sudo chown -R :podman-users /var/lib/containers
sudo chmod -R 770 /var/lib/containers
```

這樣一來，用戶 `sysmon1` 和 `sysmon2` 都可以訪問和管理 Podman 容器，包括啟動、停止、刪除和檢查容器。
```

你可以將上述內容保存為一個 `.md` 文件，這樣就可以方便地使用和分享了。如果有其他需要修改或添加的部分，請隨時告訴我。