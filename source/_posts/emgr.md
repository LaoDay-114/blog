---
title: 我搭建了一个自己的电子邮件服务
---

## 前言
这花费了我一天的时间
当然搭建他也很简单，但是为什么我还是花了一天时间呢？
因为我提前帮你们把坑都踩完了）

## 示例
我的电子邮件服务是在 `mail.bitcat.dpdns.org` 你们也是可以去看看的
示例:
```url
https://mail.bitcat.dpdns.org/<发送给谁>/<邮件标题>/<邮件内容 HTML 使用<br>换行>
?key=<使用密钥 获取教程在后面>&user=<身份>
```
例如:
```url
https://mail.bitcat.dpdns.org/924403444@qq.com/test/114<br>514/?key=<密钥我就不告诉你们了>
```
这时候我的QQ邮箱就会收到一份邮件 如下图:
![emgr1](emgr1.png)
当然，每次使用api后都会为你生成一份专属的id:
```json
{
  "id": "388a37cc-d0f3-4233-8783-435ce97d8eaa",
  "meta": {
    "keyUsed": <保护密钥>,
    "from": "bitcat114@bitcat.dpdns.org",
    "user": null,
    "subject": "test"
  }
}
```
如果你觉得手动输入麻烦 请看下面:
## Emgr V1
可以使用我编写的 `Emgr` 工具来使用我的邮件API
<a href="EMgr_V1.exe" download>点我下载 Emgr V1 工具</a>
可以使用 Emgr V1 来更方便的使用我的API发送邮件。
由于篇幅问题，这里就不过多介绍 Emgr V1 工具了。

## 获取密钥
可以从 `QQID: 924403444` 来获取密钥
也就是联系我

## 搭建
> 如果你们也想搭建像我一样的 API 那就看这里，如果你不想搭建的话，那实际上你就看完了
### 需要准备的
- 一个 Cloudflare 账号
- 一个 Resend 账号
- 一个域名（什么域名都行）

我们将会使用 Cloudflare Worker + Resend 来构建属于我们自己的邮件API

### Resend
登入你的Resend控制台 如果你是新账号的话他应该会给你个api key
你按照正常流程新建，之后应该会让你们绑定域名
按照提示给域名添加DNS(不同的人可能不一样 因此我就不给你们图片步骤了)
添加完DNS后点击检测，过一会就添加成功了。
这时候Resend的准备基本上就完成了。
如果你想启用接收邮件，就在域名这里点击启用接收，之后按照提示添加这一个DNS，之后你的域名就可以接收邮件了
你们也可以给bitcat114@bitcat.dpdns.org发邮件，也就是给我发邮件

### Cloudflare Worker
登入 Cloudflare 控制台
点击侧边栏的 `计算` -> `Workers 和 Pages`
点击创建应用程序。
点击 `从 Hello World 开始`创建
创建完后点击 `编辑代码` 按钮
这里我就给你推荐代码：(可以直接复制)
```js
export default {
  async fetch(request) {
    try {
      const url = new URL(request.url);
      const parts = url.pathname.split('/').filter(p => p);
      const user = url.searchParams.get("user") || "";

      if(parts.length < 3) {
        return Response.json({
          error: "参数不足",
          usage: "GET /收件邮箱/标题/html内容?text=纯文本&key=xxx&user=testuser",
          note: "user为可选项；HTML<br>换行，纯文本%0A换行"
        }, { status: 400 });
      }

      const to = decodeURIComponent(parts[0]);
      const subject = decodeURIComponent(parts[1]);
      const htmlRaw = decodeURIComponent(parts[2]);
      let textRaw = url.searchParams.get("text") ?? htmlRaw;

      let finalSubject = subject;
      if(user){
        finalSubject = `${subject} [${user}]`;
      }

      fromEmail = "<这里填啥都行>@<这里是你的域名>";

      const payload = {
        from: fromEmail,
        to: [to],
        subject: finalSubject,
        html: htmlRaw,
        text: textRaw
      };

      const res = await fetch("https://api.resend.com/emails", {
        method: "POST",
        headers: {
          "content-type": "application/json",
          "Authorization": "Bearer <这里是在Resend控制台复制的密钥>"
        },
        body: JSON.stringify(payload)
      });

      const rawText = await res.text();
      let data;
      try {
        data = JSON.parse(rawText);
      } catch (e) {
        data = { error: "非JSON返回", raw: rawText, status: res.status };
      }

      return Response.json({
        ...data,
        meta:{
          from: fromEmail,
          user: user||null,
          subject: finalSubject
        }
      });
    } catch (err) {
      return Response.json({ fatal: true, msg: err.message }, { status: 500 });
    }
  }
}
```
之后你可以在右侧测试，测试满意后点击 `部署` 按钮来保存。
之后你的邮件API就部署完成了！你可以绑定到你在 Cloudflare 的域名，如果你在 Cloudflare 没有域名
那就只能用Worker的ip了
> Worker的ip在国内延迟巨高

可以使用DNS来链接Worker的ip，有可能成功。
好了，本篇完成了。
