# GPUとは
- Graphics Processing Unit
- 画像処理を担当する主要な部品
- 高速なVRAM(ビデオメモリ；グラフィックボード上のGPU専用メモリ)と接続され、グラフィックシェーディングに特化したプロセッサが多く集まった構造を持っている
- **大量のデータを複数のプロセッサで同時かつ並列処理する**ことができる。

## GPU開発企業
- Intel
- NVIDIA
- AMD/ATI
- Matrox
- VIA/S3 GRAPHICS
    - **NVIDIAAMD**の二強。

### NVIDIA
シリーズ名|型番例|用途例
---|---|---
GeForce|GeForece GTX 680,580|ゲームフィジクス、動画編集、CUDA入門
Quadro|Quadro 6000,K5000|デザイン/3DCD/CAD/ビデオアプリケーション
Tesla|Tesla C2075,K20|計算科学/CAE/バイオインフォマティクス、ファイナンス

#### GeForceシリーズ
いわゆるコンシューマ向けで**DirectXに最適化されている**。

#### Quadro
**OpenGLに最適化**されており、3DCG作成やCADなどを利用するのに適している。逆に性能的にGeForceなどと変わらなくてもゲームは苦手だったりする。

#### Teslaシリーズ
グラフィックボードから映像出力機能を覗いたもので、まさに**GPUコンピューティングのために開発されたもの**。高い計算精度が求められるCAE、金融などの分野で利用される。

### AMD
シリーズ名|型番例|用途例
---|---|---
Radeon|Radeon HD6090,6070|ゲームフィジクス/動画編集
FirePro|FirePro V7900,V5900|デザイン/3DCD/CAD/ビデオアプリケーション
FireStream|FireStream 9370|計算科学/CAE/バイオインフォマティクス/ファイナンス



## GPUコンピューティングとは？
- 一言でいえば**計算機での数値計算においてGPUを利用すること**。

## CPUとGPUの違い？
- **CPUは連続的な計算が得意で、大してCPUは並列的な演算が得意**

## CUDAとは？
- **Compute Unified Device Archtecture**の略
- NVIDIAが提供する**GPUコンピューティング向けの統合開発環境**
- プログラム記述、コンパイラ、ライブラリ、デバッガなどから構成されており、プログラム言語はC言語ベースに拡張を加えたものであるため、C言語によるプログラミング経験があれば扱いやすくなっている
- CUDAは**無料で使用**することができる
- CUDAでプログラムを書けばCUDAをサポートするすべてのGPU上で動作させることもできる。

## GPGPU
- グラフィックス処理以外の演算を行わせるようにしたもの
- NVIDIAのGeForce、Quadroシリーズが対応しており、TeslaはGPGPU専用

## 参考サイト
- http://www.gdep.jp/page/view/249
- http://www.orixrentec.jp/helpful_info/detail.html?id=92