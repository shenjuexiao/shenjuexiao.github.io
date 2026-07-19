# Resolving "npm package not found" Errors

This error occurs when npm cannot locate a required package in your project, often due to missing installations, incorrect paths, or misconfigured project settings.

##Example

```bash
$ npm run build
Error: NPM packages not found. Please confirm npm packages which need to build are in the correct directory
```

A common cause is that dependencies listed in package.json have not been installed. Running npm install in the project root usually resolves this by downloading all required packages into the node_modules directory.

If you are working with environments like WeChat Mini Programs, the issue can also arise if npm packages are outside the miniprogramRoot directory. In such cases, you may need to adjust project.config.json:

```json
"packNpmManually": true,
"packNpmRelationList": [
{
"packageJsonPath": "./package.json",
"miniprogramNpmDistDir": "./miniprogram/"
}
]
```

After updating, restart the development tool and rebuild npm.

Sometimes, the error appears because npm itself is not installed or not accessible. Verify installation with:

```bash
npm -v
```

If it fails, reinstall Node.js (which includes npm) and ensure its binary path is added to your system’s PATH environment variable.

For corrupted or incomplete installs, delete node_modules and package-lock.json, then reinstall:

```bash
rm -rf node_modules package-lock.json
npm install
```

In summary, ensure dependencies are installed, paths are correctly configured, and npm is properly set up in your environment. Restarting your IDE or build tool after changes often helps apply configurations.

## 了解详细信息
- [blog.csdn.net](https://blog.csdn.net/qq_43492356/article/details/130863125)
-  [developers.weixin.qq.com](https://developers.weixin.qq.com/community/minihome/doc/0002c4582ac3303e4413d1da561c00)
- [geeksforgeeks.org](https://www.geeksforgeeks.org/node-js/how-to-resolve-npm-command-not-found-error-in-node-js/)