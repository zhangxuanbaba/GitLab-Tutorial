
### 1：手动备份到本地
  当gitLab安装成功后，默认的备份目录位于 /var/opt/gitlab/backups 下,可以在 /etc/gitlab/gitlab.rb中修改其路径（搜索backup）
    
    gitlab_rails['backup_path'] = '/var/opt/gitlab/backups'
  
  修改完成后保存，并使用如下命令完成重载配置文件
    
    sudo gitlab-ctl reconfigure
    
  在上述操作完成后，可以使用如下命令完成手动备份（如果没有更改默认备份目录的需求，直接进行此步操作即可）
  
    gitlab-rake gitlab:backup:create
    
### 2：自动定时备份到本地
  自动备份到本地其实很简单，只需要两部操作，第一步写一个脚本执行备份操作，第二部配置crontab的定时任务即可
  
#### 2.1：编写[备份脚本文件](https://pan.baidu.com/s/1pMm7O2B)（我将脚本目录放到了/root下）

    cd /root
    vim gitlab_auto_backup_to_local.sh

    #!/bin/bash
    # gitLab备份命令
    gitlab-rake gitlab:backup:create
    
#### 2.2：赋予脚本可执行权限，并手动执行该脚本测试是否可以备份

    sudo chmod 770 /root/gitlab_auto_backup_to_local.sh
    
#### 2.3：编辑/etc/crontab 文件
  在crontab文件里面，每一行代表一个任务，每行的每个字段代表一项设置，共有六个字段，前五段时时间段，第六段是要执行的命令段，每个字段之间以空格隔开，不使用的段用 * 代替 
  
    m h dom mon dow user command
    
  其中      
  - m：表示分钟，0-59   
  - h：表示小时，0-23   
  - dom：表示日期，1-31   
  - mon：表示月份，1-12     
  - dow：表示周几，0-6（周日可以是 0 或 7）     
  - user：表示执行的用户      
  - command：要执行的命令，可以时系统命令，也可以是脚本文件    
  
  例如每天凌晨两点进行自动打包：     
  
    vim /etc/crontab
    
    0 2 * * * root /root/gitlab_auto_backup_to_local.sh
    
  在这里，你可以先用date命令查看当前系统时间，然后将定时任务时间设置为稍后的几分钟，测试定时任务与自动备份是否可以正常正确执行      
  
#### 2.4：重启定时任务

    service crond restart
  
### 3：自动定时备份到远程服务器

#### 3.1：服务器秘钥配对，取消scp传输文件密码限制（这里使用的用户都是root）

  此时有两台服务器A，B。其中A为gitlab服务器，B为远程备份服务器
  
##### 3.1.1：A,B服务器分别生成公钥私钥

    cd /root
    ssh-keygen -t rsa(没有特殊要求一路回车就可以)
    
  执行上述命令后，会在/root/.ssh 目录下生成 公钥 id_rsa.pub 私钥 id_rsa (如果是其他用户，则在其用户家目录下的 .ssh文件夹)
  
##### 3.1.2：A(master),B(backup)服务器分别保存对方公钥
  将A服务器的公钥发送到B服务器     
  
    cd /root/.ssh
    cp id_rsa.pub id_rsa.pub.master
    scp id_rsa.pub.master root@B.IP:/root/.ssh
  
  在scp命令执行后，会有警告询问是否接受B服务器地址到 known_hosts，同意即可，之后会要求输入B服务器的root登录密码，成功输入密码后，文件会成功拷贝的B服务器的/root/.ssh 路径下     
  
  进入B服务器将A服务器的公钥保存到authorized_keys文件中
  
    cd /root/.ssh
    cat id_rsa.pub.master >> authorized_keys
    chmod 644 authorized_keys
    
  注意：authorized_keys的文件权限不可以设置为777,否则scp还是会要求输入密码     
  
  然后将B服务器的公钥发送到A服务器，并在A服务器下将B服务器的公钥保存到authorized_keys中（B公钥名称可以cp为id_rsa.pub.backup）     
  
##### 3.1.3：编写将gitlab[备份同步到远程服务器的脚本](https://pan.baidu.com/s/1i6fY1Qd)      

  下面脚本查找 本地备份目录下 时间为60分钟之内的，并且后缀为.tar的gitlab备份文件      
  此脚本文件逻辑仍不完善！！！！！

    cd /root
    vim gitlab_auto_backup_to_remote.sh
    
    #!/bin/bash
    #local default gitlab-server backup dir,you can change it with vim /etc/gitlab/gitlab.rb ,in this file ,to find backup
    LOCAL_BACK_DIR=/var/opt/gitlab/backups

    #remote backup-server backup dir
    REMOTE_BACK_DIR=/home/gitlab-runner/backup

    #remote backup-server login account
    REMOTE_USER=root

    #remote backup-server ip
    REMOTE_IP=47.100.46.209
    
    #local system date
    DATE=`date +%y-%m-%d_%T`

    #log save filename
    LOG_FILE=$LOCAL_BACK_DIR/log/$DATE.log

    #find local backup dir,time less than 60min,end with .tar backup file
    BACKUP_FILE_TO_REMOTE=`find $LOCAL_BACK_DIR -type f -mmin -60 -name *.tar`

    #create new log
    touch $LOG_FILE

    #append log to logfile
    echo  "Gitlab auto backup to remote server, start at $DATE" >> $LOG_FILE

    #copy to remote server
    scp $BACKUP_FILE_TO_REMOTE $REMOTE_USER@$REMOTE_IP:$REMOTE_BACK_DIR
    
##### 3.1.5:赋予脚本可执行权限，并手动执行该脚本测试是否可以自动备份到远程服务器

    chmod 777 /root/gitlab_auto_backup_to_remote.sh
    
  先修改定时任务中的时间，在本地备份自动执行完毕后，手动执行此脚本测试是佛成功拷贝到远程服务器
  
  
##### 3.1.6：编辑/etc/crontab文件，添加自动拷贝备份到远程服务器任务

    vim /etc/crontab
    
    #a crontabs about gitlab backup to lcoal
    0 2 * * * root /root/gitlab_auto_backup_to_local.sh -D 1 

    #a crontab aboout gitlab backup to remote
    55 2 * * * root /root/gitlab_auto_backup_to_remote.sh -D 1
    
  
##### 3.1.6 重启定时任务

    service crond restart
 
