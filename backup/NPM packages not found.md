	# NPM packages not found

## 内容来源

-   [TDesign小程序看板设计](https://chat.deepseek.com/share/75k9rg5oxr7uivdn3h)[（Deepseek）](https://chat.deepseek.com/a/chat/s/1baeab24-723a-4953-be69-007b610bc7ed)

![NPM packages not found](https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/wechat_devtools_NPM_packages.png)

> NPM packages not found. Please confirm npm packages which need to build are belong to `miniprogramRoot` directory. Or you may edit project.config.json's `packNpmManually` and `packNpmRelationList`
appid: wx429151859caeb046
openid: o6zAJsws4oaOjHasDK8qBirpUw8Q
ideVersion: 2.01.2510290
osType: win32-x64

开发者工具找不到需要构建的 npm 包。 
 
微信web开发者工具：wechat_devtools_2.01.2510290_win32_x64.exe  

 IDE 版本（2.01.2510290），这套新版的配置方式确实需要特别注意。
 
## 🎯根本原因
 `project.config.json` 中可能同时存在 `miniprogramRoot` 配置，而 `packNpmRelationList` 中的路径没有与之匹配，导致工具找不到 `node_modules`。

##✅ 解决方案（三步走）
### 第一步：确认项目结构
典型的微信小程序项目结构：
```json
你的项目文件夹/
├── miniprogram/	← 小程序代码（app.json, pages/ 等）
├── node_modules/	← npm 包（安装后自动生成）
├── package.json
├── project.config.json
└── ...
```
如果 `miniprogram/` 存在，说明 `miniprogramRoot` 已经指向了 `miniprogram` 文件夹。

### 第二步：修改 `project.config.json`（关键）
在 `project.config.json` 中，按以下方式配置，注意 `miniprogramNpmDistDir` 的值要根据你的实际 `miniprogramRoot` 来调整。

```json
{
	// 假设你的代码在 miniprogram 文件夹
  "miniprogramRoot": "miniprogram/",   
  "setting": {
    "packNpmManually": true,
    "packNpmRelationList": [
      {
        "packageJsonPath": "./package.json",
        "miniprogramNpmDistDir": "./miniprogram/"
      }
    ]
  }
}
```
关键点说明：  

|字段|值|含义|
|-----|-----|------|
|miniprogramRoot|"miniprogram/"|小程序代码文件夹|
|packageJsonPath|"./package.json"|package.json 在项目根目录|
|miniprogramNpmDistDir|"./miniprogram/"|构建后的 miniprogram_npm 在 miniprogram/ 下面，小程序能引用|
	
	
💡 如果你的小程序代码直接在根目录（没有 `miniprogram/` 文件夹），那么 `miniprogramRoot` 应该是 "./"，`miniprogramNpmDistDir` 也设为 "./" 即可。

### 第三步：重新构建

保存 `project.config.json` 文件。

点击微信开发者工具菜单栏：`工具` -> `构建 npm`。

等待构建完成，项目根目录（或 `miniprogram/` 下）会出现 `miniprogram_npm` 文件夹。

### 🔧 “杀手锏”
如果还是报错，试试这个“杀手锏”。

有些情况下，开发者工具缓存了旧配置，导致新配置不生效。可以这样做：

完全关闭微信开发者工具。

手动删除项目下的 `miniprogram_npm` 文件夹（如果有）。

重新打开开发者工具，先不要点击`构建 npm`。

先点击菜单栏 项目 -> 重新打开此项目，让工具重新加载所有配置。

然后再点击 `工具` -> `构建 npm`。

##  修改后的 `project.config.json`

```json
{
  "miniprogramRoot": "miniprogram/",
  "cloudfunctionRoot": "cloudfunctions/",
  "setting": {
    "urlCheck": true,
    "es6": true,
    "enhance": true,
    "postcss": true,
    "preloadBackgroundData": false,
    "minified": true,
    "newFeature": true,
    "coverView": true,
    "nodeModules": false,
    "autoAudits": false,
    "showShadowRootInWxmlPanel": true,
    "scopeDataCheck": false,
    "uglifyFileName": false,
    "checkInvalidKey": true,
    "checkSiteMap": true,
    "uploadWithSourceMap": true,
    "compileHotReLoad": false,
    "useMultiFrameRuntime": true,
    "useApiHook": true,
    "useApiHostProcess": true,
    "babelSetting": {
      "ignore": [],
      "disablePlugins": [],
      "outputPath": ""
    },
    "enableEngineNative": false,
    "useIsolateContext": true,
    "useCompilerModule": true,
    "userConfirmedUseCompilerModuleSwitch": false,
    "userConfirmedBundleSwitch": false,
    "packNpmManually": true,
    "packNpmRelationList": [
      {
        "packageJsonPath": "./package.json",
        "miniprogramNpmDistDir": "./miniprogram/"
      }
    ],
    "minifyWXSS": true,
    "compileWorklet": false,
    "minifyWXML": true,
    "localPlugins": false,
    "disableUseStrict": false,
    "useCompilerPlugins": false,
    "condition": false,
    "swc": false,
    "disableSWC": true
  },
  "appid": "wx429151859caeb046",
  "projectname": "quickstart-wx-cloud",
  "libVersion": "3.17.0",
  "cloudfunctionTemplateRoot": "cloudfunctionTemplate/",
  "condition": {
    "search": {
      "list": []
    },
    "conversation": {
      "list": []
    },
    "plugin": {
      "list": []
    },
    "game": {
      "list": []
    },
    "miniprogram": {
      "list": [
        {
          "id": -1,
          "name": "db guide",
          "pathName": "pages/databaseGuide/databaseGuide"
        }
      ]
    }
  },
  "compileType": "miniprogram",
  "srcMiniprogramRoot": "miniprogram/",
  "packOptions": {
    "ignore": [],
    "include": []
  },
  "editorSetting": {
    "tabIndent": "insertSpaces",
    "tabSize": 2
  },
  "simulatorPluginLibVersion": {}
}
```