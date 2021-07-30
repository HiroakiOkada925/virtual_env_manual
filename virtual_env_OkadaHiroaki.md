# 環境構築手順書マークダウン

## バージョン一覧
| 名前 | バージョン | 
| -------- | -------- | 
| PHP     | PHP7.3     | 
| Nginx | 1.21.1 |
| MySQL     | MySQL5.7     |
| Laravel | Laravel6.0 |
| OS     | CentOS7     |

## VirtualBoxのインストール
まず最初に下記のサイトからそれぞれのdmgファイルをダウンロード後、インストールを進めます。
[Virtual Box公式](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0)
OS X hosts を選択します。
以下のコマンドを実行してVirtualBoxのウィンドウが表示されれば正常にインストールされています。

    $ virtualbox
コマンド実行後は入力を受け付けない状態となるため、 Control + c を押してください。

## Vagrantのインストール

    $ brew install --cask vagrant
上記のコマンドでVagrantをインストールします。
インストールが終わったら vagrant コマンドが使用可能かどうか確認するため以下のコマンドを実行します。

    $ vagrant -v
バージョンの確認ができたら問題なく使用が可能な状態です。

## vagrant boxのダウンロード
今回はLinuxのCentOSのバージョン7のbox名 centos/7 を指定して実行してみましょう。

    $ vagrant box add centos/7
コマンドを実行すると、下記のような選択肢が表示されます。

    1) hyperv
    2) libvirt
    3) virtualbox
    4) vmware_desktop

    Enter your choice: 3
今回使用するソフトはVirtualBoxのため、3を選択してenterを押しましょう。
下記のように表示されたら完了です。

    Successfully added box 'centos/7' (v1902.01) for 'virtualbox'!

## Vagrantの作業ディレクトリを用意する
Vagrantの作業用ディレクトリを作成します。
以下のいずれかのディレクトリの下にディレクトリを作成しましょう。
自分の作業用ディレクトリ
デスクトップ
今回はvagrant_test2という名前で作成します。

※ 作成したディレクトリをのちのちに移動すると、ゲストOSへのログインが上手くいかなくなってしまうので注意してください。

    mkdir vagrant_test２
作成したフォルダの中で以下のコマンドを実行します。

    cd vagrant_test
vagrant init box名で先ほどダウンロードしたcentos/7を使用します。

    vagrant init centos/7
    
## Vagrantfileの編集
今回行う編集は、三箇所です。
変更点１

    config.vm.network "forwarded_port", guest: 80, host: 8080

変更点２

    config.vm.network "private_network", ip: "192.168.33.19"
上記二箇所の # が付いているのを外します。
今回はipに"192.168.33.19"を指定します。
<br>
また以下の箇所はコメントインし、変更を加えてください。
変更点3

    config.vm.synced_folder "../data", "/vagrant_data"
↓ 以下に編集

    config.vm.synced_folder "./", "/vagrant", type:"virtualbox"

## Vagrant プラグインのインストール
今回は vagrant-vbguest というプラグインをインストールします。
vagrant-vbguestは初めに追加したBoxの中にインストールされているGuest Additionsというもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグインです。

    vagrant plugin install vagrant-vbguest
vagrant-vbguestのインストールが完了しているか下記のコマンドを実行して確認しましょう。

    vagrant plugin list
    
## Vagrantを使用してゲストOSの起動
以上で仮想環境を構築する準備は整いました。
Vagrantfileがあるディレクトリにて以下のコマンドを実行して、早速起動してkernel をアップデートしましょう。

    vagrant up
    vagrant ssh
    sudo yum -y update kernel
    exit
    vagrant reload --provision
    
## パッケージをインストール
では早速グループパッケージをインストールしていきましょう。
下記のコマンドを実行してください。

    sudo yum -y groupinstall "development tools"
このコマンドを実行することによりgitなどの開発に必要なパッケージを一括でインストールできます。

## PHPのインストール
次はPHPをインストールしていきます。

