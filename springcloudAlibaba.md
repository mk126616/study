# 										SpringCloudAlibaba

netflix的中微服务组件停更进维后，alibaba推出的全套微服务架构解决方案。

# Nacos

Nacos名称含义为naming+configuration+service，是一个服务注册中心加服务配置中心。

## docker方式安装

docker run --restart=unless-stopped --env MODE=standalone --name nacos-server -d -p 8848:8848 nacos/nacos-server:1.1.4

