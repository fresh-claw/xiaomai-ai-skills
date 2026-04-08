# Doubao Web 2 API

作者：**小马AI**

把已登录的豆包网页版变成统一的命令接口，方便本机 Agent 直接发起对话、创建新会话、读取最新回复。网页版不可用时，可回退到豆包桌面版。

适合：
- 已经在本机登录豆包
- 想让能执行本地命令的 Agent 直接调用豆包
- 需要同时兼容 `Windows` 和 `macOS`

不适合：
- 高并发批量任务
- 强依赖稳定接口的生产链路
- 自动绕过验证码、风控或登录限制

## 能做什么

- 检查豆包是否已登录并可调用
- 新建会话
- 发送问题并读取完整回复
- 在网页版和桌面版之间自动切换
- 统一输出 JSON，方便继续封装

## 当前支持

- `login-check`
- `new`
- `ask`
- `read`
- `reset`

第一版只做文本问答，不包含文件上传、图片理解和复杂会话管理。

## 运行前准备

必须满足这 4 个条件：

1. 已安装 `opencli`
2. 已接通 `opencli doubao` 或 `opencli doubao-app`
3. 豆包已手动登录
4. 本机可运行 Python

详细安装步骤见：

- [安装与接通](skills/doubao-web-to-api/references/setup.md)

## 快速开始

### macOS

```bash
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py login-check
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py new
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "帮我总结这段内容"
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py read
```

### Windows

```powershell
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py login-check
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py new
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "帮我总结这段内容"
python skills/doubao-web-to-api/scripts/doubao_web_to_api.py read
```

## 参数说明

### `--adapter`

- `auto`：先试网页版，再试桌面版
- `web`：只用 `opencli doubao`
- `app`：只用 `opencli doubao-app`

默认值：`auto`

### `--timeout`

等待超时时间，默认 `180` 秒。  
如果问题长、回复慢，可以调大。

示例：

```bash
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "分析这段文案" --adapter web --timeout 300
```

## 返回结果

所有命令统一返回 JSON。

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

## 工作方式

默认会先尝试豆包网页版；如果失败，再尝试豆包桌面版。  
最近一次成功的适配器会被记录下来，后续优先复用。

状态文件位置：

- `~/.doubao-web-to-api/state.json`

## 常见问题

### 1. 找不到 `opencli`

先执行：

```bash
opencli list
```

如果失败，说明 `opencli` 没装好，或者没有加入 PATH。

### 2. `login-check` 失败

优先检查：
- 豆包是否已登录
- OpenCLI 扩展是否连上
- 当前浏览器或桌面版是否就是 OpenCLI 使用的目标

### 3. `ask` 超时

把超时调大：

```bash
python3 skills/doubao-web-to-api/scripts/doubao_web_to_api.py ask "问题" --timeout 300
```

### 4. 出现验证码或风控

不要继续自动化，直接人工处理。

## 仓库内容

```text
doubao-web-to-api/
├── README.md
└── skills/
    ├── catalog.json
    └── doubao-web-to-api/
        ├── SKILL.md
        ├── scripts/
        │   └── doubao_web_to_api.py
        └── references/
            ├── setup.md
            └── maintenance.md
```

## 相关文件

- [SKILL.md](skills/doubao-web-to-api/SKILL.md)
- [安装与接通](skills/doubao-web-to-api/references/setup.md)
- [维护说明](skills/doubao-web-to-api/references/maintenance.md)
- [Skill 清单](skills/catalog.json)

## 后续新增 Skill

后续会继续按同样方式管理新的 skill。  
每个 skill 单独放在 `skills/<skill-name>/`，并统一写安装、使用、维护文档。
