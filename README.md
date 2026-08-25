# AI-MC 服务器智能管理脚本

一个基于 Python 开发的轻量级脚本，通过调用大语言模型（LLM）API，让你可以用**自然语言**控制 Minecraft 服务器（例如：“踢掉那个熊孩子”、“把大家都传送到主城”、“备份世界”）。脚本通过 RCON 协议与服务器通信，安全高效。
两个版本目前，图形化和自己配置
# 环境要求
- **Python 版本**：3.8 或更高版本
- **Minecraft 服务端**：支持 RCON 的 Spigot / Paper / Purpur 等（原版 Vanilla 需安装 RCON 模组）
- **网络环境**：能够访问你配置的 LLM API 地址（如 OpenAI / 国内中转 / 本地 Ollama）

---

# 第一步：开启服务端 RCON 远程控制

脚本通过 RCON 协议发送指令给 MC 服务端，因此必须提前在服务端配置文件中开启此功能。
当然，我的发布版本里也给了可以直接替换方便使用的服务器配置

1. **关闭**你的 MC 服务器（如果正在运行）。
2. 用文本编辑器打开服务端根目录下的 `server.properties` 文件。
3. 找到并修改以下三项内容（如果没有则手动添加在末尾）：

```properties
# 开启 RCON 远程控制
enable-rcon=true

# 设置 RCON 密码（请勿使用弱密码，此密码稍后要填入 setting.py）
rcon.password=你的强密码_例如_Minecraft2026

# 设置 RCON 通信端口（默认 25575，确保该端口未被防火墙拦截）
rcon.port=25575
```
4. **保存**文件并**重启服务器**，使配置生效。
**好啦，到这里图形化的配置就完成了呢，如果你使用dist，享受ai的便利吧！就现在**
---

# 第二步：填写脚本配置文件 (setting.py)

解压项目目录后，找到 `setting.py` 文件，用文本编辑器（推荐 VSCode 或 Notepad++）打开。

**请务必只修改双引号 `""` 内部的内容**，参考如下对照表进行填写：

```python
# ==================== MC 服务器连接配置 ====================
MC_HOST = "127.0.0.1"          # 服务器IP，本地填 127.0.0.1，远程填公网IP
MC_PORT = 25575                # 必须与 server.properties 里的 rcon.port 完全一致
RCON_PASSWORD = "你的强密码_例如_Minecraft2026"  # 必须与 server.properties 里的密码完全一致

# ==================== AI 大模型接口配置 ====================
AI_API_KEY = "sk-xxxxxxxxxxxxxxxx"      # 你的 API 密钥（请严格保密）
AI_BASE_URL = "https://api.openai.com/v1" # API 地址（若使用国内中转或 Ollama，请修改此处）
AI_MODEL = "gpt-4o-mini"                # 模型名称（如 gpt-3.5-turbo, deepseek-chat）

# ==================== 系统提示词与安全设置 ====================
SYSTEM_PROMPT = "你是一位Minecraft服务器管理员助理，只负责执行玩家的合理指令，禁止执行stop或save-off等危险操作。"
ADMIN_WHITELIST = ["玩家名1", "玩家名2"] # 只有在此列表中的玩家名可以触发敏感指令（可留空）
```

>  **特别警告**：千万不要把 `setting.py` 直接上传到公开的 GitHub 仓库！否则你的 RCON 密码和 API 密钥将全网公开。建议在 `.gitignore` 中添加 `setting.py`，并提供 `setting.example.py` 模板供用户复制。

---

# 第三步：安装 Python 依赖库

1. 打开命令行工具（Windows 按 `Win + R` 输入 `cmd`；Mac/Linux 打开终端）。
2. 使用 `cd` 命令切换到你的项目解压目录（例如 `cd C:\Users\你的名字\Desktop\AI-MC`）。
3. 输入以下命令一键安装所有需要的库：

```bash
pip install -r requirements.txt
```

> 如果网络较慢，可以换用国内源：`pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple`

---

# 第四步：运行启动脚本 (llm.py)

依赖安装完成后，就可以启动脚本了。推荐以下几种运行方式（根据你的使用习惯选择）：

### 方式一：前台调试运行（适合测试）
在项目目录下执行：
```bash
python llm.py
```
启动成功后，命令行会显示 `[AI] 已连接至服务器 RCON` 等信息。此时你可以在命令行输入中文指令（如“查询在线人数”），AI 会自动解析并执行。

### 方式二：后台挂载运行（适合 7x24 小时开服）
- **Windows 系统**：将 `python llm.py` 保存为 `.bat` 文件，或使用 `pythonw.exe llm.py`（无黑框）。
- **Linux / Mac 系统**：推荐使用 `screen` 或 `tmux` 保持后台运行：
  ```bash
  screen -S mc_ai
  python llm.py
  # 按 Ctrl + A 然后按 D 切出后台
  ```
  如需重新进入查看日志：`screen -r mc_ai`

---

# 常见问题排查（FAQ）

| 问题现象 | 解决办法 |
| :--- | :--- |
| **连接 RCON 失败 (Connection refused)** | 检查 `server.properties` 中 `enable-rcon=true` 是否开启；检查防火墙是否放行了 `25575` 端口；确认服务器已重启。 |
| **AI 返回乱码或报错 401** | 检查 `AI_API_KEY` 是否复制正确（注意没有多余空格）；检查 `AI_BASE_URL` 是否对应你的模型提供商。 |
| **提示 `ModuleNotFoundError`** | 说明依赖未安装完全，请重新执行 `pip install -r requirements.txt`。 |
| **脚本执行指令无反应** | 检查 `RCON_PASSWORD` 是否包含特殊字符（如 `&`、`#`），如有特殊字符建议在 `setting.py` 中加引号并用转义符 `\`。 |

---

如果还有其他不清楚的地方，欢迎提交 Issue 或直接查看源码中的注释！如果觉得好用，别忘了给个 Star ⭐ 支持一下！
<img width="450" height="450" alt="-9lddQ2u-3h1wX11Za6T3cSci-ci" src="https://github.com/user-attachments/assets/aadb8dd6-85d0-42f3-a517-d2ae0e7ddc44" />
https://drive.google.com/file/d/1hCvF8V1P5AnxZcfA1wx3htdzRC2Bj8dz/preview


