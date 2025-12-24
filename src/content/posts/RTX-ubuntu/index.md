---
title: RTX5080搭載PCにUbuntuを入れた話
published: 2025-12-24
description: 'RTX5080の乗ったPCにUbuntuを入れようとしたところ一筋縄ではいかなかった'
image: ''
tags: [ubuntu, environment, MMA-Advent-Calendar-2025, Advent-Calendar]
category: '環境'
draft: false 
lang: 'ja'
---

<iframe src="https://adventar.org/calendars/11429/embed" width="620" height="362" frameborder="0" loading="lazy"></iframe>

>この記事は[MMA Advent Calendar 2025 - Adventar](https://adventar.org/calendars/11429) の16日目の記事です。

こんにちは。MMAのkofeです。今回はRTX5080搭載PCにUbuntuを入れた話を書きます。

## はじめに

最新のRTX5080を搭載したPCにUbuntuをインストールしようとしたところ、ドライバの互換性の問題でGRUBの次の画面が真っ黒になり、一筋縄ではいかなかったので、その解決方法を記録します。

## 環境

- GPU: NVIDIA RTX 5080
- CPU: AMD Ryzen 9 3950X（内蔵GPU非搭載）
- ストレージ: 新規SSD

## 発生した問題

RTX5080は新しいGPUのため、Ubuntuのデフォルトドライバ（Nouveau）との互換性がなく、GRUBメニューの次の画面で真っ黒になってしまいました。

通常は内蔵GPUに切り替えて対処する方法がありますが、Ryzen 9 3950Xには内蔵グラフィックスが搭載されていないため、別の対策が必要でした。

参考にした記事：
- [RTX 5000シリーズでのUbuntuインストール](https://qiita.com/chiral/items/a0d71d2971dece3199d1)
- [RTX 5080搭載PCへのUbuntu 22.04インストール方法](https://notes.rdu.im/howtos/install-ubuntu22.04-on-pc-with-nv-5080/)

## インストール準備

### 1. Ubuntu Desktop 日本語 Remixのダウンロード

[Ubuntu Desktop 日本語 Remix](https://www.ubuntulinux.jp/download/ja-remix)からISOイメージをダウンロードします。

### 2. ブータブルUSBの作成

[Rufus](https://rufus.ie/ja/)などのツールを使ってUSBにISOイメージを焼きます。

## UEFI設定の変更

RTX 50シリーズを搭載していて内蔵グラフィックスがない場合、デフォルトドライバではGRUBの後に真っ黒になるため、以下のUEFI設定を変更します。

### AMDプロセッサの場合の設定

```
BOOT-CSM: OFF
SR-IOV: ENABLED
Above 4G decoding: ENABLED
Re-size BAR Support: DISABLED
```

参考：
- [AskUbuntu - RTX 5060 Tiでの起動問題](https://askubuntu.com/questions/1547947/ubuntu-dont-start-with-nvidia-rtx-5060-ti-graphic-card)
- [NVIDIA Developer Forums - RTX 5090での空白画面問題](https://forums.developer.nvidia.com/t/ubuntu-24-04-2-amd-threadripper-blank-screen-while-install-with-nvidia-rtx-5090-zotac/326242/22)

### 追加の設定

```
Secure Boot: DISABLE
Fast Boot: DISABLE
起動順序: USBを一番上に設定
```

まあ、いつものですね。
設定を保存してUEFIから出ます。

## Ubuntuのインストール

### 1. 起動と言語選択

GRUBメニューから「Try Ubuntu with Japanese input support」を選択します。

### 2. インストーラーの起動

デスクトップから「Ubuntuをインストール」を選択し、言語とキーボードレイアウトを設定して、通常のインストールで進めます。

### 3. パーティションの設定

ポータブルなシステムにするため（SSDを他のPCに挿しても動くように）、インストールの種類で「それ以外」を選択し、カスタムパーティションを作成します。

参考：[Ubuntu 22.04 LTS UEFIインストール方法](https://kledgeb.blogspot.com/2022/04/ubuntu-2204-lts-5-uefi.html?m=1)

#### EFIパーティションの作成

新しいSSD上にEFIパーティションを作成します。これにより、Windowsの既存のEFIパーティションに依存せず、SSDを別のPCに移植可能になります。

#### ルートパーティションの作成

空き領域の半分程度（約2TB）でルートパーティションを作成します。残りはホームディレクトリやスワップ領域として使用できます。

### 4. ブートローダーの設定

ブートローダーのインストール先として、新しく作成したEFIパーティションのあるSSDを指定します。これにより、Windowsの既存のEFIパーティションに影響を与えずにインストールできます。

### 5. インストールの完了

タイムゾーンとユーザー情報を設定してインストールを完了します。

再起動時に「Please remove the installation medium, then press ENTER:」と表示されたら、USBを抜いてEnterを押します。

## インストール後の設定

### 1. システムの更新

初期設定を完了したら、ターミナルを開いて以下を実行します：

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install build-essential dkms
```

再起動します。

### 2. NVIDIAドライバのインストール

```bash
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install nvidia-driver-580-open
```

Open版ドライバを選択した理由：
- Open版が推奨されている
- Server版はQuadro向け
- 580はRTX 5080で検証済み（らしい）

### 3. Nouveauドライバの無効化

デフォルトのNouveauドライバを無効化します：

```bash
sudo sh -c "echo 'blacklist nouveau
options nouveau modeset=0' > /etc/modprobe.d/blacklist-nouveau.conf"
sudo update-initramfs -u
```

### 4. 動作確認

再起動後、以下のコマンドでGPUが正しく認識されているか確認します：

```bash
nvidia-smi
```

GPUの情報が表示されれば成功です。

## まとめ

RTX 5080のような最新GPUでは、デフォルトドライバとの互換性の問題が発生することがあります。UEFI設定の調整とドライバの手動インストールにより、無事にUbuntuを動作させることができました。

同様の環境でインストールに困っている方の参考になれば幸いです。

### おまけ
最初はドライバーが入ってないからだと思い、CubicでカスタムISOを作成してドライバーを組み込もうとしましたが、UEFI設定の問題だったので無駄骨に終わりました。皆さんもUEFI設定を最初に確認しましょう！