title: 跟我学做c#皮肤美化（二）
category: CSharp
date: 2010-04-13 10:22:17
tags: [皮肤]
---
这篇介绍Button控件的制作

<!-- more -->

先来看看我们最终要做的效果图（分别对应普通、悬停、按下时的状态）：  

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx1xew8pqj20aa01i0sm.jpg)

下面就开始正式做。首先让我们新建一个控件库项目，命名为QLFUI。如图：

![](http://ww4.sinaimg.cn/large/5d7c1fa4gw1elx1y3bnlaj20sd0izwic.jpg)

然后将默认的UserControl1重命名为 Button。接下来，我们就要在这上面来做文章了。先来稍稍设置一下，让这个用户控件看起来更像一个按钮吧！  

Button的
```
Size: 78,30
BackgroundImageLayout:Stretch
```
然后拖一个label控件到这个用户控件上，并设置label1的属性为
```
AutoSize:false ,  
Dock:fill,   
TextAlign:MiddleCenter,    
BackColor: Transparent,     
Font: 宋体, 9pt
```
这几个属性。好了，是不是开始像一个按钮了呢？  

哦，差点忘了最后还要将整个控件（BUTTON）的背景色设置为Trasparent透明色。因为如果不设置成透明色那么透明的图片下面就会显示出button的背景色（默认灰色），不好看。好了，现在样子的已经大概有了，接下来就是编程了。先贴代码，然后我一个一个解释：
```
代码

 [DefaultEvent("Click")]
   public partialclass Button: UserControl
   {
        #region 变量
 
       //三种不同状态下的图片
       Image _normalImage =null;
       Image _moveImage =null;
       Image _downImage =null;
 
        #endregion
 
        #region 属性
 
       [Category("QLFSkinDll")]
       public ImageNormalImage
       {
            get
            {
                return_normalImage;
 
            }
            set
            {
                _normalImage = value;
            }
       }
 
       [Category("QLFSkinDll")]
       public ImageDownImage
       {
            get{ return _downImage; }
            set
            {
                _downImage = value;
            }
       }
 
       [Category("QLFSkinDll")]
       public ImageMoveImage
       {
            get{ return _moveImage; }
            set
            {
                _moveImage = value;
            }
       }
 
       [Category("QLFSkinDll")]
       public stringCaption
       {
            get{ returnthis.label1.Text;}   //控件运行时会自动运行get方法得到值
set
            {
                this.label1.Text= value;
            }
       }
 
        #endregion
 
        #region 构造函数
 
       public Button()
       {
            this.SetStyle(ControlStyles.AllPaintingInWmPaint | ControlStyles.OptimizedDoubleBuffer, true);
            //默认的是自带的图片样式，如果使用该按钮则必须手工指定当前按钮你想要的背景图片
            _normalImage = Image.FromStream(Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.button.btnnomal.bmp"));
            _moveImage = Image.FromStream(Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.button.btnfore.bmp"));
            _downImage = Image.FromStream(Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.button.btndown.bmp"));
            MakeTransparent(_normalImage);
            MakeTransparent(_moveImage);
            MakeTransparent(_downImage);
            InitializeComponent();
            this.BackgroundImage= _normalImage;
       }
 
        #endregion
 
        #region 辅助函数
 
       private voidMakeTransparent(Image image)
       {
            Bitmapbitmap = image as Bitmap;
            bitmap.MakeTransparent(Color.FromArgb(255, 0, 255));
       }
 
        #endregion
 
        #region 事件
 
       private voidlabel1_MouseEnter(object sender, EventArgs e)
       {
            this.BackgroundImage= _moveImage;
       }
 
       private voidlabel1_MouseDown(object sender, MouseEventArgs e)
       {
            this.BackgroundImage= _downImage;
       }
 
       private voidlabel1_MouseLeave(object sender, EventArgs e)
       {
            this.BackgroundImage= _normalImage;
       }
 
       private voidlabel1_MouseUp(object sender, MouseEventArgs e)
       {
            this.BackgroundImage= _normalImage;
       }
 
 
       private voidlabel1_Click(object sender, EventArgs e)
       {
            this.OnClick(e);
       }
 
        #endregion
 
    }
```  

**先看第一句：**`[DefaultEvent("Click")]`，这句话是什么意思呢？我们知道平常我们拖一个按钮后，在设计模式下双击这个按钮就会自动产生这个按钮的Click事件。这个默认的Click事件从哪里来的呢，毫无疑问就是`[DefaultEvent("Click"`)]这一句来设置的啦！这里默认的是Click事件，如果写成`[DefaultEvent("MouseEnter")]`就是MouseEnter事件啦！  

接下来的四句分别定义了按钮三种不同状态下的bitmap。  

下面的四个属性分别是三种不同状态下Image的属性和按钮的文字属性Caption,大家一看应该就明白。具体要解释的就是`[Category("QLFSkinDll")]`。这句话的意思是将这些属性分类，看个图片大家就都明白了:以后在项目中设置属性时，我们定义的属性就自动分类在QLFSkinDll下面了。   

