## Environment
- MacOS Catalina 10.15.1
- Homebrew 2.2.1


## Setup VuePress
```
brew install nodebrew
nodebrew install-binary latest
nodebrew use v13.5.0

# First, install VuePress globally
npm install -g vuepress
# Create a Markdown file README.md as homepage
echo '# Hello VuePress' > README.md
```

[公式サイト](https://vuepress.vuejs.org/guide/markdown.html#links)によると、VuePressは`README.md`または`index.md`を`index.html`として生成します。

READMEとindexについてさらに詳しい情報は[こちら](https://github.com/vuejs/vuepress/pull/23)

> `vuepress dev`コマンドは現在機能していません

## Enable Navbar and Sidebar


## 参考
https://blog.howar31.com/vuepress-blog-tutorial/#so-why-vuepress-and-gitlab-pages