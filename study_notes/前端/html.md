# html简介

- html（**H**yper**T**ext **M**arkup **L**anguage）：超文本标记语言
  - 超文本：除了普通文本，还可以定义图片、视频、音频信息
  - 标记语言：由标签构成的语言
- html运行在浏览器上，由浏览器来解析
- W3C标准：网页由三部分组成
  - 结构：html
  - 表现：css
  - 行为：JavaScript

​		

# html标签

```html
<!DOCTYPE html>
<html>

<head>
    <meta charset="UTF-8">
    <title>网页标题</title>
</head>

<body>
    <h1>一级标题</h1>
    <p style="color:red">Hello world</p>
    <b>加粗字体</b> <br>
    <i>斜体</i> <br>
    <u>下划线</u>
    <img src="../Resource/imag/恐龙.png">
    <audio src="../Resource/audio/路过人间.mp3" controls></audio>
    <video src="../Resource/video/快速排序" controls></video>
    <a href="http://www.baidu.com" target="_self">点击跳转</a>
    
    <form action="#" method="post">
        <p><label for="username">用户名：</label><input type="text" name="username" id="username" placeholder="请输入用户名"></p>
        <p><label for="pwd">密码：</label><input type="password" name="pwd" id="pwd"></p>

        <p>
            性别：
            <input type="radio" name="gender" value="男" id="male"> <label for="male">男</label>
            <input type="radio" name="gender" value="女" id="female"> <label for="female">女</label>
        </p>
        <p>爱好
            <input type="checkbox" value="smoking" name="hobby">抽烟
            <input type="checkbox" value="drinking" name="hobby">喝酒
            <input type="checkbox" value="perming" name="hobby" disabled>烫头
        </p>

        <p>
            <select name="country">
                <option value="">China</option>
                <option value="">America</option>
                <option value="" selected>India</option>
            </select>
        </p>

    <!--文本域-->
        <p>
            <textarea name="textarea" id="" cols="30" rows="10">text</textarea>
        </p>

    <!--文件域-->
        <p>
            <input type="file" name="files">
            <input type="button" value="上传">
        </p>

    <!--输入验证-->
        <p>邮箱:
            <input type="email" name="email">
        </p>

        <p>url:
            <input type="url" name="url">

        <p>数字:
            <input type="number" max="100">
        </p>

        <p>音量:
            <input type="range" min="0" max="100" step="3">
        </p>

        <p>搜索:
            <input type="search">
        </p>

        <p>
            <input type="submit">
            <input type="reset">
        </p>

</form>
</body>
</html>
```



