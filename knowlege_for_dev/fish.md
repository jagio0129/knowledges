fishを導入
===

### for Ubuntu
```sh
# install fish
sudo apt-add-repository ppa:fish-shell/release-3
sudo apt update
sudo apt install fish

# install fisherman
curl -Lo ~/.config/fish/functions/fisher.fish --create-dirs https://git.io/fisher

# theme 適当
fisher install oh-my-fish/theme-bobthefish

# install powerline fonts
git clone https://github.com/powerline/fonts.git
cd fonts
./install.sh
cd & rm -rf fonts
```

### for Mac
```sh
# install fish
brew install fish

# あとはubuntuと同じ
```