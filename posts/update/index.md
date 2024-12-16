# 更新博客操作流




```shell
//推到线上
PS D:\blog\my-blog&gt; hugo --theme=FixIt --baseURL=&#34;https://vazurev.github.io&#34;
PS D:\blog\my-blog&gt; cd public
PS D:\blog\my-blog\public&gt; git add .
PS D:\blog\my-blog\public&gt; git commit -m &#34;第五次提交&#34;
PS D:\blog\my-blog\public&gt; git push origin main

//本地调试
hugo server --buildDrafts
```



## git push超时

1. 设置代理

   ```
   git config --global https.proxy
   ```

2. 取消代理

   ```
   git config --global --unset https.proxy
   ```

3. 再次提交



---

> 作者: Azure  
> URL: //localhost:1313/posts/update/  

