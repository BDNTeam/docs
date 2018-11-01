# centos系统

## 系统准备
    #准备4台服务器
    #修改hostanme
    hostnamectl set-hostname t[1-4]
    
    # 修改/etc/hosts 增加以下内容
    192.168.1.201 t1
    192.168.1.202 t2
    192.168.1.203 t3
    192.168.1.204 t4

## 安装软件
### 安装python3
    #安装EPEL依赖
    yum install epel-release -y

    #安装IUS软件源
    yum install https://centos7.iuscommunity.org/ius-release.rpm
    
    # 安装python3.6
    yum install python3.6
    
    # 为了方便使用，创建python3.6 软连接
    ln -s /bin/python3.6 /bin/python3
    
    # 安装pip
    yum install python36u-pip
    
    # 创建pip3 软连接
    ln -s /bin/pip3.6 /bin/pip3

### 安装依赖
    yum install -y openssl-devel git wget unzip  openssl gcc gcc-c++ python36u-devel 
    
    
### 安装 mongodb
    yum install mongodb -y
    
    
### 安装 tendermint
    cd /tmp
    wget https://github.com/tendermint/tendermint/releases/download/v0.22.8/tendermint_0.22.8_linux_amd64.zip
    unzip tendermint_0.22.8_linux_amd64.zip
    rm -rf tendermint_0.22.8_linux_amd64.zip
    mv tendermint /usr/local/bin/
    
### 安装bigchaindb
    cd /opt/
    git clone https://github.com/BDNTeam/ChainDB
    cd ChainDB/core/
    # 安装
    pip3 install --no-cache-dir --process-dependency-links .
    
## 配置
### 配置bigchaindb
    #按照提示输入，其中ip地址都改为0.0.0.0
    bigchaindb configure
    

### 配置 tendermint
    # 初始化命令
    tendermint init
    
#### 配置.tendermint/config/genesis.json ; 用于接受其他成员的消息
    #genesis_time、chain_id 配置必须一致
    
    #查询 pub_Key
    cat .tendermint/config/priv_validator.json
    # 修改 validators
    
    "validators":[
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member 1 public key 可直接>"
         },
         "power":10,
         "name":"<Member 1 name 填写hostname>"
      },
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member 2 public key>"
         },
         "power":10,
         "name":"<Member 2 name>"
      },
      {
         "...":{

         },

      },
      {
         "pub_key":{
            "type":"AC26791624DE60",
            "value":"<Member N public key>"
         },
         "power":10,
         "name":"<Member N name>"
      }
   ],
    
#### .tendermint/config/config.toml ；用于连接其他成员
    # Member 1 node id 可通过以 tendermint show_node_id 查看

    moniker = "Name of our node  【hostname】"
    create_empty_blocks = false
    log_level = "main:info,state:info,*:error"
    
    persistent_peers = "<Member 1 node id>@<Member 1 hostname>:26656,\
    <Member 2 node id>@<Member 2 hostname>:26656,\
    <Member N node id>@<Member N hostname>:26656,"
    
    send_rate = 102400000
    recv_rate = 102400000
    
    recheck = false
    
    
## 启动
    # 启动 mongodb
    systemctl start mongodb
    
    # 启动 bigchaindb | 也可以配置systemd启动
    bigchaindb start 2>&1 > /var/log/bigchaindb.log &
    
    # 启动 tendermint | 也可以配置systemd启动
    tendermint node 2>&1 > /var/log/tendermint.log &
    
    
## bigchaindb tendermint 配置systemd | 配置后需要执行 systemctl daemon-reload
### bigchaindb.service | systemctl {start|restart|stop|status} bigchaindb
    # vi /usr/lib/systemd/system/bigchaindb.service
    [Unit]
    Description=Bigchaindb Server
    After=network.target
    
    [Service]
    User=root
    Restart=on-failure
    WorkingDirectory=/root/
    ExecStart=/usr/bin/bigchaindb start
    
    [Install]
    WantedBy=multi-user.target

### tendermint.service  | systemctl {start|restart|stop|status} tendermint
    # vi /usr/lib/systemd/system/tendermint.service
    [Unit]
    Description=Tendermint Server
    After=network.target
    
    [Service]
    User=root
    Restart=on-failure
    WorkingDirectory=/root/
    ExecStart=/usr/bin/tendermint node
    
    [Install]
    WantedBy=multi-user.target
    
    
## 检查状态
    systemctl status mongod bigchaindb tendermint 
    