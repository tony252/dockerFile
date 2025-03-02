# Docker Image CI/CD Pipeline

这个项目提供了一个高效的 CI/CD 流水线，用于构建和推送 Docker 镜像到多个容器镜像仓库。

## 功能特点

- **集中配置**: 所有镜像配置集中在一个 JSON 文件中管理
- **一体化流程**: 在同一个工作流中完成构建和推送，简化配置和操作
- **并行推送**: 同时推送到多个容器镜像仓库（DockerHub 和阿里云）
- **自动化触发**: 支持多种触发方式（代码推送、PR、手动触发）
- **构建验证**: 自动检查 Dockerfile 存在性和构建结果

## 工作流程

该项目包含一个主要工作流 `.github/workflows/docker-build.yml`，它执行以下任务:

1. **准备阶段** (prepare)
   - 检出代码
   - 从配置文件生成构建矩阵

2. **构建阶段** (build)
   - 检查 Dockerfile 是否存在
   - 构建 Docker 镜像
   - 验证构建结果
   - 将镜像保存为构件

3. **DockerHub 推送阶段** (push-to-dockerhub)
   - 下载构建好的镜像构件
   - 登录到 DockerHub
   - 推送镜像到 DockerHub

4. **阿里云推送阶段** (push-to-aliyun)
   - 下载构建好的镜像构件
   - 登录到阿里云容器镜像服务
   - 推送镜像到阿里云容器镜像服务

## 配置说明

镜像配置在 `.github/docker-images.json` 文件中定义，格式如下:

```json
{
  "images": [
    {
      "name": "镜像名称",
      "tag": "镜像标签",
      "dockerfile": "Dockerfile路径"
    }
  ]
}
```

例如:
```json
{
  "images": [
    {
      "name": "playwright-go",
      "tag": "1.24",
      "dockerfile": "playwright/go_1.24/Dockerfile"
    },
    {
      "name": "maven3.5-jdk8",
      "tag": "latest",
      "dockerfile": "maven/Dockerfile"
    }
  ]
}
```

## 必要的 Secrets

在 GitHub 仓库设置中需要配置以下 Secrets:

- `DOCKERHUB_USERNAME`: DockerHub 用户名
- `DOCKERHUB_PASSWORD`: DockerHub 密码或访问令牌
- `ALIYUN_USERNAME`: 阿里云容器镜像服务用户名
- `ALIYUN_PASSWORD`: 阿里云容器镜像服务密码或访问令牌
- `ALIYUN_NAMESPACE`: 阿里云容器镜像服务命名空间

## 使用方法

### 添加新镜像

1. 编辑 `.github/docker-images.json` 文件
2. 在 `images` 数组中添加新的镜像配置
3. 提交并推送更改

### 手动触发工作流

1. 在 GitHub 仓库页面，点击 "Actions" 标签
2. 选择 "Build and Push Docker Images" 工作流
3. 点击 "Run workflow" 按钮

## 注意事项

- 构件保留期为 1 天
- 大型镜像可能受到 GitHub 构件大小限制（通常为 2GB）
- 确保 Dockerfile 路径正确，工作流会自动检查文件是否存在
- 推送阶段需要正确配置相关的 Secrets
