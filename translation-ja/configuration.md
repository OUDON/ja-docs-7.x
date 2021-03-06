# 設定

- [イントロダクション](#introduction)
- [環境設定](#environment-configuration)
    - [環境変数タイプ](#environment-variable-types)
    - [環境設定の取得](#retrieving-environment-configuration)
    - [現在環境の決定](#determining-the-current-environment)
    - [デバッグページの環境変数非表示](#hiding-environment-variables-from-debug)
- [設定値へのアクセス](#accessing-configuration-values)
- [設定キャッシュ](#configuration-caching)
- [メンテナンスモード](#maintenance-mode)

<a name="introduction"></a>
## イントロダクション

Laravelフレームワークの全設定ファイルは、`config`ディレクトリに保存されています。各オプションには詳しいコメントが付いているので、各ファイルを一読し、使用できるオプションを把握しておきましょう。

<a name="environment-configuration"></a>
## 環境設定

アプリケーションを実行している環境にもとづき、別の設定値に切り替えられると便利です。たとえば、ローカルと実働サーバでは、異なったキャッシュドライバを使いたいことでしょう。

これを簡単にできるようにするため、LaravelではVance Lucas氏により作成された、[DotEnv](https://github.com/vlucas/phpdotenv) PHPライブラリーを使用しています。新たにLaravelをインストールすると、アプリケーションのルートディレクトリには、`.env.example`ファイルが含まれています。ComposerによりLaravelをインストールした場合は自動的に、このファイルは`.env`に名前が変更されます。Composerを使わずにインストールした場合は、名前を変更してください。

`.env`ファイルは、アプリケーションのソースコントロールに含めるべきでありません。各ユーザー／サーバは異なった環境設定が必要だからです。さらに、侵入者がソースコントロールリポジトリへアクセスすることが起きれば、機密性の高い情報が漏れてしまうセキュリティリスクになります。

チーム開発を行っている場合、`.env.example`ファイルをアプリケーションに含めたいと思うでしょう。サンプルの設定ファイルに、プレースホルダーとして値を設定しておけば、チームの他の開発者は、アプリケーションを実行するために必要な環境変数をはっきりと理解できるでしょう。さらに、`.env.testing`ファイルを作成することもできます。このファイルは、PHPUnitテスト実行時やArtisanコマンドへ`--env=testing`オプションを指定した場合に、`.env`ファイルをオーバーライドします。

> {tip} `.env`ファイルにあるすべての変数は、サーバレベルやシステムレベルで定義されている、外部の環境変数によってオーバーライドすることができます。

<a name="environment-variable-types"></a>
### 環境変数タイプ

`.env`ファイル中の全変数は、文字列としてパースされます。`env()`関数でさまざまなタイプを返すために、予約語があります。

`.env`値  | `env()`値
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

空白を含む値を環境変数に定義する場合は、ダブル引用符で囲ってください。

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### 環境設定の取得

このファイルにリストしている値は、アプリケーションがリクエストを受け取った時点で、`$_ENV` PHPスーパーグローバル変数へロードされます。しかし、設定ファイルの変数を`env`ヘルパを使用して、値を取得できます。実際にLaravelの設定ファイルを見てもらえば、このヘルパで多くのオプションが使われているのに気がつくでしょう。

    'debug' => env('APP_DEBUG', false),

`env`関数の第２引数は「デフォルト値」です。この値は指定したキーの環境変数が存在しない場合に返されます。

<a name="determining-the-current-environment"></a>
### 現在環境の決定

現在のアプリケーション環境は、`.env`ファイルの`APP_ENV`変数により決まります。`APP`[ファサード](/docs/{{version}}/facades)の`environment`メソッドにより、この値へアクセスできます。

    $environment = App::environment();

指定した値と一致する環境であるかを確認するために、`environment`メソッドへ引数を渡すこともできます。必要であれば、複数の値を`environment`メソッドへ渡せます。値のどれかと一致すれば、メソッドは`true`を返します。

    if (App::environment('local')) {
        // 環境はlocal
    }

    if (App::environment(['local', 'staging'])) {
        // 環境はlocalかstaging
    }

> {tip} 現在のアプリケーション環境は、サーバレベルの`APP_ENV`環境変数によりオーバーライドされます。これは同じアプリケーションを異なった環境で実行する場合に便利です。特定のホストに対し、サーバの設定で適切な環境を指定できます。

<a name="hiding-environment-variables-from-debug"></a>
### デバッグページの環境変数非表示

例外が補足されず、`APP_DEBUG`環境変数が`true`になっていると、すべての環境変数とその内容がデバッグページに表示されます。特定の変数は非表示にしたい場合があるでしょう。`config/app.php`設定ファイルの`debug_blacklist`オプションを更新してください。

いくつかの変数は、環境変数とサーバ／リクエストデータの両方で利用できます。そのため、`$_ENV`と`$_SERVER`両方のブラックリストへ登録する必要があります。

    return [

        // ...

        'debug_blacklist' => [
            '_ENV' => [
                'APP_KEY',
                'DB_PASSWORD',
            ],

            '_SERVER' => [
                'APP_KEY',
                'DB_PASSWORD',
            ],

            '_POST' => [
                'password',
            ],
        ],
    ];

<a name="accessing-configuration-values"></a>
## 設定値へのアクセス

アプリケーションのどこからでもグローバルの`config`ヘルパ関数を使用し、設定値へ簡単にアクセスできます。設定値はファイルとオプションの名前を含む「ドット」記法を使いアクセスします。デフォルト値も指定でき、設定オプションが存在しない場合に、返されます。

    $value = config('app.timezone');

    // 設定値が存在しない場合、デフォルト値を取得する
    $value = config('app.timezone', 'Asia/Seoul');

実行時に設定値をセットするには、`config`ヘルパへ配列で渡してください。

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 設定キャッシュ

アプリケーションをスピードアップさせるために、全設定ファイルを一つのファイルへまとめる、`config:cache` Artisanコマンドを使ってください。これによりアプリケーションの全設定ファイルのオプションが、単一のファイルに結合され、フレームワークが素早くロードできるようになります。

一般的には、本番環境へのデプロイ作業の一環として、`php artisan config:cache`コマンドを実行すべきでしょう。アプリケーションの開発期間中は設定が頻繁に変更されることも多いので、ローカルでの開発中にこのコマンドを実行してはいけません。

> {note} 開発過程の一環として`config:cache`コマンド実行を採用する場合は、必ず`env`関数を設定ファイルの中だけで使用してください。設定ファイルがキャッシュされると、`.env`ファイルはロードされなくなり、`env`関数の呼び出しはすべて`null`を返します。

<a name="maintenance-mode"></a>
## メンテナンスモード

アプリケーションをメンテナンスモードにすると、アプリケーションに対するリクエストに対し、すべてカスタムビューが表示されるようになります。アプリケーションのアップデート中や、メンテナンス中に、アプリケーションを簡単に「停止」状態にできます。メンテナンスモードのチェックは、アプリケーションのデフォルトミドルウェアスタックに含まれています。アプリケーションがメンテナンスモードの時、ステータスコード503で`MaintenanceModeException`例外が投げられます。

メンテナンスモードにするには、`down` Artisanコマンドを実行します。

    php artisan down

`down`コマンドには、`message`と`retry`オプションを付けることもできます。`message`の値はカスタムメッセージを表示、もしくはログするために使用し、`retry`の値はHTTPヘッダの`Retry-After`としてセットされます。

    php artisan down --message="Upgrading Database" --retry=60

コマンドの`allow`オプションを使用し、メンテナンスモードであっても、アプリケーションへアクセスを許すIPアドレスやネットワークを指定できます。

    php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16

メンテナンスモードから抜けるには、`up`コマンドを使います。

    php artisan up

> {tip} `resources/views/errors/503.blade.php`を独自に定義することにより、メンテナンスモードのデフォルトテンプレートをカスタマイズできます。

#### メンテナンスモードとキュー

アプリケーションがメンテナンスモードの間、[キューされたジョブ](/docs/{{version}}/queues)は実行されません。メンテナンスモードから抜け、アプリケーションが通常状態へ戻った時点で、ジョブは続けて処理されます。

#### メンテナンスモードの代替

メンテナンスモードでは、アプリケーションがその間ダウンタイムになってしまいますので、Laravelでの開発でゼロダウンタイムを実現する[Envoyer](https://envoyer.io)のような代替サービスを検討してください。
