==================================================
■ データベース内テーブルの拡張
==================================================

▼ 記事(mt_entry)に test_text1 カラムを追加する場合

  ● config.yaml の設定

    schema_version: 0.1
    object_types:
        entry:
            test_text1: string(255)

    - schema_version は初期化時にチェックされ、以前より上がっているとアップグレードが実行される
    - アップグレード時のコールバックもあるので、これを利用してデータの補正をするなど可能
      - version_limit に注意
    - schema_version の格納場所は mt_config テーブル

  コールバック MT::App::CMS::template_param.edit_entry を使って入力欄を追加すると、よしなにしてくれる
  ExtFields プラグインを field_loop で grep すると参考になる

  ● 使い方

    $entry->test_text1( 'test' );
    $entry->save or die $entry->errstr();


▼ 記事に test_meta1 カラムを追加する

  ● config.yaml の設定

    object_types:
        entry:
            test_meta1: string meta

    - この場合、schema_version 指定は不要

  ● 使い方

    $entry->test_meta1( 'test' );
    $entry->save or die $entry->errstr();


▼ 新しいオブジェクトを追加する

  schema_version: 0.1
  object_types:
      foo_obj: TestPlugin::Foo


  - plugins/TestPlugin/lib/TestPlugin/Foo.pm を作成
===========================================================
package TestPlugin::Foo;
use strict;

use base qw( MT::Object );
__PACKAGE__->install_properties( {
    column_defs => {
        'id' => 'integer not null auto_increment',
        'blog_id' => 'integer',
        'name' => 'string(255)',
        'text' => 'text',
    },
    indexes => {
        'blog_id' => 1,
        'name' => 1,
    },
    child_of => [ 'MT::Blog', 'MT::Website' ],
    datasource => 'foo', # < テーブル名
    primary_key => 'id',
    audit => 1,
} );

1;

===========================================================

  ● 使い方

    my $foo = MT->model( 'foo_obj' )->new(); # < object_types のキー
    $foo->blog_id( 1 );
    $foo->name( '名前' );
    $foo->text( 'テキスト' );
    $foo->save or die $foo->errstr();


==================================================
■ 一覧画面を表示する
==================================================

▼ 既存オブジェクトに追加の場合

  ● config.yaml の設定

    list_properties:
        entry:
            test_text1:
                label: Test Text1
                auto: 1
                order: 190

▼ 新規オブジェクトの場合

  ● config.yaml の設定

    まず左メニューに追加

      applications:
          cms:
              menus:
                  foo:
                      label: fooooo!
                      order: 10
                  foo:list_foo:
                      label: Manage
                      mode: list
                      order: 100
                      args:
                          _type: foo_obj
                      condition: |
                          sub { 1 }
                      view:
                          - user
                          - system
                          - blog
                          - website

    次に一覧画面を表示するための設定

        listing_screens:
            foo_obj: 
                object_label: Fooooo!
                primary: name
                default_sort_key: name
                condition: |
                    sub { 1 }
                view:
                    - user
                    - system
                    - blog
                    - website

    次に一覧画面での各カラムの設定

      list_properties:
          foo_obj:
              id:
                  base: __virtual.id
                  order: 100
              name:
                  label: Name
                  auto: 1
                  order: 200
              blog_name:
                  base: __common.blog_name
                  order: 300

