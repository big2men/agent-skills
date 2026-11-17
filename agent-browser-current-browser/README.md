# agent-browser-current-browser

macOS で、すでに開いている Chrome を agent-browser から操作するスキルです。開いているタブとログイン状態はそのまま使います。複数のエージェントが同じブラウザを共有でき、それぞれ専用のバックグラウンドタブで作業します。

「今開いているブラウザで」「現在のブラウザで」と頼むときや、Chrome のプロファイル名・タブグループ名を指定するときに使います。

nabinno/agent-skills の `agent-browser-current-browser`（Windows + WSL2 向け）を macOS 向けに移植したものです。出典とライセンスは `NOTICE` にあります。

## 使い始める前に

1. Chrome 144 以降を使います。標準のユーザーデータディレクトリ（`~/Library/Application Support/Google/Chrome`）を前提にしています。Chrome Beta など別の場所を使うときは `CDP_CHROME_USER_DATA_DIR` を設定してください。
2. Node 22 以降と `python3` を `PATH` に置きます。
3. 操作したい Chrome で `chrome://inspect/#remote-debugging` を開き、リモートデバッグをオンにします。そのブラウザは開いたままにします。
4. スキルとして使うには、このディレクトリを `~/.claude/skills/` にリンクします。

   ```bash
   ln -s ~/src/big2men-agent-skills/agent-browser-current-browser ~/.claude/skills/agent-browser-current-browser
   ```

5. Chrome のプロファイルごとに 1 回、付属の拡張機能を入れます。

   ```bash
   CB=~/src/big2men-agent-skills/agent-browser-current-browser/scripts/cb
   $CB install-bridge
   ```

   続いて、そのプロファイルで `chrome://extensions` を開き、デベロッパーモードをオンにして「パッケージ化されていない拡張機能を読み込む」を選びます。コマンドが表示したフォルダを指定します（ファイルダイアログでは Cmd+Shift+G でパスを貼れます）。拡張機能 ID は `haagbfgnipgidagcllndobiiiejdnpjl` です。詳しくは `cb-bridge/README.md` を見てください。

## エージェントに頼む

- 「今開いているブラウザでカレンダーを確認して」
- 「Work の『調査』グループでこの URL を開いて」

エージェントは `cb new` を 1 回実行し、表示されたセッション名を使い続け、最後に `cb <name> done` で終わります。開いたタブは残ります。

## 自分で実行する

```bash
CB=~/src/big2men-agent-skills/agent-browser-current-browser/scripts/cb
$CB new --own-group "資料確認"
$CB <表示された名前> open "https://example.com"
$CB <表示された名前> snapshot -i
$CB <表示された名前> done
```

`done` はセッションを切り離すだけです。`done --close` を付けると、そのセッションが開いたタブを閉じます。`$CB <name> show` で、そのタブを前面に出せます。

最初の接続では、Chrome が「リモートデバッグを許可しますか？」と尋ねます。1 回許可すれば、Chrome を終了するかリモートデバッグをオフにするまで再確認されません。

素の `agent-browser connect` でブラウザを操作しないでください。別のデバッグ接続ができ、ほかのセッションとぶつかります。プロファイル、タブグループ、Slack リンクについてのエージェント向けルールは `SKILL.md` にあります。
