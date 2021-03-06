#### uni-app自动化测试

uni-app提供了一批API，这些API可以操控uni-app应用，包括运行、跳转页面、触发点击等，并可以获取页面元素状态、进行截图，从而实现对uni-app项目进行自动化测试的目的。

#### 特性
开发者可以利用API做以下事情：

* 控制跳转到指定页面
* 获取页面数据
* 获取页面元素状态
* 触发元素绑定事件
* 调用 uni 对象上任意接口

**平台差异说明**

|App|H5|微信小程序|支付宝小程序|百度小程序|字节跳动小程序|QQ小程序|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|√(ios仅支持模拟器)|√|√|x|x|x|x|


目前仅 [cli](https://uniapp.dcloud.net.cn/quickstart?id=_2-通过vue-cli命令行) 工程支持。有利于持续集成。

推荐使用方式：研发提交源码到版本库后，持续集成系统自动拉取源码，自动运行自动化测试。

创建 `cli` 工程
```
# 全局安装vue-cli
$ npm install -g @vue/cli
$ cd ... // 切换到工程保存目录
$ vue create -p dcloudio/uni-preset-vue#alpha my-project
```

如果之前是HBuilderX工程，则把HBuilderX工程内的文件（除 unpackage、node_modules 目录）拷贝至 vue-cli 工程的 src 目录。
在 vue-cli 工程内重新安装 npm 依赖（如果之前使用了 npm 依赖的话）

已有 `cli` 工程
1. 更新依赖包 `@dcloudio/*` >= `2.0.0-alpha-27920200612001`
2. 安装依赖包 `uni-automator`
```
npm install uni-automator
```
3. package.json script节点新增命令
```
"test:h5": "cross-env UNI_PLATFORM=h5 jest -i",
"test:android": "cross-env UNI_PLATFORM=app-plus UNI_OS_NAME=android jest -i",
"test:ios": "cross-env UNI_PLATFORM=app-plus UNI_OS_NAME=ios jest -i",
"test:mp-weixin": "cross-env UNI_PLATFORM=mp-weixin jest -i",
"test:mp-baidu": "cross-env UNI_PLATFORM=mp-baidu jest -i"
```


#### H5平台测试流程

1. 工程目录下安装依赖
```
npm install puppeteer
```

2. 根据API编写测试的js代码，参考测试用例
API文档见：[https://uniapp.dcloud.io/collocation/auto/api](https://uniapp.dcloud.io/collocation/auto/api)

3. 运行测试
```
npm run test:h5
```

4. 测试结果
```
>> cross-env UNI_PLATFORM=h5 jest -i
...
Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        14.995s, estimated 16s
```

更多配置参考 `jest.config.js`


#### App-Android测试流程

1. 配置全局 `adb` 环境变量

2. 配置 `Hbuilder` 调试基座/自定义基座 `android_base.apk` 目录，参考 `jest.config.js`

3. 创建 `cli` 工程/现有 `cli` 工程
切换到工程目录，安装依赖包 `adbkit`
```
npm install adbkit
```

4. 编写测试代码，参考测试用例

5. 运行测试
```
npm run test:android
```

**注意**

在 windows 系统下因 adb 同步问题，提供临时方案

1-4 同上

5.1 编译工程
```
npm run dev:app-plus  -- --auto-port 9520
```
将编译后的目录 `dist/dev/app-plus` 拖到 `HBuilderX` 中，运行到设备

5.2 运行自动化测试
```
npm run test:android
```


#### App-iOS测试流程

目前仅支持 iOS 模拟器

1. 配置模拟器id，参考 `jest.config.js`

2. 配置 `Hbuilder` 调试基座/自定义基座 `android_base.apk` 目录，参考 `jest.config.js`

3. 编写测试代码，参考测试用例

4. 运行测试
```
npm run test:ios
```



#### 微信小程序测试流程

1. 创建cli项目，同H5平台 (必须配置微信小程序 appid, manifest.json -> mp-weixin -> appid)

2. 运行测试(如果微信开发者工具无法成功打开项目，请手动打开)
```
npm run test:mp-weixin
```

3. 测试结果
```
> cross-env NODE_ENV=development UNI_PLATFORM=mp-weixin vue-cli-service uni-build --watch "--auto-port" "9520"
Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        14.995s, estimated 16s
```



#### 测试示例

使用 hello uni-app 工程测试 H5 平台

1. 创建 `cli` 项目，选择 `hello uni-app`
```
$ vue create -p dcloudio/uni-preset-vue#alpha my-hello-uni-app
# 进入项目目录
$ cd my-hello-uni-app
```

2. 安装 `puppeteer`
```
npm install puppeteer
```

3. 创建测试文件 `src/__tests__/pages/tabBar/component/component.spec.js`，复制下面代码
```
describe('pages/tabBar/component/component.nvue', () => {
    let page
    beforeAll(async () => {
        // 重新reLaunch至首页，并获取首页page对象（其中 program 是uni-automator自动注入的全局对象）
        page = await program.reLaunch('/pages/tabBar/component/component')
        await page.waitFor(1000)
    })

    it('u-link', async () => {
        // 检测首页u-link的文本内容
        expect(await (await page.$('.hello-link')).text()).toBe('https://uniapp.dcloud.io/component/')
    })

    it('视图容器', async () => {
        // 检测首个 panel 是视图容器
        expect(await (await page.$('.uni-panel-text')).text()).toBe('视图容器')
        // 检测首个 panel 切换展开
        const panelH = await page.$('.uni-panel-h');
        // 不能做完全匹配，百度小程序会生成额外的class
        expect(await panelH.attribute('class')).toContain('uni-panel-h')
        await panelH.tap()
        await page.waitFor(500)
        // 已展开
        expect(await panelH.attribute('class')).toContain('uni-panel-h-on')
    })

    it('.uni-panel', async () => {
      const lists = await page.$$('.uni-panel')
      expect(lists.length).toBe(9)
    })

    it('.uni-panel action', async () => {
      const listHead = await page.$('.uni-panel-h')
      expect(await listHead.attribute('class')).toContain('uni-panel-h-on')
      await listHead.tap()
      await page.waitFor(200)
      expect(await listHead.attribute('class')).toContain(
        'uni-panel-h',
      )

      // 展开第一个 panel，点击第一个 item，验证打开的新页面是否正确
      await listHead.tap()
      await page.waitFor(200)
      const item = await page.$('.uni-navigate-item')
      await item.tap()
      await page.waitFor(500)
      expect((await program.currentPage()).path).toBe('pages/component/view/view')
      await page.waitFor(500)

      // 执行 navigateBack 验证是否返回
      expect((await program.navigateBack()).path).toBe('pages/tabBar/component/component')
    })
})
```

4. 运行测试
```
npm run test:h5
```

5. 测试结果
```
> cross-env UNI_PLATFORM=h5 jest -i
 PASS  src/__tests__/pages/tabBar/component/component.spec.js (14.789s)
  pages/tabBar/component/component.nvue
    √ u-link (8ms)
    √ 视图容器 (518ms)
    √ .uni-panel (2ms)
    √ .uni-panel action (4447ms)
Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        14.995s, estimated 16s
```



##### 屏幕截图示例
```
describe('pages/API/set-navigation-bar-title/set-navigation-bar-title.vue', () => {
    let page
    beforeAll(async () => {
        // 重新reLaunch至首页，并获取首页page对象（其中 program 是uni-automator自动注入的全局对象）
        page = await program.reLaunch('/pages/API/set-navigation-bar-title/set-navigation-bar-title')
        await page.waitFor(3000)
    })

    it('.uni-hello-text', async () => {
      var image = await program.screenshot({
        path: "set-navigation-bar-title.png"  // 默认项目根目录
      })
      console.log(image)
    })
})
```

更多测试示例见： hello uni-app

GitHub： [https://github.com/dcloudio/hello-uniapp](https://github.com/dcloudio/hello-uniapp)



#### jest.config.js
```
module.exports = {
  globalTeardown: '@dcloudio/uni-automator/dist/teardown.js',
  testEnvironment: '@dcloudio/uni-automator/dist/environment.js',
  testEnvironmentOptions: {
    compile: true,
    h5: { // 为了节省测试时间，可以指定一个 H5 的 url 地址，若不指定，每次运行测试，会先 npm run dev:h5
      url: "http://192.168.x.x:8080/h5/",
      options: {
        headless: false // 配置是否显示 puppeteer 测试窗口
      }
    },
    "app-plus": {
      android: {
        executablePath: "HBuilderX/plugins/launcher/base/android_base.apk" // apk 目录
      },
      ios: {
        // uuid 必须配置，目前仅支持模拟器，可以（xcrun simctl list）查看要使用的模拟器 uuid
        id: "",
        executablePath: "HBuilderX/plugins/launcher/base/iPhone_base.ipa" // ipa 目录
      }
    }
  },
  testTimeout: 15000,
  reporters: [
    'default'
  ],
  watchPathIgnorePatterns: ['/node_modules/', '/dist/', '/.git/'],
  moduleFileExtensions: ['js', 'json'],
  rootDir: __dirname,
  testMatch: ['<rootDir>/src/__tests__/**/*spec.[jt]s?(x)'],
  testPathIgnorePatterns: ['/node_modules/']
}

```



**注意事项**

1. 如果页面涉及到分包加载问题，`reLaunch` 获取的页面路径可能会出现问题 ，解决方案如下 ：
```javascript
// 重新 reLaunch至首页，并获取 page 对象（其中 program 是 uni-automator 自动注入的全局对象）
page = await program.reLaunch('/pages/extUI/calendar/calendar')
// 微信小程序如果是分包页面，需要延迟大概 7s 以上，保证可以正确获取page对象
await page.waitFor(7000)
page = await program.currentPage()
```

2. 微信小程序 element 不能跨组件选择元素，首先要先获取当前组件，在继续查找

```html
<uni-tag>
  <view class="test"></view>
</uni-tag>
```

```javascript
// 错误，取不到元素
await page.$('.test')

// 可以取到元素
let tag = await page.$('uni-tag')
await tag.$('.test')
```

3. 微信小程序暂不支持父子选择器
4. 百度小程序选择元素必须有事件的元素才能被选中，否则提示元素不存在
5. 分包中的页面，打开之后要延迟时间长一点，否者不能正确获取到页面信息


