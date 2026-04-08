---
name: doubao-web-to-api
description: "Use when Codex or another local agent needs to talk to Doubao through OpenCLI on Windows or macOS. Exposes Doubao Web as a JSON-returning command interface with login-check, new, ask, read, and reset, and falls back to doubao-app when needed."
---

# doubao-web-to-api

通过 OpenCLI 把豆包网页版变成统一的命令接口，必要时回退到桌面版，让本机 Agent 可以直接发起文本问答并读取回复。

适合：
- 本机已登录豆包
- 需要兼容 `Windows` 和 `macOS`
- 希望以命令行方式发起问答并读取回复

当前范围：
- `login-check`
- `new`
- `ask`
- `read`
- `reset`

第一版只做文本问答，不处理文件上传、图片理解和复杂会话管理。

## 依赖

必须满足：
- 已安装 `opencli`
- 已接通 `opencli doubao` 或 `opencli doubao-app`
- 豆包已手动登录
- 本机可运行 `python3`（Windows 可用 `python`）

如果还没准备好，先看：
- [setup.md](references/setup.md)

## 命令

### macOS

```bash
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py login-check
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py new
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "帮我总结这段内容"
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py read
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py reset
```

### Windows

```powershell
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py login-check
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py new
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "帮我总结这段内容"
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py read
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py reset
```

## 参数

- `--adapter auto|web|app`
  - `auto`：先试网页版，再试桌面版
  - `web`：只用 `opencli doubao`
  - `app`：只用 `opencli doubao-app`

- `--timeout`
  - 默认 `180` 秒
  - 长回答可以调大

示例：

```bash
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "分析这段文案" --adapter web --timeout 300
```

## 输出

所有命令统一输出 JSON。

成功示例：

```json
{
  "ok": true,
  "adapter": "web",
  "adapter_kind": "web",
  "action": "ask",
  "result": {},
  "answer": "这里是豆包回复",
  "command": ["opencli", "doubao", "ask", "问题", "-f", "json"]
}
```

失败示例：

```json
{
  "ok": false,
  "error": "opencli_not_found",
  "message": "未找到 opencli，请先安装并加入 PATH。",
  "action": "login-check"
}
```

## 调用逻辑

默认 `auto` 模式下：
1. 先试网页版
2. 网页版失败再试桌面版
3. 哪个成功，就记住上一次可用适配器
4. 下次优先复用上一次成功的适配器

状态文件位置：
- `~/.doubao-web-to-api/state.json`

## 什么时候该停下

遇到这些情况，不要继续自动化：
- 豆包没登录
- 出现验证码
- 出现风控页
- 页面结构明显变化
- OpenCLI 适配器失效

这时应直接停止，并提示人工处理。

## 维护入口

后续维护优先看：
- [setup.md](references/setup.md)
- [maintenance.md](references/maintenance.md)
