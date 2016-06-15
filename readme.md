# WordPress テーマ制作 ハンズオン

## はじめに

この手順・サンプルソースはWordPress実装初心者のノンプログラマー向けに、
短時間でざっくり理解しやすい事を前提に書かれた内容なので、
実際のサイトとしては機能不十分な面も多々ありますが予めご了承ください。

## ディレクトリの準備


1. wck-lesson-static フォルダを複製してフォルダ名を wck-lesson に変更
1. wck-lessonディレクトリを /wp-content/themes/ ディレクトリにアップ

## テーマを適用させる

WordPressのテーマには最低 style.css と index.php の２つのファイルが必要です。
まずはその２つのファイルを用意しましょう。

### style.cssを作る

WordPressではテーマの名前や説明などの基本情報をstyle.cssに記述します。
style.cssファイルを新規作成し、下記の記述例を貼り付けて、テーマの名前などを変更してみましょう。

テーマの style.css に最低限必要な記述例
~~~
@charset "utf-8";
/*
Theme Name: テーマの名前
Description: テーマの説明
Author: 作成者の名前
Author URI: http://www.xxxxx.xxx
License: GNU General Public License v2 or later
License URI: http://www.gnu.org/licenses/gpl-2.0.html
*/
~~~

### index.php を作る

とりあえず index.html のファイル名（拡張子）を index.php に変更して下さい。

### 確認してみよう

これで style.css と index.php は存在しますので、WordPressのテーマとして有効化する事ができます。
管理画面の「外観」→「テーマ」を確認してみましょう。
style.cssに記述した内容のテーマがあるはずです。
これを有効化してみましょう。

WordPressのテーマとしては適用されますが、CSSや画像がリンク切れになっているはずです。

今までindex.htmlのあったファイルを基準に相対パスで参照していたのに対して、
WordPressで設定されている公開URLでページを表示するようになったので、
公開URLからの相対パスの位置にはCSSファイルや画像ファイルがないからです。

## トップビジュアル画像のリンク切れの修正

まずはトップの画像がリンク切れになっているのを修正します。  
トップビジュアルは現状  

~~~
<img id="top_visual_image" class="thumbnail" src="images/top_visual.png" alt="" />
~~~

と書かれていますが、WordPressのテーマがあるディレクトリまでのURLは <?php echo get_template_directory_uri(); ?> で出力する事ができますので、下記のように書き換えます。

~~~
<img id="top_visual_image" class="thumbnail" src="<?php echo get_template_directory_uri(); ?>/images/top_visual.png" alt="" />
~~~

これで画像が正しく表示されるようになる事が確認できます。


## css と js の再設定

画像と同様に、cssファイルやjsファイルの参照先が静的htmlファイルの時とは異なるので、cssやjavascriptが適用されていない状態になっていますので修正します。

### wp_head() と wp_footer() の設定

index.php の headタグの閉じタグの直前に wp_head(); 関数を記述します。

~~~
<?php wp_head();?>
</head>
<body>
~~~

同様に index.php の body の閉じタグの直前に wp_footer(); 関数を記述します。

~~~
<?php wp_footer();?>
</body>
</html>
~~~

#### 補足 : wp_head() と wp_footer()

WordPressでは例えば画像をポップアップ表示するプラグインとして有名なLightbox系のプラグインをインストールして有効化すると、テーマファイルに変更を加えていないのに、公開されているサイトのHTMLを見てみるとLightboxのjavascriptやcssが読み込まれていて正常に動作するようになります。  
これはLightboxのjsファイルやcssファイルが wp_head() や wp_footer() 関数を経由して出力されているからです。  
よって、この関数が適正な位置に記載されていないとWordPressのテーマとして正しく動作しません。

### cssの出力

現在、javascriptやcssファイルの読み込みが index.php に直接記載されていますが、WordPressでは先ほど記載した wp_head() や wp_footer() を経由して出力します。

まず lesson ディレクトリの中に functions.php というファイル名でファイルを作成します。

~~~
<?php 
function lesson_theme_scripts(){
	// テーマディレクトリにある style.css を出力
	wp_enqueue_style( 'lesson_theme_style', get_stylesheet_uri() );

	// テーマ用のCSSを読み込む
	wp_enqueue_style( 'lesson-css-bootstrap', get_template_directory_uri() . '/css/bootstrap.min.css', array(), '3.3.6' );
	wp_enqueue_style( 'lesson-css', get_template_directory_uri() . '/css/wck_style.css', array('lesson-css-bootstrap'), '1.0' );
}
add_action( 'wp_enqueue_scripts', 'lesson_theme_scripts' );
~~~

