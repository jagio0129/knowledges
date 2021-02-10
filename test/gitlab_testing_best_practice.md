Testing best practices - GitLab
===

> GitLabのテストにおけるベストプラクティス記事を日本語訳したものです。
>
> https://docs.gitlab.com/ee/development/testing_guide/best_practices.html
>
> 最終閲覧日時: 2020/01/06

## [Test Design](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-design)
GitLabではテストは最優先事項[1]です。機能の設計と同様に、テストの設計を考慮することは重要と考えています。

機能を実装するとき、我々は適切な機能を適切な方法で開発することを検討します。これにより、範囲を管理可能なレベルに絞り込むことができます。機能のテストを実装する場合、適切なテスト実装を検討する必要がありますが、重要な箇所をすべてカバーするテストは難しく、すぐに管理困難な段階にまで拡大するでしょう。

テストヒューリスティック[2]は、この問題の解決に役立ちます。これは、我々のコードに潜むバグを明らかにし、簡潔に対処します。テストを設計する時、既知のテストヒューリスティックの確認に時間をかけ、我々のテスト設計手法を周知してください。[Test Engineering section](https://about.gitlab.com/handbook/engineering/quality/test-engineering/#test-heuristics)で有用なヒューリスティックのドキュメントが読めます。

## [Test speed](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-speed)

GitLabには大規模なテストスイートがあり、[並列化](https://docs.gitlab.com/ee/development/testing_guide/ci.html#test-suite-parallelization-on-the-ci)しないと実行に数時間かかることがあります。正確で効果的かつ高速なテストを書く努力をすることが重要です。

テストのパフォーマンスに関する注意事項を次に示します。

- `double`と`spy`は`FactoryBot.build(...)`より速い
- `FactoryBot.build(...)`と`.build_stubbed`は`.create`より速い
- `build`, `build_stubbed`, `attributes_for`, `spy`, `double`を使う時、`create`でオブジェクトを作成しないこと。DBの永続化は遅い。
- 本当にテストする必要がある場合以外は、JavaScriptを必要とする機能のテスト(RSpecの`:js`のような)を行わないこと。ヘッドレスブラウザでのテストは遅い。

## [RSpec](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-speed)

rspecでテストするには

```ruby
# run all tests
bundle exec rspec

# run test for path
bundle exec rspec spec/[path]/[to]/[spec].rb
```

[guard](https://github.com/guard/guard)を使用して変更を継続的に監視し、変更されたテストのみを実行します。

```ruby
bundle exec guard
```

springとguardを一緒に使用する場合は、代わりに`SPRING = 1 bundle exec guard`としてspringを使用してください。

## [General guidelines](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-speed)

- トップレベルには`describe ClassName`を一つだけ定義すること
- `describe`内では、クラスメソッドを`.method`、インスタンスメソッドを`#method`と表記すること
- ロジック的に分岐する場合は`context`を使うこと
- テスト項目の順序はプロダクトコードクラス内の順序と一致させること
- 改行を使用してフェーズを分離し、[4フェーズのテストパターン](https://thoughtbot.com/blog/four-phase-test)に従うこと
- `localhost`のようなハードコーディングはせず、`Gitlab.config.gitlab.host`のように設定変数を使うこと
- `sequence`によって生成された変数のような値に対してテストを行わないこと([落とし穴](https://docs.gitlab.com/ee/development/gotchas.html#do-not-assert-against-the-absolute-value-of-a-sequence-generated-attribute)を参照)
- `before`や`after`などのフックの引数に`:each`(alias `:example`)与えてもデフォルトで効いているため引数に指定しないこと
- `before`と`after`のフックは`:all`のスコープよりも`:context`のスコープのほうが望ましい
- 指定した要素に作用する`evaluate_script("$('.js-foo').testSomething()"`) (もしくは`execute_script`)を使うときは、Capybaraのマッチャー(例えば`find('.js-foo')`)であらかじめ要素が確実に存在することを確かめる
- `focus: true`を使ってテストしたい範囲を分離すること
- テストに複数の期待値がある場合は[aggregate_failures](https://qiita.com/jnchito/items/3a590480ee291a70027c#1-%E7%89%B9%E5%AE%9A%E3%81%AE%E3%82%A8%E3%82%AF%E3%82%B9%E3%83%9A%E3%82%AF%E3%83%86%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E7%BE%A4%E3%82%92%E3%81%BE%E3%81%A8%E3%82%81%E3%81%A6%E6%A4%9C%E8%A8%BC%E3%81%A7%E3%81%8D%E3%82%8Baggregate_failures-%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89)を使用すること

## [System / Feature tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-speed)

> Note: 新しいSystem / Feature testを書く前に一度、[書かないことを検討してください](https://docs.gitlab.com/ee/development/testing_guide/testing_levels.html#consider-not-writing-a-system-test)

- feature specのファイル名は`user_changes_password_spec.rb`のように`ROLE_ACTION_spec.rb`とすべき
- シナリオタイトルには成功ケースと失敗ケースを記載すること
- 「successfully」など、情報がないようなシナリオタイトルは避けること
  - 何がsuccessfullyなのか明記すること
- 機能のタイトルを繰り返すだけのシナリオタイトルは避けること
- データベースには必要なレコードのみを作成すること
- Happy path[3]とless happy pathだけをテストします。
- 可能な限り単体テストまたは統合テストでテストする必要がある
- ActiveRecord内部ではなくページに表示されるものを評価すること
  - もしレコードが作成されたことを確認したかったら、`Model.count`等でモデルが増えたことをテストするのではなく、その項目がページに表示されるというテストを追加すること。
- DOM要素を探してもよいがテストをより脆弱にするため乱用はしないこと


## [Live debug](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#test-speed)

`live_debug`メソッド[4]を使えば、Capybaraを一時停止して、ブラウザでウェブサイトを表示できます。デフォルトブラウザで開きます。テストの実行を再開するには、任意のキーを押します。

以下のような感じです。

```
$ bin/rspec spec/features/auto_deploy_spec.rb:34
Running via Spring preloader in process 8999
Run options: include {:locations=>{"./spec/features/auto_deploy_spec.rb"=>[34]}}

Current example is paused for live debugging
The current user credentials are: user2 / 12345678
Press any key to resume the execution of the example!
Back to the example!
.

Finished in 34.51 seconds (files took 0.76702 seconds to load)
1 example, 0 failures
```

> Note: `live_debug`はJavaScriptが使える場合でのみ動きます。


## [Run :js spec in a visible browser](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#run-js-spec-in-a-visible-browser)

以下のように`CHROME_HEADLESS=0`を付けてspecを実行します。

```
CHROME_HEADLESS=0 bundle exec rspec some_spec.rb
```

このテストはすぐに終わりますが、これにより何が起こっているのかわかります。`CHROME_HEADLESS=0`を付け `live_debug`を使って開いているブラウザを一時停止し、再び開くことができません。これは要素のデバッグと検査に使用できます。

byebugまたはbinding.pryを追加して、実行を一時停止し、テストを[ステップ実行](https://docs.gitlab.com/ee/development/pry_debugging.html#stepping)することもできます。

## [Screenshots](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#screenshots)

`capybara-screenshot`gemを使用して失敗時に自動的にスクリーンショットを撮ります。 CIでは、これらのファイルをジョブアーティファクトとしてダウンロードできます。

また、以下のメソッドを追加することにより、テストの任意の時点でスクリーンショットを手動で取得できます。不要になったら削除してください！詳細については、[ここ](https：//github.com/mattheworiordan/capybara-screenshot#manual-screenshots)を参照してください。

- `screenshot_and_save_page`
  - Capybaraが「見ているもの」のスクリーンショットを作成し、ページソース(htmlファイル等)を保存します。
- `screenshot_and_open_image `
  - Capybaraが「見ているも」のをスクリーンショット化し、画像を自動的に開きます。

これにより作成されたHTMLダンプにはCSSがありません。これにより、実際のアプリケーションとは大きく異なった外観になります。デバッグを容易にするCSSを追加する[small hack](https://gitlab.com/gitlab-org/gitlab-foss/snippets/1718469)があります。

## [Fast unit tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#fast-unit-tests)

一部のクラスはRailsから十分に分離されており、RailsやBunlderの`:default`グループのgem等によって追加されたオーバーヘッドなしでそれらをテストできるはずです。このような場合、テストファイルで`spec_helper`をreuqireする代わりに`fast_spec_helper`をreuqireできます。次の理由からテストは非常に高速に実行されるはずです。

- gemのロードをスキップ
- Railsアプリの起動をスキップ
- GitLab ShellとGitallyのスキップ
- テストリポジトリのセットアップをスキップ

`fast_spec_helper`はlibディレクトリ配下にある自動ロードクラスもサポートします。つまり、クラス/モジュールがlibディレクトリ配下のコードのみを使用している限り、依存関係を明示的にロードする必要はありません。`fast_spec_helper`は、Rails環境で一般的に使用されるコア拡張機能を含む、すべてのActiveSupport拡張機能もロードします。

コードがgemを使用している場合、または依存関係がlibにない場合、require_dependencyを使用して依存関係をロードする必要があることに注意してください。

たとえば、`Gitlab::UntrustedRegexp`クラスを呼び出しているコードをテストする場合は、内部でre2ライブラリを使用します。re2 gemを必要とするライブラリ内のファイルに`require_dependency 're2'`を追加して、この要件を作成する必要があります 明示的に指定するか、仕様自体に追加することもできますが、前者が優先されます。

`spec_helper`の場合に30秒以上かかるloadが、`fast_spec_helper`を使うことで1秒程度のロードですみます。

## [let variables](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#let-variables)

GitLabのRSpecスイートでは、重複を減らすために`let`（それに加えて、厳密な非遅延バージョン`let！`）変数を広範囲に使用しています。しかしながらこれは時々[コードを分かりにくく](https://thoughtbot.com/blog/lets-not)します。そのため、今後の使用に関するガイドラインを設定する必要があります。

- `let!`はインスタンス変数よりも好ましい。`let`は`let!`よりも好ましい。ローカル変数は`let`よりも望ましい
- `let`を使用してspecファイル全体の重複を減らせる
- 単一のテストでのみ使用される変数は`let`を使わずにitブロック内でローカル変数として定義すること
- 最上位の記述ブロック内でlet変数を定義しないこと。これは、より深くネストされたコンテキストまたは記述ブロックでのみ使用される。定義は使用する場所にできるだけ近づけること。
  - 解せない
- ある`let`変数の定義を別の`let`変数の定義で上書きしないようにする
  - これもスコープが異なっていればいいのでは？
- 別で定義されている`let`変数を定義するな(たぶん糸の異なる重複定義するなってことだと思われ)
  - 代わりにヘルパーメソッドを使うべき
- `let!`変数は定義された順序が重要な場合にのみ使用すること。それ以外は`let`で十分
  - `let`は遅延評価であり、参照されるまで評価されないことに注意して

## [Common test setup](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#common-test-setup)

場合によっては、各exampleでテスト用に同じオブジェクトを生成する必要はありません。たとえば、プロジェクトとのそのプロジェクトのゲストは、プロジェクトとゲストが関係するすべてのファイルに対してテストするために必要となります。これは、[`test-prof`](https://rubygems.org/gems/test-prof)gemで導入できる[`let_it_be`](https://test-prof.evilmartians.io/#/let_it_be)変数と[`before_all`](https://test-prof.evilmartians.io/#/before_all)フックを使うことで達成できます。

```ruby
let_it_be(:project) { create(:project) }
let_it_be(:user) { create(:user) }

before_all do
  project.add_guest(user)
end
```

これにより、このcontextに対して作成されるProject, User, ProjectMemberが一つだけになります。

`let_it_be`および`before_all`は、ネストされたcontext内でも使用できます。トランザクションロールバックにより、contextが処理された後に自動的にクリーンアップされます。

`let_it_be`ブロック内で定義されたオブジェクトを変更する場合、必要に応じてオブジェクトをリロードするか、すべての例でリロードするリロードオプションを指定する必要があることに注意してください。

```ruby
let_it_be(:project, reload: true) { create(:project) }
```

また、再検索オプションを指定して、新しいオブジェクトを完全にロードすることもできます。

```ruby
let_it_be(:project, refind: true) { create(:project) }
```

## [set variables](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#common-test-setup)

> Note: Gitlabでは`let_it_be`を支持しているため、`set`を削除しつつあります。詳細は[こちら](https://gitlab.com/gitlab-org/gitlab/issues/27922)参照してください。

場合によっては、各exampleでテスト用に同じオブジェクトを生成する必要はありません。たとえば、プロジェクトとのそのプロジェクトのゲストは、プロジェクトとゲストが関係するすべてのファイルに対してテストするために必要となります。これは、`let`を使用するのと同じ方法で`set`を使用することで実現できます。

`rspec-set`はActiveRecordオブジェクトでのみ機能し、新しいサンプルの前に必要な場合にのみモデルをリロードまたは再作成します。つまり、プロパティを変更したときまたはオブジェクトを破棄したときです。

`set`ブロック内の`let`ブロックで定義されたモデルを参照することはできませんので注意してください。

また、`:js`スペックでは、各exampleの後にデータベースの状態をクリーンアップするためにトランザクションを使用しないため、`set`はサポートされていません。

## [Time-sensitive tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#time-sensitive-tests)
[Timecop](https://github.com/travisjeffery/timecop)はRubyベースで利用でき、時間依存のテストケースに有効です。時間に依存するテストを実行・検証する場合、Timecopを使用して一時的なテストの失敗を防ぎます。

例えば、

```ruby
it 'is overdue' do
  issue = build(:issue, due_date: Date.tomorrow)

  Timecop.freeze(3.days.from_now) do
    expect(issue).to be_overdue
  end
end
```


## [Feature flags in tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#time-sensitive-tests)

すべての機能フラグはRubyベースのテストにおいて、デフォルトで有効になるようにスタブ化されています。

テストで機能フラグを無効にするには、`stub_feature_flags`ヘルパーを使用します。たとえば、テストで`ci_live_trace`機能フラグをグローバルに無効にするには、

```ruby
stub_feature_flags(ci_live_trace: false)

Feature.enabled?(:ci_live_trace) # => false
```

一部のアクターに対して機能フラグを無効にし、他のアクターに対しては無効にしないテストを設定する場合、ヘルパーに渡すオプションでこれを指定できます。たとえば、特定のプロジェクトの`ci_live_trace`機能フラグを無効にするには、

```ruby
project1, project2 = build_list(:project, 2)

# Feature will only be disabled for project1
stub_feature_flags(ci_live_trace: { enabled: false, thing: project1 })

Feature.enabled?(:ci_live_trace, project1) # => false
Feature.enabled?(:ci_live_trace, project2) # => true
```

## [Pristine test environments](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#pristine-test-environments)

1つのGitLabテストで実行されるコードは、多くのデータにアクセスして変更する場合があります。テストを実行する前に慎重に準備し、その後クリーンアップしないと、データはテストによって変更され、次のテストの動作に影響を与える可能性があります。これは必ず避けてください。幸いなことに、既存のテストフレームワークのほとんどはこのようなケースを回避しています。

テスト環境が汚染されると、一般的には[不安定なテスト](https://docs.gitlab.com/ee/development/testing_guide/flaky_tests.html)になります。テスト環境汚染の多くの場合は、スペックAの後にスペックBを実行すると確実に失敗するが、スペックBの後にスペックAを実行すると確実に成功するといった、順序の依存関係として現れます。このような場合、`rspec --bisect`[5](または手動で組み合わせをチェックする)を使用して、どのスペックに問題があるかを判断できます。問題を解決するには、テストスイートで環境がどのように維持されているかをある程度理解する必要があります。続きを読んで各データストアの詳細をご覧ください。

## [SQL database](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#sql-database)

`database_cleaner`gemによって管理しています。各スペックはトランザクションに囲まれ、テストが完了するとロールバックされます。特定のスペックでは、完了後にすべてのテーブルに対して`DELETE FROM`クエリが発行されます。これにより複数のデータベース接続(ブラウザからの操作やマイグレーションスペックなどのスペックにとって重要な)から作成された行を表示できます。

よく知られている`TRUNCATE TABLES`アプローチの代わりにこれらの戦略を使用した結果の1つに、主キーと他のシーケンスがスペック間で**リセットされない**ことがあります。したがって、スペックAでプロジェクトを作成してからスペックBでプロジェクトを作成すると、最初のプロジェクトは`id = 1`になり、2番目のプロジェクトは`id = 2`になります。

これは、スペックがIDの値またはその他のシーケンス生成列に**依存しない**ことを意味します。偶発的な競合を避けるため、スペックではこれらの種類の列に値を手動で指定することも避けてください。代わりに、未指定のままにして、行の作成後に値を検索します。

## [Redis](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#redis)

GitLabではRedisに、キャッシュされたデータとSidekiqジョブの2つのデータカテゴリを保存します。殆どのスペックではRailsキャッシュはメモリ内に存在しています。これはスペック間で置き換えられるため、`Rails.cache.read`と`Rails.cache.write`の呼び出しは安全です。ただし、スペックが直接Redis呼び出しを行う場合は、必要に応じて`:clean_gitlab_redis_cache`、`:clean_gitlab_redis_shared_state`、`:clean_gitlab_redis_queues`traitを適切に使用する必要があります。

## [Background jobs / Sidekiq](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#background-jobs--sidekiq)

デフォルトでは、Sidekiqジョブはジョブ配列にキューイングされ、処理されません。テストがSidekiqジョブをキューに入れて処理する必要がある場合は、`:sidekiq_inline`traitを使用できます。

`:sidekiq_might_not_need_inline`traitは、[Sidekiqのインラインモードがフェイクモードに変更された](https://gitlab.com/gitlab-org/gitlab/merge_requests/15479)ときに、Sidekiqが実際にジョブを処理するのに必要なすべてのテストに追加されました。このtraitを持つテストは、Sidekiq処理ジョブに依存しないように修正するか、バックグラウンドジョブの処理が必要/予想される場合、`:sidekiq_might_not_need_inline`traitを`:sidekiq_inline`に更新する必要があります。

> Note: ワーカーは`ApplicationJob`/`ActiveJob::Base`を継承していないため、`perform_enqueued_jobs`は現在使用できません。

## [Filesystem](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#filesystem)

ファイルシステムのデータは、「リポジトリ」と「その他すべて」に大まかに分けることができます。リポジトリは`tmp/tests/repositories`に保存されます。このディレクトリは、テストが実行される前、およびテストが終了した後に空になります。スペック間では空にならないため、作成されたリポジトリはプロセスの存続期間中、このディレクトリ内に蓄積されます。それらを削除するのはコストがかかりますが、注意深く管理しないと汚染につながる可能性があります。

これらを回避するには、テストスイートで[ハッシュストレージ](https://docs.gitlab.com/ee/administration/repository_storage_types.html)を有効にします。つまり、リポジトリにはプロジェクトのIDに依存する一意のパスが与えられます。プロジェクトIDはスペック間でリセットされないため、各スペックがディスク上の独自のリポジトリを取得することが保証され、スペック間で変更が表示されないようにします。

スペックでプロジェクトIDを手動で指定する場合、または`tmp/tests/repositories/`ディレクトリの状態を直接検査する場合、実行の前後にディレクトリをクリーンアップする必要があります。一般的に、これらのパターンは完全に回避する必要があります。

アップロードなど、データベースオブジェクトにリンクされた他のクラスのファイルは、通常同じ方法で管理されます。スペックでハッシュストレージが有効になっている場合、IDによって決定される場所のディスクに書き込まれるため、競合は発生しません。

一部のスペックでは、`projects`factoryに`:legacy_storage`traitを渡すことで、ハッシュストレージを無効にします。これを行うスペックは、プロジェクトまたはそのグループのパスをオーバーライドしてはなりません。デフォルトのパスにはプロジェクトIDが含まれているため、競合しません。ただし、2つの仕様が同じパスを持つ`:legacy_storage`プロジェクトを作成する場合、ディスク上の同じリポジトリを使用し、環境汚染をテストします。

その他のファイルは、スペックによって手動で管理する必要があります。たとえば、`tmp/test-file.csv`ファイルを作成するコードを実行する場合、スペックでは、クリーンアップの一環としてファイルが削除されるようにする必要があります。

## [Persistent in-memory application state](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#persistent-in-memory-application-state)

Rspecによるスペックはすべて同じRubyプロセスを共有します。つまり、スペック間でアクセス可能なRubyオブジェクトを変更することで、互いに影響を与えることができます。これはグローバル変数、および定数(クラス、モジュールなどを含む)であることを意味しています。

通常、グローバル変数は変更しないでください。どうしても必要な場合、以下のようなブロックを使用して、変更を後でロールバックできます。

```ruby
around(:each) do |example|
  old_value = $0

  begin
    $0 = "new-value"
    example.run
  ensure
    $0 = old_value
  end
end
```

スペックで定数を変更する必要がある場合は、`stub_const`ヘルパーを使用して、変更が確実にロールバックされるようにする必要があります。

`ENV`定数を変更する必要がある場合は、代わりに`stub_env`ヘルパーメソッドを使用できます。

ほとんどのRubyインスタンスはスペック感で共有されませんが、クラスとモジュールは一般的に共有されます。クラスおよびモジュールのインスタンス変数、アクセサー、クラス変数、およびその他のステートフルイディオムはグローバル変数と同じように扱われるべきです。必要がない限り変更しないでください。とくに、変更の必要性を排除するために、expectまたはstubに沿った依存関係の代入かを使うのことが望ましいです。他に選択肢がない場合は、上記のグローバル変数と同様に`around`ブロックが使用できますが、可能な限り回避する必要があります。

## [Table-based / Parameterized tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#table-based--parameterized-tests)

このスタイルのテストは、包括的な入力範囲で1つのコードを実行するために使用されます。テストケースを1回指定するだけで、入力のテーブルとそれぞれの予想出力とともに、テス​​トを読みやすく、コンパクトにすることができます。

GitLabでは[rspec-parameterized](https://github.com/tomykaira/rspec-parameterized)gemを使っています。テーブル構文を使用し、Rubyの入力範囲をチェックする短い例は、次のようになります。

```ruby
describe "#==" do
  using RSpec::Parameterized::TableSyntax

  where(:a, :b, :result) do
    1         | 1        | true
    1         | 2        | false
    true      | true     | true
    true      | false    | false
  end

  with_them do
    it { expect(a == b).to eq(result) }

    it 'is isomorphic' do
      expect(b == a).to eq(result)
    end
  end
end
```

> Caution: whereブロックの入力として単純な値のみを使用します。プロシージャ、ステートフルオブジェクト、FactoryBotで作成されたオブジェクトなどを使用すると、[予期しない結果](https://github.com/tomykaira/rspec-parameterized/issues/8)が生じる可能性があります。

## [Prometheus tests](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#prometheus-tests)

Prometheusメトリクス[6]は、テストの実行ごとに保持される場合があります。各サンプルの前にメトリクスが確実にリセットされるようにするには、Rspecテストに`:prometheus`タグを追加します。

## [Matchers](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#matchers)

カスタムマッチャーを作成して、意図を明確にし、RSpecの予想の複雑さを隠す必要があります。これは`spec/support/matchers/`に配置する必要があります。マッチャーは、特定のタイプのスペック（機能スペック、リクエストスペックなど）にのみ適用される場合はサブフォルダーに配置できますが、複数のタイプの仕様に適用する場合は配置しないでください。

## [be_like_time](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#be_like_time)

データベースから返される時間は、Rubyの時間オブジェクトと精度が異なる場合があります。そのため、スペックを比較する際に柔軟な許容範囲が必要です。`be_like_time`を使用して、時間が1秒以内であることを比較できます。

```ruby
expect(metrics.merged_at).to be_like_time(time)
```

## [have_gitlab_http_status](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#have_gitlab_http_status)

`have_http_status`よりも`have_gitlab_http_status`をお勧めします。`have_gitlab_http_status`は、ステータスが一致しない場合に常に応答本文も表示できるためです。これは、テストが落ちたときにソースコードを編集せず、テストを再実行せずとも落ちた原因を知るのに非常に役に立ちます。

特に500サーバーエラーが表示されている場合に便利です。

## [Shared contexts](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#shared-contexts)

すべての`shared context`は、`spec/support/shared_contexts/`に配置する必要があります。`shared context`は、特定のタイプのスペック(機能スペック、リクエストスペックなど)にのみ適用される場合サブフォルダーに配置できますが、複数のタイプの仕様に適用される場合はそうではありません。

各ファイルにはコンテキストが1つだけ含まれ、わかりやすい名前を付ける必要があります。
(e.g. `spec/support/shared_contexts/controllers/githubish_import_controller_shared_context.rb.`)

## [Shared examples](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#shared-examples)

すべての`shared example`は、`spec/support/shared_exampless/`に配置する必要があります。`shared examples`は、特定のタイプのスペック(機能スペック、リクエストスペックなど)にのみ適用される場合サブフォルダーに配置できますが、複数のタイプの仕様に適用される場合はそうではありません。

各ファイルにはコンテキストが1つだけ含まれ、わかりやすい名前を付ける必要があります。
(e.g. `spec/support/shared_exampless/controllers/githubish_import_controller_shared_example.rb.`)

## [Helpers](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#helpers)

ヘルパーは通常、特定のRSpecのexampleの複雑さを隠すためのメソッドを提供するモジュールです。他のスペックと共有することを意図していない場合、RSpecファイルでヘルパーを定義できます。それ以外の場合は、`spec/support/helpers/`に配置する必要があります。

特定のタイプのスペック(機能スペック、リクエストスペックなど)のみに適用される場合、ヘルパーはサブフォルダーに配置できます。

ヘルパーはRailsの命名規則/名前空間規則に従う必要があります。たとえば、`spec/support/helpers/cycle_analytics_helpers.rb`は以下のように定義する必要があります。

```ruby
module Spec
  module Support
    module Helpers
      module CycleAnalyticsHelpers
        def create_commit_referencing_issue(issue, branch_name: random_git_name)
          project.repository.add_branch(user, branch_name, 'master')
          create_commit("Commit for ##{issue.iid}", issue.project, user, branch_name)
        end
      end
    end
  end
end
```

ヘルパーでRSpecの設定を変更しないでください。たとえば、上記のヘルパーモジュールには以下を含めないでください。

```ruby
RSpec.configure do |config|
  config.include Spec::Support::Helpers::CycleAnalyticsHelpers
end
```

## [Factories](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#factories)

GitLabはテスト用のFixture[7]の代替として[factory_bot](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#factories)を使用します。

- Factoryは`spec/factories/`で定義し、対応するモデルの複数形を使用して命名します(`User`のfactoryは`users.rb`)。
- ファイルごとにトップレベルのファクトリ定義は1つだけにする必要があります。
- FactoryBotメソッドは、すべてのRSpecグループに混在しています。つまり`Factory.create(...)`の代わりに`create(...)`を呼び出すことができます(そして呼び出す必要があります)。
- [trait](https://www.rubydoc.info/gems/factory_bot/file/GETTING_STARTED.md#Traits)を使用して定義と使用方法をクリーンアップします。
- ファクトリを定義するとき、モデルに関係のないカラムを定義しないでください。
- ファクトリをインスタンス化するときは、不要なカラムを指定しないでください。
- ファクトリはActiveRecordオブジェクトに限定される必要はありません。[例](https://gitlab.com/gitlab-org/gitlab-foss/commit/0b8cefd3b2385a21cfed779bd659978c0402766d)を参照してください。

## [Fixtures](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#fixtures)

すべてのFixtureは`spec/fixtures/`の下に配置する必要があります。

## [Repositories](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#repositories)

マージリクエストのマージなどの一部の機能をテストするには、特定の状態のGitリポジトリがテスト環境に存在する必要があります。GitLabは、特定の一般的なケースに対して[gitlab-test](https://gitlab.com/gitlab-org/gitlab-test)リポジトリを維持します。プロジェクトファクトリの`:repository`トレイトでリポジトリのコピーが使用されていることを確認できます

```ruby
let(:project) { create(:project, :repository) }
```

可能な場合は、`:repository`ではなく`:custom_repo`トレイトの使用を検討してください。これにより、プロジェクトのリポジトリのmasterブランチに表示されるファイルを正確に指定できます。

```ruby
let(:project) do
  create(
    :project, :custom_repo,
    files: {
      'README.md'       => 'Content here',
      'foo/bar/baz.txt' => 'More content here'
    }
  )
end
```

これにより、デフォルトの権限と指定されたコンテンツを持つ2つのファイルを含むリポジトリが作成されます。

## [Config](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#config)

RSpecの設定ファイルは、RSpecのコンフィグ(すなわち`RSpec.configure do |config|`ブロック)を変更するファイルです。これは`spec/support/`に配置する必要があります。

各ファイルには特定のドメインに関連する必要があります。たとえば、`spec/support/capybara.rb`, `spec/support/carriewave.rb`などです。

ヘルパーモジュールが特定の種類のスペックにのみ適用される場合、`config.include`呼び出しに修飾子を追加する必要があります。たとえば、`spec/support/helpers/cycle_analytics_helpers.rb`が`:lib`および`type: :model`スペックにのみ適用される場合、次のように記述します。

```ruby
RSpec.configure do |config|
  config.include Spec::Support::Helpers::CycleAnalyticsHelpers, :lib
  config.include Spec::Support::Helpers::CycleAnalyticsHelpers, type: :model
end
```

構成ファイルが`config.include`のみで構成されている場合、これらの`config.include`を`spec/spec_helper.rb`に直接追加できます。

汎用的なヘルパーについては、`spec/fast_spec_helper.rb`ファイルで使用される`spec/support/rspec.rb`ファイルに含めることを検討してください。`spec/fast_spec_helper.rb`ファイルの詳細については、[高速ユニットテスト](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html#fast-unit-tests)を参照してください。

## 注釈
- [1] first class citizenの訳がふわっとしていた(一級市民 or 第一級オブジェクト)ので[この記事](https://news.mynavi.jp/article/20090609-mozilla/3)をみて「最優先事項」との訳にした
- [2] [ヒューリスティック評価](https://u-site.jp/usability/evaluation/heuristic-evaluation/)：経験則（ヒューリスティックス）に基づいてユーザビリティを評価し、UI上の問題を発見する手法。ここでは「テストにおける経験則・ナレッジ」の意味合いに近い
- [3] 最も使用頻度の高いユースケースのテストを[ハッピーテスト](https://twitter.com/krsna_sub/status/310588701807869953)っていうんですね。知らなんだ...
- [4] `live_debug`なんてメソッドCapybaraにもRspecにもいないぞと[思ったらこういうこと](https://www.reddit.com/r/ruby/comments/9ggna7/some_good_advices_about_rspec_and_capybara_from/)でした
- [5] [rspec --bisect](https://techracho.bpsinc.jp/hachi8833/2018_04_05/54360)
- [6] https://qiita.com/sugitak/items/ff8f5ad845283c5915d2
- [7] 初期データを投入する(https://qiita.com/itkrt2y/items/ca34fea17fc7dde56b7a)
