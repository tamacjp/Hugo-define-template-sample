Hugo の {{define}}-{{template}} を使う（サンプル）
==================================================

Hugoで  
__共通のコードはなるべく書かない。一箇所にまとめる（DRY）__  
という方針を進めるためにはどうしたらよいか。自分なりのひとつの答え。


## 方針 ##

* _single.html_ や _section.html_ など直接テンプレートとして用いられるHTMLファイルの名前空間（`.`）は
    `{{template}}` を呼び出すときに __.Config__ という名前で参照できるようにpipelineに辞書（dict）を渡す
    * これにより `{{define}}` の中や partial HTMLの中では共通して `{{.Config.Site.Title}}` などの名前でアクセスできる
    * またこの辞書には __.Title__ というページタイトルや __.Description__ というページ概要を含む
    * 例：`{{template "hoge" (dict "Config" . "Title" .Title "Description" .Description)}}`
        * __Config__ … テンプレートHTMLの名前空間全体 `.`
        * __Title__ … ページコンテンツで定義されている `.Title`
        * __Description__ … ページコンテンツで定義されている `.Description`
    * HTMLファイルの先頭でこの辞書を __$pipeline__ などの変数に入れておくと全ての `{{template}}` 呼び出しで使い回せる
    * `{{define}}` や partial の中からさらに `{{template}}` を呼び出す場合は単に名前空間（`.`）を継承するか、
        `(dict "Config" .Config)` などのように __Config__ が引き継がれるようにする 
* [_page-template.html_](layouts/page-template.html) に汎用的なページ構成とそこで用いるパーツを定義（`{{define}}`）する
    * `{{define "html-header"}}` … &lt;html&gt;〜&lt;body&gt; までのHTMLヘッダ
    * `{{define "page-header"}}` … 全ページ共通のヘッダ部分
    * `{{define "navigation"}}` … 一部のページで共通して含むナビゲーションバー
    * `{{define "page-footer"}}` … 全ページ共通のフッタ部分
    * `{{define "html-footer"}}` … HTMLフッタ &lt;/body&gt;&lt;/html&gt;
    * また、コンテンツページで共通して使う部分をさらに簡易化定義している
        * `{{define "common-header"}}` … 共通ヘッダ `"html-header"` + `"page-header"` + `"navigation"` 
        * `{{define "common-footer"}}` … 共通フッタ `"page-footer"` + `"html-footer"`
* トップページ [_index.html_](layouts/index.html) はナビゲーションバーが必要ないので
    `"html-header"` と `"page-header"` ならびに `"page-footer"` `"html-footer"` だけを呼び出す
    * また、ページタイトルは不要なので __$pipeline__ 辞書に __Title__ を含まない
* デフォルトの [_single.html_](layouts/_default/single.html) は
    共通ヘッダ表示、コンテンツ表示、共通フッタ表示だけを行うシンプルなものとなる
    * `{{define}}` に含まれない地の部分では __.Config.Content__ ではなく __.Content__ のような名前で変数を参照する
* セクション固有の _single.html_ （[例](layouts/news/single.html)）は
    デフォルトの [_single.html_](layouts/_default/single.html) と同様の構成で固有の表示を作れる
    * このとき共通部分（HTMLヘッダ〜ページヘッダ〜ナビゲーションバー、ページフッタ〜HTMLフッタ）は
     `{{template}}` 呼び出しだけなのでDRYに反するようなコードの再記述は必要ない
* デフォルトのリスト表示 [_section.html_](layouts/_default/section.html) も
    デフォルトの [_single.html_](layouts/_default/single.html) に似た構成だが、
    セクションリストの表示方法を `{{define "section-list"}}` として定義している
    * 定義した `"section-list"` を直後にすぐ `{{template "section-list" $pipeline}}` と呼び出す
    * 1件1件の表示自体も定義したり partial にしたほうが汎用性が高まる（今回はシンプルにするためそこまではしていない）
* セクション固有の _section.html_ にあたる _section/SECTION.html_（[例](layouts/section/tech.html)）は
    デフォルトの [_section.html_](layouts/_default/section.html) で定義した `"section-list"` を用いて
    リスト表示する部分は再記述することなく固有の表示も実現できる


## partial HTML と {{define}} ##

* partial HTML はパーツだけを個々のHTMLファイルにして `{{partial}}` で呼び出す
* `{{define}}` で定義したパーツは [layouts 配下のどこに配置してもよく](https://github.com/tamacjp/Hugo-define-template#readme)、
    `{{template}}` で呼び出す
* 名前空間（pipeline）の指定については変わりがない（と思われる）
* だったらファイルをバラバラと分けずひとつのHTMLやデフォルトのテンプレートの中に `{{define}}` 定義を置けたほうがいんでね？と思う次第
