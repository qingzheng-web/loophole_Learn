クラスのオートローディング
==========================

Yiiは、必要となるすべてのクラス・ファイルを特定してインクルードするにあたり、
[クラスのオートローディング・メカニズム](https://secure.php.net/manual/ja/language.oop5.autoload.php) を頼りにします。
Yii は、[PSR-4 標準](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) に準拠した、高性能なクラスのオートローダを提供しています。
このオートローダは、あなたが `Yii.php` ファイルをインクルードするときにインストールされます。

> Note: 説明を簡単にするため、このセクションではクラスのオートローディングについてのみ話します。しかし、
  ここに記述されている内容は、インタフェイスとトレイトのオートローディングにも同様に適用されることに注意してください。


Yii オートローダを使用する <span id="using-yii-autoloader"></span>
--------------------------

Yii のクラス・オートローダを使用するには、クラスを作成して名前を付けるとき、次の二つの単純なルールに従わなければなりません:

* 各クラスは [名前空間](https://secure.php.net/manual/ja/language.namespaces.php) の下になければなりません (例 `foo\bar\MyClass`)
* 各クラスは次のアルゴリズムで決定される個別のファイルに保存されなければなりません:

```php
// $className は先頭にバック・スラッシュを持たない完全修飾クラス名
$classFile = Yii::getAlias('@' . str_replace('\\', '/', $className) . '.php');
```

たとえば、クラス名と名前空間が `foo\bar\MyClass` であれば、対応するクラス・ファイルのパスの [エイリアス](concept-aliases.md) は、
`@foo/bar/MyClass.php` になります。このエイリアスがファイル・パスとして解決できるようにするためには、`@foo` または `@foo/bar`
のどちらかが、 [ルート・エイリアス](concept-aliases.md#defining-aliases) でなければなりません。

[ベーシック・プロジェクト・テンプレート](start-installation.md) を使用している場合、最上位の名前空間 `app` の下にクラスを置くことができ、
そうすると、新しいエイリアスを定義しなくても、Yii によってそれらをオートロードできるようになります。これは `@app`
が [事前定義されたエイリアス](concept-aliases.md#predefined-aliases) であるためで、`app\components\MyClass` のようなクラス名を
今説明したアルゴリズムに従って、クラス・ファイル `AppBasePath/components/MyClass.php` であると解決することが出来ます。

[アドバンスト・プロジェクト・テンプレート](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide-ja/README.md) では、各層がそれ自身のルート・エイリアスを持っています。たとえば、
フロントエンド層はルート・エイリアス `@frontend` を持ち、バックエンド層のルート・エイリアスは `@backend` です。その結果、名前空間 `frontend` の下に
フロントエンド・クラスを置き、バックエンド・クラスを `backend` の下に置けます。これで、これらのクラスは Yii のオートローダによって
オートロードできるようになります。

独自の名前空間をオートローダに追加するためには、[[Yii::setAlias()]] を使って、その名前空間のベース・ディレクトリに対するエイリアスを定義する必要があります。
例えば、`path/to/foo` ディレクトリに配置されている `foo` 名前空間に属するクラスをロードするためには、`Yii::setAlias('@foo', 'path/to/foo') を呼び出します。 

クラス・マップ <span id="class-map"></span>
--------------

Yii のクラス・オートローダは、 *クラス・マップ* 機能をサポートしており、クラス名を対応するクラス・ファイルのパスにマップできます。
オートローダがクラスをロードするときは、クラスがマップに見つかるかどうかを最初にチェックします。もしあれば、対応する
ファイル・パスは、それ以上チェックされることなく、直接インクルードされます。これでクラスのオートローディングを非常に高速化できます。
実際のところ、すべての Yii のコア・クラスは、この方法でオートロードされています。

次の方法で、 `Yii::$classMap` に格納されるクラス・マップにクラスを追加できます:

```php
Yii::$classMap['foo\bar\MyClass'] = 'path/to/MyClass.php';
```

クラス・ファイルのパスを指定するのに、 [エイリアス](concept-aliases.md) を使うことができます。クラスが使用される前にマップが準備できるように、
クラス・マップの設定は [ブートストラップ](runtime-bootstrapping.md) プロセス内でする必要があります。


他のオートローダの使用 <span id="using-other-autoloaders"></span>
-----------------------

Yii はパッケージ依存関係マネージャとして Composer を包含しているので、Composer のオートローダもインストールすることをお勧めします。
あなたが独自のオートローダを持つサードパーティ・ライブラリを使用している場合は、
それらもインストールする必要があります。

Yii オートローダを他のオートローダと一緒に使うときは、他のすべてのオートローダがインストールされた *後で* 、 `Yii.php`
ファイルをインクルードする必要があります。これで Yii のオートローダが、任意クラスのオートローディング要求に応答する最初のものになります。
たとえば、次のコードは [ベーシック・プロジェクト・テンプレート](start-installation.md) の
[エントリ・スクリプト](structure-entry-scripts.md) から抜き出したものです。
最初の行は、Composer のオートローダをインストールしており、二行目は Yii のオートローダをインストールしています。

```php
require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/../vendor/yiisoft/yii2/Yii.php';
```

あなたは Yii のオートローダを使わず、Composer のオートローダだけを単独で使用することもできます。しかし、そうすることによって、
あなたのクラスのオートローディングのパフォーマンスは低下し、クラスをオートロード可能にするために
Composer が設定したルールに従わなければならなくなります。

> Info: Yiiのオートローダを使用したくない場合は、`Yii.php` ファイルのあなた独自のバージョンを作成し、
  それを [エントリ・スクリプト](structure-entry-scripts.md) でインクルードする必要があります。


エクステンション・クラスのオートロード <span id="autoloading-extension-classes"></span>
--------------------------------------

Yii のオートローダは、 [エクステンション](structure-extensions.md) クラスのオートロードが可能です。唯一の要件は、
エクステンションがその `composer.json` ファイルに正しく `autoload` セクションを指定していることです。
`autoload` の指定方法の詳細については [Composer のドキュメント](https://getcomposer.org/doc/04-schema.md#autoload) 参照してください。

Yii のオートローダを使用しない場合でも、まだ Composer のオートローダがエクステンション・クラスをオートロードすることが可能です。