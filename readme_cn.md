# Sunlight CT 环境说明

[English](readme.md) | 简体中文

本文根据提供的测试、正式环境 `sunlight.yaml` 和部署描述整理，用于记录现有部署。配置文件已按环境分别保存；本文未对远程服务进行运行验证。

服务器 IP 已替换为文档示例地址（`192.0.2.10` 和 `2001:db8::10`），实际部署时需替换为真实的本地监听地址。

配置中的 S3 endpoint 域名已脱敏为 `https://s3-endpoint.example.invalid`，实际部署时需替换为真实 endpoint。

## 1. 项目与部署概况

- 项目：[FiloSottile/sunlight](https://github.com/FiloSottile/sunlight)
- 版本：根据提供的服务器目录记录为 `sunlight-0.9.0`，未核验运行中的二进制版本。
- 正式和测试环境区域：均为新加坡。
- 正式环境 Sunlight 服务器：24 核 CPU、64 GB 内存。
- 测试环境 Sunlight 服务器：16 核 CPU、32 GB 内存。
- 测试与正式环境采用相同架构：WAF → Nginx → Sunlight，CDN → OSS。
- 本地存储：SQLite checkpoint 数据库、各日志独立的 SQLite 去重缓存、各日志的密钥种子文件。
- 日志数据存储：阿里云 OSS，通过 S3 兼容接口写入。
- 提交链路：证书提交方 → 云 WAF → Nginx → Sunlight。
- 读取链路：监控客户端 → 云 CDN → OSS；CDN 缓存命中时直接响应。

测试与正式环境的架构相同。Nginx 是否与 Sunlight 同机、各跳 TLS 终止位置尚未确认。

## 2. 共用架构图

正式与测试环境采用相同架构，均部署在新加坡。下图统一展示两套环境的拓扑，每套环境分别使用对应的服务器、日志与配置。

- 正式环境 Sunlight 服务器：**24 核 CPU / 64 GB 内存**。
- 测试环境 Sunlight 服务器：**16 核 CPU / 32 GB 内存**。

图中不显示 IP 地址和域名。箭头表示请求或写入方向，响应沿原路返回。

![Sunlight CT Common Architecture — Production and Test](docs/images/sunlight-common-architecture.png)

## 3. 日志分片配置

| 分片 | 测试日志 / Bucket | 正式日志 / Bucket | `notafterstart` | `notafterlimit` |
| --- | --- | --- | --- | --- |
| 2026h2 | `ct-test-2026h2` | `ct-2026h2` | `2026-07-01T00:00:00Z` | `2027-01-01T00:00:00Z` |
| 2027h1 | `ct-test-2027h1` | `ct-2027h1` | `2027-01-01T00:00:00Z` | `2027-07-01T00:00:00Z` |
| 2027h2 | `ct-test-2027h2` | `ct-2027h2` | `2027-07-01T00:00:00Z` | `2028-01-01T00:00:00Z` |
| 2028h1 | `ct-test-2028h1` | `ct-2028h1` | `2028-01-01T00:00:00Z` | `2028-07-01T00:00:00Z` |
| 2028h2 | `ct-test-2028h2` | `ct-2028h2` | `2028-07-01T00:00:00Z` | `2029-01-01T00:00:00Z` |

## 4. 测试环境完整配置

独立文件：[sunlight-testing.yaml](config/sunlight-testing.yaml)。

```yaml
listen:
  - "192.0.2.10:8080"
    #  - "[2001:db8::10]:8080"
    #acme:
    #hosts:
    #- ct-test.sunlight.sheca.com
    #cache: /var/db/sunlight/autocert-staging/
checkpoints: /tank/shared/checkpoints.db
logs:
  - shortname: ct-test-2026h2
    inception: 2026-09-01
    period: 200
    submissionprefix: https://ct-test-2026h2-sunlight.sheca.com
    monitoringprefix: https://ct-test-2026h2-bucket.sheca.com
    ccadbroots: testing
    extraroots: /etc/sunlight/extra-roots-staging.pem
    secret: /tank/enc/sheca2026h2.seed.bin
    cache: /tank/caches/ct-test-2026h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-test-2026h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2026-07-01T00:00:00Z
    notafterlimit: 2027-01-01T00:00:00Z

  - shortname: ct-test-2027h1
    inception: 2026-09-01
    period: 200
    submissionprefix: https://ct-test-2027h1-sunlight.sheca.com
    monitoringprefix: https://ct-test-2027h1-bucket.sheca.com
    ccadbroots: testing
    extraroots: /etc/sunlight/extra-roots-staging.pem
    secret: /tank/enc/sheca2027h1.seed.bin
    cache: /tank/caches/ct-test-2027h1/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-test-2027h1
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2027-01-01T00:00:00Z
    notafterlimit: 2027-07-01T00:00:00Z

  - shortname: ct-test-2027h2
    inception: 2026-09-01
    period: 200
    submissionprefix: https://ct-test-2027h2-sunlight.sheca.com
    monitoringprefix: https://ct-test-2027h2-bucket.sheca.com
    ccadbroots: testing
    extraroots: /etc/sunlight/extra-roots-staging.pem
    secret: /tank/enc/sheca2027h2.seed.bin
    cache: /tank/caches/ct-test-2027h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-test-2027h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2027-07-01T00:00:00Z
    notafterlimit: 2028-01-01T00:00:00Z

  - shortname: ct-test-2028h1
    inception: 2026-09-01
    period: 200
    submissionprefix: https://ct-test-2028h1-sunlight.sheca.com
    monitoringprefix: https://ct-test-2028h1-bucket.sheca.com
    ccadbroots: testing
    extraroots: /etc/sunlight/extra-roots-staging.pem
    secret: /tank/enc/sheca2028h1.seed.bin
    cache: /tank/caches/ct-test-2028h1/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-test-2028h1
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2028-01-01T00:00:00Z
    notafterlimit: 2028-07-01T00:00:00Z

  - shortname: ct-test-2028h2
    inception: 2026-09-01
    period: 200
    submissionprefix: https://ct-test-2028h2-sunlight.sheca.com
    monitoringprefix: https://ct-test-2028h2-bucket.sheca.com
    ccadbroots: testing
    extraroots: /etc/sunlight/extra-roots-staging.pem
    secret: /tank/enc/sheca2028h2.seed.bin
    cache: /tank/caches/ct-test-2028h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-test-2028h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2028-07-01T00:00:00Z
    notafterlimit: 2029-01-01T00:00:00Z
```

## 5. 正式环境完整配置

独立文件：[sunlight-production.yaml](config/sunlight-production.yaml)。

```yaml
listen:
  - "192.0.2.10:8080"
    #  - "[2001:db8::10]:8080"
    #acme:
    #hosts:
    #- ct-test.sunlight.sheca.com
    #cache: /var/db/sunlight/autocert-staging/
checkpoints: /tank/shared/checkpoints.db
logs:
  - shortname: ct-2026h2
    inception: 2026-09-15
    period: 200
    submissionprefix: https://ct-2026h2-sunlight.sheca.com
    monitoringprefix: https://ct-2026h2-bucket.sheca.com
    ccadbroots: trusted
    secret: /tank/enc/sheca2026h2.seed.bin
    cache: /tank/caches/ct-2026h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-2026h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2026-07-01T00:00:00Z
    notafterlimit: 2027-01-01T00:00:00Z

  - shortname: ct-2027h1
    inception: 2026-09-15
    period: 200
    submissionprefix: https://ct-2027h1-sunlight.sheca.com
    monitoringprefix: https://ct-2027h1-bucket.sheca.com
    ccadbroots: trusted
    secret: /tank/enc/sheca2027h1.seed.bin
    cache: /tank/caches/ct-2027h1/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-2027h1
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2027-01-01T00:00:00Z
    notafterlimit: 2027-07-01T00:00:00Z

  - shortname: ct-2027h2
    inception: 2026-09-15
    period: 200
    submissionprefix: https://ct-2027h2-sunlight.sheca.com
    monitoringprefix: https://ct-2027h2-bucket.sheca.com
    ccadbroots: trusted
    secret: /tank/enc/sheca2027h2.seed.bin
    cache: /tank/caches/ct-2027h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-2027h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2027-07-01T00:00:00Z
    notafterlimit: 2028-01-01T00:00:00Z

  - shortname: ct-2028h1
    inception: 2026-09-15
    period: 200
    submissionprefix: https://ct-2028h1-sunlight.sheca.com
    monitoringprefix: https://ct-2028h1-bucket.sheca.com
    ccadbroots: trusted
    secret: /tank/enc/sheca2028h1.seed.bin
    cache: /tank/caches/ct-2028h1/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-2028h1
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2028-01-01T00:00:00Z
    notafterlimit: 2028-07-01T00:00:00Z

  - shortname: ct-2028h2
    inception: 2026-09-15
    period: 200
    submissionprefix: https://ct-2028h2-sunlight.sheca.com
    monitoringprefix: https://ct-2028h2-bucket.sheca.com
    ccadbroots: trusted
    secret: /tank/enc/sheca2028h2.seed.bin
    cache: /tank/caches/ct-2028h2/cache.db
    poolsize: 750
    s3region: ap-southeast-1
    s3bucket: ct-2028h2
    s3endpoint: https://s3-endpoint.example.invalid
    notafterstart: 2028-07-01T00:00:00Z
    notafterlimit: 2029-01-01T00:00:00Z
```
