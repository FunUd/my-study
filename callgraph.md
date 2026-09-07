CコードベースをGitHub Copilot Agentが高速に探索できるように、コールグラフ索引システムを実装してください。

目的

現在、Agentがコード探索時にgrep/ripgrepを多用しており、関数の呼び出し関係や関連箇所の発見に時間がかかっています。

CMakeでビルドされるCコードからコールグラフを自動生成し、Agent SkillからローカルCLIを呼び出すことで、関数のcaller/callee/call pathを即座に取得できる仕組みを構築してください。

最終的には、

「DataProcessor_Processの呼び出し元を調べる」

という要求に対して、grepでソースコードを総当たりするのではなく、コールグラフ索引を問い合わせて関連関数を発見できる状態にしてください。

重要な制約

MCP Serverは禁止

MCP Serverは使用しないでください。

Agentとの連携は、GitHub Copilot Agent SkillからローカルCLI/スクリプトを実行する方式を基本としてください。

想定構成：

CMake
→ compile_commands.json
→ Clang系解析
→ コールグラフ索引
→ ローカルCLI
→ Agent Skill
→ GitHub Copilot Agent

業務利用可能であること

使用するOSS、ライブラリ、ツールについて、業務利用可能なライセンスであることを必ず確認してください。

特に以下を確認してください。

- OSS名
- バージョン
- ライセンス
- 商用・業務利用可否
- 再配布時の義務
- NOTICE等の表示義務
- ソースコード開示義務の有無

GPL等、業務利用時のライセンス上の影響が大きいものは、安易に採用しないでください。

ライセンスが不明なライブラリは採用しないでください。

可能な限りApache-2.0、MIT、BSD系など、業務利用しやすいライセンスのOSSを優先してください。

1. 既存環境を最初に調査

実装を開始する前に、リポジトリを調査してください。

最低限、以下を確認してください。

- CMake構成
- CMakePresets
- compile_commands.jsonの生成方法
- Cソースの構成
- compiler
- Python環境
- 既存の開発用スクリプト
- 既存のAgent Skills
- GitHub Copilot Agentで利用可能なSkillの仕組み
- CI環境
- Windows/Linux双方で利用可能なツール

既存の仕組みがある場合は、それを優先して利用してください。

2. コンパイル情報を利用する

CMakeからcompile_commands.jsonを取得してください。

解析時には実際のコンパイル設定を利用し、少なくとも以下を考慮してください。

- include path
- compiler options
- preprocessor definitions
- conditional compilation
- source file

単純なgrepや正規表現によるコールグラフ生成は禁止します。

3. Cコード解析

Clang/LLVM系のAST解析を第一候補として検討してください。

少なくとも以下を取得してください。

- 関数定義
- 関数名
- ファイル
- 行番号
- caller
- callee
- call site

可能な限り以下にも対応してください。

- static関数
- macro
- #ifdef
- typedef
- function pointer

ただし、静的解析で確定できない関数ポインタの呼び出し関係等を、確定情報として扱わないでください。

4. コールグラフ索引

解析結果を毎回ソースコードから探索するのではなく、永続化してください。

最初の実装ではSQLite等のシンプルな方式を優先してください。

不要なGraph DBを導入しないでください。

最低限、以下の情報を保持してください。

functions:

- ID
- function name
- source file
- line
- static/non-static
- signature等、必要な情報

calls:

- caller
- callee
- source file
- line

関数名から高速に検索できるインデックスを作成してください。

5. CLI

Agent Skillから呼び出せるCLIを作成してください。

最低限、以下の操作を提供してください。

find-function <function>
get-callers <function>
get-callees <function>
get-call-path <from> <to>

出力はAgentが解釈しやすい機械可読形式を基本としてください。

JSONを第一候補とします。

例えば、

get-callers DataProcessor_Process

に対して、

{
  "function": "DataProcessor_Process",
  "callers": [
    {
      "function": "AplTask_Run",
      "file": "...",
      "line": 123
    }
  ]
}

のような結果を返してください。

6. Agent Skill

GitHub Copilot AgentがコールグラフCLIを利用するためのSkillを作成してください。

Skillには以下の探索ルールを定義してください。

1. 関数の呼び出し関係を調査するときは、コールグラフCLIを最初に使用する。
2. grep/ripgrepを最初の探索手段にしない。
3. callerを調べる場合はget-callersを使用する。
4. calleeを調べる場合はget-calleesを使用する。
5. 特定の2関数間の経路を調べる場合はget-call-pathを使用する。
6. コールグラフで対象関数を特定できない場合のみgrep/ripgrep等を補助的に使用する。
7. コールグラフの結果だけで実装内容を判断せず、必要なソースコードを確認する。
8. 静的解析による推定と確定した呼び出し関係を区別する。

7. 更新方式

ソースコード変更後にコールグラフが古くならないようにしてください。

以下を検討してください。

- 全再生成
- ファイル単位の差分更新
- compile_commands.json変更時の再解析
- Gitの変更ファイルを利用した差分更新

ただし、最初から複雑なインクリメンタル解析を実装する必要はありません。

まず確実に全再生成できる仕組みを完成させ、その後必要なら高速化してください。

8. Windows/Linux

開発者はWindowsを使用し、CIやAgent実行環境はLinuxになる可能性があります。

そのため、OS依存のパスやコマンドを極力避けてください。

可能であればPythonをCLI実装の基盤として使用し、Windows/Linux双方で同じコマンド体系で実行できるようにしてください。

9. テスト

実際のコードベースを対象としてテストしてください。

最低限、

- 関数を検索できる
- callerを取得できる
- calleeを取得できる
- call pathを取得できる
- static関数を扱える
- #ifdefを含むコードを解析できる
- Agent SkillからCLIを呼び出せる

ことを確認してください。

また、意図的に解析できないケースについても、異常終了するのではなく原因が分かるエラーを返してください。

10. ドキュメント

README等に以下を記載してください。

- 全体アーキテクチャ
- 使用OSS
- 各OSSのバージョン
- ライセンス
- 業務利用可否
- インストール方法
- コールグラフ生成方法
- CLI使用方法
- Agent Skillの使用方法
- 更新方法
- Windowsでの使用方法
- Linux/CIでの使用方法
- 既知の制約

実装方針

最初から巨大なKnowledge Graphを作らないでください。

まず、

CMake
→ compile_commands.json
→ Clang
→ SQLite
→ CLI
→ Agent Skill
→ GitHub Copilot Agent

という最小構成を完成させてください。

その後、必要性を確認して、

- module dependency
- global variable
- struct/type dependency
- message/event dependency
- semantic search

等へ拡張できる設計にしてください。

最重要の完成条件は、AgentがCコードを探索するとき、関数のcaller/calleeをgrepで探すのではなく、索引をCLI経由で即座に発見できることです。