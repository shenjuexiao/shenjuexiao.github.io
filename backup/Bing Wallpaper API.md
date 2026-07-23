# 必应壁纸每日一图 API

## 内容来源

-   [Bing 每日一图壁纸 API](https://chat.deepseek.com/share/uumxj61bo84xh484tc)[（Deepseek）](https://chat.deepseek.com/a/chat/s/b7fb2e58-5ba6-40a2-ab53-2d72ea012045)

获取必应每日一图，主要有**官方接口**和**第三方封装接口**两种方式，按需选择即可。

## 官方 API：最直接的数据源

Bing 官方提供了获取每日壁纸元数据的接口，这是最可靠的方式。

### 接口地址
```
https://cn.bing.com/HPImageArchive.aspx
```

### 核心参数说明

| 参数 | 说明 | 可选值示例 |
| :--- | :--- | :--- |
| `format` | **必填**。指定返回的数据格式 | `js` (JSON格式) 或 `xml` |
| `idx` | **必填**。图片的日期偏移量。`0`代表今天，`1`代表昨天，依此类推，最多可获取前16天的图片 | `0`, `1`, `2` |
| `n` | **必填**。返回的图片数量，最大为`8` | `1`, `3`, `8` |
| `mkt` | 地区/语言参数，影响图片说明和版权信息的语言 | `zh-CN` (中文), `en-US` (英文) |
| `uhd` | 是否请求4K超高清图片。`1`表示请求 | `1` |

### 请求示例和返回数据
- 获取今天的图片信息（JSON 格式）：
    `https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN`
- 返回示例
    ```json
    {
      "images": [
        {
          "startdate": "20250105",
          "url": "/th?id=OHR.RavennaBasilica_ZH-CN1406474730_UHD.jpg",
          "copyright": "被水淹没的地下室，圣弗朗西斯大教堂，拉文纳，意大利 (© Andrea Pucci/Getty Images)"
        }
      ]
    }
    ```
 
你需要在`url`前拼接上Bing的域名 `https://cn.bing.com` 才能得到完整图片地址。

## 第三方封装接口：免去解析的麻烦

如果你不想自己处理数据，可以直接使用第三方封装好的接口，它们通常会直接返回图片或更简洁的JSON数据。

### `bing.biturl.top` 功能全面的替代API

这个接口将官方数据封装得很简洁，支持指定分辨率和直接重定向到图片，非常方便。

-  获取今日4K壁纸的JSON数据
`https://bing.biturl.top/?resolution=UHD&format=json&index=0&mkt=zh-CN`
-  直接获取今日壁纸图片（重定向）
`https://bing.biturl.top/?resolution=1920&format=image`
-  参数说明
`resolution`支持`UHD`(4K)、`1920`、`1366`等；`index`支持`0`(今天)、`1`(昨天)或`random`(随机)。

### `npanuhin/Bing-Wallpaper-Archive` 强大的历史归档项目

如果你想获取更久远的壁纸，这个项目维护了完整的图片归档。它按国家和语言组织数据，并且可以直接通过日期拼接图片地址，非常实用。
- 获取归档数据*
`https://bing.npanuhin.me/CN-zh.json`

- 直接获取图片
`https://bing.npanuhin.me/CN/zh/2024-01-16.jpg` (替换日期即可)

### 其他可用API

根据搜索结果，还有一些可用的API服务，如 `https://api.1314.cool/bingimg` 和 `https://api.sunweihu.com/api/bing1/api.php`，使用时请注意其稳定性。

## 快速上手：JavaScript 示例

```javascript
// 使用官方API获取今日壁纸URL并显示
fetch('https://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1&mkt=zh-CN')
  .then(response => response.json())
  .then(data => {
    // 从返回数据中提取图片相对路径
    const imageUrl = data.images[0].url;
    // 拼接完整URL
    const fullImageUrl = 'https://cn.bing.com' + imageUrl;
    console.log('今日壁纸:', fullImageUrl);
    // 可以将该URL用于设置背景图片等
  });
```