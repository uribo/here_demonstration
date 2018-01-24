hereパッケージの導入でファイル参照のパス問題で悩むことはなくなるかもしれない
================

去年から気になっていたものの、その利点や使い道について理解できていなかった[**here**パッケージ](https://krlmlr.github.io/here/)、ようやくにして少しわかった気がする。今は声を大にして言える。**here**は良いぞ。**here**をプロジェクトに導入することで、これまでにあったWindows - macOS間でのパス表記の違いや作業ディレクトリの階層性によるパスが正しく指定できないといった課題を解決することができるかもしれない。と言う訳で布教用の記事を書く。また、テスト用のリポジトリも用意しているので、**here**を使ってそのご利益を享受されたい方はこちらをcloneなりダウンロードするなりして手前で実行してほしい。

<https://github.com/jennybc/here_here>

参考) <https://github.com/jennybc/here_here>

問題
----

Rを使っていて、ファイルの保存先を指定するのに失敗してイラつきを覚える、そんな経験は誰しもがあるだろう。Rでは、作業ディレクトリ (working directory。`getwd()`で現在の作業ディレクトリを確認する)と呼ばれる基盤となるディレクトリが設定される。これにより目的のファイルまで簡単にたどり着くことができるなどの利点もあるのだが、複数のプロジェクトを運用している場合、プロジェクトのフォルダをまたいでしまったり、作業ディレクトリを頻繁に切り替える作業が発生してしまう(`setwd()`により作業ディレクトリは変更される)。皆さんには作図したファイルの保存先に`fig`フォルダを作っていたのに、全く異なる場所にファイルを保存してしまった、ということがないだろうか?私はしばしばそんな経験をした。ここに書いたにもパスに関する多様な問題があるだろうが、もっとも頻繁に遭遇する問題だと思われる2点について、次にまとめる。

RStudioの一機能であるRプロジェクトは、プロジェクトに応じた作業ディレクトリの構築を可能にする。Rプロジェクトであることを示す `.Rproj`ファイルはプロジェクトとして扱うフォルダのトップの階層に置かれ、そこを起点とした作業ディレクトリでプロジェクトを回していくことができる。プロジェクトに関する全てのファイルをプロジェクトの中に置くことで利便性が増し、上記の問題を防ぐことが可能となる。

