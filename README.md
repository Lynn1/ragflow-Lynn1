# ragflow-lynn1

########### 欢迎使用知识库 ##############

  This is a branch forked from "[infiniflow/ragflow](https://github.com/infiniflow/ragflow)",

  where Lynn1 made some optimizations and customizations.

########################################

## 📝 Prerequisites

- CPU >= 4 cores
- RAM >= 16 GB
- Disk >= 50 GB
- Docker >= 24.0.0 & Docker Compose >= v2.26.1  [Install Docker Engine](https://docs.docker.com/engine/install/).
- Local LLMs Server [Install Ollama on GPUs server](https://github.com/ollama/ollama/blob/main/docs/linux.md)
- Requirements(Ubuntu22.04 x86_64):

  ```bash
  # Ensure `vm.max_map_count` >= 262144:
  sysctl vm.max_map_count
  sudo sysctl -w vm.max_map_count=262144
  ## Permanent set change in `/etc/sysctl.conf`
  vm.max_map_count=262144
  ## if you came up with error:`System limit for number of file watchers reached`:
  sudo sysctl -w fs.inotify.max_user_watches=52428800 

  # Install system libs:
  su
  apt update
  apt install -y ca-certificates 
  apt update
  apt install -y libglib2.0-0 libglx-mesa0 libgl1
  apt install -y pkg-config libicu-dev libgdiplus
  apt install -y default-jdk
  apt install -y libatk-bridge2.0-0
  apt install -y libpython3-dev libgtk-4-1 libnss3 xdg-utils libgbm-dev
  apt install -y python3-pip pipx nginx unzip curl wget git vim less
  # Install msodbcsql17
  curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
  curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
  apt update
  apt install -y unixodbc-dev msodbcsql17

  ## Install npm
  sudo apt purge -y nodejs npm # clean old
  sudo apt autoremove
  curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
  sudo apt update
  sudo apt install -y nodejs cargo

  ## Install poetry 
  curl -sSL https://install.python-poetry.org | python3 -

  #### Set env in `~/.bashrc`
  export PATH="~/.local/bin:$PATH"  
  export HF_ENDPOINT=https://hf-mirror.com
  export TIKA_SERVER_JAR="file:///tika-server-standard-3.0.0.jar"

  ### Install python deps with `poetry`:
  git clone https://github.com/Lynn1/ragflow-lynn1
  cd ragflow-lynn1/
  export POETRY_VIRTUALENVS_CREATE=true POETRY_VIRTUALENVS_IN_PROJECT=true
  poetry install --no-root --with=full

  #### Download full deps:
  cd ragflow-lynn1/
  python3 ./download_deps.py
  mkdir -p rag/res/deepdoc ~/.ragflow
  #### openssl
  sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
  #### tika-server
  cp tika-server-standard-3.0.0.jar $TIKA_SERVER_JAR
  #### nltk
  cp -r nltk_data ~/nltk_data
  #### tiktoken
  cp cl100k_base.tiktoken ./9b5ad71b2ce5302211f9c61530b329a4922fc6a4
  #### deepdoc
  tar --exclude='.*' -cf - huggingface.co/InfiniFlow/text_concat_xgb_v1.0 huggingface.co/InfiniFlow/deepdoc | tar -xf - --strip-components=3 -C rag/res/deepdoc
  #### rerank models
  tar -cf - huggingface.co/BAAI/bge-large-zh-v1.5 huggingface.co/BAAI/bge-reranker-v2-m3 huggingface.co/maidalun1020/bce-embedding-base_v1 huggingface.co/maidalun1020/bce-reranker-base_v1 | tar -xf - --strip-components=2 -C ~/.ragflow

  ### Install web deps with `npm`:
  cd ragflow-lynn1/web 
  npm install --force
  ```
- Check the configuration files, ensuring that:

  - The settings in **docker/.env** match those in **conf/service_conf.yaml**.
  - The IP addresses and ports for related services in **service_conf.yaml** match the local machine IP and ports exposed by the container.
  - Add the following line to `/etc/hosts` to resolve all hosts specified in **docker/.env** to `127.0.0.1`:
    ```
    127.0.0.1       es01 infinity mysql minio redis
    x.x.x.x(your ollama server ip)         localllm
    ```

## 🛠️ Launch service from source for development

To launch the service from source:

```bash
cd ragflow-lynn1
# Launch the database services (MinIO, Elasticsearch, Redis, and MySQL):
docker compose -f docker/docker-compose-base.yml up -d

# Launch backend service: 
poetry shell
export PYTHONPATH=./ 
python api/ragflow_server.py
python rag/svr/task_executor.py

# Launch frontend service: (# Update proxy.target to http://127.0.0.1:9380 in `.umirc.ts`)
cd web
npm run dev 
```

To stop services and clean all test data:

```bash
# stop docker:
docker compose -f docker/docker-compose-base.yml down

# stop docker and delete all data:
docker compose -f docker/docker-compose-base.yml down -v

# check if api/ragflow_server.py exit (kill it manually if necessary)
netstat -tlnp | grep 9380
# check if web exit
netstat -tlnp | grep 9222

```
