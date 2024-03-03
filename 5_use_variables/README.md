# 変数を使う

- [linux2を自動設定する](#linux2を自動設定する)
  - [(参考) インベントリ変数の必要性](#参考-インベントリ変数の必要性)
  - [プレイブックの実行](#プレイブックの実行)

## linux2を自動設定する

### (参考) インベントリ変数の必要性

[../4_set_up_linux](../4_set_up_linux)では`linux1`を自動構築しました。  
今回は`linux2`も自動設定できるようにプレイブックを拡張しましょう。

どのようにすれば`linux2`も自動化できるでしょうか？  
以下のように、hosts行を書き換えて`linxu2`もプレイブックの対象に加えればそれで十分でしょうか？

```yml
- name: Set up Linux
  hosts: linux
  gather_facts: false

  tasks:
    # ...
```

httpdのインストールと起動、そしてfirewalldの設定については問題なさそうです。  
ですが、IPアドレス設定の箇所には問題があります。  
今のままのプレイブックでは`linux1`と`linux2`に全く同じIPアドレスとスタティックルートが設定されてしまいます。

```yml
    - name: Configure IP address
      community.general.nmcli:
        conn_name: ens4
        type: ethernet
        method4: manual
        ip4: 192.168.1.2/24
        routes4:
          - 192.168.2.0/24 192.168.1.1
        autoconnect: true
        state: present
```

今回のように、実行対象のホストやグループに応じてプレイブック内のパラメータを変更したい場合、ホスト変数やグループ変数が役に立ちます。  
詳細は公式ドキュメントの[How to build your inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)を参照してください。

次のセクションで具体的な設定内容を示します。

### プレイブックの実行

今回は`linux1`と`linux2`という2つのホスト間で挙動を変更したいので、ホスト変数を使います。  
それぞれのホスト変数ファイルは以下のとおりです。

- [../host_vars/linux1.yml](../host_vars/linux1.yml)
- [../host_vars/linux2.yml](../host_vars/linux2.yml)

`linux1`と`linux2`の両方の自動化に対応したプレイブックを示します。  
([playbook.yml](playbook.yml))

変更したのは以下の3点です。

1. プレイブックの実行対象を`linux1`から`linux`グループに変更
2. IPアドレス指定を変数で表現
3. ルーティング設定を変数で表現

では、プレイブックを実行してみましょう。

```sh
ansible-playbook -i hosts 5_use_variables/playbook.yml
```

`linux1`と`linux2`の設定をそれぞれ確認してみましょう。  
Linux実機で以下のコマンドを実行します。

```sh
ip -br address show
ip route show proto static
```

Ansibleで確認するなら以下のコマンドになります。

```sh
ansible -i hosts linux -v -a 'ip -br address show'
ansible -i hosts linux -v -a 'ip route show proto static'
```
