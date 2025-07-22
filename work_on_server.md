# 基礎題：
## 撰寫 Dockerfile 和 docker-compose.yml，創建一個實驗環境讓你可以在 jupyter notebook 中實驗並且將相關的紀錄顯示在另一個 Container 的 tensorboard 中。
```dockerfile
# Dockerfile.jupyter
FROM jupyter/base-notebook:latest
RUN mkdir /home/jovyan/logs
RUN chown -R jovyan:users /home/jovyan/logs
# 將當前目錄掛載到 container 的工作目錄
WORKDIR /home/jovyan/work
```

```dockerfile
# Dockerfile.tensorboard
FROM tensorflow/tensorflow:latest
# 安裝 TensorBoard
RUN pip install tensorboard
# 將當前目錄掛載到 container 的工作目錄
WORKDIR /home/tensorboard
```

``` bash
# Build Docker images
docker build -t jupyter_notebook:latest -f Dockerfile.jupyter .
docker build -t tensorboard:latest -f Dockerfile.tensorboard .
```

Docker Compose 文件：
```yaml
version: '3.8'
services:
    jupyter_notebook:
        image: jupyter_notebook:latest
        container_name: jupyter_notebook
        ports:
            - "8888:8888"
        volumes:
            - ./notebooks:/home/jovyan/work
            - logs:/home/jovyan/logs
        command: start-notebook.sh --NotebookApp.token=''

    tensorboard:
        image: tensorboard:latest
        container_name: tensorboard
        ports:
            - "6006:6006"
        volumes:
            - logs:/home/tensorboard/logs
        command: tensorboard --logdir=/home/tensorboard/logs --host=0.0.0.0
volumes:
    logs:
```

```bash
docker compose up -d
# 這樣就可以啟動 Jupyter Notebook 和 TensorBoard 服務了
```
``` bash
# ssh port forwarding
ssh -L 8888:localhost:8888 -L 6006:localhost:6006 user@remote_host
# 這樣可以將本地的 8888 和 6006  port 轉發到遠程主機的相應 port 
```

# 情境題：
## 你在一個 container 中完成了訓練工作，但忘記掛載 volume，現在需要將 container 內部的資料拯救出來。請寫出你的流程。
```bash
# 首先，找到 container 的 ID 或名稱
docker ps -a  # 列出所有 container ，找到需要的 container  ID 或名稱
# 假設 container  ID 為 <container_id>
# 接下來，使用 docker cp 命令將 container 內的資料 copy 到主機上
docker cp <container_id>:/path/to/data /path/on/host
# 這樣就可以將 container 內的資料拷貝到主機上的指定路徑了
```

## 你在一個 container 中進行了多次環境設置和安裝操作，現在需要將這個配置好的環境打包成 Docker Image，但不想重寫一個 Dockerfile。請問該怎麼做？
```bash
docker commit <container_id> <new_image_name>
# 這樣可以將當前 container 的狀態保存為一個新的 Docker Image
# 例如：
docker commit <container_id> my_custom_image:latest
# 接下來可以使用 docker push 將這個新 Image 推送到 Docker Hub 或其他 harbor
docker push my_custom_image:latest
```


# 問答題：
## 什麼是 Multi-stage Build Image？請解釋其優點和用途。
Multi-stage Build Image 是 Docker 的一種構建技術，允許在一個 Dockerfile 中定義多個構建階段。每個階段可以使用不同的基礎 Image ，並且可以在最終 Image 中只包含所需的文件和依賴項。這樣可以顯著減少最終 Image 的大小，因為不需要將所有構建過程中的臨時文件和工具包含在內。
優點包括：
1. **減少 Image 大小**：只包含最終需要的文件和依賴，避免了不必要的臨時文件。
2. **提高安全性**：減少了 Image 中包含的工具和依賴，降低了潛在的攻擊面。
3. **簡化構建過程**：可以在不同階段使用不同的 base image，靈活性更高。
用途包括：
- 構建複雜的應用程序，將編譯和運行環境分離。
- 減少部署時的 Image 大小，提升傳輸效率。

## 為什麼 Docker Hub 上的官方 Image 會提供多種型號（Tags）？請解釋這些型號之間的關聯和選擇的理由。
Docker Hub 上的官方 Image 提供多種型號（Tags），是為了滿足不同用戶的需求和使用場景。這些型號之間的關聯和選擇理由包括：
1. **版本控制**：不同的 Tags 通常對應於不同的軟體版本或發行版，允許用戶選擇特定版本的 Image 以確保兼容性和穩定性。
2. **環境配置**：某些 Tags 可能針對特定的環境或配置進行優化，例如針對特定的操作系統、架構或性能需求。
3. **功能差異**：不同的 Tags 可能包含不同的功能或特性，例如某些 Tags 可能包含額外的工具或庫，而其他 Tags 則是精簡版。
4. **實驗性和穩定性**：某些 Tags 可能標記為實驗性（如 `latest` 或 `beta`），而其他 Tags 則是穩定版本（如 `1.0`, `2.0` 等），用戶可以根據自己的需求選擇使用。
這樣的設計使得用戶可以靈活地選擇最適合自己需求的 Image 版本，並且能夠在不同的開發和生產環境中保持一致性。  
以 vLLM 的官方 Image 為例，其對應的 Tag 即 vLLM 的版本號，這樣用戶可以選擇特定版本的 vLLM 進行部署和使用。
![alt text](image.png)
