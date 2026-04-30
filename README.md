# BuildSystem
ビルドシステムの基本の備忘録である。

## build.gradle.ktsについて

- **minSdk**: アプリをインストールできるAndroidの最小バージョンを指定
- **compileSdk**: コンパイル時に、アプリがどれだけ最新のAPIを使用するか指定。ただし、minSdk以上のライブラリを使用した場合、AndroidXのライブラリでなければ、条件文が必要になる場合がある。
- **targetSdk**: Runtime Behaviorを設定。targetSdkを上げるほど、位置情報の取得やセキュリティが厳しくなる。（例：[Android 10 の動作変更点](https://developer.android.com/about/versions/10/behavior-changes-all?hl=ja)）
- **versionCode**: Androidシステムが、現在デバイスにインストールされているバージョンより大きいversionCodeのAPKをインストールするためのもの。そのため、リリースを重ねるたびに、単純に1ずつ値を増やす。初期値は1。（参考：[Versioning Your App](https://developer.android.com/studio/publish/versioning?hl=ja)）
- **versionName**: ユーザーに表示されるバージョン番号。例：1.1.1。具体的な指定方法は[Semantic Versioning](https://semver.org/)に記載。

## Build variantsについて
Build variantsでは、開発環境、ステージング環境、本番環境というように、環境別に条件を指定できる。
これは、Build typesとProduct flavorsを組みあわせたもので、自動で構成される。
### Build types
プロパティとして、[applicationIdSuffix](https://developer.android.com/reference/tools/gradle-api/9.2/com/android/build/api/dsl/ApplicationVariantDimension#applicationIdSuffix())を指定してアプリケーションidを変更できたり、[isMinifyEnabled](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization)を指定してアプリの最適化ができたりします。
```build.gradle.kts
android {
    buildTypes {
        release {
            isMinifyEnabled = false
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            applicationIdSuffix = ".dubeg"
            isDebuggable = true
        }
        create("staging") {
            initWith(getByName("debug") //debugビルドタイプをコピー
            manifestPlaceholders["hostName"] = "internal.example.com" //マニフェストにビルド変数を追加する（https://developer.android.com/build/manage-manifests?hl=ja#inject_build_variables_into_the_manifest）
            applicationIdSuffix = ".debugStaging"
        }
    }
}
```
### Product Flavors
- アプリの無料版や有料版など、ユーザーにリリースできる様々なアプリバージョンを設定できる。
- （筆者の考えだが）、Build Variantsは開発環境や本番環境など開発のプロセスで条件を変えたい時に使うのに対し、
Product Flavorsは本番環境下や、開発環境下だけであっても条件を変えたい時に使うものである。
- Product Flavorsを作成する際は、必ずflavorDimensionsに属さないといけない。
- ただし、flavorDimensionsが1つの場合、dimensionプロパティは必要ない。

```build.gradle.kts
android {
    ...
    buildTypes{}
    productFlavors {
        flavorDimensions += "version" // flavorDimensionsの指定必須
        create("production") {
            dimension = "version" // flavorDimensionが1つの場合、dimensionプロパティは必須ではない。
            ...
        }
        create("development"){
            ...
        }
     }
}
```

### Build Variantsごとに値を変える方法
　resValueを各productFlavorsで使うと、Build Variantsごとに値を変えられる。ちなみに、resValueを有効化するために、buildFeaturesに
明示しなければならない。（参考：[動作の変更](https://developer.android.com/build/releases/agp-9-0-0-release-notes?hl=ja&utm_source=android-studio-app&utm_medium=app#android-gradle-plugin-behavior-changes)）
```build.gradle.kts
android {
    ...
    productFlavors {
        flavorDimensions += "version"

        create("production") {
            applicationIdSuffix = ".production"
            versionNameSuffix = "-production"
            resValue("string", "message", "This is a production App.") // 第1引数に型、第2引数に名前、第3引数に値を入れる
        }
        create("development") {
            applicationIdSuffix =".development"
            versionNameSuffix = "-development"
            resValue("string", "message", "This is a development App")
        }
    }
    ...
    buildFeatures {
        compose = true
        resValues = true //明示必須
    }
}
```
　値をコード内で取得する際には、以下のように記述する。
```
val message = stringResource(R.string.message)
```
参考：[カスタム フィールドとリソース値をアプリのコードと共有する](https://developer.android.com/build/gradle-tips?hl=ja#share-custom-fields-and-resource-values-with-your-app-code)

## アプリに署名
　署名方法は、まず[アップロード鍵とキーストアを生成する](https://developer.android.com/studio/publish/app-signing?hl=ja#generate-key)に従って鍵を作成する。
その後[鍵を使ってアプリに署名する](https://developer.android.com/studio/publish/app-signing?hl=ja#sign_release)に従って、作成した鍵を使用し、アプリに署名する。
結果、署名済みアプリが、app以下のフォルダに保存される。基本は、AAB（Android App Bundle）が推奨されるが、エミュレーターでの実行をするため、今回はAPKで作成した。

## 参考
- [ビルドバリアントを設定する](https://developer.android.com/build/build-variants?hl=ja)
