==================================================
■ DynamicMTML
==================================================
スタティック・パブリッシングとダイナミック・パブリッシングの併用
● 特徴
HTML の中に MT タグが書ける
- どういう状況でも同じ HTML のものはあらかじめ出力しておき、変化するものだけ MT タグで記述する
PHP で記述されている
- CGI よりも軽い
-- MemoryLimit などの制限に引っかかりやすいので注意
-- 関数名が重複すると死ぬ
-- 配列の最後の要素の後ろにカンマをつけると死ぬ
- 逆に、複数の言語でハンドリングしなければならないデメリットも
-- ポリシーやルールを統一しておかなくてはならない
MT テンプレートで出力していなくても処理させることができる
PHP も処理される
--------------------
<?php
echo 'test';
?>
--------------------
● 簡単な使い方
インデックステンプレートに
--------------------
<MTRawMTML>
MTDate: [<MTDate>]
</MTRawMTML>
--------------------
と記述して、スタティックに再構築すると、ファイルの内容は
--------------------
MTDate: [<MTDate>]
--------------------
になる。これを DynamicMTML で処理させると、現在時刻が表示される。
● 陥りやすいミス
--------------------
<MTSetvarblock name="entry_ids">1,2,3</MTSetvarblock>
<MTRawMTML>
MTEntries: [<MTEntries id="$entry_ids">]
</MTRawMTML>
--------------------
これは、出力ファイルが
--------------------
MTEntries: [<MTEntries id="$entry_ids">]
--------------------
になる
--------------------
{{}}
--------------------
を使うと、Smarty 的にエラーになる
● 使いどころ
HTML コンテンツについて、動的表示が必要な場面
- バックエンド処理は基本的に CGI でさせること
※ 使わなくてよいなら使わない方がよい
● 動作の仕組み
1. http://example.com/index.html にアクセスがある
2. /var/www/html/.htaccess が処理を受け取る
3. mod_rewite により処理が /var/www/cgi-bin/mt/addons/DynamicMTML/php/dynamicmtml.run.php に渡される
4. dynamicmtml.run.php が、リクエスト内容からファイルを /var/www/html/index.html だと特定する
5. /var/www/html/index.html の内容を読み込み、ダイナミック・パブリッシングのテンプレートとしてビルドして表示する
==================================================
■ テンプレートタグの作り方
==================================================
●　ファンクションタグ
plugins/TestPlugin/php/function.mtfoo.php
--------------------
<?php
function smarty_function_mtfoo($args, &$ctx) {
    // $args はモディファイア
    // $ctx は Perl 版と同じと考えてよい
    //   $blog = $ctx->stash( 'blog' );
    //   $tag = $ctx->stash( 'tag' ); <= タグ名
    return 'foo!';
}
?>
--------------------
plugins/TestPlugin/configyaml
--------------------
tags:
  function:
    foo:
--------------------
● デバッグの手段
- echo 'test';
- print_r( $args ); // NOT OBJECT
- var_dump( $args );
- print_r( debug_backtrace() );
●　ユーザの取得
global $app; // DynamicMTML オブジェクトの取得
var_dump( $app->user ); // Perl 版と同じように取得できる
● オブジェクトの load
$terms = array( 'id' => 1, 'class' => array( 'website', 'blog' ) );
$extra = array( 'limit' => 1 );
$blog = $app->load( 'Blog', $terms, $extra );
● プラグイン設定
$mt = MT::get_instance();
// $app->mt()
$data = $mt->db()->fetch_plugin_data( 'TestPlugin', 'configuration:blog:' . $blog_id );
●　ブロックタグ
php/lib/block.mtblogs.php を見ろ
● 注意点
タグ名に if/has/header/footer/previous/next を含めないこと