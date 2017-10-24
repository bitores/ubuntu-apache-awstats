# 如何在 Ubuntu 服务器中配置 AWStats



![](http://www.linuxidc.com/upload/2015_12/151203131947833.png)

AWStats 是一个开源的网站分析报告工具，可以生成强大的网站、流媒体、FTP 或邮件服务器的访问统计图。此日志分析器以 CGI 或命令行方式进行工作，并在网页中以图表的形式尽可能的显示你日志中所有的信息。它可以“部分”读取信息文件，以便能够频繁并快速处理大量的日志文件。它支持绝大多数 Web 服务器日志文件格式，包括 Apache，IIS 等。

本文将帮助你在 [Ubuntu](http://www.linuxidc.com/topicnews.aspx?tid=2) 上安装配置 AWStats。

 

#### 安装 AWStats 包

默认情况下，AWStats 的包可以在 Ubuntu 仓库中找到。

可以通过运行下面的命令来安装：

1. `sudo apt-get install awstats`

接下来，你需要启用 Apache 的 CGI 模块。

运行以下命令来启动 CGI：

1. `sudo a2enmod cgi`

现在，重新启动 Apache 以使改变生效。

1. `sudo /etc/init.d/apache2 restart`



#### 配置 AWStats

你需要为你想要查看统计的每个域或网站创建一个配置文件。在这个例子中，我们将为 “test.com” 创建一个配置文件。

要完成此步，你可以通过复制 AWStats 的默认配置文件来配置你要统计的域。

1. `sudo cp /etc/awstats/awstats.conf /etc/awstats/awstats.test.com.conf`

现在，你需要在配置文件中做一些修改：

1. `sudo vim /etc/awstats/awstats.test.com.conf`

像下面这样修改一下:

1. `#Change to Apache log file, by default it's /var/log/apache2/access.log`
2. `LogFile="/var/log/apache2/access.log"`
3. `# Change to the website domain name`
4. `SiteDomain="test.com"`
5. `HostAliases="www.test.com localhost 127.0.0.1"`
6. `# When this parameter is set to 1, AWStats adds a button on report page to allow to "update" statistics from a web browser`
7. `AllowToUpdateStatsFromBrowser=1`

保存并关闭文件。

修改配置文件后，你需要用服务器的当前日志建立初步统计。你可以这样做：

1. `sudo /usr/lib/cgi-bin/awstats.pl -config=test.com -update`

输出会是这个样子:

![awtstats](http://www.linuxidc.com/upload/2015_12/151203131947832.png)

*awtstats*

 

#### 为 Apache 配置 AWStats

接下来，你需要配置 Apache2 来显示统计数据。现在你需要将 “cgi-bin” 文件夹中的内容复制到 Apache 默认根目录下。默认它是在 “/usr/lib/cgi-bin”。

运行以下命令来完成此步:

1. `sudo cp -r /usr/lib/cgi-bin /var/www/html/`
2. `sudo chmod -R 755/var/www/html/cgi-bin/`



####  配置apache可以执行perlCGI脚本

现在，您可以通过访问 url “http://your-server-ip/cgi-bin/awstats.pl?config=test.com.” 来查看 AWStats 的页面。

它的页面像下面这样：

要么让修改/etc/apache2/sites-available/default-ssl.conf2，要么让修改mods-available里的文件，apache2.conf也都设置过了都不行，最后忽然看到/etc/apache2/下有个conf-available文件夹，进去之后有个serve-cgi-bin.conf文件，将里面的


    ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
    <directory "usr/lib/cgi-bin">
    AllowOverride None
    Options _ExecCGI -MultiViews _symLinkIfOwnerMatch
    Require all granted
    </Directory>


的前两句的路径改为自己的CGI程序目录，比如我的：

ScriptAlias /cgi-bin/ /var/www/cgi-bin/

<directory "/var/www/cgi-bin">

另外找到/etc/apache2/mods-available里的cgi.load文件，然后在/etc/apache2/mods-enabled,建立一个指向cgi.load的软链接，链接名为cgi.load。

最后在/etc/apache2/mods-available文件夹下找到mime.conf，加上一个话：

AddHandler cgi-script .cgi .pl .sh .py .php

把自己的CGI程序放在/var/www/cgi-bin目录下，服务器就可以识别了





#### 测试 AWStats

现在，您可以通过访问 url “http://your-server-ip/cgi-bin/awstats.pl?config=test.com.” 来查看 AWStats 的页面。

它的页面像下面这样：

![awstats_page](http://www.linuxidc.com/upload/2015_12/151203131947831.jpg)

*awstats_page*

 

#### 设置定时任务来更新日志

建议你创建一个定时任务，使用新创建的日志条目定期更新 AWStats 的数据库，然后统计会定期更新。这也将节省你的时间。

要做到这一点，你需要编辑 “/etc/crontab” 文件:

1. `sudo nano /etc/crontab`

添加下面那一行来让 AWStats 每十分钟更新一次。

1. `*/10 * * * * root /usr/lib/cgi-bin/awstats.pl -config=test.com -update`

保存并关闭文件。

 

#### 结论

AWStats 是一个非常有用的工具，可以让你对网站的状况了如指掌，并能协助你分析网站。它非常容易安装和配置。如果你有任何疑问，请在下面发表评论。