これでサイトを見てみましょう。CSSが適用されているのが確認出来るはずです。

もともと直接記載してあった下記のコードを削除しておきましょう。
~~~
<!-- [ Load CSS ] -->
<link href="css/bootstrap.min.css" rel="stylesheet">
<link href="css/wck_style.css" rel="stylesheet">
<!-- [ /Load CSS ] -->
~~~

#### 補足 : なぜ head に直書きでなく enqueue を使うのか？

##### 正しい順番で読み込む

いろいろなjsやcssファイルを読み込む時、正しい順番で読み込まないと正常に動作しない事があります。  
例えばjQueryのプログラムがその代表で、元となるjQueryのファイルの後で読み込まなくては正しく動作しません。  
今回のCSSファイルについてはCSSフレームワークのBootstrapのCSSを読み込んだ後で、固有のCSSを読み込む必要があります。
そこで、読み込むファイルが別のどのファイルに依存しているかを指定する事によって、WordPressが適切な順番で出力してくれます。  

<?php wp_enqueue_style( ハンドル名, 参照先, 依存するハンドル名, バージョン, メディア ); ?>

##### プラグインなど外部から操作出来るようになる

テーマで読み込まれているjsファイルやcssファイルを外部のプラグインなどから解除したい時に wp_deregister_script() や wp_deregister_style() を使えば、テーマを改変する事なく読み込みを解除する事ができるようになります。

### jsファイルの出力

CSSが正しく適用されたら次はjsファイルを読み込みます。
先ほどの lesson_theme_scripts() 関数の中に下記を追記します。

~~~
// テーマ用のjsを読み込む
wp_enqueue_script( 'lesson-js-bootstrap', get_template_directory_uri() . '/js/bootstrap.min.js', array( 'jquery' ), '20160710', true );
~~~

全体では下記にようになります。

~~~
<?php 
function lesson_theme_scripts(){
	// テーマディレクトリにある style.css を出力
	wp_enqueue_style( 'lesson_theme_style', get_stylesheet_uri() );

	// テーマ用のCSSを読み込む
	wp_enqueue_style( 'lesson-css-bootstrap', get_template_directory_uri() . '/css/bootstrap.min.css', array(), '3.3.6' );
	wp_enqueue_style( 'lesson-css', get_template_directory_uri() . '/css/wck_style.css', array('lesson-css-bootstrap'), '1.0' );

	// テーマ用のjsを読み込む
	wp_enqueue_script( 'lesson-js-bootstrap', get_template_directory_uri() . '/js/bootstrap.min.js', array( 'jquery' ), '20160710', true );
}
add_action( 'wp_enqueue_scripts', 'lesson_theme_scripts' );
~~~

これで wp_footer()経由でjsファイルが出力されるので、bodyの閉じタグの直前に記載してある下記のコードは削除します。

~~~
<!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
<!-- Include all compiled plugins (below), or include individual files as needed -->
<script src="js/bootstrap.min.js"></script>
~~~

これでjavascriptも正しく動作しているのが確認出来るはずです。

## タイトルタグの設定

続いてSEOでも重要になる head の title タグを設定します。

これも wp_head() から出力してしまうので、
index.php に記載されている下記の title タグは必要ありませんので削除します。

~~~
<title>WordCamp Kansai 2016</title>
~~~

それから wp_head() から出力するための記述を functions.php に追加します。

~~~
function lesson_theme_title() {
    add_theme_support( 'title-tag' );
}
add_action( 'after_setup_theme', 'lesson_theme_title' );
~~~

これで自動的に head 内に wp_head() を経由して title タグが出力されます。 


#### 補足 : title で出力される中身を任意に改変したい

titleタグに出力される文言は非常に重要なので、自分の思うように変更したい人は多いと思いますが、プラグインを使えば簡単に設定できます。

