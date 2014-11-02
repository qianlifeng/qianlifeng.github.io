title: 跟我学做c#皮肤美化（三）
category: CSharp
date: 2010-04-15 12:58:17
tags: [皮肤]
---
这篇介绍Checkbox控件的制作

<!-- more -->

先看最终的效果图：

![](http://ww1.sinaimg.cn/large/5d7c1fa4gw1elx2mfcjrdj208i08zglq.jpg)
![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2n1f8t4j208k093aa1.jpg)

或许大家已经猜出来我这个checkbox是怎么实现的吧？不错，就是前面的框是一个图片，后面的文字是label。经过前面button的讲解我想有能力的人完全可以单独制作出来。还不熟悉的现在就开始跟我一步一步的来。  

打开上次的项目QLFUI,新建一个名为CheckBox的用户控件。如图

![](http://ww1.sinaimg.cn/large/5d7c1fa4gw1elx2n97kc6j20j20hwdj6.jpg)

同样的，我们先设置一下，使其看起来像一个checkbox。具体设置如下：

CheckBox控件
```
Size:70,13 
MinimumSize:70,13  
BackColor:Transparent
```
然后拖一个picturebox并设置以下属性：
```
Size:13,13  
Location:0,0, 
BackgroundImageLayout:Stretch
```
再拖一个label上来设置以下属性：
```
AutoSize:falseLocation:15,0  
Size:50:13
```
最终应该是这个样子：

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2o2jwlbj204f015gle.jpg) 

好了，样子有了接下来就是编码了！还是一样，先贴代码，然后我一句一句解释：
```
代码

using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Drawing;
using System.Data;
using System.Linq;
using System.Text;
using System.Windows.Forms;
using System.Reflection;
using System.Drawing.Imaging;

namespace QLFUI
{
    public partial class CheckBox : UserControl
    {
        #region 变量

        Image _normal;
        Image _normalmove;
        Image _checkedmove;
        Image _checked;
        bool check;

        #endregion

        #region 属性

        [Category("QLFSkinDll")]
        [Description("控件是否被选中")]
        public bool Checked
        {
            get { return check; }
            set
            {

                if (value == true)
                {
                    pictureBox1.Image = _checked;
                }
                else
                {
                    pictureBox1.Image = _normal;
                }
                check = value;
            }
        }

        [Category("QLFSkinDll")]
        [Description("显示的文字")]
        public string CheckBoxText
        {
            get { return this.label1.Text; }
            set { this.label1.Text = value; }
        }

        #endregion

        #region 构造函数

        public CheckBox()
        {
            this.SetStyle(ControlStyles.AllPaintingInWmPaint | ControlStyles.OptimizedDoubleBuffer, true);

            Bitmap nl = new Bitmap(Image.FromStream(Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.checkbox.check.bmp")));
            _normal = nl.Clone(new Rectangle(0, 0, 13, 13), PixelFormat.Format64bppPArgb);
            _normalmove = nl.Clone(new Rectangle(13, 0, 13, 13), PixelFormat.Format64bppPArgb);

            Bitmap n2 = new Bitmap(Image.FromStream(Assembly.GetExecutingAssembly().GetManifestResourceStream(@"QLFUI.Res.checkbox.check2.bmp")));
            _checked = n2.Clone(new Rectangle(0, 0, 13, 13), PixelFormat.Format64bppPArgb);
            _checkedmove = n2.Clone(new Rectangle(13, 0, 13, 13), PixelFormat.Format64bppPArgb);

            InitializeComponent();
            
            pictureBox1.Image = _normal;
           
        }

        #endregion

        #region 事件

        private void pictureBox1_MouseEnter(object sender, EventArgs e)
        {
            if (check == true)
            {
                pictureBox1.Image = _checkedmove;
            }
            else
            {
                pictureBox1.Image = _normalmove;
            }
        }

        private void pictureBox1_MouseLeave(object sender, EventArgs e)
        {
            if (check == true)
            {
                
                pictureBox1.Image = _checked;
            }
            else
            {
                pictureBox1.Image = _normal;
            }
        }

        private void pictureBox1_MouseDown(object sender, MouseEventArgs e)
        {
            if (check == false)
            {
                pictureBox1.Image = _checkedmove;
                check = true;
            }
            else
            {
                pictureBox1.Image = _normalmove;
                check = false;
            }
        }

        private void label1_TextChanged(object sender, EventArgs e)
        {
            label1.Width = label1.Text.Length *((int)this.Font.Size+5);
            this.Width = label1.Width + 20;
        }

        private void CheckBox_SizeChanged(object sender, EventArgs e)
        {
            label1.Width = this.Width - 15;
        }

        #endregion 

    }
}
```
**变量：**  

CheckBox类的开头我定义了四个image的变量用来分别表示选中和未选中状态下鼠标的移进移出的图像！接下来的bool check变量就是代表该控件是否已经被选中（checkbox最重要的属性就是这个了吧？！）。

**属性：**  

接下来，就是2个属性了，我代码里面也注释的很清楚。这里只有两个地方需要解释一下:1, [Description("显示的文字")]是干什么的？看图你就明白了

![](http://ww4.sinaimg.cn/large/5d7c1fa4gw1elx2p90ba0j20cm0dgab8.jpg)

这下明白了吧? 对，就是用来解释你这个属性的（别人用你的控件都不知道什么意思，我们当然要适当的注释一下啦）!第二点要注意的就是，在设置Checked这个属性的时候要注意更换图片，选中和不选中的时候的图片不同！其他的属性我想大家经过button控件的讲解已经能很容易看明白里面的内容了。这里就不再罗嗦了。  

接下来看构造函数。第一句this.SetStyle我就不说了，不懂可以看button里面的讲解。接下来的六句话就是得到四种状态下的图片了，不过得到的好像不是很简单！这是因为我们的原始图片是这样的

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2pn1agoj206h03adfp.jpg)

看见没有，图片是画在一张图片中的，所以我们需要使用bitmap的clone函数将不同矩形区域下的图片给截取出来。这样解释大家应该明白了Clone的意思了吧。剩下的两句不罗嗦了，和button里面的一样。  

接下来该做什么呢，根据制作button控件的经验，我们就该开始编写鼠标的各种事件了，并在这些事件中处理不同的背景。  

事件里面我们一共写了三个事件，与button不同的是这里并没有MouseUp事件，为什么呢？因为不需要，哈哈！开个玩笑，因为背景对于mouseup事件并不需要改变！不过如果你高兴当然可以加上去。事件里面的内容也很简单，就是根据当前check的状态，然后依据鼠标的状态更换picturebox的背景。很简单，我就不多说了。另外要注意的就是把label的对应的事件也与生成的鼠标事件关联起来（怎么关联？直接在label的属性的对应事件里面选择那些个已经存在的事件就行了）

好了，来运行一下看看效果吧！

![](http://ww3.sinaimg.cn/large/5d7c1fa4gw1elx2q7wwo1j20cn0dcq44.jpg) 

运行的好像还不错呢，我们设置的属性都能成功应用。不过，如果我们在其他项目中引用这个控件并将CheckBoxText属性设置的稍微长一些，我们就会发现有一部分文字显示不出来了。这是因为我们的控件的宽度是固定的70个像素，当label的长度更改时超过整个控件长度的部分就会被隐藏。所以我们还需要在label的内容更改时同时改变整个控件的长度以完全显示文字的内容。  

看代码，我们写一个label文字更改时的事件`label1_TextChanged`，当label中的文字改变时就会触发这个事件。事件里面的第一句就是根据文字的个数计算label需要显示的宽度。字体的宽度是用`(int)this.Font.Size+` 5来得到的(这个宽度我测试的基本能得到要求)，得到了label的宽度我们只要将这个宽度加20(图片的宽度)就大概得到了整个控件的宽度，这样文字就能正常显示了。当然为了以防万一我们还要加一个事件(基本用不上，不过保险嘛，呵呵)就是`CheckBox SizeChanged`事件了（见代码），当我们手动拉动checkbox时手动改变label的长度，这样即使上面一个事件打不到要求有些文字没有显示出来，我们只要手动拖一下checkbox的长度那文字肯定就能显示出来了。  

OK,checkbox控件制作完成。  

下一篇就会开始先制作窗体部分。。。敬请期待！
