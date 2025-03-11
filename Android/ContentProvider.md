
# 1 什么是ContentProvider，及其使用场景
ContentProvider 的作用是跨应用数据分享提供统一接口，我们知道安卓系统是对外隔离的，要想让它应用能使用自己的数据，需要用ContentProvider  

使用场景：跨应用分享数据，从系统App对外接口读取数据，如通讯录读取联系人

# 2 ContentProvider、ContentResolver、ContentObserver之间的关系
`ContentProvider`:管理数据、提供数据增删查改操作，数据源可以是数据库、文件、XML、网络等  
`ContentResolver`:外部进程可以通过ContentResolver与ContentProvider交互  
`ContentObserver`:监听ContentProvider中数据变化

# 3 ContentProvider的实现原理

# 4 ContentProvider的优点
1. 封装  
提供对外访问数据接口，解耦了底层数据存储方式和细节，使访问更加简单、高效  

外界无需关心数据本身是如何实现、如何变更，只需要按照要求访问数据即可  

2. 提供一种跨进程数据共享方式  

相比于AIDL，更加简单，能够做到跨进程数据增删查改，以及权限控制

# 5 URI是什么
URI即 Uniform Resource Identifier，即统一资源标识符  
作用是ContentProvider唯一标识，URI分为系统预置&自定义，分别对应系统内置数据（如通讯录、日程表）和自定义数据库  

URI = content://com.xxx.xxx/user/id

- content:// 前缀是固定的，表明这是一个ContentProvider的URI  
- com.xxx.xxx 是授权名，必须与AndroidManifest中保持一致
- user和id分别是表名和要查询的id，按需自定义
