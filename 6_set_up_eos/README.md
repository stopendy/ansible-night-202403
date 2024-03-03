# ネットワーク機器の自動化

- [(参考) コネクションプラグインの概要](#参考-コネクションプラグインの概要)
- [(参考) インベントリ変数の説明](#参考-インベントリ変数の説明)
- [Arista vEOSの自動化](#arista-veosの自動化)
- [疎通確認](#疎通確認)

## (参考) コネクションプラグインの概要

Linuxとネットワークの主な違いは「コネクションプラグイン」の違いです。

実は、Linuxを自動化する際はずっと[ansible.builtin.ssh](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ssh_connection.html)コネクションプラグインを使ってきました。  
このコネクションは、Linuxを始めとするShellを持つOSを自動化するためのものです。  
内部的には以下の動作をします。

1. プレイブックをPythonスクリプトに変換する
2. SFTPやSCPによってPythonスクリプトをターゲットノードに送る
3. ターゲットノード上でPythonスクリプトを実行する

一方で、多くのネットワーク機器はPythonを実行する機能を持ちません。  
(最近のネットワーク機器はLinuxシェルを開けるものも増えていますが、まだ標準装備とまではいきません)

ではどうするかというと、ssh以外のコネクションプラグインを使う必要があるのです。  
使うべきコネクションプラグインはネットワーク機器のOSによって異なります。  
コミュニティがサポートしているOSであれば、公式ドキュメントの[Settings by Platform](https://docs.ansible.com/ansible/latest/network/user_guide/platform_index.html#settings-by-platform)に概ね載っているはずです。  
Ansibleコレクションを外部からインストールした場合は、そのコレクションの配布サイトから詳細情報を確認してください。

今回はArista EOSを自動化します。  
Arista EOSは主に[ansible.netcommon.network_cli](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/network_cli_connection.html)と[ansible.netcommon.httpapi](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/httpapi_connection.html#ansible-collections-ansible-netcommon-httpapi-connection)をサポートしていますが、今回はおそらく馴染みやすいであろう`network_cli`を使います。  
`network_cli`はネットワーク機器に対してSSH接続して自動化します。  
ただし、`ssh`コネクションとは異なりターゲットノードにPythonがインストールされていなくても動作します。

## (参考) インベントリ変数の説明

Arista vEOSを自動化するにあたり、いくつかの変数情報が必要となります。  
変数情報を格納したファイルを以下に示します。

| ファイル | 説明 |
| ------- | --- |
| [../hosts](../hosts) | インベントリファイル。Linuxのときと同じ考え方 |
| [../group_vars/eos.yml](../group_vars/eos.yml) | 認証情報に加えて、ネットワーク機器固有の設定を記述した。<br>詳細は後述 |
| [../host_vars/veos1.yml](../host_vars/veos1.yml) | 接続先のIPアドレスを記載。<br>これはLinuxと同じ考え方 |

ここで重要となるのは[../group_vars/eos.yml](../group_vars/eos.yml)です。  
中身を一部抜粋して記載します。

```yml
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: arista.eos.eos
ansible_become: true
ansible_become_password: admin
```

コネクションプラグインは`ansible_connection`で指定します。

`network_cli`や`httpapi`コネクションを使う場合、必ずOSを識別する情報をセットで指定します。  
今回は`arista.eos.eos`です。

`ansible_become: true`を指定すると、`enable`を実行するようになります。  
ちなみにLinuxなど`ansible.builtin.ssh`コネクションプラグインを指定した場合は`sudo`を実行するときに使えます。  
`enable`や`sudo`を実行してパスワードを求められたときに入力する文字列を`ansible_become_password`に記述します。

## Arista vEOSの自動化

今回はプレイブック内に3つのタスクを記述します。

| # | 利用モジュール | 処理内容 |
| - | ------------ | ------ |
| 1 | [arista.eos.eos_config](https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_config_module.html) | IPv4ルーティングの有効化 (`ip routing`) |
| 2 | [arista.eos.eos_interfaces](https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_interfaces_module.html) | ルーテッドポート設定 (`no switchport`)。ついでに`description`も設定する |
| 3 | [arista.eos.eos_l3_interfaces](https://docs.ansible.com/ansible/latest/collections/arista/eos/eos_l3_interfaces_module.html) | ルーテッドポートに対するIPアドレス設定 (`ip address x.x.x.x/xx`) |

2と3はinterface configuration modeに設定するので同時にできそうなものですが、Ansibleモジュールが分かれているので別タスクになります。

では、以下のコマンドでプレイブックを実行しましょう。  
([playbook_1_set_up_eos.yml](playbook_1_set_up_eos.yml))

```sh
ansible-playbook -i hosts 6_set_up_eos/playbook_1_set_up_eos.yml
```

参考までに、これまでのLinuxとネットワークの両方のタスクを連結したプレイブックも掲載しておきます。  
プレイブック2つを繋げただけの内容です。  
([playbook_2_linux_and_veos.yml](playbook_2_linux_and_veos.yml))

## 疎通確認

これまでの自動化により、以下の構成が完成しました。  
せっかくなので疎通確認してみましょう。

![../images/network_diagram.png](../images/network_diagram.png)

`linux1`にログインしてping, curl, tracepathを実行してみます。

`ping`が通ることから、`linux2`に疎通性があることがわかります。

```sh
ping -c2 192.168.2.2
# PING 192.168.2.2 (192.168.2.2) 56(84) bytes of data.
# 64 bytes from 192.168.2.2: icmp_seq=1 ttl=63 time=20.4 ms
# 64 bytes from 192.168.2.2: icmp_seq=2 ttl=63 time=4.42 ms

# --- 192.168.2.2 ping statistics ---
# 2 packets transmitted, 2 received, 0% packet loss, time 1002ms
# rtt min/avg/max/mdev = 4.416/12.387/20.359/7.971 ms
```

`curl`を実行すると、`linux2`のコンテンツが返ります。

```sh
curl 192.168.2.2
# linux2
```

`tracepath`の実行結果から、`veos1 (192.168.1.1)` でルーティングされてから`linux2 (192.168.2.2)`に到達していることがわかります。

```sh
tracepath -l 1500 -n 192.168.2.2
 1:  192.168.1.1                                           6.496ms 
 2:  192.168.2.2                                          11.966ms !H
     Resume: pmtu 1500 
```