**接下来是构造函数**：public Button(){}中的内容。  

第一句是开始了窗体的双缓冲。双缓冲的意思就是现在内容中将图像画好了然后再显示到界面上去。在c#中图像一多最怕的就是图像闪烁的问题，开启了双缓冲虽说不能完全避免图像闪烁，但起码也能有一定的效果，我们就先开着吧^ ^！  

接下来的三句就是给三个状态的图像赋值了，这里我是把图像嵌入进来了，并没有放置在外部。要应用嵌入的资源分两步走：第一步在项目中点击图片的属性，然后将“生成操作”改为嵌入的资源。  

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2c1rs6zj207q0ietao.jpg)

第二步应用就是我们用到的代码啦：  
```
Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.button.btnnomal.bmp")
```
这句话前面的照写，后面的路径规则是“命名空间+文件夹名+你的文件名+文件名后缀”，当然如果你的文件直接放在项目下就没有文件夹名了。聪明的大家应该明白吧？呵呵！其中我们要用到的文件大家可以从我给的项目例子中复制。接下来的 
```
MakeTransparent(_normalImage);
MakeTransparent(_moveImage);
MakeTransparent(_downImage);
```
三句先不看，可以注释掉，下面会讲解它的作用的。

第八句`InitializeComponent()`是系统自带的初始化控件一些代码，我们不用去管它。最后一句`this.BackgroundImage =_normalImage;`就是设置按钮的其实的图片的样子啦！  

好啦，写了这么久，咱们还是先来看看能运行出个什么样子出来吧！看看目前的状态下，它离我们的目标还差多远！见图：  

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2dq5591j20co0dggmq.jpg)

从图中我们可以看到明显的一个问题，就是按钮的边缘有粉红色。而且如果你也正好做到这里也会发现鼠标经过按钮时，按钮没有任何反应（废话，我们还什么都没做呢）。

**好了，接下来就有目标了，解决这两个问题我们这个按钮美化的就差不多了！**  

先来解决第一个问题，去掉粉红色。我们需要用到Bitmap的一个函数MakeTransparent(Color),这个函数的作用是使指定的颜色对 Bitmap 透明。也就是说我们只要将这个函数的Color设置为我们需要去掉的粉红色不就行了?!  

继续看代码，我们先写一个函数private void MakeTransparent(Imageimage)，这个函数的作用就是将传进来的图片的指定的颜色进行透明处理。函数里就两句话，第一句话先将Image转为Bitmap。第二句话就是进行透明处理，这里的Color.FromArgb(255, 0, 255)就是我们要去的粉红色啦。那什么时候进行去色呢？毫无疑问，当然是在给三种状态赋值后紧接着就去色啦！所以我们在构造函数中的5,6,7句加上透明处理。(倒过来讲这个作用，大家习惯不习惯呢？)好了，现在我们再来运行一下：看图：  

![](http://ww2.sinaimg.cn/large/5d7c1fa4gw1elx2ea9cuzj20cm0dd75e.jpg) 

呵呵，正如预料的那样，粉红色没有了（图中的灰色其实是透明的，只要在另外一个项目中引用一下就知道了），第一个问题解决！  

再来看第二个问题，要实现按钮的变色肯定是跟鼠标的事件相关啦，比如说鼠标一进入按钮的范围之内我们就让按钮的背景改变，按下和离开按钮的时候也响应的改变背景。所以我们用到这四个事件（注意，事件都是label1的事件，因为我们将label覆盖在按钮上，所以我们点击的其实是label1）：MouseEnter，MouseDown, MouseLeave, MouseUp。具体的事件里面执行的代码也很简单，就是更换按钮的背景图片。好了，让我们再运行一下看看效果：  

![](http://ww1.sinaimg.cn/large/5d7c1fa4gw1elx2eosdpaj20cl0db0tt.jpg)

OK!两个问题都解决了，我们的按钮运行的好像也很正常！鼠标进入，移出，按下的时候也会变换背景！但是，如果你另外建一个项目，并双击产生点击事件并编写相应代码时就会发现点击事件没作用了。多么郁闷的一件事情啊，点击事件都没作用了，我们还要这个按钮干嘛啊！别急，让我们来解决它。首先要明白的就是我们在其他项目中引用的这个的button控件的点击事件是整个用户控件的点击事件（还记得我们在整个button类的上方设置了默认的点击事件吗？），而当我们去点击按钮时我们实际点击的只是label1。问题很明了了，我们点击的是label1，设置的却是整个用户控件的事件，当然触发不了事件了。既然问题已经搞明白了，下面就去解决它。继续看代码：  

既然我们点击的是label1那么我们就在label1的点击事件中做文章，如代码所示，我们在label1的点击事件中触发了整个类（this）的onclick事件，然后这个onclick会去找相应的类的click事件，就是我们在其他项目中直接双击的那个事件啦！

OK，至此这个按钮空间的美化就做好了看看效果吧！

![](http://ww1.sinaimg.cn/large/5d7c1fa4gw1elx2fic2y5j20gl0aw74r.jpg)  

 
