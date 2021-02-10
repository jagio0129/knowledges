# Jupyterの導入
```
pip install jupyter
```

## 起動
```
jupyter notebook --ip=0.0.0.0
```

## パスワード設定
notebookパッケージが4.3になってからセキュリティが強化されたようでパスワード入力を求められる。

tokenを平分で指定する場合とpasswordを暗号化して設定する2種類がある

```
# 設定ファイルを作成
jupyter notebook --generate-config

# tokenの指定
vim ~/.jupyter/jupyter_notebook_config.py
  + c.NotebookApp.token = 'xxx'
=> 起動後に192.168.33.20:8888?token=xxxにアクセス
```

## 参考サイト

- https://qiita.com/SaitoTsutomu/items/aee41edf1a990cad5be6