* [Yoast SEO](https://ja.wordpress.org/plugins/wordpress-seo/)
* [All in One SEO Pack](https://ja.wordpress.org/plugins/all-in-one-seo-pack/)

使い方は「 [Yoast SEO 使い方](https://www.google.co.jp/search?q=Yoast+SEO+%E4%BD%BF%E3%81%84%E6%96%B9&oq=Yoast+SEO+%E4%BD%BF%E3%81%84%E6%96%B9&aqs=chrome..69i57.362j0j7&sourceid=chrome&ie=UTF-8) 」などで検索してみてください。

## ヘッダーのタイトル／リンク／キャッチフレーズを設定

■サイト名を表示  
<?php echo get_bloginfo( 'name' ); ?>

■サイトURLを表示  
<?php echo home_url( '/' ); ?>

■ヘッダー上部にディスクリプションを設定  
<?php echo get_bloginfo( 'description' ); ?>

下記のように変更してください。

~~~
<h1 class="wck_head_logo"><a href="<?php echo home_url( '/' ); ?>"><?php echo get_bloginfo( 'name' ); ?></a></h1>
<h2 class="wck_head_description"><?php echo get_bloginfo( 'description' ); ?></h2>
~~~

## 投稿のループの作成

ブログではトップページやカテゴリーページに、記事が新しい順で数件表示されているのが一般的です。
ここではWordPressに登録されている投稿を呼び出す記述をします。

index.php のコメントの『記事のループ』の直後に下記を貼り付けます。

~~~
<!-- [ 記事のループ ] -->
<?php if ( have_posts() ) { ?>
<?php while( have_posts() ) : the_post(); ?>
<article>★ 投稿の内容が繰り返されます ★</article>
<?php endwhile; ?>
<?php } ?>
~~~

これで確認すると、WordPressに登録されている投稿の数だけ  
『★ 投稿の内容が繰り返されます ★』  
が表示されるはずです。

続いて、

~~~
<article>★ 投稿の内容が繰り返されます ★</article>
~~~
の部分を、その後ろの article の1件分の内容に差し替え、それ以外の article の部分は削除します。

### 投稿の情報を反映させる

article の中の投稿のタイトル名などを下記を参考に書き換えてください。

■ 投稿のタイトル名  
<?php the_title(); ?>

■ 記事詳細ページへのリンク
<?php the_permalink(); ?>

■ 記事の投稿日  
<?php the_date(); ?>

■ 記事の投稿者  
<?php the_author(); ?>

■ 記事のカテゴリー名とリンク 
<?php the_category(','); ?>

■ 記事の本文  
<?php the_content(); ?>

これでWordPressの管理画面で投稿されている内容が反映されるのが確認できます。

## ページングの設定

投稿のループの終わりである <?php endwhile; ?> の後に  

~~~
<?php the_posts_pagination(); ?>
~~~

を記述すると、ページ送りが表示されます。

## サイドバーの設定（カテゴリーリスト/アーカイブリスト）

サイドバーはコメントで [ #sub ] と書いてある部分に記述してあります。

### カテゴリーリスト

サイドバーに表示されているカテゴリーリストは

~~~
<h4 class="wck_sub_section_title">カテゴリー</h4>
<ul>
<li><a href="#">カテゴリー1</a></li>
<li><a href="#">カテゴリー2</a></li>
<li><a href="#">カテゴリー3</a></li>
</ul>
~~~

と記載されていますが、  
<?php wp_list_categories(); ?>  
関数を使うと、カテゴリーの一覧とリンクを取得できるので下記のように書き換えます。

~~~
<h4 class="wck_sub_section_title">カテゴリー</h4>
<ul>
<?php wp_list_categories('title_li='); ?>
</ul>
~~~

### 月別アーカイブリスト

カテゴリー同様に

~~~ 
<h4 class="wck_sub_section_title">月別</h4>
<ul>
<li><a href="#">2016年7月</a></li>
<li><a href="#">2016年6月</a></li>
<li><a href="#">2016年5月</a></li>
</ul>
~~~

と記載されている箇所を

~~~
<h4 class="wck_sub_section_title">月別</h4>
<ul>
<?php wp_get_archives('type=monthly'); ?>
</ul>
~~~

に書き換えます。これで月別アーカイブへのリストが出力されます。

## グローバルメニューにカスタムメニューを設定する

まず、カスタムメニューが利用出来るように functions.php に下記を貼り付けます。

~~~
function lesson_theme_custom_menu() {
    register_nav_menus( array( 'Header Navigation' => 'Header Navigation', ) );
}
add_action( 'after_setup_theme', 'lesson_theme_custom_menu' );
~~~

これで管理画面の「外観」→「メニュー」から Header Navigation というメニューエリアにメニューを設定する事が出来るようになります。

しかし、これだけでは公開されているサイトには反映されません。

ヘッダーのメニューに該当する下記の部分を...

~~~
<ul class="nav navbar-nav">
  <li><a href="#">Home</a></li>
  <li><a href="#">Page A</a></li>
  <li><a href="#">Page B</a></li>
  <li><a href="#">Page C</a></li>
</ul>
~~~

カスタムメニューを表示する wp_nav_menu() 関数を使って下記のように呼び出します。

~~~
<?php
$args = array(
    'theme_location' => 'Header Navigation',
    'items_wrap'     => '<ul id="%1$s" class="%2$s nav navbar-nav wck_nav">%3$s</ul>',
);
wp_nav_menu( $args ) ;
?>
~~~

これでカスタムメニューが反映されます。

- - -

# 時間があれば

## パンくずリストの作成（条件分岐）

トップの画像の下にあるパンくずリストについて、ページによって表示内容が変わるように設定していきましょう。

まずは HOME のリンク先を <?php echo home_url( '/' ); ?> に変更します

### 詳細ページでのみ表示する条件分岐

次に HOME の次のカテゴリー名と投稿のタイトルは、投稿の詳細ページでしか必要ないので、

~~~
<?php if ( is_single() ) { ?>
<li><a href="#">カテゴリー名</a></li>
<li>投稿のタイトル</li>
<?php } ?>
~~~

とします。

if ( is_single() ) で投稿の詳細ページの場合のみ実行するという条件分岐ができます。

トップページのパンくずでは HOME だけが表示されて、記事のタイトルをクリックして投稿の詳細ページに移動すると、カテゴリー名と投稿タイトルまで表示されている事が確認できます。

『カテゴリー名』の部分は a タグも含めて <?php the_category(','); ?> タグに置き換え、
『投稿タイトル』の部分を <?php the_title();?> に書き換えます。

~~~
<ol class="breadcrumb">
  <li><a href="<?php echo home_url( '/' ); ?>"><span class="glyphicon glyphicon-home" aria-hidden="true"></span>HOME</a></li>
  <?php if ( is_single() ) { ?>
  <li><?php the_category(','); ?></li>
  <li><?php the_title();?></li>
  <?php } ?>
</ol>
~~~

これで投稿の詳細ページでカテゴリー名や記事タイトルが反映されているのが確認出来ます。

### アーカイブページでの条件分岐

次に、カテゴリーや投稿月別のアーカイブページでの条件分岐を追加します。

~~~
<?php if ( is_single() ) { ?>
~~~

の部分を

~~~
<?php if ( is_archive() ) { ?>
<li><?php the_archive_title(); ?></li>
<?php } else if ( is_single() ) { ?>
~~~

に置き換えます。

<?php if ( is_archive() ) { ?> タグで カテゴリー別や月別などのアーカイブの時の条件分岐をして、<?php the_archive_title(); ?> タグでアーカイブタイトルを出力しています。

他にも様々な条件分岐タグがあるので、調べてみてください。

[条件分岐タグ - WordPress Codex 日本語版](https://wpdocs.osdn.jp/%E6%9D%A1%E4%BB%B6%E5%88%86%E5%B2%90%E3%82%BF%E3%82%B0)

## トップビジュアルとパンくずの表示切り替え（条件分岐）

### トップビジュアルがトップページでだけ表示できるように

ヘッダーのメニューの下の画像をトップページでだけ表示するようにします。

下記のように <?php if ( is_front_page() ) { ?> と <?php } ?> で囲います。

~~~
<?php if ( is_front_page() ) { ?>
<!-- [ トップビジュアル ] -->
（略）
<!-- [ /トップビジュアル ] -->
<?php } ?>
~~~

そうすると、トップページでのみ囲われている部分を実行します。

### パンくずの表示に関する条件分岐

パンくずリストはトップのビジュアルとは逆にトップページ以外でのみ表示してみます。

先ほど書いたトップビジュアルの下記の部分を

~~~
<!-- [ /トップビジュアル ] -->
<?php } ?>
<!-- [ パンくずリスト ] -->
~~~

else を使って下記のように条件分岐を追加します。

~~~
<!-- [ /トップビジュアル ] -->
<?php } else { ?>
<!-- [ パンくずリスト ] -->
（略）
<!-- [ /パンくずリスト ] -->
<?php } ?>
~~~

これによって、is_front_page() に該当しなかった場合に、次の { } で囲われた部分（パンくずリスト）を実行します。

トップページでは画像が、記事の詳細ページではパンくずが表示される事が確認できるはずです。

## もっと詳しく知りたい方は・・・

今回のテーマは限られた時間の中で出来そうな事に留めていますので、もっと興味がある人はいろいろ調べてみてください。

Thank you.



