---
title: vim
---

<head>
<link rel='stylesheet' href='/style/github2.css'/>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>

我的轻度定制vim
=============

作者：[gaopenghigh](http://gaopenghigh.github.com)
，转载请注明出处。
[（原文地址）](http://gaopenghigh.github.io/posts/vim.html)

vim的学习曲线又陡又长，但它的功能和可定制性实在太强大，一个熟练的用户加上一个高度定制化的vim能够达到很高的效率，在加上熟练地运用vim的各种高级功能是很酷的一件事，于是我决定近期内不再尝试其它的编辑器，逐步地打造适合自己的vim。下面记录的，都是我自己觉得很有用的，或者是容易忘记的，这篇文章的内容也会是逐步丰富的。 

### 技巧们：

#### 快捷键
* `gd`跳到变量声明的地方`<Ctrl> + ]`跳到定义的地方，需要ctags事先生成tag文件
* `<Ctrl> + o`返回之前的位置
* `5 + <Ctrl> + ^`跳到第5号buffer
* `<Ctrl> + PgUp/PgDn`在tab间跳
* `:ls`列出buffer
* `<Ctrl> + g`显示当前编辑文件中当前光标所在行位置以及文件状态信息
* `:r FILENAME`向当前文件中插入另外的文件的内容
* `J`把两行连起来
* `f/F`单字符查找命令，最有用的移动命令之一，"fx" 命令向前查找本行中的字符 x。"F" 命令则用于向左查找。
* `tx`命令与`fx`相似，但它只把光标移动到目标字符的前一个字符上。
* `H,M,L`分别代表移到当前视野的Home, Middle, Last处
* `:qall`全部退出
* `:wqall`全部保存退出


#### vim中的替换:

`%`表示全文匹配:

* `s/old/new/g`当前行中所有old替换成new
* `:%s/old/new/`表示将全文中old替换成new，但每行只替换第一个单词
* `:%s/old/new/g`表示将全文中所有出现过的old替换成new (所有都替换)
* `%s/old/new/gc`全文替换, 替换前询问

`d`表示删除：

* `g/china/d`
* `:g/string/d` 删除所有包含`string`的行
* `:v/string/d` 删除所有不包含`string`的行


我的vimrc如下： 

    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " => 全局设置
    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " Turn backup off
    set nobackup
    set nowb
    set noswapfile
    
    set autoread        " Set to auto read when a file is changed from the outside
    set hid             "Change buffer - without saving
    
    " map leader键设置
    " With a map leader it's possible to do extra key combinations
    " like <leader>w saves the current file
    let mapleader = ","
    let g:mapleader = ","
    
    set showcmd         " Show (partial) command in status line.
    set showmatch       " Show matching brackets.
    "set ignorecase     " Do case insensitive matching
    set linebreak       " 整词换行
    set smartcase       " Do smart case matching
    set incsearch        " 输入字符串就显示匹配点
    set hlsearch        " high light search results
    set autowrite       " 自动把内容写回文件
    "set hidden         " Hide buffers when they are abandoned
    "set mouse=a        " Enable mouse usage (all modes)
    set nu
    
    "--状态行设置--
    set title           " show title in console title bar
    " 总显示最后一个窗口的状态行
    " 设为1则窗口数多于一个的时候显示最后一个窗口的状态行
    " 设为0不显示最后一个窗口的状态行
    set laststatus=2
    
    " 标尺，用于显示光标位置的行号和列号，逗号分隔。
    " 每个窗口都有自己的标尺。
    " 如果窗口有状态行，标尺在那里显示。否则，它显示在屏幕的最后一行上。
    set ruler
    
    " 编码设置
    set fenc=utf-8
    set fencs=utf-8,usc-bom,euc-jp,gb18030,gbk,gb2312,cp936
    
    " 重新打开一个文件时跳到上一次编辑的地方
    " Uncomment the following to have Vim jump to the last position when                                                       
    " " reopening a file
    if has("autocmd")
       au BufReadPost * if line("'\"") > 0 && line("'\"") <= line("$")
        \| exe "normal! g'\"" | endif
    endif
    
    
    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " => 界面设置
    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    runtime! debian.vim
    syntax on
    set background=dark
    "设置配色方案，vim自带的配色方案保存在/usr/share/vim/vim73/colors目录
    colorscheme default 
    " Python 的关键字设置
    let python_highlight_all = 1
    au FileType python syn keyword pythonDecorator True None False self
    " 列数为80的地方高亮显示
    if exists('+colorcolumn')
        set colorcolumn=80
    else
        au BufWinEnter * let w:m2=matchadd('ErrorMsg', '\%>80v.\+', -1)
    endif
    
    
    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " => 格式设置tabs and indent
    """"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    set tabstop=4
    set shiftwidth=4
    set smarttab
    set expandtab       "输入:re可以把tab替换为空格
    set autoindent
    set ai              "Auto indent
    set si              "Smart indet
    set wrap            "Wrap lines
    
    " 删除末尾的空格，对python等很有用
    func! DeleteTrailingWS()
      exe "normal mz"
      %s/\s\+$//ge
      exe "normal `z"
    endfunc
    
    autocmd BufWrite *.py,*.t2t,*.sh :call DeleteTrailingWS()
    
    """""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " => 在tabs和windows之间移动
    """""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    
    " Tab设置, <leader> 已经被设为','
    map <leader>tn :tabnew<cr>
    map <leader>te :tabedit
    map <leader>tc :tabclose<cr>
    map <leader>tm :tabmove
    
    " 按<F2>在新tab中编辑文件, 注意下一行末尾是有个空格的:)
    nnoremap <F2> :tabedit
    
    " 按 ,<Tab> 和 ,` 移动到下一个/上一个tab
    set switchbuf=usetab
    nnoremap <leader><Tab> :sbnext<CR>
    nnoremap <leader>` :sbprevious<CR>
    
    " 按 ,1 ,2 ,3等跳到相应的tab
    map <leader>1 1gt
    map <leader>2 2gt
    map <leader>3 3gt
    map <leader>4 4gt
    map <leader>5 5gt
    map <leader>6 6gt
    map <leader>7 7gt
    map <leader>8 8gt
    map <leader>9 9gt
    
    
    """""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " => Cope, 还不太理解怎么用这个东东
    """""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
    " Do :help cope if you are unsure what cope is. It's super useful!
    map <leader>cc :botright cope<cr>
    map <leader>n :cn<cr>
    map <leader>p :cp<cr>
    
    """"""""""""""""""""""""""""""
    " => 插件设置
    """"""""""""""""""""""""""""""
    
    set autoindent
    
    " 用vundle来管理插件
    set nocompatible
    filetype off " required!
    
    set rtp+=~/.vim/vundle.git/
    call vundle#rc()
    
    " Use Vundle to Manage Vundle
    Bundle 'gmarik/vundle'
    " 安装的插件 
    Bundle 'genutils'
    Bundle 'taglist.vim'
    Bundle 'TaskList.vim'
    Bundle 'django.vim'
    Bundle 'jQuery'
    Bundle 'a-new-txt2tags-syntax'
    Bundle 'python.vim'
    Bundle 'Syntastic'
    Bundle 'pyflakes'
    Bundle 'L9'
    Bundle 'FuzzyFinder'
    Bundle 'vim-powerline'
    Bundle 'c.vim'
    
    filetype plugin indent on
    
    " Syntastic
    "let g:syntastic_python_checker = 'pylint'
    "let g:syntastic_python_checker_args = '--rcfile /etc/pylint.conf -d C0301'
    "let g:syntastic_quiet_warnings=1
    
    " NERDTree, 这个插件没法用vundle安装
    let NERDTreeShowBookmarks = 1
    " 按F3打开文件导航窗口
    map <silent> <F3> :NERDTreeFind<cr>
    
    " FuzzyFinder
    map <leader>F :FufFile<CR>
    map <leader>f :FufTaggedFile<CR>
    map <leader>g :FufTag<CR>
    map <leader>b :FufBuffer<CR>
    
    " txt2tags
    au BufNewFile,BufRead *.t2t set ft=txt2tags
    
    "  ctags 和 taglist
    " 按下F4重新生成tag文件，并更新taglist
    map <F4> :!/usr/bin/ctags-exuberant -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR><CR> 
                \:TlistUpdate<CR>
    imap <F4> <ESC>:!/usr/bin/ctags-exuberant -R --c++-kinds=+p --fields=+iaS --extra=+q .<CR><CR> 
                \:TlistUpdate<CR>
    set tags=tags
    set tags+=./tags "add current directory's generated tags file
    let Tlist_Ctags_Cmd = '/usr/bin/ctags-exuberant'
    let Tlist_Show_One_File = 1            "不同时显示多个文件的tag，只显示当前文件的
    let Tlist_Exit_OnlyWindow = 1          "如果taglist窗口是最后一个窗口，则退出vim
    let Tlist_Show_One_File=0              "让taglist可以同时展示多个文件的函数列表
    let Tlist_File_Fold_Auto_Close=1       "非当前文件，函数列表折叠隐藏
    let Tlist_Use_Right_Window = 1         "在右侧窗口中显示taglist窗口
    let Tlist_Process_File_Always = 1      "aglist始终解析文件中的tag，不管taglist窗口有没有打开
    " 用 F9 来打开/关闭taglist页面
    map <silent> <F9> :TlistToggle<cr>
    
    " pyflakes
    " map <silent> <F7> :call pyflakes()<CR>
    
    " python.vim
    " Shortcuts:
    "   ]t      -- Jump to beginning of block
    "   ]e      -- Jump to end of block
    "   ]v      -- Select (Visual Line Mode) block
    "   ]<      -- Shift block to left
    "   ]>      -- Shift block to right
    "   ]#      -- Comment selection
    "   ]u      -- Uncomment selection
    "   ]c      -- Select current/previous class
    "   ]d      -- Select current/previous function
    "   ]<up>   -- Jump to previous line with the same/lower indentation
    "   ]<down> -- Jump to next line with the same/lower indentation
    
    " TaskList.vim
    " <leader>t打开TODO的list window
    "
    " powerline{
    "set guifont=PowerlineSymbols\ for\ Powerline
    " set nocompatible
    set t_Co=256
    let g:Powerline_symbols = 'unicode'
    "}
    
    """""""""""""""""""""""""""""""""""""""""""""
    " =>自动运行这个文件(python, bash, lua, perl)
    """""""""""""""""""""""""""""""""""""""""""""
    
    " 写python或shell时经常需要做单元测试, 设置按下<F10>就用相应的解释器运行这个文件
    map <F10> :call AutoRun(input('argv : '))<cr>
    
    func AutoRun(par)
        let par = a:par
        exec "w"
        if &filetype == 'sh'

----

<div id="disqus_thread"></div>
<script type="text/javascript">
/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = 'gaopenghigh'; // required: replace example with your forum shortname

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
<script>
  (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
  (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
  m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
  })(window,document,'script','//www.google-analytics.com/analytics.js','ga');

  ga('create', 'UA-40539766-1', 'github.com');
  ga('send', 'pageview');

</script>
