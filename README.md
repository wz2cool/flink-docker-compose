# Flink Docker Compose

这是一个使用 Docker Compose 部署 Apache Flink 集群的完整解决方案。

## 架构

- **JobManager**: Flink 主节点，负责协调分布式执行
- **TaskManager**: Flink 工作节点，负责执行实际的数据处理任务

### 版本信息

- Flink 版本: 1.20.0
- Scala 版本: 2.12
- Java 版本: Java 11

## 快速开始

### 1. 启动集群

```bash
docker-compose up -d
```

### 2. 查看状态

```bash
docker-compose ps
```

### 3. 访问 Flink Web UI

打开浏览器访问: `http://localhost:8081`

### 4. 查看日志

```bash
# 查看所有日志
docker-compose logs

# 查看 JobManager 日志
docker-compose logs jobmanager

# 查看 TaskManager 日志
docker-compose logs taskmanager
```

### 5. 停止集群

```bash
docker-compose down
```

### 6. 停止并删除所有数据

```bash
docker-compose down -v
```

## 提交作业

### 准备作业文件

将你的 Flink 作业 JAR 文件放到 `jobs/` 目录下。

### 通过 Web UI 提交

1. 访问 `http://localhost:8081`
2. 点击 "Submit New Job"
3. 上传 `jobs/` 目录下的 JAR 文件
4. 配置作业参数并提交

### 通过命令行提交

```bash
# 进入 JobManager 容器
docker exec -it flink-jobmanager bash

# 提交作业
flink run -d /opt/flink/jobs/your-job.jar
```

## 配置说明

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `jobmanager.rpc.address` | jobmanager | JobManager RPC 地址 |
| `jobmanager.memory.process.size` | 1600m | JobManager 内存大小 |
| `taskmanager.memory.process.size` | 1728m | TaskManager 内存大小 |
| `taskmanager.numberOfTaskSlots` | 2 | 每个 TaskManager 的 Slot 数量 |
| `parallelism.default` | 2 | 默认并行度 |

### 端口

| 端口 | 说明 |
|------|------|
| 8081 | Flink Web UI 和 REST API |

### 数据卷

| 卷名 | 用途 |
|------|------|
| flink-checkpoints | 存储 Checkpoint 数据 |
| flink-savepoints | 存储 Savepoint 数据 |

## 扩展 TaskManager

如需添加更多 TaskManager，在 docker-compose.yml 中添加更多 taskmanager 服务：

```yaml
taskmanager2:
  image: flink:1.18.0-scala_2.12-java11
  container_name: flink-taskmanager2
  depends_on:
    - jobmanager
  command: taskmanager
  environment:
    - |
      FLINK_PROPERTIES=
      jobmanager.rpc.address: jobmanager
      taskmanager.memory.process.size: 1728m
      taskmanager.numberOfTaskSlots: 2
  volumes:
    - flink-checkpoints:/opt/flink/checkpoints
    - flink-savepoints:/opt/flink/savepoints
    - ./jobs:/opt/flink/jobs
  networks:
    - flink-network
```

然后重新启动：

```bash
docker-compose up -d
```

## 故障排查

### TaskManager 无法连接到 JobManager

检查网络配置，确保容器在同一个网络中。

### 作业提交失败

1. 检查 Flink Web UI 确认集群状态正常
2. 查看日志: `docker-compose logs`
3. 确认作业 JAR 文件与 Flink 版本兼容

### 内存不足

在 docker-compose.yml 中调整 `jobmanager.memory.process.size` 和 `taskmanager.memory.process.size`。

## 参考资料

- [Apache Flink 官方文档](https://flink.apache.org/)
- [Flink Docker Hub](https://hub.docker.com/_/flink)
