## 拡張機能
```
Brancket Pari Colorizer
endwise
Excel Viewer
Go to Spec
HTML Snippets
Markdown Preview Github Styling
Rails
Remote - SSH
Remote - SSH Editing Configuration Files
Remote - SSH Explorer
Ruby
Ruby Solargraph
ruby-rubocop
settings Sync
Slim
vscode-gemfile
```

## setting json
```
{

  // ミニマップ非表示
  "editor.minimap.enabled": false,
  // インデント可視化
  "editor.renderIndentGuides": true,
  // マウスホイール+Ctrlでエディタサイズの可変有効化
  "editor.mouseWheelZoom": true,
    //タブサイズ
  "editor.tabSize": 2,
   // スペースの表示
  "editor.renderWhitespace": true,
  // 折り返し表示
  "editor.wordWrap": "on",
  // vscode全体のサイズ指定。Ctrl+(+),(-)で調整できる
  "window.zoomLevel": -1,
  // パンくずリストの表示
  "breadcrumbs.enabled": true,

  
  // setting syncs
  "explorer.confirmDragAndDrop": false,
  "sync.gist": "6a45da0944064a928316accfea2ff4ff",

  // ターミナルをGitBashに変更
  "terminal.integrated.shell.windows": "C:\\Program Files\\Git\\bin\\bash.exe",  
  // ターミナルの文字色変更
  "workbench.colorCustomizations": {
    "terminal.foreground": "#00FF00",
  },
  // ターミナルで選択したテキストを自動コピーする
  "terminal.integrated.copyOnSelection": true,

  // マークダウンでスニペット機能を有効にする
  "[markdown]":  {
  "editor.wordWrap": "on",
  "editor.quickSuggestions": true
  },

  // 改行コードは必ずLF
  "files.eol": "\n",

  "[ruby]": {
    "format": "rubocop",
    "intellisense": "rubyLocate",
    "specCommand": "bundle exec bin/rspec",
  },

  "gitlens.advanced.messages": {
    "suppressGitVersionWarning": true
  },

  "command": "extension.runFileSpecs",
  "title": "Run File Specs",
  "key": "cmd+shift+r",

  "git.ignoreLegacyWarning": true,

  "solargraph.useBundler": true,
  "ruby.rubocop.onSave": false,
  "solargraph.autoformat": true,
  "explorer.confirmDelete": false,

}
```