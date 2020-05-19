# Chrome远程调试

## Android Chrome浏览器

1. 手机设置：设置>开发人员工具>USB调试

2. USB线连接手机和电脑

3. 电脑端Chrome浏览器：chrome://inspect页面

![remoteTarget](https://github.com/GrowLegend/MyNote/blob/master/static/images/chrome远程调试/remoteTarget.png)

## iPhone Safari浏览器

### 安装scoope

```powershell
// powershell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

### 安装remotedebug-ios-webkit-adapter

```powershell
scoop bucket add extras
scoop install ios-webkit-debug-proxy
npm install remotedebug-ios-webkit-adapter -g
```

### 安装iTunes：官网下载

### 连接调试：

1. iphone：设置>Safari浏览器>高级>网页检查器

2. 打开iTunes连接iphone。在iPhone上选择信任该电脑

3. 执行命令，启动适配器

```powershell
remotedebug_ios_webkit_adapter --port=9000
```

4. 在iphone中打开Safari浏览器打开待调试页面

5. 电脑端Chrome进入chrome://inspect页面，Discover network targets configure配置localhost:9000后刷新后可见remote Target