# はじめに

Rubocop は Ruby の静的コード解析ツールです。導入したらすぐに [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) で取り扱われているガイドラインのほとんどが適用されるでしょう。

それらの振舞いのほとんどは様々な設定オプションによって微調整が可能です。

コードの問題を指摘することの他に Rubocop は自動的にいくつかの問題を修正することもできます。

## 導入

RuboCop の標準的な導入の方法は以下の通りです。

    $ gem install rubocop

もし、bundler を使って導入する場合 Gemfile において require する必要はありません。

    gem 'rubocop', require: false

## 基本的な使い方

引数無しで `rubocop` を実行するとカレントディレクトリにある全ての Ruby ソースファイルをチェックします。

    $ rubocop

別の方法として、チェックするファイルやディレクトリのリストを `rubocop` に渡すこともできます。

    $ rubocop app spec lib/something.rb

Rubocop の挙動を見てみましょう。以下の Ruby のソースコードを前提とします。

    def badName
      if domething
        test
        end
    end

Rubocop を実行すると以下のレポートを出力するでしょう。

    Inspecting 1 file
    W
    
    Offenses:
    
    test.rb:1:5: C: Use snake_case for method names.
    def badName
        ^^^^^^^
    test.rb:2:3: C: Use a guard clause instead of wrapping the code inside a conditional expression.
      if something
      ^^
    test.rb:2:3: C: Favor modifier if usage when having a single-line body. Another good alternative is the usage of control flow &&/||.
      if something
      ^^
    test.rb:4:5: W: end at 4, 4 is not aligned with if at 2, 2
        end
        ^^^
    
    1 file inspected, 4 offenses detected

さらに詳細なチェックを行なうためのコマンドラインオプションは

    $ rubocop -h

の出力を参照して下さい。

# Cops

Rubocop においては code 上で実行される様々なチェックがあり、それらは cops と呼ばれています。

## Style

略。

## Lint

Lint cops はコードの中にある非常に悪い慣習およびエラーの可能性をチェックします。Rubocop は移植性のある方法で全ての build-in MRI lint チェック (ruby -wc) を実装し、自身が拡張された lint check の多くを追加します。あなたは lint cops のみをこのように実行できます。

    $ rubocop -l

-l/--lint オプションは有効になっている全ての lint cops に他の cops の選択を加える場合に --only と一緒に使うことができます。

lint cops のいずれかを disable にすることは一般的には悪い考えです。

## Metrix

略。

## Rails

Rails cops は Ruby on Rails フレームワーク固有のものです。style あるいは lint cops とは違い、デフォルトでは使用されず、使う事を明示する必要があります。

    $ rubocop -R

あるいは以下のディレクティブをあなたの `.rubocop.yml` に記載する必要があります。

    AllCops:
      RunRailsCops: true

# 設定

Rubocop の挙動は .rubocop.yml という設定ファイルによって制御することができます。チェックの enable/disable や任意のパラメータを受け取る場合、それらの挙動を変える事が可能です。ファイルはホームディレクトリ、あるいはいくつかのプロジェクトディレクトリに配置することが可能です。Rubocop は検査するファイルがあるディレクトリの中およびルートディレクトリに向けて上方向にで設定ファイルを探すでしょう。

設定ファイルは以下のフォーマットになります。

    inherit_from: ../.rubocop.yml
    
    Style/Encoding:
      Enabled: false
    
    Metrics/LineLength:
        Max: 99

注：cop の名前の修飾子 (例えば Style) が推奨されていますが全ての type の中で cop の名前が一意であれば必ずしもそうしなければならない訳ではありません。

## 継承

任意のディレクティブ inherit_from は一つまたは複数のファイルから設定を読み込むために使われます。これによりプロジェクトルートの .rubocop.yml にある共通のプロジェクト設定とサブディレクトリのルールからの差分を持つことが可能になります。

ファイルは相対パス乃至絶対パスによってそれらを参照することができます。inherit_from 以降のディレクティブは読み込まれたファイルの設定で上書きされます。

複数のファイルが include されている時、最初のファイルが一番優先順位が低くなり、最後のものが一番高くなります。

複数の記述が以下になります。

    inherit_from:
      - ../.rubocop.yml
      - ../conf/.rubocop.yml

