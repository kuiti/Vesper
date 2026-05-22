# 夕语（Vesper）

> 一个隐私优先的本地 AI 桌面伙伴 — 长期记忆 + 情感 + 人设 + 陪伴 + 功能性 + 温度感

---

## 特性

- **智能聊天**：流式输出，上下文记忆，支持多种 AI 模型
- **情感系统**：AI 情感追踪、好感度/信任度、9 种情绪状态、性格自主演化
- **主动交互**：多条件智能触发（情绪关怀、活跃问候、提醒关联）
- **天气关怀**：每天三次天气推送
- **生活工具**：待办清单、7 级提醒、笔记、倒计时
- **4 个主题**：暗色、亮色、樱花粉、夕语星光
- **系统托盘**：最小化到托盘，后台运行

---

## 快速开始

### EXE 运行（推荐）
从 [Releases](../../releases) 下载最新 `Vesper_v*_windows_x64.zip`，解压双击 `Vesper.exe`。

### 源码运行
```bash
git clone https://github.com/kuiti/Vesper-XiYu.git
cd Vesper-XiYu
python -m venv venv
pip install -r requirements.txt
python launcher.py
```

---

## 技术栈

- **后端**：Python 3.10 + FastAPI + SQLite + ChromaDB
- **前端**：Vue 3 + Vite
- **桌面**：pywebview (WebView2)
- **AI**：OpenAI 兼容 API（DeepSeek 等）

---

## 许可证

MIT License
