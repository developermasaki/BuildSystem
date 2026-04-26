# BuildSystem

## build.gradle.ktsについて

<dl>
    <dt>minSdk</dt>
    <dd>アプリをインストールできるAndroidの最小バージョンを指定</dd>
    <dt>compileSdk</dt>
    <dd>コンパイル時に、アプリがどれだけ最新のAPIを使用するか指定。ただし、minSdk以上のライブラリを使用した場合、AndroidXのライブラリでなければ、条件文が必要になる。</dd>
    <dt>targetSdk</dt>
    <dd>Runtime Behaviorを設定。targetSdkを上げるほど、位置情報の取得やセキュリティが厳しくなる。（例：https://developer.android.com/about/versions/10/behavior-changes-all?hl=ja）</dd>
    <dt>versionCode</dt>
    <dd>Androidシステムが、現在デバイスにインストールされているバージョンより大きいversionCodeのAPKをインストールするためのもの。そのため、リリースを重ねるたびに、単純に1ずつ値を増やす。初期値は1。（https://developer.android.com/studio/publish/versioning?hl=ja）</dd>
    <dt>versionName</dt>
    <dd>ユーザーに表示されるバージョン番号。例：1.1.1。具体的な指定方法は（https://semver.org/）</dd>
</dl>
