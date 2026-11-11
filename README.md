# DSH MCP 市场（自建聚合货架）

把自己挑的连接器来源合并成一份 `catalog.json`，用来充实 DSH 连接器货架。
装什么仍然按需点——这里只管"货架上有货"。

## 目录结构（仓库为扁平布局；本地工作区可自由分层）

```
build_catalog.py             # 聚合器：远程底座 + 本地精选 + 官方Registry 转换
custom/                      # 手工精选条目（*.json，模板文件会被自动跳过）
  example-entry.template.json
catalog.json                 # 产物：托管的是这个文件
README.md
```

## 生成货架

```bash
# 底座（duhu2000 社区市场）+ 本地精选 + 官方 Registry 关键词扩充
py -X utf8 build_catalog.py \
  --remote https://cdn.jsdelivr.net/gh/duhu2000/dsh-mcp-connector-registry@main/catalog.json \
  --custom custom \
  --official-search "tencent" \
  --official-limit 20 \
  --out catalog.json
```

- 合并规则：按 `id` 去重，**后面的来源覆盖前面的**（想覆盖默认市场的某条描述，在 `custom/` 里放同 id 文件即可）
- 校验：缺字段、重复 id、疑似密钥都会在构建时报告；**条目里禁止放任何真实密钥**
- `published: false` 的条目保留在目录里但不出现在货架

## 托管（三选一）

| 方式 | 做法 |
| --- | --- |
| GitHub + jsDelivr（推荐） | 把本目录推到 GitHub 仓库，货架地址即 `https://cdn.jsdelivr.net/gh/<user>/<repo>@main/catalog.json` |
| EdgeOne Pages / COS 静态托管 | 上传 `dist/catalog.json`，用分配的 HTTPS 域名 |
| 内网 nginx | 放到内网 HTTPS 站点；明文 HTTP 或私有 IP 需在 DSH 中显式放行 |

## 切换 DSH 货架地址

编辑 `~\.dsh\profiles\desktop\cordis.patch.yml`，追加（已存在 id 时合并进该条）：

```yaml
- id: mcp-connector
  config:
    catalogUrl: 'https://<你的托管地址>/catalog.json'
```

然后**重载 DSH（或重启）**，会话里执行一次「刷新连接器目录」强刷缓存即可看到新货架。

回滚：删掉上面这段 YAML 再重载，即恢复默认 116 条货架。

## 更新节奏

- 上游市场有新货：重跑 `build_catalog.py` → 推送仓库（可配 GitHub Actions 定时）
- 货架缓存默认 5 分钟（refreshSkewMs），着急可在会话里强刷
