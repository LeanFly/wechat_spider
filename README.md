# wechat_spider 微信爬虫

基于 Node.js 的微信爬虫，通过中间人代理的原理，批量获取微信文章数据，包括阅读量、点赞量、在看数、评论和文章正文等数据。

使用代理模块 AnyProxy。代码已支持 AnyProxy 4 版本。

支持 Docker 部署。

项目可运行在个人电脑上，也可部署在服务器上。

## 开始

### 安装前准备

- 安装 Node，版本大于 8.8.1
- 安装 MongoDB，版本大于 3.4.6
- 安装 Redis

### 安装

```bash
git clone https://github.com/lqqyt2423/wechat_spider.git
cd wechat_spider
npm install
```

本项目基于代理模块 AnyProxy，解析微信 HTTPS 请求需在电脑和手机上都安装证书。可参考：[AnyProxy 文档](http://anyproxy.io/cn/#%E8%AF%81%E4%B9%A6%E9%85%8D%E7%BD%AE)。

### 通过 Docker 部署

```bash
git clone https://github.com/lqqyt2423/wechat_spider.git
cd wechat_spider
# build image
docker-compose build
# 运行实例（mongo数据存储地址需通过环境变量MONGO_PATH传入）
MONGO_PATH=/data/mongo docker-compose up
# 终止运行
docker-compose down
```

## 全Docker化部署
```bash
    部署MongoDB：docker run -dit --name mongodb -v $PWD/:/data/db -p 27017:27017 mongo:4.4.4
    部署Redis：docker run -dit --name redis -v $PWD/data:/data -p 6379:6379 redis
    修改config： 自定义内容请依次配置，比如：44行MongoDB数据修改成自己创建的MongoDB；49、50行的Redis数据修改成自己创建的Redis；69、70的文章发布时间范围；81行的公众号biz；87行的文章保存形式；101、104行的时间；108行公众号biz；131、132行MongoDB和Redis；
    自己编译docker镜像：docker build -t xxx/yyy:tag .
    使用已有镜像：docker pull leanfly/wechat_spider:latest
    启动：docker run --name wechat_spider -v $PWD/config.js:/app/config.js -p 18101:8101 -p 18102:8102 -p 18104:8104 leanfly/wechat_spider
    查看日志：docker logs -f wechat_spider
    安装证书：浏览器访问 http://192.168.5.200:18102
    配置代理：服务器-192.168.5.200，端口-18101
    微信访问公众号链接，注意这里的biz需要自己配置：https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=Mzg4NzYxNTQzNw==&scene=124#wechat_redirect
    点开一个详情文章，刷新，然后会看到自动开始刷新入库
    访问web：http://192.168.5.200:18104
```

- `Dockerfile` 中已经设置了在 `Linux` 环境的 Docker 中添加根证书的操作步骤，所以接下来仅需在手机上安装 https 证书即可。
- 最终手机上设置的代理 ip 还是需要以自己电脑上的 ip 为准，需忽略 Docker 实例中打印的 ip 地址
- 可编辑 `Dockerfile` 和 `docker-compose.yml` 改变部署规则

## 使用

```bash
cd wechat_spider
npm start
```

1. 确保电脑和手机连接同一 WIFI，`npm start` 之后，命令行输出`请配置代理: xx.xx.xx.xx:8101` 类似语句，手机设置代理为此 IP 和端口
2. 手机上测试打开任一公众号历史文章详情页和文章页，观察电脑命令行的输出，查看数据是否保存至 MongoDB

> - 如需测试自动翻页，可先多次分别打开不同的公众号的历史详情页，等数据库中有了翻页的基础公众号信息之后，再随便进入历史页等待翻页跳转
> - 翻页逻辑仅支持公众号历史页面跳公众号历史页面，微信文章页面跳微信文章页面，两个不同页面不能互相跳转

### 针对微信新版需注意

1. 历史页面可自行拼接后发送至微信中打开，拼接规则为：

```javascript
var biz = 'MzI4NjQyMTM2Mw==';
var history_page = 'https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=' + biz + '&scene=124#wechat_redirect';
// https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzI4NjQyMTM2Mw==&scene=124#wechat_redirect
```

2. 进入微信文章页面先刷新一下

### 自定义配置

可编辑 `config.js` 文件进行自定义配置，文件中每个配置项都有详细的说明。

可配置项举例如下：

- 控制是否开启文章或历史详情页自动跳转
- 控制跳转时间间隔
- 根据文章发布时间控制抓取范围
- 是否保存文章正文内容
- 是否保存文章评论

需注意，本项目修改了 AnyProxy 的默认端口。连接代理的端口改为 8101，AnyProxy 管理界面的端口改为 8102，且仅在 `NODE_ENV=development` 时才会开启 AnyProxy 的管理界面功能。如需修改，可编辑 `config.js`。

### 可视化界面

前端页面已打包好，启动项目后，如无修改默认 `server port` 配置，浏览器直接访问 `http://localhost:8104` 即可。检测数据有无抓取保存直接刷新此页面即可。

![可视化界面](./imgs/posts_screenshot.png)

前端页面由 `React` 编写，如需修改，可编辑 `client` 文件中的代码。

### MongoDB 数据信息

数据库 database: wechat_spider

数据表 collections:

- posts - 文章数据
- profiles - 公众号数据
- comments - 评论数据

### 从 MongoDB 导出数据

#### 命令行直接导出数据

```bash
mongoexport --db wechat_spider --collection posts --type=csv --fields title,link,publishAt,readNum,likeNum,likeNum2,msgBiz,msgMid,msgIdx,sourceUrl,cover,digest,isFail --out ~/Desktop/posts.csv
```

#### 脚本导出

可参考文件 `/test/exportData.js` 。

## 感谢

感谢此文章提供思路：[微信公众号文章批量采集系统的构建](https://zhuanlan.zhihu.com/p/24302048)

## 赞赏

如本项目对你有所帮助，可扫码赞赏。

![赞赏](./imgs/pay.JPG)

## 联系作者

由于平时比较忙，如果是一些基础的问题，尽量不要联系我，加我微信问我我也不会回复的，有问题不懂先看上面的文档和代码。合作、商务方面等求可添加我为好友。

![添加微信](./imgs/wx.JPG)

## License

[MIT](LICENSE)