# my-docker-compose

## Description
docker-composeでサクッとWordPress開発環境を構築する

## Requirement
- docker 17.03.1-ce
- docker-compose version 1.11.2

https://docs.docker.com/docker-for-mac/

## Usage
### ディレクトリをつくる
```
$ mkdir my-wordpress
$ cd my-wordpress

$ mkdir .docker/backup .docker/data .docker/log
```
- backup: WordPressプラグインで作成されるデータのバックアップ先
- data:   WordPressが利用するデータベース(MySQL)のデータ格納先
- log:    WordPressのアプリケーションログ格納

### docker-compose.ymlを編集する

- 各containerをサービスとして定義する。
mysqlとWordPressの二つのcontainerを定義する。
 
- networkは、アプリケーションに対しデフォルトで１つ設定されている。
各containerは、networkに参加することで同一network上の他のcontainerから接続できるようになる。
明示的に名前をつけるために、デフォルトのアプリケーション用のnetworkを使っていない。
 
- volumeには、ローカルのファイルとdocker container内を同期する設定を書く。
アプリケーションに必要なすべてのファイルをローカルの/.docker以下に同期している。

### WordPressとMySQLの共有networkをつくる
```
$ docker network create --driver bridge my_wordpress_link
```

### docker containerを起動する
docker-compose.ymlのあるディレクトリで下記コマンドを叩くと、containerが起動しアプリケーションがbuildされる。
```
$ cd my-wordpress-pass/
$ docker-compose up -d

$ docker container ls -a // containerが期待通り起動したか確認する 
```

### localhostでアクセスして初期設定をする
http://localhost:1003

1. 日本語を選択する
 
2. データベース設定をする

docker-compose.ymlに合わせ、以下のように設定をする。ここで設定したものをもとにwp-config.phpがつくられる。
```
データベース名: wordpress
ユーザー名: wordpress
パスワード: wordpress
データベースのホスト名: db:3306
```
開発環境を想定して簡単なパスワードにしている。

3. WordPressの設定をする
あとで管理画面から変更できるが、パスワードを忘れると管理画面にログインできなくなるので注意する。
```
サイトのタイトル: My WordPress Site
ユーザー名: { 自由につける }
パスワード: { 忘れないものにする }
メールアドレス: { メール }
```

### wp-configを書き換える
#### 環境設定を追記する
- 環境はdevelop/stage/productionの3種類定義している。
docker-compose.ymlのenvironmentに、MY_WORDPRESS_ENVIRONMENTという名前で記載している。
 
- MY_WORDPRESS_ENVIRONMENTという名前を指定し、定数を定義する。
- mywp_get_environmentというメソッドを用いて環境変数を取得する。
```
/**
 * Environment
 *
 * develop
 * staging
 * production
 */
define( 'MY_WORDPRESS_ENVIRONMENT', mywp_get_environment( 'MY_WORDPRESS_ENVIRONMENT' ) ); 
```

- mywp-get-environmentメソッドは、パラメータにした環境変数の値を返す関数である。
```
function mywp_get_environment( $name ) {

  return getenv( $name ) ? getenv( $name ) : '';
}
```

#### 開発用にデバッグ設定を追記する
```
/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the Codex.
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */

define( 'WP_DEBUG', true );

if ( WP_DEBUG ) {

  define( 'WP_DEBUG_LOG', true );              // エラーをdebug.logに書き出す
  if ( MY_WORDPRESS_ENVIRONMENT == 'production' ) { // 本番環境ではエラーをブラウザに表示しない

    define( 'WP_DEBUG_DISPLAY', false );@ini_set( 'display_errors', 0 );
  }
} 
```

以下のものを設定する。

```
define( 'WP_HOME',          mywp_get_environment( 'WORDPRESS_URL' ) );
define( 'WP_SITEURL',       mywp_get_environment( 'WORDPRESS_URL' ) );
define( 'FORCE_SSL_ADMIN',  false );
define( 'SAVEQUERIES',      true );
```

WP_HOME: ブログアドレス(URL)を定義する。
WP_SITEURL: WordPressのコアファイルが存在するURLを定義する。
FORSE_SSL_ADMIN: 管理画面とログイン画面にSSL通信を要求するかどうか定義する。
SAVEQUERIES: 分析のためにデータベースクエリを配列に保存し表示させる(開発環境以外ではfalseにすること)

## Author
mom0tomo