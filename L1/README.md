# Полная инструкция: Где и как запускать Hadoop в Docker

## 1. **Где работать:**

### Если у вас **Linux/Mac**:
Откройте **Терминал** и выполняйте команды:

### Если у вас **Windows**:
1. Откройте **PowerShell** или **Command Prompt**
2. Или установите **Git Bash**
3. Или используйте **Docker Desktop Terminal**

## 2. **Пошаговая установка:**

### Шаг 1: Откройте терминал и создайте папку
```bash
# Перейдите в домашнюю директорию
cd ~

# Создайте папку для Hadoop
mkdir hadoop-docker
cd hadoop-docker
```

### Шаг 2: Создайте файл `docker-compose.yml`
```bash
# Для Linux/Mac:
nano docker-compose.yml

# Для Windows (в PowerShell):
notepad docker-compose.yml
```

**Содержимое файла:**
```yaml
version: '3.8'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    restart: always
    ports:
      - "9870:9870"    # HDFS Web UI
      - "9000:9000"    # HDFS Service
      - "8020:8020"    # HDFS NameNode
    environment:
      - CLUSTER_NAME=hadoop-cluster
    volumes:
      - namenode_data:/hadoop/dfs/name
    env_file:
      - ./hadoop.env

  datanode1:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode1
    restart: always
    depends_on:
      - namenode
    env_file:
      - ./hadoop.env
    volumes:
      - datanode1_data:/hadoop/dfs/data

  datanode2:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    container_name: datanode2
    restart: always
    depends_on:
      - namenode
    env_file:
      - ./hadoop.env
    volumes:
      - datanode2_data:/hadoop/dfs/data

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    restart: always
    depends_on:
      - namenode
    ports:
      - "8088:8088"    # YARN Web UI
    env_file:
      - ./hadoop.env

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    restart: always
    depends_on:
      - resourcemanager
    env_file:
      - ./hadoop.env

volumes:
  namenode_data:
  datanode1_data:
  datanode2_data:
```

### Шаг 3: Создайте файл `hadoop.env`
```bash
# Создаем файл с настройками
echo 'CORE_CONF_fs_defaultFS=hdfs://namenode:8020
CORE_CONF_hadoop_http_staticuser_user=root
HDFS_CONF_dfs_webhdfs_enabled=true
HDFS_CONF_dfs_permissions_enabled=false
YARN_CONF_yarn_log___aggregation___enable=true
YARN_CONF_yarn_resourcemanager_recovery_enabled=true
YARN_CONF_yarn_resourcemanager_store_class=org.apache.hadoop.yarn.server.resourcemanager.recovery.FileSystemRMStateStore
YARN_CONF_yarn_resourcemanager_fs_state___store_uri=/rmstate
YARN_CONF_yarn_nodemanager_remote___app___log___dir=/app-logs
YARN_CONF_yarn_log_server_url=http://resourcemanager:19888/jobhistory/logs
YARN_CONF_yarn_timeline___service_enabled=true
YARN_CONF_yarn_timeline___service_generic___application___history_enabled=true
YARN_CONF_yarn_resourcemanager_system___metrics___publisher_enabled=true
YARN_CONF_yarn_resourcemanager_hostname=resourcemanager
YARN_CONF_yarn_timeline___service_hostname=historyserver' > hadoop.env
```

## 3. **Запуск кластера:**

```bash
# Запускаем кластер (в той же папке где docker-compose.yml)
docker-compose up -d

# Проверяем что все контейнеры запущены
docker-compose ps
```

## 4. **Где найти веб-интерфейсы:**

После запуска откройте в браузере:

### **HDFS NameNode UI:**
```
http://localhost:9870
```

### **YARN ResourceManager UI:**
```
http://localhost:8088
```

## 5. **Проверка работы:**

```bash
# Зайти внутрь контейнера
docker exec -it namenode bash

# Проверить HDFS
hdfs dfsadmin -report

# Создать тестовую директорию
hdfs dfs -mkdir /test
hdfs dfs -ls /
```

## 6. **Остановка кластера:**

```bash
# Остановить
docker-compose down

# Остановить и удалить данные
docker-compose down -v
```

## 7. **Где смотреть логи:**

```bash
# Логи NameNode
docker logs namenode

# Логи всех сервисов
docker-compose logs
```

