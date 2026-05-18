# 外部定时器触发 GitHub Actions

本仓库的每日分析工作流已改为通过外部云端定时器触发，避免 GitHub Actions 自带 `schedule` 在高负载时延迟或丢触发。

电脑关机不影响执行。只要外部定时器服务在云端运行，它会按时调用 GitHub API，GitHub Actions 也会在云端完成分析和报告推送。

## 1. 创建 GitHub Token

推荐创建 Fine-grained personal access token：

- Repository access: 只选择 `LittleWhaleRx/daily_stock_analysis`
- Repository permissions: `Contents` 设为 `Read and write`

如果使用 classic token，私有仓库需要 `repo` 权限。

不要把 token 写进代码或公开页面，只填到外部定时器服务的请求头里。

## 2. 配置外部定时器

可以使用 cron-job.org、EasyCron、Cloudflare Workers Cron、腾讯云函数、阿里云函数等云端定时器。

定时时间：

- 如果服务支持时区：`Asia/Shanghai`，周一到周五 `08:15`
- 如果服务只支持 UTC：周一到周五 `00:15`

HTTP 请求配置：

- Method: `POST`
- URL: `https://api.github.com/repos/LittleWhaleRx/daily_stock_analysis/dispatches`
- Headers:
  - `Accept: application/vnd.github+json`
  - `Authorization: Bearer <YOUR_GITHUB_TOKEN>`
  - `X-GitHub-Api-Version: 2022-11-28`
- Body:

```json
{
  "event_type": "daily_stock_analysis",
  "client_payload": {
    "mode": "full",
    "force_run": "false"
  }
}
```

## 3. 可选参数

`mode` 可选：

- `full`: 股票分析和大盘复盘
- `market-only`: 仅大盘复盘
- `stocks-only`: 仅股票分析

`force_run` 可选：

- `false`: 遵守交易日检查
- `true`: 跳过交易日检查，非交易日也运行

## 4. 验证

外部定时器触发成功后，GitHub 仓库的 `Actions -> 每日股票分析` 里会出现一条由 `repository_dispatch` 触发的运行记录。
