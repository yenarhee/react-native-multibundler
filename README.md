# react-native-multibundler
Based on the configuration development of react native's metro bundler to handle subcontracting, it supports iOS and Android, and supports remote loading. Metro bundler is an official packaging tool. The official subcontracting method is more flexible and stable, and more practical and reliable than some methods on the Internet.

Support debug, optional module path or increment id as module id

Metro official: https://facebook.github.io/metro/


Support react native 0.57~0.63.2, because the official metro unpacking is adopted, theoretically future rn versions can be compatible without modification

Both iOS and Android have loaded multiple bundle instances, which are stable and reliable after testing

### Demo instructions：

     1. Enter the project folder: npm install

      2. android: use android studio to open the android project iOS: use xcode to open the iOS project

      3. Run the android or iOS project directly, the jsbundle package has been pre-printed
     
<img src="https://github.com/smallnew/react-native-multibundler/raw/master/imgs/readme/demo-android.gif" width="250" alt="Demo Android"></img>
<img src="https://github.com/smallnew/react-native-multibundler/raw/master/imgs/readme/demo-ios.gif" width="250" alt="Demo iOS"></img>
     

### How to access the original project：
#### android

    1. Copy all code files except the demo folder to the project
    
    2. Customize platformDep.js and platform57.config.js according to your own needs, and determine the js module included in the basic package here
    
    3. Determine your business entry js and buz57.config.js according to your needs, and determine the js code contained in the business package here
    
    4. Packing: Pack according to the packing command given below
    
    5. The UI entry of the business uses the activity inherited from AsyncReactActivity, rewrites getScriptPath and getScriptPathType to determine the business bundle path, and rewrites getMainComponentName to determine the loaded business module
    
    6. Load the basic package in advance as needed. If the basic package is not loaded in advance, AsyncReactActivity will automatically load the basic package code as follows
    
    ReactInstanceManager reactInstanceManager = ((ReactApplication)getApplication()).getReactNativeHost().getReactInstanceManager();
        reactInstanceManager.createReactContextInBackground();//The basic package platform.android.bundle will be loaded first, or not
        
      
    7. Rewrite ReactApplication to return the location of the base package
    
#### iOS

    1. Expose the executeSourceCode method of RCTBridge by adding the RCTBridge in this project to your own project
    
    2～4. same as Android, see above
    
    5. Load the basic package in advance:
    
    jsCodeLocation = [[NSBundle mainBundle] URLForResource:@"platform.ios" withExtension:@"bundle"];
    bridge = [[RCTBridge alloc] initWithBundleURL:jsCodeLocation
                                 moduleProvider:nil
                                  launchOptions:launchOptions];
                                  
    6. Load the business package:
    
    NSURL *jsCodeLocationBuz = [[NSBundle mainBundle] URLForResource:bundleName withExtension:@"bundle"];
      NSError *error = nil;
      NSData *sourceBuz = [NSData dataWithContentsOfFile:jsCodeLocationBuz.path
                                             options:NSDataReadingMappedIfSafe
                                               error:&error];
      [bridge.batchedBridge executeSourceCode:sourceBuz sync:NO];
      
    7. Create RCTRootView and bind business code
    
    RCTRootView* view = [[RCTRootView alloc] initWithBridge:bridge moduleName:moduleName initialProperties:nil];
    

### DEBUG
```
The debug function has been added since version 2.1. The main working principle is: copy the business module code that needs to be debugged into the debug entry file MultiDebugEntry.js, and then load the entry file on the native side for debugging;
Steps for usage:
1. Configure the business entry file DegbugBuzEntrys.json to be debugged, and add the relative path of the business entry js file to the main project in this json array
2. Set the MULTI_DEBUG variable in ScriptLoadUtil.java in Android to true, or set MULTI_DEBUG to true in iOS
3. Execute under the main project directory: node multiDebug.js
4. Start your native app and start debugging

```

