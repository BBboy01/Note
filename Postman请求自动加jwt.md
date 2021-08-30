1. 在登录接口获取得到`token`

![image-20210830181808178](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210830181808178.png)

2. 在`test`中获取到该`token`，并添加到全局中

![image-20210830181914921](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210830181914921.png)

3. 在其他接口的`Authorization`中的`token`中填入该变量

![image-20210830182040467](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210830182040467.png)

---

之后每当重新调用获取`token`的请求后，全局的`token`也会跟着改变，从而达到自动携带`token`

