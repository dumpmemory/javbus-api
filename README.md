# JavBus API <!-- omit in toc -->

一个自我托管的 [JavBus](https://www.javbus.com) API 服务

## 目录 <!-- omit in toc -->

- [用途](#用途)
- [使用](#使用)
  - [部署与启动](#部署与启动)
    - [Docker 部署（推荐）](#docker-部署推荐)
    - [Node.js 部署](#nodejs-部署)
      - [使用 PM2 保持服务后台常驻](#使用-pm2-保持服务后台常驻)
  - [配合 web 服务器](#配合-web-服务器)
- [API 文档](#api-文档)
  - [/api/v1/movies](#apiv1movies)
    - [method](#method)
    - [参数](#参数)
    - [请求举例](#请求举例)
    - [返回举例](#返回举例)
  - [/api/v1/movies/search](#apiv1moviessearch)
    - [method](#method-1)
    - [参数](#参数-1)
    - [请求举例](#请求举例-1)
    - [返回举例](#返回举例-1)
  - [/api/v1/movies/{id}](#apiv1moviesid)
    - [method](#method-2)
    - [请求举例](#请求举例-2)
    - [返回举例](#返回举例-2)
  - [/api/v1/stars/{id}](#apiv1starsid)
    - [method](#method-3)
    - [参数](#参数-2)
    - [请求举例](#请求举例-3)
    - [返回举例](#返回举例-3)

## 用途

- 可以用来搭建自己的视频信息网站
- 可以作为移动端 App 的 API 服务
- 可以作为爬虫的数据源
- 可以用来制作 iOS/macOS 快捷指令
- 可以用来开发 Telegram 机器人
- 等等...

## 使用

注意：本程序仅仅是 JavBus 的一个在线转换服务，因此不依赖数据库服务，每个请求会实时请求 JavBus 对应的网页，解析之后返回对应的 json 数据

**所以需要保证部署本程序的机器有访问 JavBus 的能力，否则请求会失败**

### 部署与启动

#### Docker 部署（推荐）

[Docker Hub 地址](https://hub.docker.com/r/ovnrain/javbus-api)

```shell
$ docker pull ovnrain/javbus-api
$ docker run -d \
    --name=javbus-api \
    --restart=unless-stopped \
    -p 8922:3000 \
    ovnrain/javbus-api
```

启动一个 Docker 容器，将其名称设置为 `javbus-api`，端口设置为 `8922`，并且自动重启

#### Node.js 部署

```shell
$ git clone https://github.com/ovnrain/javbus-api.git
$ cd javbus-api
$ nvm use # 可选，使用 .nvmrc 中指定的 Node.js 版本，关于 nvm 的安装与使用，请参考 https://github.com/nvm-sh/nvm
$ pnpm install # 或者 npm install 或者 yarn install
$ npm run build
$ echo "PORT=8922" > .env # 可选，默认端口为 `3000`
$ npm start
```

##### 使用 PM2 保持服务后台常驻

```shell
$ npm install -g pm2
$ pm2 start npm --name javbus-api -- start
```

_关于 PM2 的详细使用方法，请参考 [PM2 官方文档](https://pm2.keymetrics.io/docs/usage/quick-start/)_

服务启动后，在浏览器中访问 [http://localhost:8922](http://localhost:8922) 即可获取结果

### 配合 web 服务器

以上两种方式都可以配合 nginx 等一起使用，以实现 https 访问等，例如 nginx 配置如下：

```nginx
location /api {
  proxy_pass http://localhost:8922;
  proxy_http_version 1.1;
  proxy_set_header Host $host;

  add_header cache-control "no-cache";
}
```

## API 文档

### /api/v1/movies

获取影片列表

#### method

GET

#### 参数

| 参数        | 是否必须 | 可选值                                                                       | 默认值   | 说明                                                                                                                                                              |
| ----------- | -------- | ---------------------------------------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| page        | 是       |                                                                              |          | 页码                                                                                                                                                              |
| magnet      | 是       | `exist`<br />`all`                                                           |          | `exist`: 只返回有磁力链接的影片<br />`all`: 返回全部影片                                                                                                          |
| filterType  | 否       | `star`<br />`genre`<br />`director`<br />`studio`<br />`label`<br />`series` |          | 筛选类型，必须与 `filterValue` 一起使用<br />`star`: 演员<br />`genre`: 类别<br />`director`: 导演<br />`studio`: 制作商<br />`label`: 发行商<br />`series`: 系列 |
| filterValue | 否       |                                                                              |          | 筛选值，必须与 `filterType` 一起使用                                                                                                                              |
| type        | 否       | `normal`<br />`uncensored`                                                   | `normal` | `normal`: 有码影片<br />`uncensored`: 无码影片                                                                                                                    |

#### 请求举例

    /api/v1/movies?page=1&magnet=exist

返回有磁力链接的第一页影片

    /api/v1/movies?page=1&filterType=star&filterValue=rsv&magnet=all

返回演员 ID 为 `rsv` 的影片的第一页，包含有磁力链接和无磁力链接的影片

    /api/v1/movies?page=2&filterType=genre&filterValue=4&magnet=exist

返回类别 ID 为 `4` 的影片的第二页，只返回有磁力链接的影片

    /api/v1/movies?page=1&magnet=exist&type=uncensored

返回无码影片的第一页，只返回有磁力链接的影片

#### 返回举例

<details>
<summary>点击展开</summary>

```jsonc
{
  // 影片列表
  "movies": [
    {
      "date": "2023-04-28",
      "id": "YUJ-003",
      "img": "https://www.javbus.com/pics/thumb/9n0d.jpg",
      "title": "夫には言えない三日間。 セックスレスで欲求不満な私は甥っ子に中出しさせています。 岬ななみ",
      "tags": ["高清", "字幕", "3天前新種"]
    }
    // ...
  ],
  // 分页信息
  "pagination": {
    "currentPage": 1,
    "hasNextPage": true,
    "nextPage": 2,
    "pages": [1, 2, 3]
  },
  // 筛选信息，注意：只有在请求参数包含 filterType 和 filterValue 时才会返回
  "filter": {
    "name": "岬ななみ",
    "type": "star",
    "value": "rsv"
  }
}
```

</details>

### /api/v1/movies/search

搜索影片

#### method

GET

#### 参数

| 参数    | 是否必须 | 可选值                     | 默认值   | 说明                                                     |
| ------- | -------- | -------------------------- | -------- | -------------------------------------------------------- |
| keyword | 是       |                            |          | 搜索关键字                                               |
| page    | 是       |                            |          | 页码                                                     |
| magnet  | 是       | `exist`<br />`all`         |          | `exist`: 只返回有磁力链接的影片<br />`all`: 返回全部影片 |
| type    | 否       | `normal`<br />`uncensored` | `normal` | `normal`: 有码影片<br />`uncensored`: 无码影片           |

#### 请求举例

    /api/v1/movies/search?keyword=三上&page=1&magnet=exist

搜索关键词为 `三上` 的影片的第一页，只返回有磁力链接的影片

    /api/v1/movies/search?keyword=三上&page=1&magnet=all

搜索关键词为 `三上` 的影片的第一页，包含有磁力链接和无磁力链接的影片

#### 返回举例

<details>
<summary>点击展开</summary>

```jsonc
{
  // 影片列表
  "movies": [
    {
      "date": "2020-08-15",
      "id": "SSNI-845",
      "img": "https://www.javbus.com/pics/thumb/7t44.jpg",
      "title": "彼女の姉は美人で巨乳しかもドS！大胆M性感プレイでなす術もなくヌキまくられるドMな僕。 三上悠亜",
      "tags": ["高清", "字幕"]
    }
    // ...
  ],
  // 分页信息
  "pagination": {
    "currentPage": 2,
    "hasNextPage": true,
    "nextPage": 3,
    "pages": [1, 2, 3, 4, 5]
  },
  "keyword": "三上"
}
```

</details>

### /api/v1/movies/{id}

获取影片详情

#### method

GET

#### 请求举例

    /api/v1/movies/SSIS-406

返回番号为 `SSIS-406` 的影片详情

#### 返回举例

<details>
<summary>点击展开</summary>

```jsonc
{
  "id": "SSIS-406",
  "title": "SSIS-406 才色兼備な女上司が思う存分に羽目を外し僕を連れ回す【週末限定】裏顔デート 葵つかさ",
  "img": "https://www.javbus.com/pics/cover/8xnc_b.jpg",
  // 封面大图尺寸
  "imageSize": {
    "width": 800,
    "height": 538
  },
  "date": "2022-05-20",
  // 影片时长
  "videoLength": 120,
  "director": {
    "id": "hh",
    "name": "五右衛門"
  },
  "producer": {
    "id": "7q",
    "name": "エスワン ナンバーワンスタイル"
  },
  "publisher": {
    "id": "9x",
    "name": "S1 NO.1 STYLE"
  },
  "series": {
    "id": "xx",
    "name": "xx"
  },
  "genres": [
    {
      "id": "e",
      "name": "巨乳"
    }
    // ...
  ],
  // 演员信息，一部影片可能包含多个演员
  "stars": [
    {
      "id": "2xi",
      "name": "葵つかさ"
    }
  ],
  // 磁力链接列表
  "magnets": [
    {
      "link": "magnet:?xt=urn:btih:A6D7C90FAB7E4223C61425A2E4CDF9E503CEDAA2&dn=SSIS-406-C",
      // 是否高清
      "isHD": true,
      "title": "SSIS-406-C",
      "size": "5.46GB",
      // bytes
      "numberSize": 5862630359,
      "shareDate": "2022-05-20",
      // 是否包含字幕
      "hasSubtitle": true
    }
    // ...
  ],
  // 影片预览图
  "samples": [
    {
      "alt": "SSIS-406 才色兼備な女上司が思う存分に羽目を外し僕を連れ回す【週末限定】裏顔デート 葵つかさ - 樣品圖像 - 1",
      "id": "8xnc_1",
      // 大图
      "src": "https://pics.dmm.co.jp/digital/video/ssis00406/ssis00406jp-1.jpg",
      // 缩略图
      "thumbnail": "https://www.javbus.com/pics/sample/8xnc_1.jpg"
    }
    // ...
  ]
}
```

</details>

### /api/v1/stars/{id}

获取演员详情

#### method

GET

#### 参数

| 参数 | 是否必须 | 可选值                     | 默认值   | 说明                                                           |
| ---- | -------- | -------------------------- | -------- | -------------------------------------------------------------- |
| type | 否       | `normal`<br />`uncensored` | `normal` | `normal`: 有码影片演员详情<br />`uncensored`: 无码影片演员详情 |

#### 请求举例

    /api/v1/stars/2xi

返回演员 `葵つかさ` 的详情

    /api/v1/stars/2jd?type=uncensored

返回演员 `波多野結衣` 的详情

#### 返回举例

<details>
<summary>点击展开</summary>

```jsonc
{
  "avatar": "https://www.javbus.com/pics/actress/2xi_a.jpg",
  "id": "2xi",
  "name": "葵つかさ",
  "birthday": "1990-08-14",
  "age": "32",
  "height": "163cm",
  "bust": "88cm",
  "waistline": "58cm",
  "hipline": "86cm",
  "birthplace": "大阪府",
  "hobby": "ジョギング、ジャズ鑑賞、アルトサックス、ピアノ、一輪車"
}
```

</details>
