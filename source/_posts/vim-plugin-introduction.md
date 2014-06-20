title: Vim插件推荐
category: 软件
date: 2014-01-18 16:22:17
tags: vim
---

陆陆续续使用Vim也差不多快一年多了，从一开始的几次放弃到现在越用越习惯。在这当中也折腾了许久，最主要的就是Vim中各种各样的插件。
<!-- more -->

对于软件，我[在这里](http://blog.youxu.info/2012/02/02/software-tools-1/)看到一种观点比较有趣。作者把好的软件大致分为两类：**瑞士军刀**和**工具链**。所谓瑞士军刀指的是那些在某些特定领域做的很强大的软件。这类软件不需要折腾又需要折腾。所谓不需要折腾是因为一般这类软件在此领域中都很强悍，你折腾来折腾去最终一定会折腾到这个软件上面，一旦熟练使用了此软件那么其他的类似软件已经完全不需要折腾了。而所谓需要折腾是指通常这类软件都有一个比较陡峭的学习曲线，从不会不习惯到熟练使用的过程就是一个折腾与磨合的过程。

大概Vim就是这样的一个软件，而我磨合的结果就是下面列出的插件。

#[vundle](https://github.com/gmarik/vundle)#
----------
推荐星级：★★★★★

Vim插件管理工具。使用此工具后安装Vim插件会变成一件很简单的事。只需要在vimrc中声明一行类似于这样的代码：
```
Bundle 'kien/ctrlp.vim'
```
然后使用命令：`BundleInstall`。此时vundle会自动安装声明的插件，简单方便。我下面介绍的插件都是使用这种方式安装的。

#[ctrlp.vim](https://github.com/kien/ctrlp.vim)#
----------
推荐星级：★★★★★

<img src="/Images/vimplugin-photo/ctrlp.png" />
这是一款用于模糊查找文件的插件。它可以搜索文件，buffer列表等。我最喜欢的功能是打开MRU列表，即我最近经常使用的文件列表。下面是我对于它的一些配置文件，主要是把快捷键换成F2了。
```
Bundle 'kien/ctrlp.vim'
"MRU default 
let g:ctrlp_map = '<F2>'
let g:ctrlp_regexp = 1
let g:ctrlp_cmd = 'CtrlPMRU'
let g:ctrlp_custom_ignore = '\.git$\|\.hg$\|\.svn$\|.rvm$'
"let g:ctrlp_working_path_mode=0
let g:ctrlp_match_window_bottom=1
let g:ctrlp_max_height=15
let g:ctrlp_match_window_reversed=0
let g:ctrlp_mruf_max=500
let g:ctrlp_follow_symlinks=1
```

#[The-NERD-Commenter](https://github.com/vim-scripts/The-NERD-Commenter)#
----------
推荐星级：★★★★★

代码注释插件。这款插件很强大，基本不用考虑你目前编辑的代码语言，它会自动找到符合该语言的注释方式。我目前使用的c#,python,css,html,js等无一例外。
配置文件：
```
" 这里使用的绑定快捷键同VS
Bundle 'The-NERD-Commenter'
:nmap <C-K><C-C> <leader>c<space>
:imap <C-K><C-C> <Esc><leader>c<space>i
:nmap <C-K><C-U> <leader>c<space>
:imap <C-K><C-U> <Esc><leader>c<space>i
```

#[The-NERD-tree](https://github.com/scrooloose/nerdtree)#
----------
推荐星级：★★★★★

树形文件浏览插件。应该是很出名的一个插件了，在文件目录浏览的时候很有用。
<img src="/Images/vimplugin-photo/nerdtree.png" />
配置文件：
```
Bundle 'The-NERD-tree'
function! NTFinderP()
    "" Check if NERDTree is open
    if exists("t:NERDTreeBufName")
        let s:ntree = bufwinnr(t:NERDTreeBufName)
    else
        let s:ntree = -1
    endif
    if (s:ntree != -1)
        "" If NERDTree is open, close it.
        :NERDTreeClose
    else
        "" Try to open a :Rtree for the rails project
        if exists(":Rtree")
            "" Open Rtree (using rails plugin, it opens in project dir)
            :Rtree
        else
            "" Open NERDTree in the file path
            :NERDTreeFind
        endif
    endif
endfunction
map <silent> <F11> :call NTFinderP()<CR>
```

#[neocomplcache.vim](https://github.com/Shougo/neocomplcache.vim)#
----------
推荐星级：★★★★★

自动补全插件。我前前后后试过很多补全插件，这款用的时间最长。支持很多种补全方式。具体的截图大家可以去官网看。下面是我的配置：
```
Bundle 'Shougo/neocomplcache.vim'
"自动完成映射为Ctrl+J
imap <C-J> <C-X><C-u>
" Use neocomplcache.
let g:neocomplcache_enable_at_startup = 1
let g:neocomplcache_enable_smart_case = 1
" Use camel case completion.
let g:neocomplcache_enable_camel_case_completion = 1
" Use underscore completion.
let g:neocomplcache_enable_underbar_completion = 1
" Sets minimum char length of syntax keyword.
let g:neocomplcache_min_syntax_length = 1
let g:neocomplcache_enable_auto_close_preview = 0
"let g:neocomplcache_force_overwrite_completefunc = 1
" AutoComplPop like behavior.
let g:neocomplcache_enable_auto_select = 1
" <CR>: close popup and save indent.
inoremap <expr><CR> pumvisible() ? neocomplcache#close_popup() : "\<CR>"
inoremap <expr>.  neocomplcache#close_popup() . "."
inoremap <expr>(  neocomplcache#close_popup() . "("
inoremap <expr>)  neocomplcache#close_popup() . ")"
inoremap <expr><space>  neocomplcache#close_popup() . " "
inoremap <expr>;  neocomplcache#close_popup() . ";"
" <TAB>: completion.
inoremap <expr><TAB>  pumvisible() ? "\<C-n>" : "\<TAB>"
" <C-h>, <BS>: close popup and delete backword char.
inoremap <expr><C-h> neocomplcache#smart_close_popup()."\<C-h>"
inoremap <expr><BS> neocomplcache#smart_close_popup()."\<C-h>"
inoremap <expr><C-y>  neocomplcache#close_popup()
inoremap <expr><C-e>  neocomplcache#cancel_popup()
inoremap <expr><ESC> pumvisible() ? neocomplcache#cancel_popup() : "\<esc>"

autocmd FileType css setlocal omnifunc=csscomplete#CompleteCSS
autocmd FileType html,markdown setlocal omnifunc=htmlcomplete#CompleteTags
autocmd FileType javascript setlocal omnifunc=javascriptcomplete#CompleteJS
autocmd FileType python setlocal omnifunc=pythoncomplete#Complete
autocmd FileType xml setlocal omnifunc=xmlcomplete#CompleteTags
" Enable heavy omni completion, which require computational power and may stall the vim. 
if !exists('g:neocomplcache_omni_patterns')
  let g:neocomplcache_omni_patterns = {}
endif
let g:neocomplcache_omni_patterns.cs = '.*'
```

#[tagbar](https://github.com/majutsushi/tagbar)#
----------
推荐星级：★★★★

浏览代码tag的插件。所谓tag指的是那些方法名，类名的定义。有人可能觉得应该评5星，但是我在实际使用的过程中并不是十分依赖他，基本都喜欢ctrl+F搜索解决了。
<img src="/Images/vimplugin-photo/tagbar.png" />
配置文件：
 ```
Bundle 'majutsushi/tagbar'
"js support:
"   1. git clone --recursive https://github.com/mozilla/doctorjs.github
"   2. add tool/jsctags.bat to path (need to change some variables inside it)
nmap <silent><F10> :TagbarToggle<CR>
 ```


#[vim-powerline](https://github.com/Lokaltog/vim-powerline)#
----------
推荐星级：★★★★

状态栏美化插件。使用此插件后B格立马提升，推荐使用。需要注意的是为了一些特殊字体的显示，需要安装补丁字体，可以到这边[下载安装](https://github.com/qianlifeng/SQVIM/tree/master/Tools)。
<img src="https://github-camo.global.ssl.fastly.net/63f9947cac196ec7e6e3d790fd3cd1e1463a7b9b/687474703a2f2f692e696d6775722e636f6d2f4d737549422e706e67" />
配置文件：
```
Bundle 'Lokaltog/vim-powerline'
Bundle 'eugeneching/consolas-powerline-vim'
"字体设置,先到到tool下安装consolas字体
set guifont=Consolas\ for\ Powerline\ FixedD:h11
set laststatus=2
set encoding=utf-8
set t_Co=256
let g:Powerline_symbols = 'fancy'
```

#[syntastic](https://github.com/scrooloose/syntastic)#
----------
推荐星级：★★★★

<img src="https://raw.github.com/scrooloose/syntastic/master/_assets/screenshot_1.png" />
实时语法检查插件。支持包括python,css,javascript等等各种语言了。安装此插件后，vim有可能变慢。各位酌情使用。


#[matchit.zip](https://github.com/vim-scripts/matchit.zip)#
----------
推荐星级：★★★

这个插件是用来匹配成对出现的标签的插件。举个例子，html代码中经常有成对出现的代码，html，body等等标签都是成对出现的。安装此插件后，在<html>上按%后会自动跳转到</html>上面。当然括号匹配也不在话下。


#[surround.vim](https://github.com/tpope/vim-surround)#
----------
推荐星级：★★★

<img src="/Images/vimplugin-photo/surround.gif" />
这个插件是用来匹配成对出现的标签的插件。举个例子，html代码中经常有成对出现的代码，html，body等等标签都是成对出现的。安装此插件后，在`<html>`上按%后会自动跳转到`</html>`上面。当然括号匹配也不在话下。


#[vim-numbertoggle](https://github.com/jeffkreeftmeijer/vim-numbertoggle)#
----------
推荐星级：★★★

<img src="/Images/vimplugin-photo/vim-numbertoggle.png" />
相对行号插件。Vim中的相对行号在上下跳转的时候非常有用，免去了计算的麻烦。而这款插件可以在插入模式的时候显示绝对行号，而正常模式的时候显示相对行号。


#[rainbow_parentheses.vim](https://github.com/kien/rainbow_parentheses.vim)#
----------
推荐星级：★★★

彩虹括号。这个名字很有意思啊，安装此插件后所有的括号都会使用不同的颜色来显示。效果图：
<img src="/Images/vimplugin-photo/rainbow_parentheses.png" />
配置文件：
```
Bundle 'kien/rainbow_parentheses.vim'
let g:rbpt_colorpairs = [
            \ ['brown',       'RoyalBlue3'],
            \ ['Darkblue',    'SeaGreen3'],
            \ ['darkgray',    'DarkOrchid3'],
            \ ['darkgreen',   'firebrick3'],
            \ ['darkcyan',    'RoyalBlue3'],
            \ ['darkred',     'SeaGreen3'],
            \ ['darkmagenta', 'DarkOrchid3'],
            \ ['brown',       'firebrick3'],
            \ ['gray',        'RoyalBlue3'],
            \ ['black',       'SeaGreen3'],
            \ ['darkmagenta', 'DarkOrchid3'],
            \ ['Darkblue',    'firebrick3'],
            \ ['darkgreen',   'RoyalBlue3'],
            \ ['darkcyan',    'SeaGreen3'],
            \ ['darkred',     'DarkOrchid3'],
            \ ['red',         'firebrick3'],
            \ ]
let g:rbpt_max = 16
let g:rbpt_loadcmd_toggle = 0
au VimEnter * RainbowParenthesesToggle
au Syntax   * RainbowParenthesesLoadRound
au Syntax   * RainbowParenthesesLoadSquare
au Syntax   * RainbowParenthesesLoadBraces
```
