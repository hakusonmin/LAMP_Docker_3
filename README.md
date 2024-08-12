## 概要
・<https://github.com/erpushpinderrana/dockerize-existing-drupal-project>
のリポジトリをベースとして改良したものである。上記リポジトリはdockerでphp mysql apache の
環境を再現している。

## 工夫した点
・apacheにおいてはlocal.apache.confを変更した。ProxyPassMatchはDirectoryIndexやPATH_INFO
に影響がでるようなのでSetHandlerを利用した。

・mysqlの設定ファイルであるmy.cnfが存在しなかったためutf8mb4などの設定が記載されているmy.cnfを
追加した.またdocker.compose.yamlでmy.cnfはコンテナ側にcopyしている。

・phpはdockerfileがphp7系のもののライブラリを利用していたので8系に変更した。
php.iniにおいてはopcacheを有効化した。またphp.iniを追加したためコンテナ側にCOPYした。

### Docker自体においての工夫した点

・LandoやDDEVによる環境構築を行なった.ddevはdocker Scoutがバックグラウンドで起動してしまいリソース
をかなり消費するので断念した。landoは単体で利用する分には問題がなかったが複数の環境をLandoで立ち上げると重くなってしまった。Dockerのままの利用が複数環境利用しても高速に動作し、一番快適であった。

・Dockerで複数環境を立ち上げるために工夫した。具体的には<br>
①compose.yamlにnameを記載した。
compose.yamlは直上のディレクトリ名を元に区別を行っている。そのため同じリポジトリからdockerの
リポジトリをクローンし、それを複数環境で起動すると別々のリソースが起動せず、同期してしまった。
これを防ぐためcompose.yamlにnameを記載した。これにより"docker compose -p project-name up"
とすることで独立したリソースを展開することができるようになった。<br>
②またportsでホスト側のPORT番号を環境変数にすることで複数環境でもportのコンフリクトが起こらなくなった。

・Dockerでmysqｌを利用するためにRosettaを利用した。

・VirtioFSを利用することによってMac上においても高速に動作した。

### drupalの起動において工夫した点
・通常通りにGUI上でサイトをインストールしようとしたらすでにDBがinstallされていますというエラーが
出てしまった。そのためdrushを使ってsite installを行うことで成功した。
(おそらく原因はdocker.compose.yamlでdrupalのdbを起動とともに作成していたからだと考えられる)

・またdrushでのsite install時にhostの部分をlocalhostや127.0.0.1とするとエラーが出てしまった。
これに対してdocker ps をするとmysqlとmysqlのhost名に指定されていた。そしてhostの部分にmysqlを記入すると
動作した。調査したところcompose.yamlファイルのcontainer_nameがコンテナ起動時のhost名になるとの
ことであった。(厳密に言うとhost名では無く、dockerネットワーク内のDNSにより名前解決される。実際に
docker exec -it して コンテナ内で hostname コマンドを打った場合は container_nameとは違う
値が表示された) (docker_networkの名前解決できるServiceDiscovery <https://dev.classmethod.jp/articles/docker-service-discovery/>)

・docker compose exec -it で dockerfileやcompose.yamlを書き換えた後<br>
コンテナ内が期待した状態になっているのか確認した。

・phpのイメージはalpaineLinuxのベースを利用しているため docker exec -it php_2 ash つまり
bashではなくashを利用する必要がある。また、パッケージインストール時にはapkを利用するべき。
また、alpine-LInuxはよく使うパッケージが抜けていることがあるため、軽量かつパッケージが充実している
debian-slimをベースイメージに利用することが推奨されているようだ。

