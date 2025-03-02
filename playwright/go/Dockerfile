FROM golang:1.24

# 设置时区为上海
ENV TZ=Asia/Shanghai
RUN apt-get update && apt-get install -y --no-install-recommends \
    tzdata ca-certificates \
    # Playwright 依赖
    chromium \
    fonts-freefont-ttf \
    libfreetype6 \
    libharfbuzz0b \
    libnss3 \
    wget \
    && rm -rf /var/lib/apt/lists/*

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /app

# 设置环境变量，只使用 Chromium
ENV PLAYWRIGHT_BROWSERS_PATH=/ms-playwright
ENV PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=/usr/bin/chromium

# 安装playwright
RUN go install github.com/playwright-community/playwright-go/cmd/playwright@latest && \
    # 只安装 Chromium 的依赖
    playwright install-deps chromium