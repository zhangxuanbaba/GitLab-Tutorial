
### 0：写在前面    
0：[官方参考](https://docs.gitlab.com.cn/runner/ "https://docs.gitlab.com.cn/runner/") 

1：GitLab的自动化部署时需要结合GitLab-Runner来使用的。        

2：自`GitLab 8.0`之后，GitLab已经默认集成了CI，所以我们只需要安装配置GitLab-Runner就可以完成相应的自动化部署。    

3：尽管GitLab-Runner可以与GitLab安装在同一服务器，但还是不推荐这样做，尽量安装在不同的服务器上。   

4：GitLab-Runner 可以在GNU/Linux, macOS, FreeBSD 和 Windows 上安装使用。 这里有三种安装方式 使用 Docker， 手动下载二进制文件，或使用 GitLab 提供的 rpm/deb 仓库进行[安装](https://docs.gitlab.com.cn/runner/install/ "https://docs.gitlab.com.cn/runner/install/")。    

5：GitLab Runner 实现了许多可用于在不同场景中运行您的 build (构建)的 executors (执行器)，这里使用的是最简单的shell。      

6：对于不同语言写成的代码，如Java,PHP，包括Vue等，其需要的解释环境，或者编译环境；运行环境其要求是不同的，这就要求我们在GitLab-Runner的安装服务器上安装对应的（解释）编译，运行环境。比如运行java的spring框架代码，一般就要求有jdk,maven,tomcat(或其他容器)环境，对于Vue的前端代码，则需要node.js环境来运行。

### 1：安装GitLab-Runner

> #### 1.1：[windows环境](https://docs.gitlab.com.cn/runner/install/windows.html "https://docs.gitlab.com.cn/runner/install/windows.html")安装GitLab-Runner        

>> ##### 1.1.1：在系统某个盘符或者文件目录下新建一个文件夹 GitLab-Runner       

>> ##### 1.1.2：下载符合你系统位数的安装包[x64](https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-386.exe "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-386.exe")或者[x86](https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-386.exe "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-386.exe")到你方才创建的文件夹中并将安装包重命名为`gitlab-runer.exe`。     

>> ##### 1.1.3：以管理员身份运行方才下载的.exe文件即可<br><br>
> #### 1.2：[GNU/Linux环境](https://docs.gitlab.com.cn/runner/install/linux-repository.html "https://docs.gitlab.com.cn/runner/install/linux-repository.html")安装GitLab-Runner(Centos)      

>> ##### 1.2.1：添加GitLab-Runner官方仓库     

    curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash   
    
>> ##### 1.2.2：安装最新版本的GitLab-Runner

    sudo yum install gitlab-runner
    
或者是安装指定版本的GitLab-Runner

    yum list gitlab-runner --showduplicates | sort -r
    sudo yum install gitlab-runner-$version     
    
安装成功后，默认会新建gitlab-runner的用户和组，即/home/gitlab-runner<br><br>

### 2：注册GitLab-Runner与各命令选项含义[官方教程](https://docs.gitlab.com.cn/runner/register/index.html "https://docs.gitlab.com.cn/runner/register/index.html")

> #### 2.1：注册GitLab-Runner    

    sudo gitlab-runner register

> #### 2.2：各命令选项含义    

>> ##### 2.2.1:输入GitLab实例URL

    Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
   
 此处如果是注册GitLab的共享runner，则该URL可GitLab的管理区域找到     
 ![共享runner的url](https://pan.baidu.com/s/1c3IIs1M)


    
