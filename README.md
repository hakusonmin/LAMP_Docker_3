## 概要
・<https://github.com/erpushpinderrana/dockerize-existing-drupal-project>
のリポジトリをベースとして改良したものである。上記リポジトリはdockerでphp mysql apache の
環境を再現している。

## 工夫した点
・apacheにおいてはlocal.apache.confを変更した。ProxyPassMatch形式はDirectoryIndexやPATH_INFO
に影響が出るようなのでSetHandler形式を利用した。

・mysqlの設定ファイルであるmy.cnfが存在しなかったため、utf8mb4などの設定が記載し、my.cnfを
追加した.またdocker.compose.yamlでmy.cnfはコンテナ側にcopyした。

・phpはdockerfileがphp7系のライブラリを利用していたので8系に変更した。
php.iniにおいてはopcacheを有効化した。またphp.iniを追加したためdockerfile内でコンテナ側にCOPYした。

### Docker自体においての工夫した点

・LandoやDDEVによる環境構築を行なった.ddevはdocker Scoutがバックグラウンドで起動してしまいリソース
をかなり消費するので断念した。landoは単体で利用する分には問題がなかったが複数の環境をLandoで立ち上げると重くなってしまった。Dockerのままの利用が複数環境利用しても高速に動作し、一番快適であった。

・Dockerで複数環境を立ち上げるために工夫した。具体的には<br>
①compose.yamlにnameを記載した。
compose.yamlは直上のディレクトリ名を元にリソースの判別を行っている。そのため同じリポジトリからdockerの
リポジトリをクローンし、それを複数環境で起動すると別々のリソースが起動せず、同期してしまう。
これを防ぐためcompose.yamlにnameを記載した。これにより"docker compose -p project-name up"
とすることで独立したリソースを展開することができるようになった。<br>
②またportsでホスト側のPORT番号を環境変数にすることで複数環境でもportのコンフリクトが防いだ。

・Dockerでmysqｌを利用するため、Rosettaを利用した。これによりmysqlでコンテナ起動時にエラ０が発生しなくなった
が、あくまでarm側でのベストエフォートのため注意が必要なようだ。

・VirtioFSを利用することによってMac上においても高速に動作した。

### drupalの起動において工夫した点
・通常通りGUI上でサイトをインストールしようとすると、「すでにDBがinstallされています」というエラーが
出てしまった。そのためdrushを使って、site installを行うことでDBをリセットしてサイトを起動することが
できるようなった、
(おそらく原因はdocker.compose.yamlでコンテナの起動と同時に、drupalのdbを作成するように記載していた
ことによると考えられる)

・またdrushでのsite install時に初期設定画面において、hostの部分をlocalhostや127.0.0.1とするとエラーが出てしまった。
これに対してhostの部分にmysqlを記入すると動作した。調査したところcompose.yamlファイルのcontainer_nameがコンテナ起動時のhost名になるとのことであった。(厳密に言うとhost名では無く、dockerネットワーク内のDNSによる名前解決によって、container_nameがhost名のような形で利用されるようだ。実際に
docker exec -it を行い、コンテナ内で hostname コマンドを打った場合は container_nameとは違う
値がhost名として表示された。)

・docker compose exec -it で dockerfileやcompose.yamlを書き換えた後
コンテナ内が期待した状態になっているのか確認した。

・phpのイメージはalpaineLinuxのベースを利用しているため docker exec -it php_2 ash つまり
bashではなくashを利用する必要がある。また、パッケージインストール時にはapkを利用するべき。
そして、alpine-LInuxはよく使うパッケージが抜けていることがあるため、軽量かつパッケージが充実している
debian-slimをベースイメージに利用することが推奨されているようだ。

