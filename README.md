```
docker build -t stack-driver .
docker run --rm -v (pwd):/etc/google/auth -v (pwd):/fluentd/etc/ stack-driver
```

# 目的
GCPでもAWSでもない環境のfluentdからのログをStackDriverに送りたい。

1. まずはクリーンな環境でプラグインが動くことを試したい
2. そしてログが送れるところまで確認したい

# 経緯
## StackDriverの準備
https://cloud.google.com/monitoring/accounts/guide
に従いアカウントを作る

https://console.cloud.google.com/iam-admin/projects
プロジェクトを作る

https://app.google.stackdriver.com
これがStackDriverのコンソール

https://cloud.google.com/logging/docs/agent/authorization
ここで認証情報取得。`application_default_credentials.json`という名前で保存

## fluentd-google-cloud-loggingが動くDockerイメージをつくる
https://github.com/mookjp/docker-fluentd-google-cloud-logging
を使おうと思ったがbaseイメージが変わりすぎていてビルドできない

https://hub.docker.com/r/fluent/fluentd/
を参考にプラグイン入りのイメージを作る手順を実行。
当初Alpineベースで試したGlibcのインストールが必要。

https://github.com/sgerrand/alpine-pkg-glibc
の手順をためしたがwgetもうまく動かないので、Debianベースに切り替え

ビルドは通るがプラグインのパラメータが足りない。
https://github.com/GoogleCloudPlatform/fluent-plugin-google-cloud/blob/master/lib/fluent/plugin/out_google_cloud.rb#L286

project_idはcredentialから取ってくれる。
zoneとvm_idは設定が必要。

https://gist.github.com/sonots/c54882f73e3e747f4b20#fluentenginerun_configure
outputプラグインにパラメータを渡すのはmatchディレクティブ
fluent.confに設定して起動した

# ログの確認
https://console.cloud.google.com/logs/viewer
に出る予定
