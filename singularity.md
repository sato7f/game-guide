
- root権限がないサーバーでのSingularityでのMySQLやg++の環境構築
- singularityでsandboxを作ってその中でMySQLの起動もC++のコンパイルもする
- Singulalrityでは以下のディレクトリが自動でマウントされる（マウント元もマウント先も同じディレクトリ名）

    - $HOME
    - /sys
    - /proc
    - /tmp
    - /var/tmp
    - /etc/resolv.conf
    - /etc/passwd
    - $PWD

1. sandboxを置くディレクトリを作成

    ただし/home配下かつ/private以外（privateは外部の共有記憶メモリなので通信分遅い）

    ```shell:bash
    mkdir singularity-dir
    ```
<br>

2. 作成したディレクトリ内で以下のファイルを作成
    ```shell:bash
    cd singularity-dir
    touch singularity-def
    ```

    ```shell:singularity-def
    Bootstrap: docker
    From: ubuntu:20.04
    
    %post
       export DEBIAN_FRONTEND=noninteractive
       apt-get update && apt-get upgrade -y
       apt-get install -y mysql-client mysql-server tzdata vim curl git build-essential libmysqlcppconn-dev cmake
       # service mysql start
       # mysql -uroot -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY 'root';"
       # mysql -uroot -ppassword -e "CREATE DATABASE test_db;"
    
    %environment
       export MYSQL_ROOT_PASSWORD=root

    %runscript
       service mysql start && exec "$@"
    ```

    `#` でコメントアウトした場所はコメントアウトしなくてもいいかも（佐藤はコメントアウトした）
<br>

3. 以下のサイトでアカウントを作る
    - https://cloud.sylabs.io/
<br>

4. 以下のサイトでトークンを取得（テキストとしてコピーして後でコマンドラインに貼り付ける）
    - https://cloud.sylabs.io/tokens
<br>

5. singularityのリモートにログイン
    ```shell:bash
    singularity remote login
    ```
    
    トークンを要求されるのでここでペースト
<br>

6. リモートでsandboxを作成する（約10~15分かかる）
    ```shell:bash
    singularity build --remote --sandbox mysql_cpp_env.sandbox singularity-def
    ```
    `singularity-dir/mysql_cpp_env.sandbox`となる
<br>

7. 仮想環境に入って初期設定をする
    1. **仮想環境にに入る**
        ```shell:bash
        singularity shell --writable mysql_cpp_env.sandbox
        ```
    <br>
    
    2. MySQLの状態を確認
        ```shell:bash
        service mysql status
        ps aux | grep mysqld
        ```
        動いていたら一応再起動する（`service mysql stop`か`ps aux | grep mysqld`ででたプロセスidをkill）
    <br>

    3. セーフモードでMySQLを起動
        ```shell:bash
        mysqld_safe --skip-grant-tables &
        ```
    <br>

    4. rootとしてMySQLに入る
        ```shell:bash
        mysql -uroot
        ```
    <br>

    5. MySQLの初期設定をする
        ```shell:bash
        FLUSH PRIVILEGES;
        ALTER USER 'root'@'localhost' IDENTIFIED WITH 'mysql_native_password' BY 'root'
        FLUSH PRIVILEGES;
        ```
    <br>

    6. MySQLを出る
        ```shell:bash
        exit
        ```
<br>

7. 設定できた確認のためMySQLを再起動して入ってみる
    1. **MySQLの再起動**
        ```shell:bash
        service mysql stop
        mysqld --user=mysql &
        ```
        プロセスidが出力されるはず，その場合ちゃんと起動されてます
    <br>
    
    2. **設定したパスワードでMySQLに入る**
        ```shell:bash
        mysql -uroot -proot
        ```
    <br>

    3. C++のコンパイル
        ```sehll:bash
        g++ test.cpp -I/usr/include -L/usr/lib/x86_64-linux-gnu -lmysqlcppconn -fopenmp
        ./a.out
        ```

        ```cpp:test.cpp
        #include <algorithm>
        #include <bitset>
        #include <cassert>
        #include <filesystem>
        #include <fstream>
        #include <iostream>
        #include <iterator>
        #include <map>
        #include <queue>
        #include <set>
        #include <sstream>
        #include <stack>
        #include <string>
        #include <unordered_map>
        #include <unordered_set>
        #include <vector>
        // database用
        #include <mysql_driver.h>
        #include <mysql_connection.h>
        #include <cppconn/statement.h>
        #include <cppconn/prepared_statement.h>
        #include <cppconn/resultset.h>
        #include <memory>
        #include <omp.h> 
        using namespace std;
        int main(){
            sql::mysql::MySQL_Driver *driver;
            sql::Connection *con;
            driver = sql::mysql::get_mysql_driver_instance();
            cout << "Hello, World!　あア亜 (^w^)" << endl;
        return 0;
        }
        ```

        問題なく実行できれば完成！
        あとは，いい感じにコードを書くだけ！
