

**1、全局检索LicenseIndex文件**

一般在`typora`安装目录下的`resources/page-dist/static/js`

```shell
sudo find / -type f -name "*.js" |grep "LicenseIndex"
```

修改`LicenseIndex`开头的`js`文件



**2、修改文件中的hasActivated**

```js
由 hasActivated="true"==e.hasActivated 改为 hasActivated="true"=="true"
```

