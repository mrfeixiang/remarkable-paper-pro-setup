# 我如何用 Claude 把 reMarkable Paper Pro 打造成终极生产力工具

上周我做了一件疯狂的事：我让 AI 帮我深度改造了 reMarkable Paper Pro。整个过程充满了挑战，但最终的结果让我惊叹不已。

## 起因

reMarkable Paper Pro 是一款出色的电子墨水平板，但开箱即用的功能相对有限。我想要更多——浏览器、终端、电子书阅读器、甚至一个会写回信的 AI 日记本。听起来很疯狂？确实。但有了 Claude，一切都变得可能。

## 改造之旅

我直接把需求告诉 Claude："帮我在 reMarkable Paper Pro 上安装 Riddle。"接下来发生的事情超出了我的预期。

### 挑战一：只读文件系统

reMarkable Paper Pro 的根文件系统是只读的，还有一个 `/etc` 覆盖层，每次重启都会重置。Claude 发现需要先卸载覆盖层（`umount -R /etc`），才能将配置写入真正的根文件系统。这个发现花了不少时间，但最终解决了 xovi 扩展加载器无法持久化的问题。

### 挑战二：版本兼容性噩梦

安装过程中，平板自动从 OS 3.24 更新到了 3.27。所有配置瞬间被清空。更糟的是，AppLoad v0.5.3 不兼容 3.24，而 v0.4.2 不兼容 3.27。Claude 通过解析 ELF 二进制文件的动态符号表，找到了版本不匹配的根本原因，并在 OS 更新后迅速用正确的版本重建了整个环境。

### 挑战三：Qt 资源哈希表

AppLoad 需要一个哈希表来定位 xochitl（reMarkable 的主 UI 进程）中的 QML 组件。这个哈希表需要针对每个 OS 版本单独构建，过程大约需要 2 分钟。Claude 编写了一个后台脚本来自动完成这个过程，避免了 SSH 连接中断导致的失败。

### 挑战四：中韩文支持

安装完所有应用后，我发现中文和韩文文件名显示为方块。Claude 不仅安装了 Noto Sans CJK 字体，还发现字体需要同时放在 `/usr/share/fonts/`（系统 UI 使用）和 `/home/root/.local/share/fonts/`（持久化备份）两个位置。

## 最终成果

经过几个小时的协作，我的 reMarkable Paper Pro 现在拥有：

**应用（共 6 个）：**
- **Riddle（AI 日记）** — 用笔写字，墨迹渐隐，AI 以流畅的手写体回复。就像哈利波特里汤姆·里德尔的日记。连接任何 OpenAI 兼容 API
- **Chromium（网页浏览器）** — 完整的电子墨水浏览器，支持触控点击、滑动滚动和内置键盘。在电子墨水屏上浏览网页，前所未有的宁静
- **KOReader（电子书阅读器）** — 支持 PDF、EPUB、MOBI、FB2、DOC、RTF、HTML、DjVu、CBZ 等格式。内置 PDF 重排、字典查询、书签和高亮。比原生阅读器强大太多
- **PaperTerm（终端）** — 带屏幕键盘的终端模拟器，支持快速局部刷新。可以直接在平板上 SSH 到服务器
- **Store（应用商店）** — 直接在设备上浏览和安装应用，无需电脑
- **Bad Apple（电子墨水基准测试）** — 经典的 Bad Apple 视频在电子墨水屏上以最大帧率播放，挑战显示极限

**系统增强（共 4 项）：**
- **reLuminate** — 解锁超出出厂限制的更高亮度等级
- **CJK 字体（Noto Sans）** — 安装 Noto Sans CJK SC 和 KR 字体，中文和韩文在文件标题和文档中完美显示
- **rmtemplate** — 命令行工具，通过 SSH 上传和管理自定义笔记本模板
- **自动更新已禁用** — 防止 OS 更新破坏所有配置和自定义

## Claude 做了什么

说实话，这不是简单的"帮我运行几个命令"。整个过程中，Claude：

1. **诊断底层问题** — 当 LD_PRELOAD 被 systemd 静默忽略时，Claude 用 Python 解析 ELF 文件分析动态符号，找到了根本原因
2. **适应环境变化** — 平板中途自动更新 OS，Claude 迅速调整策略，重新匹配正确的组件版本
3. **编写持久化方案** — 针对只读文件系统和易失性覆盖层，设计了跨重启持久化的解决方案
4. **处理连接中断** — SSH 经常因为重启而断开，Claude 使用 `nohup` 后台脚本和状态文件来确保操作不会中途失败

这是我第一次真正感受到 AI 作为"结对编程伙伴"的强大之处。它不只是执行命令，而是在不断理解、调试、适应。

## 给想要尝试的人

如果你也有 reMarkable Paper Pro 并且开启了开发者模式：

1. 安装 remagic：`curl -fsSL https://raw.githubusercontent.com/maximerivest/remagic/main/get.sh | sh`
2. 或者，直接告诉 Claude 你想要什么——它会帮你搞定剩下的

最重要的建议：**禁用自动更新**。OS 更新会清除所有根文件系统的修改，包括 systemd 配置和系统字体。

## 写在最后

这次经历改变了我对 AI 辅助开发的看法。以前我觉得 AI 只能处理常规的编程任务。但这次，Claude 帮我解决了涉及嵌入式 Linux、ELF 二进制分析、systemd 服务管理、Qt 框架和设备固件兼容性的复杂问题。

如果你在考虑要不要用 AI 来处理那些看起来"太复杂"的技术任务——试试看，结果可能会让你惊喜。

---

**项目链接：**
- Riddle: https://github.com/MaximeRivest/riddle
- remagic: https://github.com/maximerivest/remagic
- KOReader: https://github.com/koreader/koreader

#reMarkable #AI #Claude #开源 #电子墨水 #生产力工具 #嵌入式Linux #深度定制
