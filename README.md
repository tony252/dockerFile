# Docker Image CI/CD Pipeline

这个项目提供了一个高效的 CI/CD 流水线，用于构建和推送 Docker 镜像到多个容器镜像仓库。

## 功能特点

- **集中配置**: 所有镜像配置集中在一个 JSON 文件中管理
- **分离构建与推送**: 构建和推送过程分离，提高效率和可靠性
- **并行推送**: 同时推送到多个容器镜像仓库（DockerHub 和阿里云）
- **自动化触发**: 支持多种触发方式（代码推送、PR、手动触发）
- **缓存优化**: 利用 GitHub Actions 缓存加速构建过程

## 工作流程

该项目包含两个主要工作流:

1. **构建工作流** (.github/workflows/docker-build.yml)
   - 根据配置文件构建 Docker 镜像
   - 将构建好的镜像保存为构件

2. **推送工作流** (.github/workflows/docker-push.yml)
   - 在构建成功后自动触发
   - 并行推送镜像到 DockerHub 和阿里云容器镜像服务

## 配置说明

镜像配置在 `.github/docker-images.json` 文件中定义:

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

### 手动触发构建

1. 在 GitHub 仓库页面，点击 "Actions" 标签
2. 选择 "Build Docker Images" 工作流
3. 点击 "Run workflow" 按钮

### 手动触发推送

1. 在 GitHub 仓库页面，点击 "Actions" 标签
2. 选择 "Push Docker Images" 工作流
3. 点击 "Run workflow" 按钮

## 注意事项

- 推送工作流依赖于构建工作流生成的构件
- 构件保留期为 1 天
- 大型镜像可能受到 GitHub 构件大小限制（通常为 2GB）
