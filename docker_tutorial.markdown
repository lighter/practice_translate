# 網頁開發者如何使用Docker

這是篇我覺得蠻清楚，而且又解說的蠻清楚的文章，因此嘗試翻成中文當作我的筆記！如果有哪裡不好歡迎指教，謝謝。有些像是`Container`, `Volume`, `Image`我就不特別翻成`容器`, `數據`, `映像檔`了，因為我始終覺得怪怪的。 

***

原文的出處：[http://blog.osteel.me/posts/2015/12/18/from-vagrant-to-docker-how-to-use-docker-for-local-web-development.html](http://blog.osteel.me/posts/2015/12/18/from-vagrant-to-docker-how-to-use-docker-for-local-web-development.html)

你有時常注意科技新知，Docker這一個名詞對你來說應該不陌生。

如果沒有，你可以參考一下[維基百科](http://www.wikiwand.com/zh-tw/Docker_(%E8%BB%9F%E9%AB%94)的簡述

> Docker是一個開放原始碼軟體專案，讓應用程式布署在軟體容器下的工作可以自動化進行，藉此在Linux作業系統上，提供一個額外的軟體抽象層，以及作業系統層虛擬化的自動管理機制。

酷！但我還是不太明白！請繼續說下去

> Docker利用Linux核心（kernel）中的資源分離機制，例如cgroups，以及Linux核心命名空間（name space），來建立獨立的軟體容器（containers）。這可以在單一Linux實體下運作，避免啟動一個虛擬機器造成的額外負擔。

OK，有點概念了。

目前在我的Mac，我使用的Vagrant來建立我每一個專案(如果你不知道什麼是Vagrant，可以先看一下[這篇文章](http://blog.osteel.me/posts/2015/01/25/how-to-use-vagrant-for-local-web-development.html)，尤其是[這一部份](http://blog.osteel.me/posts/2015/01/25/how-to-use-vagrant-for-local-web-development.html#vagrant))。

不論何時，任何專案，我只要使用`vagrant up`指令，短時間內，我就可以得到一個獨立的虛擬機器(virtual machine, VM)，非常的便利。

但仍然要一點時間等待虛擬機器設定完成和啟動，而且每個專案的執行時間都不長，相對的也佔了不少硬碟資源。

你可以採取Laravel的初步作法[Homestead](https://laravel.com/docs/master/homestead)，將每個專案都運行在同一台虛擬機器上，因此開發環境就無法個別獨立開來。

那麼跟Docker有什麼關係呢？

Docker其實就是將環境個別獨立，並且是運行在同一台虛擬機器(使用[Boot2Docker](https://github.com/boot2docker/boot2docker))上。

那麼...我可以將我的Vagrant通通轉移到Docker，並且只使用一台虛擬機器囉?

當然囉！

下面我們會一步一步來操作。

## 大綱

* 安裝
* 設定
* Nginx
* PHP
* Data
* Volumes 和 data containers
* MySQL
* phpMyAdmin
* 掌管多個專案
* 故障排除
* 結論
* 參考

## 安裝

如同之前所提的，我使用的是Mac，所以下面的教程環境使已Mac為主。雖然我們是在Mac的環境，但是安裝完成後與Windows不會有太大的差異。如果你是使用Mac或Windows，可以參考[這裡](https://www.docker.com/docker-toolbox)，依據你的作業系統下載Docker Toolbox(如果你是Linux可以直接參考[這裡](https://docs.docker.com/linux/)，無須下載Docker Toolbox)。

Docker Toolbox會幫你安裝好所有需要的資源。

筆記：你可能會發現到一個錯誤的訊息：

```
Network timed out while trying to connect to https://index.docker.io/
v1/repositories/library/hello-world/images. You may want to check your 
internet connection or if you are behind a proxy.
```

針對這個錯誤訊息，你只需要再執行一次`docker-machine restart default`指令。

完成之後，你應該有一個虛擬機器在[VituralBox](https://www.virtualbox.org/)執行(虛擬機器的名稱為default)，Docker的所有操作都可以在這個名為default的虛擬機操作。

到目前還不是很了解。如何知道開啟的網頁要對應到哪台機器，資料(database)在哪?

## 設定

接下來的所有設定我都會放到[GitHub](https://github.com/osteel/docker-tutorial)上，你可以隨時參考(甚至你可以直接使用)。

此外，或許對於虛擬機器(VM)和Docker，你會感到有點困惑。其實差異不大。

平常我都是使用[LEMP stack](https://lemp.io/)，而且我使用的是PHP7，所以我們需要有Linux/PHP7/Nginx/MySQL(我們將會在另外一篇文章看到如何建立一個混合的框架)。

我們可以使用[Docker Compose](https://docs.docker.com/compose/)來達到Linux/PHP7/Nginx/MySQL各自獨立在Container運行。

很多教程在一開始會教你如何使用Dokcer的指令設定一個Container，或者如何串連多個Container，但實際運作上通常有一個或兩個Container以上，如果使用Docker指令一台一台的去串連，這是件很痛苦的事。

透過YAML設定檔，可以讓你自己設定各自Container如何連接。我們將會將各個服務放到各自的Contrainer：

* Nginx 一個Container
* PHP-FPM 一個Container
* MySQL 一個Container
* phpMyAdmin 一個Container
* 一個Container 使MySQL資料庫的資料
* 一個Container 放我們的程式碼

在其他教程，通常都會使用他們自己的images，如果沒有特別說明原因，對於剛接觸Docker的人或許有些困惑或疑問。

下面我將會使用官方的images和Dockerfiles來實作。

## Nginx

一開始，為了確保可以正確的運行，先從最簡單的設定做起。

建立一個資料夾做為你的專案目錄(這裡我命名為docker-tutorial)，然後在docker-tutorial內建立一個`docker-compose.yml`檔案。加入下面的設定到`docker-compose.yml`：

```
nginx: 
	image: nginx:latest
	ports: 
		- 80:80
```

接著執行下面的指令(記得切換到你的專案目錄下)

```
$ docker-compose up -d
```

等待Nginx image下載完成後就會自動執行，完成後執行下面的指令

```
$ docker-machine ip default
```

你會得到一個IP，在瀏覽器開啟這個IP，你就可以看到下面的畫面

![](images/pic_1.png)

到這裡，我們做了什麼?

我們告訴Docker Compose我們想要一個名為`nginx`的Container，並且使用官方Nginx最新的image，port指定為80。

接著要求Docker Compose依據`docker-compose.yml`的設定建置和啟動Container。`-d`是將Container丟到背景執行。

最後取得IP，Docker是運行在`default`這台虛擬機上(如果你想確認你的Docker是運行在哪台機器上，你可以使用`docker-machine ls`來取得)。

因為使用port 80，我們可以直接使用瀏覽器開啟IP位置來開啟網頁。

最後一件事情，執行`docker ps`指令，你可以看到如下圖：

![](images/pic_2.png)

上圖你可以看到一個Container使用了哪一個image正在運行。而這個Container名為`dockertutorial_nginx_1`。Docker compose是使用了資料夾加上image名稱和一個數字來命名。

## PHP

在安裝PHP之前，先建立一個`index.php`檔案(與`docker-compose.yml`同一層資料夾)。

將下面的設定取代原本的`docker-compose.yml`

```
nginx:
	build: ./nginx/
	ports:
		- 80:80
	links:
		- php
	volumes:
		- .:/var/www/html

php:
	image: php:7.0-fpm
	expose:
		- 9000
	volumes:
		- .:/var/www/html
```

我們增加一個名為`php`的Container，並指定使用`7.0-fpm`的版本。並指定expose為9000。

`expose`和`ports`有什麼不同?`expose`是用來指定Container之間連接的接口，`ports`則是外部連接機器的接口。

`volumes`，將目前的目錄`.`掛載到`/var/www/html`下，也就是我們專案的目錄會同步到Container的`/var/www/html`。

`nginx` Container設定`volumes`如同`php`(Nginx的Container必須知道哪個服務目錄的路徑)。`links`，是告訴`nginx` Container要連結到`php`(如果還是不了解，下面很快會告訴你)。

我們將`image`取代成`build`，並指向`nginx/`目錄，這裡是告訴Docker Compose不要使用已存在的image，而是使用`nginx/`下的
`Dockerfile`去建立一個新的image。

如果你有看過新手指南，你應該對於Dockerfile有點了解。你可以透過Dockerfile來告訴Docker你要安裝的image和你要執行的指令等等。

建立一個名為`Dockerfile`的檔案在`nginx/`目錄下，下面為Dockerfile的內容

```
FROM nginx:latest
COPY ./default.conf /etc/nginx/conf.d/default.conf
```

第一行，指定使用的image為nginx。第二行將`default.conf`複製取代到`/etc/nginx/conf.d/default.conf`。

建立`default.conf`檔案在`nginx/`資料夾下，下面是`default.conf`的內容：

```
server {
    listen 80 default_server;
    root /var/www/html;
    index index.html index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/error.log error;

    sendfile off;

    client_max_body_size 100m;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

這是Nginx的基本設定。

可以特別看一下這行設定：

```
fastcgi_pass php:9000;
```

要求Nginx透過9000 port連接到`php` Container，而要求`nginx`連接的設定`links`已經在`docker-compose.yml`檔案中。

接著還需要建立`index.php`檔案，檔案內容如下：

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Hello World!</title>
    </head>
    <body>
        <img src="http://blog.osteel.me/images/2015/12/18/docker-tutorial3.gif" alt="Hello World!" />
    </body>
</html>
```

這裡只有簡單的HTML代碼，用來確認PHP檔案可以執行。

下圖是目前的目錄結構：

![](images/pic_3.png)

再次執行`docker-compose up -d`。Docker Compose 會依據設定的，再次建立和啟動Container(這次會下載PHP image)：

![](images/pic_4.png)

如果你已經忘記IP，可以執行`docker-machine ip default`取得IP

現在執行`docker ps`，可以看到目前有哪些Container正在運行：

![](images/pic_5.png)

現在可以試著重啟網頁，可以看到`index.php`的title，"Hello Universel!"。

接著加入database了！

## Data

### Volumes 和 data containers

在設定MySQL之前，先來看看`volumes`。`nginx`和`php` container都有一樣的設定，這樣的設定方式很常見。

另外一種方式，將資料的來源獨立出來給其他container使用。

改變`docker-compose.yml`設定，設定如下：

```
nginx:
    build: ./nginx/
    ports:
        - 80:80
    links:
        - php
    volumes_from:
        - app

php:
    image: php:7.0-fpm
    expose:
        - 9000
    volumes_from:
        - app

app:
    image: php:7.0-fpm
    volumes:
        - .:/var/www/html
    command: "true"
```

首先新增一個名為`app`的Container，`volumes`的設定與原本的`nginx`和`php`一樣。將應用程式的code給獨立成一個Container。當Docker Compose 建立起這個`app` Container，會立即停止，因為這個Container沒有做任何動作，為了不讓它停止特別指定了`command: "true"`。這可以讓我們只存取Container，也不會無謂的浪費資源。

除此之外，你也可以發現我們使用了與`php`一樣的image，同樣的image不會重複下載，而是使用了相同的image，避免不必要的浪費。

在`nginx`和`php`的設定中，原本的`volumes`改為`volumes_from`，而且指定為`app` Container。其下之意就是告訴Docker Compose要掛載的`volumes`是來自`app` Container。

再次執行`docker-compose up -d`。執行完成時執行`docker ps`可以看到目前有哪些container正在運行，如下圖：

![](images/pic_6.png)

如果對於data Container和volumes有興趣，我推薦你這邊[文章(作者Adrian Mouat)](http://container-solutions.com/understanding-volumes-docker/)(在文章最後的引用也可以找到這篇文章)。

## MySQL

緊接著來看到MySQL。

開啟`docker-compose.yml`，加上下面的設定：

```
mysql:
	image: mysql:latest
	volumes_from:	
		- data
	environment:
		MYSQL_ROOT_PASSWORD: secret
		MYSQL_DATABASE: project
		MYSQL_USER: project
		MYSQL_PASSWORD: project
		
data:
	image: mysql:latest
	volumes:
		- /var/lib/mysql
	command: "true"
```

更新`php`的`links`，改成`mysql`，然後使用Dockerfile建置image：

```
php:
	build: ./php/
	expose:
		- 9000
	links:
		- mysql
	volumes_from:	
		- app
```

你已經知道`links`了，所以接著看到新的Dockerfile：

```
FROM php:7.0-fpm
RUN docker-php-ext-install pdo_mysql
```

這只是基本的安裝pdo_mysql套件，透過這個套件我們可以連接database(如何安裝PHP套件，你可以參考這篇文章 [image's doc](https://hub.docker.com/_/php/))。將這個檔案放置到新的`php/`目錄下。

往下看到MySQL的設定，你可以先看看這篇文章[official MySQL image](https://hub.docker.com/_/mysql/)。`environmet`是我們目前沒看過的設定，它允許我們定義環境變數供Container使用。這邊我們特別定義了MySQL的root密碼和名稱(`project`)，以及一位使用者的密碼和database名稱(所有可用的變數都列在[image's documentation](https://hub.docker.com/_/mysql/))。

如同前面所提到的，`data` Container的目的要管理MySQL Container存放在`/var/lib/mysql`的資料(並且重複使用MySQL image以節省硬碟空間)。或許你會發現，我們並沒有宣告本機要掛載到`/var/lib/mysql`(也就是冒號前本機路徑)，目錄位於何處，我們交由Docker Compose來控制，只需知道資料內容是存在的。

如果`MYSQL_DATABASE`已經存在了，`MYSQL_ROOT_PASSWORD`的設定會被忽略，保持既有的設定。

為了測試是否正能夠連到database，`index.php`需要做些調整：

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Hello World!</title>
    </head>
    <body>
        <img src="http://blog.osteel.me/images/2015/12/18/docker-tutorial3.gif" alt="Hello World!" />
        <?php
        $database   = $user = $password = "project";
        $host       = "mysql";
        $connection = new PDO("mysql:host={$host};dbname={$database};charset=utf8", $user, $password);
        $query      = $connection->query("SELECT TABLE_NAME FROM information_schema.TABLES WHERE TABLE_TYPE='BASE TABLE'");
        $tables     = $query->fetchAll(PDO::FETCH_COLUMN);

        if (empty($tables)) {
            echo "<p>There are no tables in database \"{$database}\".</p>";
        } else {
            echo "<p>Database \"{$database}\" has the following tables:</p>";
            echo "<ul>";
            foreach ($tables as $table) {
                echo "<li>{$table}</li>";
            }
            echo "</ul>";
        }
        ?>
    </body>
</html>
```

這段code會連接到database，並列出目前所有的table。

完成後再次執行`docker-compse up -d`(可能需要等待一段時間，因為需要下載MySQL image)，完成後執行`docker ps -a`。你應該可以看到5個Container，其中兩個是退出(Exited)的：

![](images/pic_7.png)

現在開啟網頁是顯示*"There are no tables in database 'project'"*。

下一步是建立phpMyAdmin，他是一個圖形化的介面供你管理MySQL。現在我先在MySQL的Container來操作MySQL command line。

從上圖的結果，找到MySQL Container的ID(在我的環境是`5207587d116b`)並且複製他；執行下面的指令：

```
$ docker exec -it 5207587d116b /bin/bash
```

接著你可以在這個Container執行指令(你也可以用Container的name來取代ID執行)。

`docker exec`允許你的指令在Container中執行，`-t`讓Docker分配一個假的終端機(pseudo-tty)並綁定到Contanier的標準輸入(參考[Docker —— 从入门到实践](https://www.gitbook.com/book/yeasy/docker_practice/details))，`-i`允許與終端機互動。`/bin/bash`是建立一個在Container的`bash`介面。

你也可以在其他Container使用這段指令。

然後使用`mysql -uroot -psecret`進入MySQL CLI。執行`show database;`列出所有的database，如下圖：

![](images/pic_8.png)

切換database到`project`，並且建立一個table：

```
$ mysql> use project
$ mysql> CREATE TABLE users (id int);
```

重新整理網頁，就可以看到`users`會顯示在網頁上。

你可以使用`\q`離開MySQL CLI，`ctrl + d`離開container終端機。

MySQL的data存在本機的位置在哪呢?

看到`docker-compose.yml`的`data` container設定：

```
data:
	image: mysql:latest
	volumes:
		- /var/lib/mysql
	command: "true"
```

Docker Compose會將本機掛載到`/var/lib/mysql`的目錄下，但本機位置是?

再次執行`docker ps -a`，複製MySQL Container狀態為退出(Exited)的ID(在我的環境是`7970b851b07a`)，然後執行：

```
$ docker inspect 7970b851b07a
```

接著你可以找到JSON格式的資料。找到`Mounts`的部分：

```
...,
"Mounts": [
    {
        "Name": "0cd0f26f7a41e40437019d9e5514b237e492dc72a6459da88d36621a9af2599f",
        "Source": "/mnt/sda1/var/lib/docker/volumes/0cd0f26f7a41e40437019d9e5514b237e492dc72a6459da88d36621a9af2599f/_data",
        "Destination": "/var/lib/mysql",
        "Driver": "local",
        "Mode": "",
        "RW": true
    }
],
...
```

`"Source"`是資料存在於Container的路徑。

我懂了。

但如果我們移除這個Container，會發生什麼事情呢?

問得很好！

事實他們仍然存在於硬碟裡。

有2個解決方法，1. 我們要確保移除Container時volume一同被移除，我們可以使用`-v`的參數：

```
$ docker rm -v containerid
```

如果不使用`-v`，可以改用下面的指令：

```
$ docker volume rm $(docker volume ls -qf dangling=true)
```
使用`docker volume ls`指令加上`q`及`f`參數刪除打上`none`標籤的volume(`dangling=true`)。

在[Docker command line documentation](https://docs.docker.com/engine/reference/commandline/cli/)可以得到更多資訊。

## phpMyAdmin

為了方便管理MySQL，接下來就要在Docker上安裝phpMyAdmin！

開起`docker-compose.yml`，加上下列的設定：

```
phpmyadmin:
	image: phpmyadmin/phpmyadmin
	ports:
		- 8080:8080
	links:
		- mysql
	environment:
		PMA_HOST: mysql
```

如同之前的操作，指定好官方的[phpMyAdmin image](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)。我們發佈的port 8080對應到虛擬機的8080，以及連結`mysql` Container。最後設定`PMA_HOST`環境變數。

儲存並再次執行`docker-compose up -d`。等待phpMyAdmin image下載，當一切就緒後，開啟網頁，這次在網址後加上`:8080`，就可以看到phpMyAdmin登入畫面：

![](images/pic_9.png)

帳號密碼輸入分別輸入`root`和`secret`，即可成功登入(如同`mysql` Container的設定，`project` / `project`一樣也可以，而且可以存取`project`的database)。

很容易，對吧?

休息一下！回想消化一下！不要忘了繼續看下去！因為它可以幫助你更清楚。

設定檔放在[GitHub repository](https://github.com/osteel/docker-tutorial)，歡迎自行取用。

## 管理多個專案

Docker有一個很大優勢，當你有多個Container使用相同的image時，使用的硬碟空間只會增加一點點，通常只有幾MB。

除非你想要用PostgreSQL取代成MySQL，不然只會繼續使用本機已存在的image。

如何在同一台虛擬機管理不同的專案呢?

但Docker只有一個私有IP，你可以使用port來區分專案，無需每個專案都執行在port 80上。

在上述的過程中你可以發現我們一直輪流使用`docker`和`docker-compose`指令，這或許讓你有點混淆。`docker-compose`就如同執行`docker`的指令；`docker`可針對個別Container做操作，`docker-compose`可以同時操作`docker-compose.yml`設定內的Container。

例如

```
docker ps -a
```

這指令會顯示目前在Docker上所有的Container包含沒有在運行的Container。

而

```
docker-compose ps
```

他與上面的目的是一樣的，但是僅會列出`docker-compose.yml`設定內的Container(`-a`參數是非必要的，而且顯示順序與設定檔不同)。

當你有很多Container需要管理時，他就可以派上用場了，例如：

```
docker-compose stop
```

這將會停止目前所有在`docker-compose.yml`內的Container。所有的Container停止後，同時也會釋出port 80。

要再次啟動可以使用先前提到的`docker-compose up -d`指令。

要刪除目前已經停止運作的Container，你可以執行：

```
docker-compose rm
```

這個指令相對於`docker rm`，如果你想要同時刪除對應的volume，你可以加入`-v`參數(你也可以刪除打上none標籤的volume，如同前面所說的)。

在`docker`和`docker-compose`指令下有更多細節的說明。

## 故障排除

到目都可以正常運作了！Docker compose有可能會不建置一個image或者啟動Container，而且通常顯示的訊息不是有很大的幫助。

當有錯誤發生時，試著執行：

```
docker-compose logs
```

他會顯示目前`docker-compose.yml`設定檔的logs。你也可以執行`docker-compose ps`檢查目前所有Container的`State`欄位狀態，如果有一個exit後的代碼數字不是`0`，代表那個Container有些問題。

這時你可以針對這個Container顯示logs：

```
docker logs containerid
```

Container的name也一樣可行。

## 結論

Docker是一個很了不起的技術(雖然剛起步，但相信我，它是非常有前途的)。

這裡我們使用Docker Toolbox和Boot2Docker，一切看似容易，因為它已經自動幫我做好了很多事情。而我們也可以安裝Docker在Vagrant box上。

但這麼做意義不大。Docker成長速度之快，資源也很豐富。

目前我仍是個初學者，如果你發現發現到任何錯誤，非常歡迎讓我知道，並且修正它。

![](http://blog.osteel.me/images/2015/12/18/docker-tutorial14.gif)


## 參考資源

* [Discovering Docker (e-book)](https://discoveringdocker.com/)
* [How I develop in PHP with CoreOS and Docker](https://www.jverdeyen.be/docker/how-php-symfony-coreos-docker/)
* [Docker for PHP Developers](http://www.newmediacampaigns.com/blog/docker-for-php-developers)
* [Understanding Volumes in Docker](http://container-solutions.com/understanding-volumes-docker/)
* [Manage data in containers](https://docs.docker.com/engine/userguide/dockervolumes/)
* [Tips for Deploying NGINX (Official Image) with Docker](https://blog.docker.com/2015/04/tips-for-deploying-nginx-official-image-with-docker/)
* [Use the Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)
* [Compose CLI reference](https://docs.docker.com/compose/reference/)
* [Dockerfile reference](https://docs.docker.com/engine/reference/builder/)

