# MkDocs
静的サイトジェネレータ。pythonで動く。markdwonで記述できる

- 公式
  - https://www.mkdocs.org/

## 環境構築(Mac)
```
# python 
git clone git://github.com/yyuu/pyenv.git ${HOME}/.pyenv
cat << EOF >> ${HOME}/.profile
export PYENV_ROOT=\${HOME}/.pyenv
export PATH=\${PYENV_ROOT}/bin:\${PATH}
eval "\$(pyenv init -)"
EOF
source ${HOME}/.profile

pyenv install 3.6.8
pyenv global 3.6.8
pyenv rehash
easy_install pip

# mkdocs
pip install mkdocs-material
```

## 起動
```
mkdocs new dashbordDocs
cd dashbordDocs
mkdocs serve
```

## 詳細設定
ここ見れば代替できる
- https://note.mu/team_csm/n/n576168a2a8bd