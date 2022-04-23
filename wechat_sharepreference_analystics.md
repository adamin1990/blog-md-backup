# Android wechat SharedPreferences 分析 
SharedPreferences文件一般存储在/data/data/package name/shared_prefs目录下，微信sharedpreferences文件存储在/data/data/com.tencent.mm/shared_prefs目录下.

### com.tencent.mm_preferences.xml
以下是过滤到的有用信息
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <string name="login_user_name">1895xxxx205</string>  //用户名
    <string name="last_login_bind_mobile">1895xxxx205</string>  //手机号
    <boolean name="isLogin" value="true" />  //是否登录
    <string name="last_avatar_path">/data/user/0/com.tencent.mm/MicroMsg/last_avatar_dir/user_ed886d7be47fa46343e68941d965c5a3.png</string> //头像地址
    <string name="last_login_uin">1687xx301</string>  //uin，唯一标识
    <string name="login_weixin_username">xxxian</string>  //微信号
    <string name="language_key">zh_CN</string>   //语言
    <string name="last_login_bind_qq">1902xxx1</string> //绑定qq
    <string name="last_login_nick_name">xxx器</string>   //微信昵称 
</map>
```