一方で、ファイルの参照形式はOS間で異なることがあり、参照がうまくできないことがしばしばある。次のコードはWindowsにおいてdataフォルダ中の`iris.csv`を読み込むための処理だが、これをUNIXベースのOSで実行するとエラーとなる。UNIXではフォルダの区切り文字に`/`を使うのが習慣であり、他方Windowsでは区切り文字に`/`の他、`\` (`￥`)の使用が認められるためである。

``` r
# Windowsで実行可能な読み込みはUNIX環境ではエラーとなる
read.csv("data\iris.csv")
```

またプロジェクトの外部で、ドライブからパスを指定しなくてはいけない時も、Windowsでは次のように指定する。

``` r
# ドライブからファイルを参照する
read.csv("C:/documents/uribo/here_demonstration/data/iris.csv")
```

UNIXではユーザディレクトリは `/home/<username>`であったり、`/Users/<username>`となっていたりするがホームディレクトリは `~`で表記を省略することもできる。また省略されたパスを`path.expand()`により完全な形で示せる。

``` r
# ホームディレクトリからファイルを参照する
read.csv("~/here_demonstration/data/iris.csv")
```

``` r
path.expand("~/here_demonstration/data/iris.csv")
```

    ## [1] "/home/rstudio/here_demonstration/data/iris.csv"

これはプロジェクト機能をもってしても残り続ける課題である。

またRmdファイルをプロジェクト内のフォルダに保存しておくと、プロジェクトの作業ディレクトリよりも深い階層でRコードを実行することになるため、コンソールでの実行とRmdファイル中のコードの記述が異ってしまう。具体的には、`data/iris.csv`を参照するには、一つ階層を遡って`../data/iris.csv`としなくてはいけない。また、Rmdファイルをより深い階層においた場合、さらに遡って`../../data/iris.csv`と記述することとなる。これに対応するのは手間である。

hereによるファイル参照
----------------------

こうした課題を解決するために、**here**パッケージは優れた機能を提供する。**here**はCRANNからインストールができる。この記事を書いた際のバージョンは0.1である。Windows環境にインストールした際、**backports**パッケージの古いバージョンをインストールしておかないといけないとエラーになったので、[参考のリンク](https://stackoverflow.com/questions/46416458/backports-1-1-1-package-fails-to-install)を貼っておく。

``` r
library(here)
```

    ## here() starts at /home/rstudio/here_demonstration

**here**パッケージを読み込むと、上記のメッセージが出力される。出力先のパスが基点を示している。Rプロジェクトが同じ階層内にある場合、その階層が基点にみなされる。

**here**の考え方としては、あるディレクトリをプロジェクトとみなした場合、ファイルの参照を常にプロジェクトの基点から表記する、というものである。関数`here`はパスを表現するのに用いられるが、従来の区切り文字で表記する方法の他にRの文字列としてフォルダ名を与えていくことでパスを表現することもができる。

``` r
# 従来のパス表記方法
here("data/iris.csv")
```

    ## [1] "/home/rstudio/here_demonstration/data/iris.csv"

``` r
# 文字区切りでフォルダ、ファイルを示す
here("data", "iris.csv")
```

    ## [1] "/home/rstudio/here_demonstration/data/iris.csv"

パスを出力するだけなので、ファイルの有無は気にしない。

``` r
file.exists("aaa/bbb/ccc.csv")
```

    ## [1] FALSE

``` r
here("aaa", "bbb", "ccc.csv")
```

    ## [1] "/home/rstudio/here_demonstration/aaa/bbb/ccc.csv"

`c()`を使ってこんなこともできる。複数の文字列を指定した際は各要素の位置に対応したパスが出力される。

``` r
here("data", c("aa", "bb"), "iris.csv")
```

    ## [1] "/home/rstudio/here_demonstration/data/aa/iris.csv"
    ## [2] "/home/rstudio/here_demonstration/data/bb/iris.csv"

``` r
here("data", c("aa", "bb"), c("iris.csv", "mtcars.csv"))
```

    ## [1] "/home/rstudio/here_demonstration/data/aa/iris.csv"  
    ## [2] "/home/rstudio/here_demonstration/data/bb/mtcars.csv"

`here()`がその効力を発揮するのは、特に深い階層にあるRコードやRmdファイルを実行する時である。サンプルリポジトリに作成した`hoge/fuga/sample2.Rmd`から`data/iris.csv`を参照するには`../../data/iris.csv`と記述しなくてはいけなかったところを`here("data", "iris.csv")`の記述に置き換えられる。`data/iris.csv`を参照するには、常に`here("data", "iris.csv")`とすれば良いのである。

``` r
dim(read.csv(here("data", "iris.csv")))
```

    ## [1] 150   6

``` r
# 作業ディレクトリを変更しても data/iris.csvの参照方法は変わらない
setwd("hoge/fuga/")
# The working directory was changed to /home/rstudio/here_demonstration/hoge/fuga inside a notebook chunk. The working directory will be reset when the chunk is finished running. Use the knitr root.dir option in the setup chunk to change the working directory for notebook chunks.
dim(read.csv(here("data", "iris.csv")))
```

    ## [1] 150   6

``` r
# 作業ディレクトリの変更は維持される
getwd()
```

    ## [1] "/home/rstudio/here_demonstration/hoge/fuga"

なお、RMarkdownのチャンクでは、チャンク内で完結し、チャンクが切り替わると基点に戻る。

``` r
getwd()
```

    ## [1] "/home/rstudio/here_demonstration"

素晴らしい！

### Rプロジェクトの外部でhere!

**here**はRプロジェクトのあるフォルダを基点とするのが基本だが、他にも基点となり得るファイルやフォルダが認められている。GitリポジトリやRパッケージ開発に必要なDESCRIPTIONファイル(`^Package:`で始まること)などである。加えて、独自の`.here`ファイルがあるフォルダも基点にすることができる。これはRプロジェクトを用いない分析プロジェクトでは有効だろう。任意の作業ディレクトリで`set_here()`を実行あるいは*path*引数にパスを指定して、`.here`を作成できる。

パス問題は、これまで多く悩まされてきたものなので、Rプロジェクトの台頭によって大分ストレスがなくなったが、**here**を導入することでさらにストレスフリーで作業を進められそうである。

Enjoy!