yumコマンドを使用してPHPをインストールした場合、古いバージョンのPHPがインストールされてしまいます。
Laravelを動作させるにはPHPのバージョン7以上をインストールする必要があるため
yumではなく外部パッケージツールをダウンロードして、そこからPHPをインストールしていきます。
今回はphp7.3をインストールしたいのでremi-php73という形で指定します。

    sudo yum -y install epel-release wget
    sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
    sudo rpm -Uvh remi-release-7.rpm
    sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip
    php -v
PHPのインストールは、以上で終わりです。

## composerのインストール
次にPHPのパッケージ管理ツールであるcomposerをインストールしていきます。

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"

どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行います

    sudo mv composer.phar /usr/local/bin/composer
    composer -v
composer のバージョンが確認できましたらOKです。
以上の作業でゲストOS内にPHPとcomposerコマンドの実行環境が整いました。

## Laravelアプリケーションインストール
今回はLaravelの６.０をインストールしたいので下記コマンドを実行してインストールします。

    cd /vagrant
    composer create-project --prefer-dist laravel/laravel blog "6.*"

下記コマンドでLaravelの6.0のインストールが確認できれば成功です。

    php artisan -V

## データベースのインストール
今回インストールするデータベースはMySQLとなります。versionは5.7を使用します。
rpmに新たにリポジトリを追加し、インストールを行います。

    sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
    sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm
    sudo yum install -y mysql-community-server
    mysql --version
versionの確認ができましたらインストール完了です。
次にMySQLを起動し接続を行います。

    sudo systemctl start mysqld
    mysql -u root -p
    Enter password:
今回はデフォルトでrootにパスワードが設定されてしまっています。
まずはpasswordを調べ、接続しpassswordの再設定を行っていく必要があります。

    sudo cat /var/log/mysqld.log | grep 'temporary password'  # このコマンドを実行したら下記のように表示されたらOKです
    2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge
hogehoge と記載されている箇所に存在するランダムな文字列がパスワードとなります。
では先程出力したランダム文字列をコピー後、再度以下のコマンドを実行し、パスワード入力時にペーストしてください。

    $ mysql -u root -p
    $ Enter password:
    mysql >
問題なく接続できたでしょうか？
次に接続した状態でpasswordの変更を行います。
新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上の設定をする必要があります。
MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。

    $ sudo vi /etc/my.cnf
<br>
    # 省略

    [mysqld]

    # 省略

    # read_rnd_buffer_size = 2M
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock

    # 下記の一行を追加
    validate-password=OFF
編集後はMySQLサーバの再起動が必要です。

    $ sudo systemctl restart mysqld
再度MySQLにログインしてパスワードの初期設定を行えば、簡単なパスワードで登録ができます。

    mysql > set password = "新たなpassword";
以上で、MySQLの導入と設定が完了となります。

## Laravelを動かす
laravel_appディレクトリ下の .env ファイルの内容を以下に変更してください。

    DB_PASSWORD=
    # ↓ 以下に編集
    DB_PASSWORD=登録したパスワード
では、/vagrant/laravel_appディレクトリに移動して php artisan migrate を実行します。

## Nginxのインストール
Nginxの最新版をインストールしていきます。
viエディタを使用して以下のファイルを作成します。

    $ sudo vi /etc/yum.repos.d/nginx.repo
書き込む内容は以下になります。

    [nginx]
    name=nginx repo
    baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/
    gpgcheck=0
    enabled=1
書き終えたら保存して、以下のコマンドを実行しNginxのインストールを実行します。

    $ sudo yum install -y nginx
    $ nginx -v
Nginxのバージョンは確認できたでしょうか？
ではNginxの起動をしましょう。

    $ sudo systemctl start nginx
ブラウザにて http://192.168.33.19 と入力し、NginxのWelcomeページが表示されましたでしょうか？
表示されたら問題なく動いていますので次に進みましょう。

## Laravelを動かす
では、まずNginxの設定ファイルを編集していきます。
/etc/nginx/conf.d ディレクトリ下の default.conf ファイルが設定ファイルとなります。

    $ sudo vi /etc/nginx/conf.d/default.conf
