基本的には、他の人に見てもらえる環境を使うこと
  - AWS デモ環境
  - Azure のサブスクリプション
まずは、ツールコマンド
  - SSH ログインする
  - タスクを実行する
    - Apache で実行
  - システムログを確認
サンプルツール
  - https://github.com/PowerCMS/PowerCMS/blob/develop/powercms/tools/sample-tool
  - デバッグの仕方
    - システムログ
      - MT->log( 'test' );
    - 変数のダンプ
      use Data::Dumper; Dumper( $obj );
      MT::Util::YAML::Dump( $hash );

MT オブジェクトについて
  https://github.com/movabletype/Documentation/wiki/Japanese-plugin-dev-3-4
  my $entry = MT->model( 'entry' )->load( { id => 1 } );
    - terms
      - ダイレクトな条件指定
        - foo => 1
          - foo カラムの値が 1
        - foo => { op => '>', value => '1' }
          - foo カラムの値が 1 より大
        - foo => [ 1, 2, 3, 4 ]
          - foo カラムの値が  1 か 2 か 3 か 4
        - foo => ['201103010000', '201104010000']
          - foo カラムの値が 201103010000 以上 201104010000 以下
          - args で range の指定が必要
    - args
      - オプション
        - 件数制限
        - 順番
        - このカラムは範囲指定あつかい
        - join
  $entry->title;
  phpMyAdmin 等でデータの構造を把握しておくこと
  カスタムフィールドは別
    - field.foo
プラグインの形
  config.yaml
  コールバックの指定
    - cms_post_save.entry
      - パラメータの取得
        - my $app = MT->instance();
        - $app->param( 'foo' );
      - 環境変数の取得
        - MT->config->foo;
