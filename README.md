# 目的

GCPでもAWSでもない環境のfluentdからのログをStackDriverに送りたい。

1. まずはクリーンな環境でプラグインが動くことを試したい
2. そしてログが送れるところまで確認したい

# 準備

## StackDriverの準備

1. <https://cloud.google.com/monitoring/accounts/guide> に従いアカウントを作る

1. <https://console.cloud.google.com/iam-admin/projects> プロジェクトを作る

1. <https://app.google.stackdriver.com> これがStackDriverのコンソール

1. <https://cloud.google.com/logging/docs/agent/authorization> ここで認証情報取得。`application_default_credentials.json`という名前で保存

# 実行

```
docker build -t stack-driver .
docker run --rm -v (pwd):/etc/google/auth stack-driver
```

```
2017-04-04 07:07:53 +0000 [info]: reading config file path="/fluentd/etc/fluent.conf"
2017-04-04 07:07:53 +0000 [info]: starting fluentd-0.12.34
2017-04-04 07:07:53 +0000 [info]: gem 'fluent-plugin-google-cloud' version '0.6.0'
2017-04-04 07:07:53 +0000 [info]: gem 'fluentd' version '0.12.34'
2017-04-04 07:07:53 +0000 [info]: adding match pattern="**" type="google_cloud"
2017-04-04 07:08:54 +0000 [info]: Unable to determine platform
2017-04-04 07:08:54 +0000 [info]: Set Project ID from credentials: ubiquitous-server
2017-04-04 07:08:54 +0000 [info]: Logs viewer address: https://console.cloud.google.com/logs/viewer?project=ubiquitous-server&resource=/instance_id/1234
2017-04-04 07:08:54 +0000 [info]: adding source type="forward"
2017-04-04 07:08:54 +0000 [info]: using configuration file: <ROOT>
  <source>
    type forward
  </source>
  <match **>
    type google_cloud
    zone hoge
    vm_id 1234
  </match>
</ROOT>
2017-04-04 07:08:54 +0000 [info]: listening fluent socket on 0.0.0.0:24224
```

# 経緯

## fluentd-google-cloud-loggingが動くDockerイメージをつくる

<https://github.com/mookjp/docker-fluentd-google-cloud-logging> を使おうと思ったがbaseイメージが変わりすぎていてビルドできない

<https://hub.docker.com/r/fluent/fluentd/> を参考にプラグイン入りのイメージを作る手順を実行。 当初Alpineベースで試したGlibcのインストールが必要。

<https://github.com/sgerrand/alpine-pkg-glibc> の手順をためしたがwgetもうまく動かないので、Debianベースに切り替え

ビルドは通るがプラグインのパラメータが足りない。 <https://github.com/GoogleCloudPlatform/fluent-plugin-google-cloud/blob/master/lib/fluent/plugin/out_google_cloud.rb#L286>

project_idはcredentialから取ってくれる。 zoneとvm_idは設定が必要。

<https://gist.github.com/sonots/c54882f73e3e747f4b20#fluentenginerun_configure> outputプラグインにパラメータを渡すのはmatchディレクティブ fluent.confに設定して起動した

# ログの確認

<https://console.cloud.google.com/logs/viewer> に出る予定
