# 変数を使う

- [linux2を自動設定する](#linux2を自動設定する)
  - [(参考) インベントリ変数の必要性](#参考-インベントリ変数の必要性)
  - [プレイブックの実行](#プレイブックの実行)
- [index.html](#indexhtml)

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
([playbook_1_use_variables.yml](playbook_1_use_variables.yml))

変更したのは以下の3点です。

1. プレイブックの実行対象を`linux1`から`linux`グループに変更
2. IPアドレス指定を変数で表現
3. ルーティング設定を変数で表現

では、プレイブックを実行してみましょう。

```sh
ansible-playbook -i hosts 5_use_variables/playbook_1_use_variables.yml
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

## index.html

追加でもう一つ処理を加えてみましょう。  
ここではindex.htmlを作成します。

なぜこのタイミングで実施するかというと、この処理にも変数を使うためです。

今回は[ansible.builtin.copy](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html)モジュールを使います。  
このモジュールは、リモートにファイルを生成する機能を持ちます。  
基本的にはローカルからリモートにファイルコピーするのが基本機能ですが、`content`パラメータを指定することで「任意の文字列」を含むファイルをターゲットノード上に作ることができます。

今回は、このモジュールを使って「ホスト名が書いてある`index.html`」を配置します。  
ホスト名は`inventory_hostname`という変数を使って表現します。  
`inventory_hostname`は[Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)と呼ばれ、定義せずとも初めから値が代入されている特殊な変数です。  
今回のケースでは[../hosts](../hosts)に記述されたターゲットノード名 (`linux1`か`linux2`) が代入されています。

では、プレイブックを実行してみましょう。  
([playbook_2_add_index_html.yml](playbook_2_add_index_html.yml))

```sh
ansible-playbook -i hosts 5_use_variables/playbook_2_add_index_html.yml --start-at-task 'Put index.html'
```

プレイブック実行後、`ansible1`から`linux1`にcurlを実行すると以下の応答が帰るようになったはずです。

```sh
curl 192.168.0.12
# linux1
```

以上でLinuxの自動化は完了です。  
お疲れ様でした。
