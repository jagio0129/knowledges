GitLab Perfomace Tool(GPT)
===

[ここ](https://about.gitlab.com/blog/2020/02/18/how-were-building-up-performance-testing-of-gitlab/?utm_medium=social&utm_source=twitter&utm_campaign=blog)の要約

- GitLabはサービスのパフォーマンステストを行う環境を構築する必要があった。
- [パフォーマンステストのためのツール](https://gitlab.com/gitlab-org/quality/performance)を作成し一般公開。
- GPTを使用して多数の負荷テストを実行し、GitLab環境のパフォーマンスを検証できる
  - 目的の環境が処理できるスループットを把握することが必要
- GPTはloadimpact/k6を使用している
- GitLabの[負荷試験結果を公開](https://gitlab.com/gitlab-org/quality/performance/-/wikis/Benchmarks/Latest)している
- 各バージョンごとの[ベンチマーク比較結果](https://gitlab.com/gitlab-org/quality/performance/-/wikis/Benchmarks/GitLab-Versions)も公開
  - 全体に渡って改善できた。12.6でのある機能では92%もレスポンス速度を改善できた
- [10kユーザ想定のアーキテクチャリファレンス](https://docs.gitlab.com/ee/administration/high_availability/#10000-user-configuration)で[List Group Projects API](https://docs.gitlab.com/ee/api/groups.html#list-a-groups-projects)を実行すると以下のような実行画面になる
  - https://asciinema.org/a/O96Wc5fyxvLb1IDyviTwbujg8?autoplay=1
    - vu200の設定で60秒間、1秒間隔でテストを実行しているっぽい
    - 40endpoint
- [GitLab k6_samples](https://gitlab.com/gitlab-org/quality/performance/-/tree/master)
  - トリガーはruby。k6スクリプトをRubyが実行している
  - k6のサンプルがあるので嬉しい