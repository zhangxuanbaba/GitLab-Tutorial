
### 1：写在前面
  在gitLab的安装过程中，踩了一个很大的坑，就是官网上明确说明服务器`至少要给gitLab分配4G内存`，最开始使用1G服务器安装成功，但浏览器访问一直是502，所以，安装gitlab切记服务器至少4G内存。<br>
	本教程服务器系统为Centos7，安装的是gitlab9.5.5版本
	
### 2：GitLab是什么
  个人的理解：GitLab是建立在git这套分布式的版本控制系统基础之上的，拼装了其他一些软件，比如Nginx，redis等等，从而达到了类似于GitHub的可以视图网页化操作的一个中央托管仓库			
	
  git相对于其他版本控制系统（如SVN）的优点：      
  * 版本库本地化    
  * 分支建立管理更简便     
### 3：GitLab的安装
 * 3.1:安装并配置必要的依赖关系     
    
		sudo yum install curl policycoreutils openssh-server openssh-clients
		sudo systemctl enable sshd
		sudo systemctl start sshd
		sudo yum install postfix
		sudo systemctl enable postfix
		sudo systemctl start postfix
		sudo firewall-cmd --permanent --add-service=http
		sudo systemctl reload firewalld
		
* 3.2：使用清华大学的[镜像源安装](https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/ "https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/")

		wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-9.5.5-ce.0.el7.x86_64.rpm
		rpm -i gitlab-ce-9.5.5-ce.0.el7.x86_64.rpm			
		
  安装成后根据版本的不同提示会稍有不同，大抵是9.0之后的版本安装成功后会出现一个标点符号组成的GITLAB，而在9.0版本之前不会有这个图标。其他的提示都是类似的：			
	
	
	thank you for installing gitlab		
	
	gitlab was unable to detect a valid hostname for your instance					
	
	please configure a URL for you gitlab instance by setting `external_url` configurtion in /etc/gitlab/gitlab.rb
	
	
	
	
### 4：GitLab配置并启动			

正如安装成功后的提示，我们打开`/etc/gitlab/gitlab.rb`并编辑`external_url`为当前的服务器Ip
	
		vim /etc/gitlab/gitlab.rb
		
		external_url "http://xx.xxx.xxx.xxx"

保存退出，执行 `gitlab-ctl reconfigure`，稍后，gitLab服务器就会重新配置成功并启动，此时可以在浏览器输入刚刚设置的地址进行访问，对于低版本的
gitlab会让你输入root的密码，登录后重新设置密码，这个密码是`root/5iveL!fe`。而对于高版本的gitlab会直接让你设置root的密码，设置成功后即可使用
设置的密码登录


### 5：GitLab的汉化
#### 5.1:8-8版本之前的汉化[参考网址](http://www.21yunwei.com/archives/4351 "http://www.21yunwei.com/archives/4351")

##### 5.1.1:：确认gitlab的版本
	
		cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
		
##### 5.1.2：下载[汉化包]( https://git.oschina.net/qiai365/gitlab-L-zh.git " https://git.oschina.net/qiai365/gitlab-L-zh.git")，此操作可能会因为网络原因失败，请多次尝试（记得先安装git `yum install git`）

		git clone  https://git.oschina.net/qiai365/gitlab-L-zh.git
		
##### 5.1.3：切换到与当前gitlab版本相同的分支（进入到文件夹就不要出来了哦，所有操作一直在此文件夹完成）

		cd gitlab-L-zh
		git branch
		git checkout -b 8-5-zh origin/8-5-zh
		
##### 5.1.4：备份gitlab自身的配置文件，以防汉化失败

		cp -r /opt/gitlab/embedded/service/gitlab-rails{,.ori}
		
##### 5.1.5：停止gitlab,将汉化包中的相应文件覆盖到gitlab对应的文件夹位置

		gitlab-ctl stop
		yes| cp -rf ../gitlab-L-zh/* /opt/gitlab/embedded/service/gitlab-rails/
		
##### 5.1.6:重新配置，启动，浏览器访问查看是否汉化成功

		gitlab-ctl reconfigure
		gitlab-ctl start
		
#### 5.2：8-8版本之后的汉化

##### 5.2.1：确认gitlab的版本

		cat /opt/gitlab/embedded/service/gitlab-rails/VERSION
##### 5.2.2：下载[汉化包]( https://gitlab.com/xhang/gitlab.git " https://gitlab.com/xhang/gitlab.git")，此操作可能会因为网络原因失败，请多次尝试（记得先装git哦：yum install git）

		git clone https://gitlab.com/xhang/gitlab.git
	
##### 5.2.3：切换到与gitLab同版本的分支（进入到文件夹就不要出来了哦，所有操作一直在此文件夹完成）

		cd gitlab
		git branch
		git checkout -b 9-5-stable-zh
		
##### 5.2.4：查看差异，得到补丁

		git diff v9.5.5 v9.5.5-zh > ../9.5.5-zh.diff
		
##### 5.2.5：停止gitlab，打补丁（同样不要忘记备份啊，不然就要重新装gitlab了）

		gitlab-ctl stop
		yum install patch (如果没有补丁工具，需要安装)
		patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < ../9.5.5-zh.diff
		
##### 5.2.6：重新配置，启动，浏览器访问查看是否汉化成功
	
		gitlab-ctl reconfigure
		gitlab-ctl start
		

### 6：GitLab的邮件配置
### 7：GitLab找回root密码



    
    
    
    
[官网社区安装参考](https://about.gitlab.com/installation/ "https://about.gitlab.com/installation/") 

[中文官网社区安装参考](https://www.gitlab.com.cn/installation/ "https://www.gitlab.com.cn/installation/")       

[本文档参考地址](http://www.21yunwei.com/archives/4351 "http://www.21yunwei.com/archives/4351")    
  
