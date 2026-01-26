# Git 同步与推送卡死问题排查日志

## 问题现象

1.  **历史分叉**：本地 `main` 分支比远程多 33 个提交，远程比本地多 26 个提交。
2.  **推送卡死**：`git push --force` 在写入对象 100% 后无响应，Github 未返回确认。
3.  **IDE 异常**：VS Code 显示正在 Rebase，且有大量 Untracked 文件（如 `frontend/.vite/`）。

## 原因分析

1.  **远程污染**：远程的 26 个新提交实为另一个项目（`my-tech-notebook`）的内容，可能是误推导致的。
2.  **认证组件过期**：Git 报错 `credential-manager-core is not a git command`，导致推送后的认证环节卡死。
3.  **网络/体积问题**：仓库包含约 80MB 数据（主要由 `frontend/.vite/`, `docs/dita/output` 等构建产物构成），加上网络环境不佳，导致大文件推送时连接超时或被阻断。
4.  **操作残留**：尝试 `git rebase` 失败后进入了中间状态，导致 IDE 显示异常。

## 解决方案

1.  **恢复本地状态**
    - 创建备份分支：`git branch backup/pre-sync-divergence`
    - 放弃变基：`git rebase --abort`
    - 硬重置：`git reset --hard backup/pre-sync-divergence`

2.  **仓库瘦身 (优化传输)**
    - 将构建产物加入 `.gitignore`：
        - `frontend/.vite/`
        - `docs/dita/output`
        - `playwright-screenshots/node_modules`
        - `whisper/.venv`
        - `whisper/upload`
    - 移除缓存：`git rm -r --cached ...`

3.  **修复配置与推送**
    - 修复认证器：`git config --global credential.helper manager`
    - 强制推送：`git push --force origin main`（配合网络调整）

## 经验总结

> [!TIP]
> 1. **推送前检查**：大版本变动前先 `git status` 检查是否有意外的大文件。
> 2. **网络环境**：推送大文件（>50MB）若卡住，优先检查代理设置 (`git config --global http.proxy`)。
> 3. **遇到 Rebase 冲突**：如果历史完全不相关（如本项目情况），不要尝试 Rebase，直接确认保留哪一边，然后 Reset + Force Push。

---

*创建日期: 2026-01-25*
