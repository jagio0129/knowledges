### textlint on vscode
- npmのインストール方法
  - https://qiita.com/taiponrock/items/9001ae194571feb63a5e
- npm 6.9.0

```
npm init -y
npm install --save-dev \
    textlint \
    textlint-rule-preset-ja-spacing \
    textlint-rule-preset-ja-technical-writing \
    textlint-rule-spellcheck-tech-word
```

vscodel-textlint install.

### textlint on docker 
```
# docker setup
docker build -t jagio0129/textlint .

# testlint run
./run <target path>
```