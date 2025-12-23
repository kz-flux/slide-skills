# Tmux Multi-Agent Skill

WSL + Tmux で複数の Claude Code エージェントを並列実行するためのスキル。

## 重要な前提知識

### Enter/C-m が効かない問題
Claude Code の入力UIでは `Enter` や `C-m` が動作しないことがある。
**必ず Space + load-buffer + paste-buffer 方式を使用すること。**

### 推奨コマンドパターン

```bash
# 最も確実なタスク送信方法
tmux send-keys -t team:0.$PANE Space
sleep 0.3
tmux send-keys -t team:0.$PANE "$MESSAGE"
sleep 0.2
printf "\r" | tmux load-buffer -
tmux paste-buffer -t team:0.$PANE
```

---

## セットアップ手順

### 1. 前提条件確認
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'node --version && claude --version && tmux -V'
```

### 2. ディレクトリ準備
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'mkdir -p ~/tmux-team/{tmp,logs,workspace/input,workspace/output/v1.0/results,instructions}'
```

### 3. Tmuxセッション作成（quick-start.sh使用推奨）
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'cd ~/tmux-team && ./quick-start.sh 11'
```

### 3b. 手動でセッション作成
```bash
# Step 1: セッション作成
wsl -d Ubuntu -u azuhidekitake -- bash -c 'tmux start-server && tmux new-session -d -s team -c ~/tmux-team/workspace'

# Step 2: ペイン分割（人数-1回）
wsl -d Ubuntu -u azuhidekitake -- bash -c 'for i in {1..10}; do tmux split-window -t team:0 -c ~/tmux-team/workspace; tmux select-layout -t team:0 tiled; done'

# Step 3: Claude起動（1ペインずつ順次、8秒待機）
wsl -d Ubuntu -u azuhidekitake -- bash -c 'tmux send-keys -t team:0.0 "claude --dangerously-skip-permissions" Enter && sleep 8'
# 各ペイン個別に実行（並列NG）
```

---

## タスク送信

### agent-send.sh 使用（推奨）
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'cd ~/tmux-team && ./agent-send.sh 0 "タスク内容"'
```

### 直接送信
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c '
tmux send-keys -t team:0.0 Space
sleep 0.3
tmux send-keys -t team:0.0 "タスク内容をここに"
sleep 0.2
printf "\r" | tmux load-buffer -
tmux paste-buffer -t team:0.0
'
```

---

## 状況確認

### ペイン状態確認
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'tmux list-panes -t team:0 -F "#{pane_index}: #{pane_current_command}"'
```

### ペイン内容確認（最新30行）
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'tmux capture-pane -t team:0.0 -p -S -30'
```

### 出力ファイル確認
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'ls -la ~/tmux-team/workspace/output/v1.0/results/'
```

---

## 同期

### Windows → WSL
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'cp -r /mnt/c/Users/KazuhideAkitake/.claude/projects/PROJECT_NAME/input/* ~/tmux-team/workspace/input/'
wsl -d Ubuntu -u azuhidekitake -- bash -c 'cp -r /mnt/c/Users/KazuhideAkitake/.claude/projects/PROJECT_NAME/instructions/* ~/tmux-team/instructions/'
```

### WSL → Windows
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'mkdir -p /mnt/c/Users/KazuhideAkitake/.claude/projects/PROJECT_NAME/output/tmux_results && cp -r ~/tmux-team/workspace/output/v1.0/results/* /mnt/c/Users/KazuhideAkitake/.claude/projects/PROJECT_NAME/output/tmux_results/'
```

---

## トラブルシューティング

### 問題: タスクが実行されない
**原因**: Enter/C-m が効かない
**解決**: Space + load-buffer + paste-buffer 方式を使用

### 問題: Claudeが起動しない
**解決**: Enter を明示的に指定
```bash
tmux send-keys -t team:0.0 "claude --dangerously-skip-permissions" Enter
```

### 問題: セキュリティ確認で止まる
**解決**: Space + load-buffer方式
```bash
tmux send-keys -t team:0.0 Space && sleep 0.3 && printf "\r" | tmux load-buffer - && tmux paste-buffer -t team:0.0
```

### 問題: 一部ペインだけ動かない
**原因**: 並列送信時の競合
**解決**: 各ペインを個別コマンドで順次実行、8秒待機

### 問題: セッションが消える
**解決**: tmux start-server で明示的にサーバー起動後セッション作成

### 全ペインリセット
```bash
wsl -d Ubuntu -u azuhidekitake -- bash -c 'cd ~/tmux-team && ./reset-all.sh'
```

---

## 動作中のサイン

Claudeが作業中の場合、以下のような表示が見える：
- `thought for Xs` - 思考中
- `Simmering...`, `Deliberating...`, `Pontificating...` - 処理中
- `● Read` - ファイル読み込み中
- `● Bash` - コマンド実行中
- `● Write` - ファイル書き込み中

---

## ベストプラクティス

| 項目 | 推奨 | 非推奨 |
|------|------|--------|
| タスク送信 | **Space + load-buffer + paste-buffer** | Enter/C-mのみ |
| Claude起動 | 1ペインずつ順次（8秒待機） | ループで一括 |
| セッション作成 | tmux start-server 後 | 直接作成 |
| セキュリティ承認 | Space + load-buffer方式 | Down + Enter |
| 並列タスク | 各ペイン個別に0.5秒間隔 | 同時送信 |

---

## 参考リンク

- リポジトリ: https://github.com/kz-flux/tmux-multi-agent
- トラブルシューティング詳細: docs/troubleshooting.md
