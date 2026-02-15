# nodejs-proxy

基于serverless实现的vless+ trojan+ shadowsocks 三协议，无内核，nodejs通用项目

用于nodejs环境的玩具和容器，基于nodejs的axios,ws库，集成哪吒探针服务(v0或v1)，可自行添加环境变量

https://domain/${SUB_APTH} or  https://domain:port/${SUB_APTH} 查看节点信息，（SUB_PATH为自行设置的订阅路径，置默认为sub）

## [web-hosting部署指南](https://github.com/eooce/node-ws/blob/main/web-hosting.md) （适用于所有带nodejs App功能面板）

设置环境变量
  | 变量名        | 默认值 | 备注 |
  | ------------ | ------ | ------ |
  | NEZHA_SERVER |        | 哪吒v1填写形式：nz.abc.com:8008   哪吒v0填写形式：nz.abc.com|
  | NEZHA_PORT   |        | 哪吒v1没有此变量，v0的agent端口| 
  | NEZHA_KEY    |        | 哪吒v1的NZ_CLIENT_SECRET或v0的agent端口 |
  | UUID         |        | 开启了哪吒v1,请修改UUID|
  | PORT         |  3000  | 监听端口                    |
  | NAME         |        | 节点名称前缀，例如：Glitch |
  | DOMAIN       |        | 项目分配的域名或已反代的域名，不带协议前缀 https:// |
  | SUB_PATH     |  sub   | 订阅路径   |
  | AUTO_ACCESS  |  false | 是否开启自动访问保活,false为关闭,true为开启,需同时填写DOMAIN变量 |



> [!TIP]
> js混肴地址：https://obfuscator.io/legacy-playground


## 制作docker镜像

1. 将index.js填写需要的变量后混淆，再保存
2. 在.github/workflows/build-image.yml 39行修改镜像名称，运行github Action

`Dockerfile` 文件内容如下:
```
FROM node:lts-alpine3.23

WORKDIR /app

COPY index.js index.html package.json ./

RUN apk update && apk add --no-cache bash openssl curl &&\
    chmod +x index.js &&\
    npm install

CMD ["node", "index.js"]
```

参考: huggingface视频教程地址：https://youtu.be/XERxg9AODeo


## 使用cloudflare workers 或 snippets 反代域名给节点套cdn加速
```
export default {
    async fetch(request, env) {
        let url = new URL(request.url);
        if (url.pathname.startsWith('/')) {
            var arrStr = [
                'change.your.domain', // 此处单引号里填写你的节点伪装域名
            ];
            url.protocol = 'https:'
            url.hostname = getRandomArray(arrStr)
            let new_request = new Request(url, request);
            return fetch(new_request);
        }
        return env.ASSETS.fetch(request);
    },
};
function getRandomArray(array) {
  const randomIndex = Math.floor(Math.random() * array.length);
  return array[randomIndex];
}
```
