---
title: 一个httpwebrequest异步下载的例子
date: 2010-04-23 09:47:17
---

最近经常要用到模拟网页的登录，网页的抓取。一开始都是用同步的方法获取，同步在请求量比较小的情况下还可以接受。但是比如我一下子请求上百个，上千个页面就显得力不从心了。昨天终于狠下心来研究了一下异步的获取方式。虽说没有同步的方法简单，不过效率上真的提高了很多。现在记下来，以便以后查阅吧。

```
namespace 异步调用方法
{
    class Program
    {

        static void Main(string[] args)
        {
            for (int i = 0; i < 1111; i++)
            {
                HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://www.cnblogs.com/qianlifeng/");
                request.BeginGetResponse(new AsyncCallback(OnResponse), request);
                Console.WriteLine("请求" + i + "完成");
            }
            Console.Read();
        }

        static void OnResponse(IAsyncResult ar)
        {
            HttpWebRequest request = (HttpWebRequest)ar.AsyncState;
            HttpWebResponse response = (HttpWebResponse)request.EndGetResponse(ar);
            Stream stream = response.GetResponseStream();
            StreamReader sr = new StreamReader(stream, Encoding.UTF8);
            Console.WriteLine( sr.ReadToEnd());
        }
    }
}
```
