# アドオン・プラグイン

Movable Type には、アドオンやプラグインといった形で、機能を拡張できる仕組みが備わっています。

- $MT_DIR/addons の下にあるのがアドオン
- $MT_DIR/plugins の下にあるのがプラグイン

と呼ばれます。プラグインは管理画面から有効/無効を切り替えられますが、アドオンはできません(ファイルやディレクトリを削除する必要があります)。アドオンもプラグインも、できることには大きな違いはありませんが、必ずアドオンが先に初期化されます。

PowerCMS は、ブログウェアとしての Movable Type に、エンタープライズ向け機能を追加するアドオン・プラグインのセットです。
また、Movable Type の機能のうち、カスタムフィールドはアドオンとして提供されています。`$MT_DIR/addons/Commercial.pack` が該当です。

# レジストリ

## Movable Type でのレジストリ

Movable Type は、「レジストリ」と呼ばれるハッシュ構造体に機能や設定などの情報を保持しています。具体的には、下記の `$core_registry` が参考になるでしょう。この変数に代入されているハッシュ構造体が、そのままレジストリとして保持されます。

- https://github.com/PowerCMS/platform/blob/master/lib/MT/Core.pm#L19

例えば、記事のオブジェクトはこのように、object_types で定義されています。

```
        object_types => {
            'entry'           => 'MT::Entry',
```

この定義により、例えば下記のようなデータベースの読み出し処理が可能になっています。

```
my $obj = MT->model( 'entry' )->load( $terms, $args );
```

※ この読み出し処理については、下記をあわせて参照してください。
- https://github.com/PowerCMS/Study/blob/master/MT%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E6%93%8D%E4%BD%9C.md

## プラグインによるレジストリへの追加

このハッシュ構造体の情報を、プラグインから追加することができます。プラグインでは、

- $MT_DIR/plugins/[PLUGIN_NAME]/config.yaml

にレジストリ情報を記述します。

例えば、オブジェクト「human」を追加したい場合は、下記のように記述します。

```
object_types:
  human: MT::Human
```

この定義により、先の $core_registry に、下記のように記述されていたのと同じ動作となります。これが、プラグインでレジストリを定義する、ということです。

```
        object_types => {
            'entry'           => 'MT::Entry',
            'human'           => 'MT::Human',
```

config.yaml にオブジェクト「human」を定義したことで、下記のようなデータベースの読み出し処理が可能になります。

```
my $obj = MT->model( 'human' )->load( $terms, $args );
```

# プラグインでできること

## レジストリ定義の拡張

Movable Type でレジストリ定義されていることは概ねできます。言い換えれば、ほとんどなんでもできる、ということでもあります。

- ページの追加
- オブジェクトの定義
- 環境変数の追加
- 左メニューの追加
- 一覧画面の定義
- テンプレートタグの追加
- タスクでの定期実行処理の追加

など

## コールバックを利用した処理

コールバックは、Movable Type 本体が、特定のタイミングでプラグインの処理を呼び出してくれるものです。そのタイミングは、ソースコードを `run_callbacks` で検索するとわかります。  

コールバックはプラグインから利用するための仕組みなので、Movable Type 本体ではレジストリ定義されていません。例えば下記のようなコールバックがあります。

- アプリケーション初期化時(init_app、init_request、pre_run、post_run など)
  - レジストリを動的に追加
  - 環境変数を動的に変更

- テンプレートのビルド時(template_source、template_param、template_output など)
  - 記事編集画面や、一覧画面で、HTML の <head> に新しい css の読み込みを埋め込む
  - 任意のテンプレート変数を追加する

プラグインでは、どのコールバックを使って、何の処理をする、の形で config.yaml に定義します。下記をあわせて参照してください。
- https://github.com/PowerCMS/Study/blob/master/%E3%82%B3%E3%83%BC%E3%83%AB%E3%83%90%E3%83%83%E3%82%AF%E3%81%AB%E3%82%88%E3%82%8B%E7%AE%A1%E7%90%86%E7%94%BB%E9%9D%A2%E6%93%8D%E4%BD%9C.md

# プラグイン開発時の注意点

- 既存の処理で、類似の機能がないか探し、あれば参考にするようにしてください。

# 参考になるドキュメント

- Movable Type が公開している開発者ガイド
  - https://www.movabletype.jp/documentation/mt6/developer/
