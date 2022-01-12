### 苹果超级签-相关日志
```shell
$ vim /opt/html/qf-xiaoke/public/download/log/20211215/pack_log.log
$ vim /opt/html/qf-xiaoke/public/download/log/20211215/sign_error_log.log
$ tree /home/admin/logs/php/seaslog/xiaokr/
```

### 苹果超级签-原理
- [iOS超级签名-基本流程](https://www.yuque.com/togettoyou/cjqm/rbk50t)
- [火速文档-小氪后台-02-添加账号列表](https://doc.huosdk.com/web/#/p/268b9640f0179a85ff32e99593d32bf0)
- [iOS证书签名原理分析，签名过程，重签名原理](https://www.jianshu.com/p/7ce397c2717d)
- [使用openssl向苹果申请证书](https://www.jianshu.com/p/0c0b457cde92)
- [苹果证书机制问题](https://blog.csdn.net/jbr5740/article/details/88895526)
- [Evernote笔记-【功能】iOS超级签](https://app.yinxiang.com/fx/85a87916-d296-4f36-bf22-e6d4cbfc13a5)

### 苹果超级签-相关代码
```shell
# 管理后台+API+H5页面等与苹果超级签相关的代码均放在统一的一个composer扩展包中(huosdk-iossign)
ll simplewind/vendor/huosdk/huosdk-iossign

# zsign重签名代码是单独放在下载服public/download中的
ll public/download/sub.php
```

### 苹果超级签-与苹果服务器交互的相关代码
- [App Store Connect API](https://developer.apple.com/documentation/appstoreconnectapi)     
- [App Store Connect API 密钥](https://appstoreconnect.apple.com/access/api)
- [OpenAPI specification](https://developer.apple.com/documentation/appstoreconnectapi)
- [借助 App Store Connect API 实现工作流程自动化](https://developer.apple.com/cn/app-store-connect/api/)
- [开发者帐户帮助](https://help.apple.com/developer-account/?lang=zh-cn)
```php
// 1、与苹果服务器交互的请求方法：
\huosdk\iossign\controller\appstoreconnect\AppStoreConnectApi::http()
```

### 苹果超级签-使用zsign对ipa包重签名
```php
\Sub::iosSign()
$_cmd = '/usr/local/bin/zsign -z 9 -f -k '......
```

### 苹果超级签-使用openssl导出证书
```php
\huosdk\iossign\controller\AppleCertificate::exportPem()
$cmd1 = 'openssl pkcs12 -in ' . $_pfx_path . ' -out ' . $_certificate_path . ' -clcerts -nokeys -password pass:' . $_pwd;
$cmd2 = 'openssl pkcs12 -in ' . $_pfx_path . ' -out ' . $_key_pem_path . ' -nocerts -nodes -password pass:' . $_pwd;
```

### 苹果超级签-数据清理
> 注：下述危险操作禁止在正式服执行！！！
>
- 1、替换embedded.mobileprovision（若已过期的话，未过期无需替换，可在[蒲公英网站上](https://www.pgyer.com/tools/udid)下载一个最新的）
```shell
$ /opt/html/qf-xiaoke/public/upload/mobileconfig/embedded.mobileprovision
```

- 2、删除signed0kejingou.cn.mobileconfig（此文件会自动生成）
```shell
$ /opt/html/qf-xiaoke/public/upload/mobileconfig/6045/signed0kejingou.cn.mobileconfig

# 需要保留：/opt/html/qf-xiaoke/public/upload/mobileconfig/embedded.mobileprovision
$ rm -rf /opt/html/qf-xiaoke/public/upload/mobileconfig/*
```

- 3、删除本地缓存证书
```shell
$ rm -rf /opt/html/qf-xiaoke/public/upload/iossign/*
$ rm -rf /opt/html/qf-xiaoke/public/download/cert/*
```

- 4、清空MySQL数据库苹果超级签相关表数据
```sql
truncate h_apple_account;
truncate h_apple_bundle;
truncate h_apple_certificate;
truncate h_apple_device;
truncate h_apple_profile;
truncate h_apple_profile_udid;
truncate h_apple_udid_ipa;
truncate h_apple_udid_ipa_log;
truncate h_apple_udid_list;
truncate h_apple_user;
```

- 5、清空Redis全部缓存
```shell
$ select 3
$ flushdb
```

- 6、清空苹果开发者后台相关数据
> a. 清空所有证书 [Certificates](https://developer.apple.com/account/resources/certificates/list)  
> b. 清空所有包名 [Identifiers](https://developer.apple.com/account/resources/identifiers/list)   
> c. 清空所有描述文件 [Profiles](https://developer.apple.com/account/resources/profiles/list)

- 7、考虑升级 zsign，以确保在新版本的iOS系统下能够正常完成打包签名
```shell
1、下载官方 zsign 压缩包：https://github.com/zhlynn/zsign

2、替换小氪环境部署docker-compose依赖的zsign.zip：qf-miscellany\docker\qf.xiaoke\php-7.1-fpm-centos-7.7\zsign\zsign.zip

3、在服务器上重建构建php docker镜像：docker-compose build php && docker-compose up -d
```