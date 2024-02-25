# Ansibleのインストール

- [実行コマンド](#実行コマンド)
- [解説](#解説)

## 実行コマンド

```sh
sudo dnf install python3-pip sshpass
pip3 install --user ansible ansible-lint paramiko
```

## 解説

`sudo dnf install python3-pip`  
venvの外でpipコマンドを使うためにインストールします。  
venvを使う場合は不要です。

`pip3 install --user ansible`  
ユーザー権限で`~/.local/`配下にansibleをインストールします。  
今回は「小さく始めるAnsible」という企画なので、venvを使わない方法で始めます。  
venvをすでによくご存じの方は、venvを使っても問題ありません。

`pip3 install --user ansible-lint`  
Ansible専用のLinterをインストールします。  
Ansibleの動作には必須ではありませんが、[../3_use_vs_code](../3_use_vs_code)にて導入するAnsible拡張機能を正しく動作させるために必要です。  
Linterとは、コーディング規約違反を指摘し、読みやすいソースコード記述を支援してくれるツールの総称です。

`sudo dnf install sshpass`  
Linuxに対してパスワード認証でSSHログインするため、`sshpass`をインストールします。  
SSH鍵認証でパスワードなしの認証を行う場合には不要です。

`pip3 install --user paramiko`  
NW機器にSSH接続するため、`paramiko`をインストールします。  
`ansible-pylibssh`でも結構です。