<br>
    server {
      listen       80;
      server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
      
      root /vagrant/laravel_app/public; # 追記
      index  index.html index.htm index.php; # 追記

      #charset koi8-r;
      #access_log  /var/log/nginx/host.access.log  main;

      location / {
          #root   /usr/share/nginx/html; # コメントアウト
          #index  index.html index.htm;  # コメントアウト
          try_files $uri $uri/ /index.php$is_args$args;  # 追記
      }

      # 省略

      # 該当箇所のコメントを解除し、必要な箇所には変更を加える
      # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

      location ~ \.php$ {
      #    root           html;
          fastcgi_pass   127.0.0.1:9000;
          fastcgi_index  index.php;
          fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
          include        fastcgi_params;
      }

      # 省略
Nginxの設定ファイルの変更は、以上です。
次に php-fpm の設定ファイルを編集していきます。

    $ sudo vi /etc/php-fpm.d/www.conf
変更箇所は以下になります。

    ;24行目近辺
    user = apache
    # ↓ 以下に編集
    user = nginx

    group = apache
    # ↓ 以下に編集
    group = nginx
設定ファイルの変更に関しては、以上となります。
では早速起動しましょう(Nginxは再起動になります)。

    $ sudo systemctl restart nginx
    $ sudo systemctl start php-fpm
そして、以下のコマンドを実行して nginx というユーザーでもログファイルへの書き込みができる権限を付与してあげましょう。

    $ cd /vagrant/laravel_app
    $ sudo chmod -R 777 storage
ブラウザにて、 http://192.168.33.19 を入力して確認してください。
ララベルのwellcome画面が表示されていれば成功です。

## Laravelの認証機能作成
Laravel6.0での認証機能を実装していきます。
以下のコマンドを実行します。

    $ composer require laravel/ui 1.*
    $ php artisan ui vue --auth
    
下記のように表示されれば成功です。

    Vue scaffolding installed successfully.
    Please run "npm install && npm run dev" to compile your fresh scaffolding.
    Authentication scaffolding generated successfully.
これでLaravelのwellcome画面の上部にloginとregisterの表示が追加されたはずです。

## Class ‘PDO’ not found
ユーザー登録する時にClass ‘PDO’ not foundのエラーが発生した際は

    yum install --enablerepo=remi-php73 php-pdo
    yum install --enablerepo=remi-php73 php-pdo_mysql
上記のコマンドを実行してあげましょう。
このエラーは何かしらの理由でPHPインストール時にphp-pdoがインストールされていない場合に発生する物なので上記のコマンドでpdoをインストールしてあげれば大丈夫です。

## 環境構築の所感
最初はエラーに怯えていたが、どんなエラーにも必ず理由が記述されているので、そこを調べていけば必ず解決できるという事に気づく事ができた。
環境構築は一見とても複雑な作業に見えるが、始めにある程度段取りを決めて一つ一つ丁寧に取り組んでいけば必ず達成できるので、エラーが出ても恐れず取り組んでいきたいと思った。
また、１度もエラーを出さずに進めていくことは不可能なので、始めからエラーが出る前提で取り組んでいけば気持ち的にも楽になれると気づく事ができた。
まだまだ知識が足りず、数をこなしてどんどん慣れていかないといけないと痛感したので、これからも環境構築の勉強に積極的に取り組んでいきたい。

## 参考サイト
[Vagrant + CentOS7 + PHP7.3 でLaravelの開発環境を構築する](https://mikepon.jp/vagrant-centos7-php7-laravel/)
<br>
[laravelをバージョン指定(laravel6)でインストールする方法](https://qiita.com/megukentarou/items/d62bf85822cd57c75bff)
<br>
[Laravel6 ログイン機能を実装する](https://qiita.com/ucan-lab/items/bd0d6f6449602072cb87#laravelui-%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)
<br>
[Vagrant + VirtualBOx で 最新のCentOS7 vbox(centos/7 2004.01)でマウントできない問題](https://qiita.com/mao172/items/f1af5bedd0e9536169ae)
<br>
[php-fpmの設定](http://www.ajisaba.net/php/php-fpm/php74-fpm_conf.html)
