# Ansible Night 2024.3

- [概要](#概要)
- [構成図](#構成図)
- [使い方](#使い方)
- [デモの詳細](#デモの詳細)

## 概要

[【リモート開催】Ansible Night 2024.3 これから始めるAnsible](https://ansible-users.connpass.com/event/310794/)のイベント登壇用資料です。  
『小さく始めるAnsible』で実施したデモのソースコードをこちらに置きました。

## 構成図

![images/network_diagram.png](images/network_diagram.png)

## 使い方

1. 環境に合わせてインベントリファイルを書き換える
   - [hosts](hosts)
   - [host_vars/](host_vars/)
   - [group_vars/](group_vars/)
2. 設定ファイルが邪魔であれば削除する ([ansible.cfg](ansible.cfg))
3. 配下ディレクトリのプレイブックを実行する

## デモの詳細

| ディレクトリ | やること概要 |
| ---------- | ---------- |
| [1_install_ansible](1_install_ansible) | Ansibleをインストールする |
