# classdrop · 班级文件中转站

网页地址（开启 Pages 后）：https://hetao-127.github.io/classdrop/

文件存在本仓库的 Release（tag `classdrop`）里，所以任何能调 GitHub API 的工具都能读写。

## 1. 列出文件（不需要凭证）

```
GET https://api.github.com/repos/Hetao-127/classdrop/releases/tags/classdrop
```

返回 `id`（Release 编号）和 `assets[]`（每个文件有 name / size / created_at / browser_download_url）。
返回 404 说明还没有 Release，网页上传第一个文件时会自动创建。

## 2. 下载文件（不需要凭证）

直接 GET `browser_download_url`，形如 `https://github.com/Hetao-127/classdrop/releases/download/classdrop/xxx.pdf`。
公开仓库不要求登录，Python 的 urllib 能直接拿到文件字节。

## 3. 上传文件（需要 PAT）

```
POST https://uploads.github.com/repos/Hetao-127/classdrop/releases/{release_id}/assets?name=文件名
Authorization: Bearer 你的PAT
Content-Type: application/octet-stream

（请求体就是文件原始二进制，不需要 base64）
```

Python 示例：

```python
import urllib.request, json
PAT = "替换成你的PAT"
REL = "https://api.github.com/repos/Hetao-127/classdrop/releases/tags/classdrop"
rel = json.load(urllib.request.urlopen(REL))
url = "https://uploads.github.com/repos/Hetao-127/classdrop/releases/%d/assets?name=test.pdf" % rel["id"]
req = urllib.request.Request(url, data=open("test.pdf", "rb").read(),
    headers={"Authorization": "Bearer " + PAT,
             "Content-Type": "application/octet-stream"}, method="POST")
print(urllib.request.urlopen(req).status)
```

## 4. 删除单个文件（需要 PAT）

```
DELETE https://api.github.com/repos/Hetao-127/classdrop/releases/assets/{asset_id}
```

## 5. 自动清理

`.github/workflows/cleanup.yml` 每天 UTC 19:00（北京时间 03:00）删除超过保留期的文件，天数在文件里的 `KEEP_DAYS`。

## 限制

- 仓库必须保持 Public，改成私有后下载要登录、页面链接失效。
- 单文件越大上传越慢，网络差时容易断，建议先压缩。
- 仓库连续 60 天没有提交，GitHub 会自动暂停定时任务，到 Actions 页面点一次 Run workflow 即可恢复。
