---
layout: post
title: 用 Jenkins 搭建Flutter 自动打包
---


1. 下载安装Jenkins
https://www.jenkins.io

2. 配置Jenkins流水线
- 浏览器打开 localhost:8080 进入Jenkins
- 打开 “我的视图”，新建流水线，打开“项目配置”
- ”源码管理“填写git代码URL、密码，”执行分支“
- ”构建环境“，钩上“Delete workspace before build starts”， “在构建日志中添加时间戳前缀”
- “Build Steps”，选择“shell”，并填写shell脚本
- “构建后操作”，选择“归档成品”， 

``` bash
#ios
build/ios/ipa/*.ipa, build/ios/ipa/*.png

#android
build/app/outputs/apk/release/*.apk
```


3. shell打包脚本

```bash
#android
FLUTTER_PROJECT_DIR="$PWD"
formatted_datetime=$(date +"%Y-%m-%d_%H%M%S")
feishu_api=https://open.feishu.cn/open-apis/bot/v2/hook/xxx

source "/Users/xxx/.bash_profile"
# 切换到 Flutter 项目目录
cd "$FLUTTER_PROJECT_DIR"

# 清除之前的构建
flutter clean
flutter pub get
flutter gen-l10n
# 构建 release 版本的 APK
flutter build apk --release
# 构建 release 版本的 aab
# flutter build appbundle --release
echo "bigp_v17_1.1.6-release_formalAD_$formatted_datetime"

echo "~~~~~~~~~~~~~~~~~~~飞书通知~~~~~~~~~~~~~~~~~~~"
curl -X POST \
$feishu_api \
-H 'Content-Type: application/json' \
-d '{"msg_type": "post","content": {"post": {"zh_cn": {"title": "","content": [[{"tag": "text","text": "打包完成"}]]}}}}'

```

``` bash
#iOS
#!/bin/bash
# 设置变量
FLUTTER_PROJECT_DIR="$PWD"
API_TOKEN="xxx" # fir token
SCHEME_NAME="Runner"
FORMATTED_DATETIME=$(date +"%Y-%m-%d_%H%M%S")
FEISHU_API="xxx"

source "/Users/xxx/.bash_profile"

echo "~~~~~~~~~~~~~~~~~~~flutter打包~~~~~~~~~~~~~~~~~~~"
cd "$FLUTTER_PROJECT_DIR"
flutter clean
flutter pub get
flutter gen-l10n
flutter build ipa --release \
    --export-options-plist=$FLUTTER_PROJECT_DIR/ios/ExportOptions.plist

echo "~~~~~~~~~~~~~~~~~~~上传 fir.im~~~~~~~~~~~~~~~~~~~"
commitInfo="$FORMATTED_DATETIME"
fir p "build/ios/ipa/AI Comic.ipa" -T $API_TOKEN -c "$commitInfo" --feishu-access-token=$FEISHU_API --feishu-custom-message="打包完成"

mv "build/ios/ipa/xxx.ipa" build/ios/ipa/xxx_$formatted_datetime.ipa
echo "~~~~~~~~~~~~~~~~~~~iOS打包完成~~~~~~~~~~~~~~~~~~~"
```

Jenkins, 启动命令, 支持局域网内其他电脑通过IP访问
/opt/homebrew/opt/openjdk@17/bin/java -Dmail.smtp.starttls.enable=true -jar /opt/homebrew/opt/jenkins-lts/libexec/jenkins.war --httpListenAddress=0.0.0.0 --httpPort=8080