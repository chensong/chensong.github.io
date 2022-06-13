---
layout: post
title: docker 多阶段构建
categories: [Blog, Docker]
description: docker 多阶段构建
keywords: Dockerfile, From
---

## 1. Docker 多阶段构建

### 1.1 多阶段构建 npm 工程

```Dockerfile
FROM node:8 AS build-stage
COPY . /code
WORKDIR /code
# 代码构建
RUN npm install -g cnpm --registry=https://registry.npm.taobao.org
RUN cnpm install
RUN npm run build

FROM nginx:1.21.4-alpine
ENV TZ="Asia/Shanghai"

COPY docker/conf.d/* /etc/nginx/conf.d/
COPY docker/templates/* /etc/nginx/templates/
COPY docker/docker-entrypoint.d/* /docker-entrypoint.d/
RUN chmod +x /docker-entrypoint.d/*

# 复制 build-stage 中的文件
COPY --from=build-stage /code/dist /usr/share/nginx/html/
```
