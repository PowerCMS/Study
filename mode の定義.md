# mode の定義

URL パラメータが `?__mode=sample_methods` の場合に動作する処理を作成する。左メニューからアクセスしたり、プラグインアクションから実行される場合もある。

※ sample_methods を定義せずにアクセスすると「不明なアクション」としてエラーになる

```
# config.yaml
applications:
  cms:
    methods:
      sample_methods: $sampleplugin::SamplePlugin::CMS::_mode_sample_methods
```

```
## SamplePlugin::CMS
sub _mode_sample_methods {
  return 1;
}
```

この定義で `?__mode=sample_methods` にアクセスすると、「1」とだけ表示される。管理画面のビジュアルに即した画面を表示する場合、テンプレートファイルを用意し、header.tmpl や foooter.tmpl をインクルードする。

```
## SamplePlugin::CMS
sub _mode_sample_methods {
    # ここで必要な処理を実行

    # 実行結果を表示
    my $tmpl = 'sample_methods.tmpl';
    my %param;
    $param{ foo } = 'bar'; # テンプレート変数としてセット
    return $app->build_page( $tmpl, \%param );
}
```

```
## tmpl/sample_methods.tmpl
<mt:include name="include/header.tmpl" id="header_include">
<mt:var name="foo"> <!-- 「bar」が出力される -->
<mt:include name="include/footer.tmpl" id="footer_include">
```
