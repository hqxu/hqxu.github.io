# hqxu.github.io
Xiaoqiu's homepage

小秋的个人Blog
平常写文章很方便呀。 在source/_posts撰写自己的文章即可。然后
hexo s 实时动态预览文章内容， 觉得满意以后：
hexo g 生成静态网页至public， 然后推送到github上:
hexo d

==================
Hexo博客备份 (在新的机器，或另外一台电脑时. 需要安装git, nodejs, nmp, hexo) 
https://blog.itswincer.com/posts/7efd2818/

使用 Hexo 在 GitHub Pages 搭建博客时，博客作为一个单独的 GitHub 仓库存在，但是这个仓库只有生成的静态网页文件，并没有 Hexo 的源文件。
这样一来换电脑或者重装系统后，再想找回源文件就比较麻烦了，这里推荐一种比较完美的方法解决备份问题。

* 创建两个分支：master 和 hexo；
* 将刚刚创建的新分支hexo clone 至本地，将之前的 hexo 文件夹中的 ** _config.yml、themes/、source/、scaffolds/、package.json 和 .gitignore 复制至 WincerChan.github.io 文件夹；
* 将 themes/next/（我用的是 NexT 主题）中的 .git/ 删除，否则无法将主题文件夹 push**（也可以将主题文件夹使用子模块的方式添加到该仓库)；
* 执行 git add、git commit -m ""、git push origin hexo 来提交 hexo 网站源文件；
