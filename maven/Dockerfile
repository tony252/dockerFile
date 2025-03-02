FROM maven:3.5.0-jdk-8-alpine

# 创建默认的 settings.xml 文件
RUN echo '<?xml version="1.0" encoding="UTF-8"?>\
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"\
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"\
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">\
  <mirrors>\
    <mirror>\
      <id>aliyun-public</id>\
      <mirrorOf>central</mirrorOf>\
      <name>Aliyun Public</name>\
      <url>https://maven.aliyun.com/repository/public</url>\
    </mirror>\
  </mirrors>\
</settings>' > /usr/share/maven/conf/settings.xml