## 既定値

Rubocop のホームディレクトリ配下の config/default.yml というファイルはすべての構成から継承するデフォルト設定を保持しています。プロジェクトや個人的な .rubocop.yml ファイルはデフォルトとは異なる設定のみを記載する必要があります。

プロジェクト乃至ホームディレクトリに .rubocop.yml が無い場合、config/default.yml が使われます。

## ファイルの Including/Excluding

Rubocop は実行されたディレクトリ、あるいはコマンドライン引数で渡されたディレクトリから始まる再帰的な探索で発見された全てのファイルをチェックします。しかし、それだけで .rb という拡張子を持つファイルや `#! .*ruby` という宣言のある Ruby ファイルとして認識します。

隠しディレクトリ (. で開始する名前を持つもの) はデフォルトでは探索されません。

デフォルトで探索対象となっていないファイルを対象とするには、コマンドライン引数でそれらを渡すか、 AllCops/Include の下にそれらをエントリに追加する必要があります。ファイルおよびディレクトリは AllCops/Exclude によって無視することもできます。

以下は Rails プロジェクトのために使われる例です。

    AllCops:
      Include:
        - '**/Rakefile'
        - '**/config.ru'
      Exclude:
        - 'db/**/*'
        - 'config/**/*'
        - 'script/**/*'
        - !ruby/regexp /old_and_unused\.rb$/
    
    # other configuration
    # ...

ファイルやディレクトリは .rubocop.yml からの相対位置として記述されます。

注: Rakefile のようにファイル名のみのパターンは任意のディレクトリにおいてマッチするがこのパターンスタイルは非推奨です。任意のディレクトリにおいてファイル名をマッチさせる正しい方法はカレントを含めた `**/Rakefile` という書き方です。

注: `config/**` というパターンは config 以下の全てのファイルについて再帰的にマッチしますが、このパターンスタイルは非推奨となり、`config/**/*` に置換されるべきです。

注: Include および Exclude パラメータは特別です。それらは定義されている場所から始まるディレクトリツリーのために有効になっています。それらはサブディレクトリにある他の .rubocop.yml の Include および Exclude の設定によって隠されることはありません。これは 検査のための設定は一番近くの .rubocop.yml から、あるいは上向き探索という RuboCop の一般原則に従う他の全てのパラメータとは異なります。

cops は必要な時に (例えばあなたが Rails のモデルを app/models/*.rb のパスにマッチするファイルのみチェックしたい) 特定のファイルのセットのみを実行することができます。全ての cops は Include というパラメータをサポートします。

    Rails/DefaultScope:
      Include:
        - app/models/*.rb

cops は必要な時に (例えばあなたが特定のファイルにおいていくつかの cop のみを実行したい) 特定のファイルのセットのみを除外することもできます。全ての cops は Exclude というパラメータをサポートします。

    Rails/DefaultScope:
      Exclude:
        - app/models/problematic.rb

## 一般的な設定パラメータ

Include および Exclude に加えて、以下に示すパラメータは全ての cop のために有効になっています。

### Enabled

特定の cops はその特定の cop のため Enabled を false に設定することで無効にすることができます。

    Metrics/LineLength:
        Enabled: false

### 重要度

それぞれの cop は属する部署に基づいた既定の重要度を持ちます。レベルは Lint の waring や他の全ての convention です。cops はそれらの重要度レベルをカスタマイズできます。許可されているパラメータは refactor、convention、warning、error そして fatal です。上記の一般的なルールから一つの例外があり、それは Lint/Syntax という他の cop が呼び出される前に構文エラーをチェックする特別な cop です。これは無効にできず、fatal という重要度は設定で変更することができません。

    Metrics/CyclomaticComplexity:
      Severity: warning

## 自動で生成される設定

もしあなたが圧倒的な量の駄目なコードベースを持っているのであれば `rubocop --auto-gen-config` の使用および `inherit_from: .rubocop_todo.yml` をあなたの `.rubocop.yml` に追加するのが良い方法でしょう。生成された `.rubocop_todo.yml` はコードから検出された不備がある全ての cops を無効にします。次にあなたは生成されたエントリを一つづつ削除することで問題の解決を開始することができます。
