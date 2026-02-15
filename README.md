# nodejs-proxy

- nodejs通用项目
- 用于nodejs环境 or 容器，基于nodejs的axios,ws库，
- 基于serverless实现的vless+ trojan+ shadowsocks 三协议
- 集成哪吒探针服务(v0或v1)，
- 可自行添加环境变量

🔖设置环境变量
  | 变量名        | 默认值 | 备注 |
  | ------------ | ------ | ------ |
  | NEZHA_SERVER |空 | 哪吒v1填写形式：nz.abc.com:8008   哪吒v0填写形式：nz.abc.com|
  | NEZHA_PORT   |空 | 哪吒v1没有此变量，v0的agent端口为{443,8443,2096,2087,2083,2053}其中之一时开启tls| 
  | NEZHA_KEY    |空 | 哪吒v1的NZ_CLIENT_SECRET或v0的agent端口 |
  | UUID         |必須 | UUID、运行哪吒v1时在不同的平台需要改UUID,否则会被覆盖|
  | WSPATH       || 节点路径，默认获取uuid前8位|
  | DOMAIN       |空 | 填写项目域名 or 反代域名，不带协议前缀 https://，例如：abc-domain.com， |
  | SUB_PATH     |空 | 订阅路径   |
  | NAME         |空 | 节点名称前缀 |
  | PORT         |3000 | http和ws服务端口 |  
  | AUTO_ACCESS  |false| 是否开启自动访问保活,false为关闭,true为开启,需同时填写DOMAIN变量 |



> [!TIP]
> js混肴地址：https://obfuscator.io/legacy-playground



## nodejs App功能面板設定
[web-hosting部署指南](https://github.com/eooce/node-ws/blob/main/web-hosting.md) 


## 制作docker镜像

1. 将index.js填写需要的变量后混淆，再保存
2. 在.github/workflows/build-image.yml 39行修改镜像名称，
3. 运行github Action
> [!TIP]
>  参考: huggingface视频教程地址：https://youtu.be/XERxg9AODeo

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
