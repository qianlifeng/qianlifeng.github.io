title: 动态编译C#代码
date: 2012-05-15 15:27
tags: [DotNet]
---

动态编译C#代码的一次尝试

<!--more-->

#需求描述
---------
现在一个页面有4个输入框，每个输入框都绑定有一个验证规则。要求在用户输入完毕后根据每个输入的规则对用户输入进行验证。而且这些验证规则是用户在后台可以动态更改的。如下图所示：
<img src="/Images/validate-user-input-by-dynamic-compling-code/1.png"/>  

#最初的想法
---------
我们最初的想法使用正则表达式实现。后台数据库中只要存储验证的正则表达式就行了。需要验证的时候，从后台数据库中取得对应的正则表达式即可。
想法很好，就是真正到了实现的时候有点问题。比如上图中的第一个要求如果使用正则表达式来实现就会比较复杂，在网上查了一些资料也没有发现比较通用的能够检测数字范围的正则表达式。最要命的是上面第一个是可以更改的，如果用户突然哪一天更改为：>34.5 && < 999999.9 那这个正则又该怎么写呢？目前还没有找到一个比较通用的验证数字范围的正则表达式。所以这种不可行。  
 
不过上面有一点肯定的是，将验证规则提取到数据库中达到可配置这个是一定的，只是现在缺少一种行之有效并且简单的表达式。表达式啊表达式...突然想到lambda表达式不也是一种表达式么，如果如果让他来进行类似于范围判断的话也会简单很多。例如，如果要进行上面的范围判断可以写成：o=> o >100 && o < 499，比正则表达式不知道高到哪里去了。
现在需要考虑的问题就是如何让后台代码从数据库中取出这段lambda表达式，并且能够成功执行返回结果。自然而然的想到了动态编译这个词，只要能够动态编译这些代码片段，那么就能执行并返回结果了。

#动态编译c#代码
-------------
初步的想法有了，下面就是验证其是否可行了。google了一番动态编译c#代码，在博客园园里面搜索到这篇文章[http://www.cnblogs.com/jailu/archive/2007/07/22/827058.html](http://www.cnblogs.com/jailu/archive/2007/07/22/827058.html)。
很详细的教程，在这里谢谢作者。作者给出的代码显然还不能直接用到我们这里。我们需要的接口应该是这样的`bool Validate(string lambda, string userInput);` 第一个参数是从数据库取出的lambda表达式字符串，第二个参数是用户输入的需要进行验证的值。
```
static bool validateInput(string lambda, string input)
{
    // 1.CSharpCodePrivoder
    CSharpCodeProvider objCSharpCodePrivoder = new CSharpCodeProvider(new Dictionary<string, string>() { { "CompilerVersion", "v3.5" } });
    // 2.ICodeComplier
    ICodeCompiler objICodeCompiler = objCSharpCodePrivoder.CreateCompiler();
    // 3.CompilerParameters
    CompilerParameters objCompilerParameters = new CompilerParameters();
    objCompilerParameters.ReferencedAssemblies.Add("System.dll");
    objCompilerParameters.ReferencedAssemblies.Add("System.Core.dll");
    objCompilerParameters.ReferencedAssemblies.Add("System.Xml.Linq.dll");
    objCompilerParameters.GenerateExecutable = false;
    objCompilerParameters.GenerateInMemory = false;
    // 4.CompilerResults
    CompilerResults cr = objICodeCompiler.CompileAssemblyFromSource(objCompilerParameters, GenerateCode(lambda, input));
 
    if (cr.Errors.HasErrors)
    {
        Console.WriteLine("编译错误：");
        foreach (CompilerError err in cr.Errors)
        {
            Console.WriteLine(err.ErrorText);
        }
    }
    else
    {
        Assembly objAssembly = cr.CompiledAssembly;
        object objHelloWorld = objAssembly.CreateInstance("DynamicCodeGenerate.DynamicValidate");
        MethodInfo objMI = objHelloWorld.GetType().GetMethod("Validate");
 
        return (bool)objMI.Invoke(objHelloWorld, null);
    }
 
    return false;
 
}
```
通过CompileAssemblyFromSource这个方法就能动态编译代码了。因为需要用到lambda表达式，所以在创建CSharpCodeProvider的时候指定编译器的版本为3.5的。编译完了之后，可以通过反射进行调用相应的动态编译的方法。在本例中，我们需要重点注意的还是怎么样构造一个可以执行lambda表达式的代码片段。
```
static string GenerateCode(string lambda, string input)
{
    StringBuilder sb = new StringBuilder();
    sb.Append("using System;");
    sb.Append(Environment.NewLine);
    sb.Append(Environment.NewLine);
    sb.Append("namespace DynamicCodeGenerate");
    sb.Append(Environment.NewLine);
    sb.Append("{");
    sb.Append(Environment.NewLine);
    sb.Append("    public class DynamicValidate");
    sb.Append(Environment.NewLine);
    sb.Append("    {");
    sb.Append(Environment.NewLine);
    sb.Append("        public delegate TResult Func<T, TResult>(T arg);");
    sb.Append(Environment.NewLine);
    sb.Append("        public bool Validates(Func<string, bool> f)");
    sb.Append(Environment.NewLine);
    sb.Append("        {");
    sb.Append(Environment.NewLine);
    sb.Append("              return f(\"" + input + "\");");
    sb.Append(Environment.NewLine);
    sb.Append("        }");
    sb.Append(Environment.NewLine);
    sb.Append("        public bool Validate()");
    sb.Append(Environment.NewLine);
    sb.Append("        {");
    sb.Append(Environment.NewLine);
    sb.Append("              return Validates(" + lambda + ");");
    sb.Append(Environment.NewLine);
    sb.Append("        }");
    sb.Append(Environment.NewLine);
    sb.Append("    }");
    sb.Append(Environment.NewLine);
    sb.Append("}");
 
    string code = sb.ToString();
    Console.WriteLine(code);
    Console.WriteLine();
 
    return code;
}
```
可以看到，我声明了一个Validates方法，这个方法接受一个委托Func。输入是字符串，返回值是bool。然后再声明一个Vlidate调用Validates方法并传入lambda。通过这样的方式就可以成功动态编译并执行我们指定的lambda表达式了。

#关于效率
---------
这段代码既用到动态编译，又用到反射效率上肯定不是太高。不过灵活性和效率向来就很难兼得。如果还有其他更好的兼得的方法欢迎不吝赐教！
