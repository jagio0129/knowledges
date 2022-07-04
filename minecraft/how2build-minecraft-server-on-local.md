MODテストサーバの作成
===

稼働サーバに適応する前にMODの作成・動作確認を行う環境を用意する

## 環境の準備

稼働サーバとOS、スペックを合わせる。ここでは以下の環境を稼働サーバと仮定する

- Ubuntu 20.01
- CPU Core 3
- Memory 2GB

開発環境ではVagrantとVirtualBoxで仮想環境を作成する。

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  config.vm.network "private_network", ip: "192.168.33.11"

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 3
    vb.memory = "2048"
  end
end
```

## minecraft-server.jarを開発環境で立ち上げる

稼働サーバと同じバージョンを使用すること。ここではv1.17.1を例とする

```sh
# 仮想環境にて
sudo apt update

# download minecraft server
wget https://launcher.mojang.com/v1/objects/a16d67e5807f57fc4e550299cf20226194497dc2/server.jar
mv server.jar minecraft_server.1.17.1.jar

# install Java16
sudo apt install openjdk-16-jre

# start minecraft server
java -Xmx1024M -Xms1024M -jar minecraft_server.1.17.1.jar nogui
[16:27:34] [main/ERROR]: Failed to load properties from file: server.properties
[16:27:34] [main/WARN]: Failed to load eula.txt
[16:27:34] [main/INFO]: You need to agree to the EULA in order to run the server. Go to eula.txt for more info.
```

エラーとなるのでeula.txtを以下のように修正

```
  #By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula).
  #Thu Sep 16 16:27:34 UTC 2021
- eula=false
+ eula=true
```

改めて起動

```sh
# start minecraft server
java -Xmx1024M -Xms1024M -jar minecraft_server.1.17.1.jar nogui

16:30:15] [main/INFO]: Environment: authHost='https://authserver.mojang.com', accountsHost='https://api.mojang.com', sessionHost='https://sessionserver.mojang.com', servicesHost='https://api.minecraftservices.com', name='PROD'
[16:30:16] [main/WARN]: Ambiguity between arguments [teleport, location] and [teleport, destination] with inputs: [0.1 -0.5 .9, 0 0 0]
[16:30:16] [main/WARN]: Ambiguity between arguments [teleport, location] and [teleport, targets] with inputs: [0.1 -0.5 .9, 0 0 0]
[16:30:16] [main/WARN]: Ambiguity between arguments [teleport, destination] and [teleport, targets] with inputs: [Player, 0123, @e, dd12be42-52a9-4a91-a8a1-11c01849e498]
[16:30:16] [main/WARN]: Ambiguity between arguments [teleport, targets] and [teleport, destination] with inputs: [Player, 0123, dd12be42-52a9-4a91-a8a1-11c01849e498]
[16:30:16] [main/WARN]: Ambiguity between arguments [teleport, targets, location] and [teleport, targets, destination] with inputs: [0.1 -0.5 .9, 0 0 0]
[16:30:16] [main/INFO]: Reloading ResourceManager: Default
[16:30:17] [Worker-Main-3/INFO]: Loaded 7 recipes
[16:30:17] [Worker-Main-3/INFO]: Loaded 1137 advancements
[16:30:19] [Server thread/INFO]: Starting minecraft server version 1.17.1
[16:30:19] [Server thread/INFO]: Loading properties
[16:30:19] [Server thread/INFO]: Default game type: SURVIVAL
[16:30:19] [Server thread/INFO]: Generating keypair
[16:30:20] [Server thread/INFO]: Starting Minecraft server on *:2556
[16:30:20] [Server thread/INFO]: Using epoll channel type
[16:30:20] [Server thread/INFO]: Preparing level "world"
[16:30:27] [Server thread/INFO]: Preparing start region for dimension minecraft:overworld
[16:30:27] [Worker-Main-4/INFO]: Preparing spawn area: 0%
[16:30:27] [Worker-Main-3/INFO]: Preparing spawn area: 0%
.
.
.
```
