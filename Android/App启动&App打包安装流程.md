# 1 应用启动流程  

1. Launcher进程请求AMS  

2. AMS发送创建应用进程请求，Zygote进程接受请求并fork应用进程  

3. App进程通过Binder向AMS（SystemServer）发起attachApplication请求，AMS绑定ApplicationThread  

4. AMS发送启动Activity的请求  

5. ActivityThread的Handler处理启动Activity请求  

# 2 Apk组成和Android打包流程  


# 3 Android的签名机制，签名如何实现，V2相比V1签名机制的改变  



# 4 APK安装流程  