### Module ID
```
Since version 2.2, an option to increment index as module id has been added.
Steps for usage:
1. The switch of this option is set to true in the useIndex in the getModulelId.js file to enable the function of incrementing index as moduleId
2. Configure the ModuleIdConfig.json file, the format is the entry file js: initial moduleId, as follows:
{
   "index.js":100000,
   "index2.js":200000,
   "index3.js":300000
}
The moduleId of the basic package is fixed from 0, so the starting value of the moduleId of the business package is recommended to start from 100000 to prevent duplication with the basic
Avoid duplication of the starting moduleId of different business packages
3. Execute the packaging command or use the UI to package, the ID mapping table corresponding to the platformMap.json, indexMap.json and other modules will be generated in the multibundler directory for subsequent module ID incremental packaging
The benefits of using incremental index as moduleId:
1. It is shorter than the path name as the moduleId, reducing the package size
2. The module name is protected after packaging
3. You can use the debug mode to package, that is, in the package command --dev can be true, which is convenient for debugging

```
### Remote bundle loading
```
The remote bundle loading function has been added since v3.0.
Steps for usage:
1. Package the remote package. The remote package is a zip compressed package. The packaging command is the same as that of the ordinary business package.
It is best to modify the directory where the bundles and assets are saved, so that you can directly compress them in a special directory when compressing, and you need to manually compress the bundles after packaging.
When compressing, put the bundle file and assets at the first level (that is, there should be no upper directory in the compressed package)
2. Put the compressed zip package on the network, rewrite AsyncReactActivity (Android) or create a ReactController object (iOS),
Specify the load type as network, and specify the link, module name, and bundle name
3. Start this Activity or Controller to successfully load the remote business package
friendly reminder:
1. By the way, this function solves the "requirements for placing different business packages in different directories", which is also attributed to the new react-native-smartassets
2. The remote bundle loading function does not do md5 verification, this needs to be solved by the developer himself, mainly because md5 mainly needs the information returned by the server.
As a general unpacking open source project, md5 verification will not be provided
3. The rn 0.62 version will have a hot issue after testing, mainly because the newly added LogBox module runsApplication without authorization and causes a crash. The latest RN version 0.63 has no such problem.

```
### js项目结构：

```
.
├── App.js               业务界面1
├── App2.js              业务界面2
├── App3.js              业务界面3
├── LICENSE
├── README.md
├── android              android项目目录
├── app.json 
├── buzDep.json        UI打包中，打业务包的中间产物，这里面包含的是当前业务包的依赖
├── buz.config.js      业务包的打包配置
├── buz-ui.config.js     UI打业务包配置
├── index.js             业务1入口js
├── index2.js            业务2入口js
├── index3.js            业务3入口js
├── ios ios目录
├── multibundler         包含着debug配置和公用方法模块
├── multiDebug.js        debug node命令行工具
├── multiDebugEntry.js   debug生成的rn调试入口，里面拼接着需要调试模块的入口代码     
├── multibundler_cmd.txt 打包命令
├── package.json
├── platform-ui.config.js UI打基础包配置
├── platformDep-ui.js    UI打基础包入口
├── platform.config.js 基础包打包配置
├── platformDep.js       基础包打包入口
├── platform-import.js   UI打包中生成的基础包依赖的模块import代码
└── platformDep.json     UI打包中生成的基础包所依赖模块的配置文件
```
### android目录结构
```
.
├── AndroidManifest.xml
├── assets
│   ├── index.android.bundle   业务包
│   ├── index2.android.bundle  
│   └── platform.android.bundle 基础包
└── java
    └── com
        ├── facebook
        │   └── react
        │       ├── AsyncReactActivity.java 重要！rn业务加载入口，业务activity重写该类
        │       ├── ReactUtil.java
        │       └── bridge
        └── reactnative_multibundler
            ├── FileUtils.java
            ├── ScriptLoadUtil.java
            └── demo   demo目录，集成项目可删除
                ├── Buz1Activity.java
                ├── Buz2Activity.java
                ├── MainActivity.java
                └── MainApplication.java
```




### UI packaging(Now supports mac os，windows)：
     Usage: Download https://github.com/smallnew/RN-MultiBundler-UI, and run it according to the readme in the project
      After selecting the packaging option, click packaging. This method can replace command packaging and help calculate business package dependencies and remove duplicates.
<img src="https://github.com/smallnew/react-native-multibundler/raw/master/imgs/readme/package-ui-demo.png" width="650" alt="Demo Android"></img>


The packaging command is as follows：
## android
### Play android basic package
node ./node_modules/react-native/local-cli/cli.js bundle --platform android --dev false --entry-file platformDep.js --bundle-output ./android/app/src/main/assets/platform.android.bundle --assets-dest android/app/src/main/res/ --config /{Your absolute path}/platform.config.js
### Play android business package
node ./node_modules/react-native/local-cli/cli.js bundle --platform android --dev false --entry-file index.js --bundle-output ./android/app/src/main/assets/index.android.bundle --assets-dest android/app/src/main/res/ --config /{Your absolute path}/buz.config.js


## iOS
### Play iOS basic package
node ./node_modules/react-native/local-cli/cli.js bundle --platform ios --dev false --entry-file platformDep.js --bundle-output ./ios/platform.ios.bundle --assets-dest ./ios/ --config /{Your absolute path}/platform.config.js
### Play iOS business package
node ./node_modules/react-native/local-cli/cli.js bundle --platform ios --dev false --entry-file index.js --bundle-output ./ios/index.ios.bundle --assets-dest ./ios/ --config /{Your absolute path}/buz.config.js
