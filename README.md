# Learn the Chef basics on Ubuntu

Original Source: http://learn.chef.io/learn-the-basics/ubuntu/

## UbuntuでChefの基礎を学ぶ
Chef を使えばインフラストラクチャーのポリシー(サーバ上のソフトウェアをどのように運用管理するか)をコードとして表現することができます。

Chef を学ぶにあたって最も手っ取り早い方法は、何はさておき Server あるいは Node にログインして直接 Chef を設定・実行してみることです。

この練習を終えた後は、あなたはすでに以下のことが出来るようになっているでしょう。

- Chef が実行されると何が起こるか、その基礎を理解している
- 基本ポリシーに従って、Chef のコードを書くことができるようになる
- そのポリシーを Sever に適用することができるようになる

## セットアップ
練習では、Chef がどのように動作するのか感覚をつかんでもらうためにサーバーを直接的に管理していきますが、実際には、Windows や Linux や Mac OS などの端末からリモートでサーバーを管理します。

1. 仮想マシンを入手する

  - 方法1: Chef が提供している仮想マシンを使う
    
    ChefDK がすでにインストールされた仮想マシンが提供されているのでそれを使う。
    
  - 方法2: 自分で用意した仮想マシンを使う

    [システム要件](https://docs.chef.io/chef_system_requirements.html)を確認し、ChefDK を自分でインストールする。
    https://downloads.chef.io/chef-dk/
    
    ```sh:Terminal
    $ curl -O *.deb
    $ sudo dpkg -i *.deb
    $ echo 'eval $(chef shell-init bash) >> .bash_profile'
    ```

2. テキストエディタを入手する

  仮想マシン上のテキストエディタを設定する必要があります。Chefのコードを書くため、シンタックスハイライト機能や行番号表示機能があるテキストエディタの利用を推奨します。

  - `gedit &`
  - `Atom`
  - `Sublime Text`
  - `Vim`
  - `Emacs`
  - `nano`

## *Resource* を設定する
> A Chef resource describes some piece of infrastructure, such as a file, a template, or a package. A Chef recipe is a file that groups related resources, such as everything needed to configure a web server, database server, or a load balancer.

To get started, let's look at a basic configuration management project. You'll learn how to manage the Message of the Day (MOTD) file for your organization. The MOTD file is an example of a resource.

1. 作業ディレクトリを設定する

  ホームディレクトリ `~/` に `chef-repo` ディレクトリを作成する
  
  ```:Terminal
  $ mkdir ~/chef-repo
  $ cd ~/chef-repo
  ```

2. MOTD ファイルを作成する

  `~/chef-repo` ディレクトリで `hello.rb` ファイルを作成し、下記の内容で保存する
  
  ```ruby:hello.rb
  file 'motd' do
    content 'hello world'
  end
  ```
  
  `chef-apply` コマンドを実行してファイルに書いた内容を適用させる
  
  ```sh:Terminal
  $ chef-apply hello.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * file[motd] action create
      - create new file motd
      - update content in file motd from none to b94d27
      --- motd    2015-07-07 04:49:55.940112458 +0000
      +++ ./.motd20150707-2007-pa3g36     2015-07-07 04:49:55.940112458 +0000
      @@ -1 +1,2 @@
      +hello world
  ```
  
  `more` コマンドでファイルの中身を確認する
  
  ```sh:Terminal
  $ more motd
  hello world
  ```
  
  もう一度 `chef-apply` コマンドを実行してみる
  
  ```sh:Terminal
  $ chef-apply hello.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * file[motd] action create (up to date)
  ```
  
  先ほどとは違った出力が得られたことでしょう。
  
  これは、すでに適用済みの処理を再び実行する必要がないと判断される場合、Chef は何もしないからです。
  
  `motd` ファイルが既に存在し、内容にも変更がない場合、Chef は新たにファイルを作成したり変更を加えたりすることはありません。
  
  Chef は現在の設定を見て、Server の状態が設定内容と異なる場合に変更を適用します。 

3. MOTDファイルの内容を更新する

  `hello.rb` ファイルの内容を下記の通り変更します
  (`hello world` が `hello chef` に変更されています)
  
  ```ruby:hello.rb
  file 'motd' do
    content 'hello chef'
  end
  ```
  
  `chef-apply` を実行します
  
  ```sh:Terminal
  $ chef-apply hello.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * file[motd] action create
      - update content in file motd from b94d27 to c38c60
      --- motd    2015-07-07 04:49:55.940112458 +0000
      +++ ./.motd20150707-2597-ntuyvx     2015-07-07  05:37:21.564094130 +0000
      @@ -1,2 +1,2 @@
      -hello world
      +hello chef
  ```

  `hello.rb` ファイルの内容を書き換えたので、設定内容と合致するように、Chefが `motd` ファイルの内容を更新したことが分かります。

4. MOTDファイルの内容が Chef 以外によって書き換えられないことを確認する

  MOTDファイルが他の方法で変更できないことを確認していきます。
  
  共同で作業をしている誰かが手動で `motd` ファイルに変更を加えた場面を想定してください。内容は `hello chef` から `hello robots` に書き換えられたこととします。テキストエディタでファイルを開いて編集するか、下記コマンドラインのような手法で内容を変更してください。
  
   ```sh:Terminal
   $ echo 'hello robots' > motd
   ``` 

  そして、`chef-apply` を実行します
  
  ```sh:Terminal
  $ chef-apply hello.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * file[motd] action create
      - update content in file motd from 548078 to c38c60
      --- motd    2015-07-07 05:58:50.406336126 +0000
      +++ ./.motd20150707-2884-2cr1lv     2015-07-07 05:59:05.950336026 +0000
      @@ -1,2 +1,2 @@
      -hello robots
      +hello chef
  ```

  Chefによってオリジナルの設定が復元されました。実はこれがChefの良い所なのです。もしChef以外の他の方法でリソースの状態が変更されても、Chefはリソースの状態が *Resource* ファイルに記述した通りになることを保証してくれるのです。Chefを使えば、サーバーに新しい設定を適用したり、もしくは現在の状態を保ち続けることができます。

5. MOTDファイルを削除する

  さて、MOTDによる練習をひと通り終えたので、掃除をしましょう。`~/chef-repo` ディレクトリで `goobye.rb` という名前のファイルを作成して以下の内容で保存しましょう。
  
  ```ruby:goobye.rb
  file 'motd' do
    action :delete
  end
  ```

  ファイルを削除するために `goodbye.rb` ファイルを適用しましょう
  
  ```sh:Terminal
  $ chef-apply goodbye.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * file[motd] action delete
      - delete file motd
  ```
  
  さて、上記出力結果によれば `motd` ファイルはなくなったようです。しかし、本当に削除されたのかどうか、下記コマンドでそれを検証しましょう。
  
  ```sh:Terminal
  $ more motd
  motd: No such file or directory
  ```

### まとめ

- Resources describe the what, not the how

  Your policy declares what state each resource should be in, but not how to get there. In this lesson, you declared that the file motd must exist and what its contents are, but you didn't specify how to apply that policy. This layer of abstraction can not only make you more productive, but it can also make your work more portable across platforms.

  A recipe declares what state each resource should be in but not how to achieve that state. Chef handles these complexities for you.
- *Resource* は *Action* を持つということ
  
  ファイルを削除した際に `:delete` という Action を見たかと思います。
    
  Chef の全ての Resource はデフォルトの Action を持ちます。デフォルトの Action は多くの場合、例えばファイルの `create` や、パッケージの `install`、サービスの `start` など、Action のうちで最も __一般的かつ肯定的__ なものがデフォルトになっています。
    
  ファイルを作成した際に `:create` と明示しなかったのは、file リソースにおいて `:create` はデフォルトの Action だからです。もちろん、明示的に書くこともできます。
    
  それぞれの Resource 種別に関して、デフォルトの Action について説明がなされています[^1]。例えば `file` の場合は[こちら](https://docs.chef.io/resource_file.html)に説明があります。
    
[^1]: [About Resources and Providers & Chef Docs](http://docs.chef.io/resource.html#resources)

- *Resource* を *Recipe* で管理する

  Chef では `hello.rb` を Recipe と呼び、Server の構成状態を連続的に書き連ねたファイルです。
  Recipe には、Webサーバーやデータベースサーバー、もしくはロードバランサーなどを構成するために必要な、関連する全ての状態の記述が含まれています。
  
  この練習で使用した Recipe は、MOTDファイルを管理するために必要な全てを示しています。 `chef-apply` は コマンドラインから Recipe を適用させるために使用しました。

## Package と Service を設定する

1. Apache パッケージをインストールする

  Apache パッケージ `apache2` をインストールしましょう。`~/chef-repo` ディレクトリで、以下の内容の Recipe を `webserver.rb` というファイル名で作成します。
  
  ```ruby:webserver.rb
  package 'apache2'
  ```
  
  ここで Action を指定する必要はありません。`:install` がデフォルトの Action だからです。`chef-apply` で Recipe を適用させましょう。
  
  ```sh:Terminal
  $ sudo chef-apply webserver.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * apt_package[apache2] action install
      - install version 2.4.7-1ubuntu4.4 of package apache2
  ```
  
  > :information_source: パッケージのインストールが伴う場合、root 権限で実行する必要があるので `sudo` を付けなければなりません。もしすでに仮想マシンを root で実行している場合は `sudo` は不要です。
  
  ここでもう一度 Recipe を実行してみましょう。
  
  ```sh:Terminal
  $ sudo chef-apply webserver.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * apt_package[apache2] action install (up to date)
  ```
  
  パッケージはすでにインストールされているので、Chef は何もしない事が分かります。

2. Apache サービスを有効にして開始させる

  Apache サービスを有効にし、Server の起動時に開始するようにしましょう。`webserver.rb` を以下のように変更します。
  
  ```ruby:webserver.rb
  package 'apache2'

  service 'apache2' do
    supports :status => true
    action [:enable, :start]
  end
  ```
  
  上記コードでは複数の Action を定義しています。
  
  > :computer: Ubuntu 14.04 では2つの init system を提供しています。`supports :status => true` の部分では、`apache2` の init script が `status` メッセージをサポートしていることを Chef に教えています。
  
  Recipe を適用させてみましょう。
  
  ```sh:Terminal
  $ sudo chef-apply webserver.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * apt_package[apache2] action install (up to date)
    * service[apache2] action enable (up to date)
    * service[apache2] action start (up to date)
  ```
  
  パッケージはすでにインストールされているので、何も実行されません。同様に、サービスもすでに開始され、有効になっています。いくつかの Linux ディストリビューションでは、Apache がインストールされても起動または有効にならないものがありますが、Chef を使えば、サービスが起動しているのか、有効になっているのか確認が容易です。

3. ホームページを追加する

  独自のホームページを整えて追加しましょう。
  すでに `file` Resource の書き方はお分かりかと思います。`webserver.rb` の最後にデフォルトホームページ `/var/www/html/index.html` の設定を追加してください。Recipe 全体としては以下のようになります。
  
  > :+1: 一連の操作を覚えるのに、コードやコマンドを手打ちするのはとても良い方法です。しかしながらコードやコマンドのテキストをコピーしてSSHセッションに貼り付けることも可能ですので、覚えておいてください。
  
  ```ruby:webserver.rb
  package 'apache2'

  service 'apache2' do
    supports :status => true
    action [:enable, :start]
  end

  file '/var/www/html/index.html' do
    content '<html>
    <body>
      <h1>hello world</h1>
    </body>
  </html>'
  end
  ```
  
  > :computer: バージョン 14.04 以前の Ubuntu では、`/var/www/index.html` がデフォルトのホームページでした。お使いの仮想マシンに合わせて Recipe を調整してください。
  
  用意した Recipe を適用させてみましょう。
  
  ```sh:Terminal
  $ sudo chef-apply webserver.rb
  Recipe: (chef-apply cookbook)::(chef-apply recipe)
    * apt_package[apache2] action install (up to date)
    * service[apache2] action enable (up to date)
    * service[apache2] action start (up to date)
    * file[/var/www/html/index.html] action create
      - update content in file /var/www/html/index.html from 538f31 to 2914aa
      --- /var/www/html/index.html        2015-07-07 09:21:17.183400886 +0000
      +++ /var/www/html/.index.html20150707-6974-1jh614b  2015-07-07 12:01:57.428958053 +0000
      @@ -1,379 +1,6 @@
      - :
      - :
      - ~
      - :
      +<html>
      +  <body>
      +    <h1>hello world</h1>
      +  </body>
      +</html>
  ```

4. 作成した Web ページが正常に動作しているか確認する

  `curl` コマンドを使って、ページが閲覧可能かどうか確認します。
  
  ```sh:Terminal
  $ curl localhost
  <html>
    <body>
      <h1>hello world</h1>
    </body>
  </html>
  ```

  もしくは、他のマシンの Web ブラウザから Server 上のページにアクセスして確認することもできます。

### まとめ

`package`,`service` Resource がどのように動作するのかを見ていきました。ここまでで、`file`,`package`,`service` の3種類の Resource を学んだことになります。

また、複数の Action を適用する方法も説明しました。しかし、Chef の Resource と Action の実行順序はどのようになっているのでしょうか?

__Chef は記述した通りの順序で処理を実行します__





