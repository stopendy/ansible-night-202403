# Hello World

- [実行コマンド](#実行コマンド)
- [解説](#解説)
- [実行結果](#実行結果)

## 実行コマンド

```sh
ansible-playbook 2_hello_world/playbook.yml
```

## 解説

`ansible.builtin.debug`モジュールは、任意の文字列を画面上に表示します。  
他のプログラミング言語で言うところのprint文です。

今回は`Hello World!`とだけ表示するプレイブックを実行してみます。

## 実行結果

![result.png](result.png)

※上記画像の出力はYAML形式で表示されていますが、デフォルト設定ではJSONになります
