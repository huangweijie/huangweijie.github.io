title: linux ubuntu安装iNode
date: 2016-05-27 13:04:57
tags: 经验

---

    本来在宿舍只需要使用舍友“外坏”就可以在ubuntu上安心上网的，可惜最近舍友“外坏”开不流畅，而且校园网也炸炸的，那么就决定在ubuntu上装好iNode自己来开网了，装iNode一天游开始了。

在广工网管中心下到了linux版本的iNode包回来解压后，先

    cd iNode解压包目录
    ./install.sh  安装iNode
    ./iNodeClient 运行iNode

接下来就是看它抛出的错误了，我估摸着肯定会少这少那，反正我是。查阅了很多资料，似乎是跟系统64位而这个iNode只是32位应用有关，于是要解决它的库问题。

    sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0 libjpeg62:i386 libpangoxft-1.0-0:i386 libpangox-1.0-0:i386 libsm6:i386 libidn11:i386

这里需要注意i386，i386通常被用来作为对Intel（英特尔）32位微处理器的统称。后续缺少的东西也是要注意加这个，不然没用。
装好这些后基本上没什么问题了，要是还是有缺什么就找什么补什么吧。下面这个问题就神奇了。

    libtiff.so.3: cannot open shared object file: No such file or directory

但是自己查找的时候发现还是有这么个文件的，其实并不是因为缺少了这个文件，而是ubuntu中这个库文件升级了，引用不到。

    cd /usr/lib/i386-linux-gnu
    ls libtiff*

输入以上命令查看本机的libtiff的版本是多少的，我的是5的那就拿5作事例。

    cd /usr/lib/i386-linux-gnu/ cd到这个目录
    sudo rm libtiff.so.3 删除原有的libtiff.so.3
    sudo ln -s /usr/lib/i386-linux-gnu/libtiff.so.5 /usr/lib/i386-linux-gnu/libtiff.so.3 创建一个libtiff.so.3是链接到libtiff.so.5的

然后这块基本就没什么问题了。接下来我们再次

    ./iNodeClient

如果有`Gtk-Message: Failed to load module "canberra-gtk-module"`出现

    sudo apt-get install libcanberra-gtk-module:i386
同样留意i386，如果有类似的出现，查找相关命令下载即可
当执行`./iNodeClient`出现客户端界面就基本上完成安装啦

看到客户端的界面，选择左上角的新建添加链接
![界面][1]
输入用户名和密码，按照下图点击选项，确定保存
![创建连接][2]
新的链接已经建立好了，选中链接，点击上方的“连接”按钮开始连接
![连接][3]
看到“You have passed the identity authentication”的字样时就表示认证成功，就可以上网啦！


  [1]: http://img1.ph.126.net/GZknSWgmRimiJ3kge7FBaA==/647955396405058911.jpg
  [2]: http://img3.ph.126.net/ZT9_EE9UGU5fLlqLSkzCvQ==/993325192829036033.jpg
  [3]: http://img4.ph.126.net/SAP09r3GcPMaNdDDFMyvMg==/2710041075787328544.jpg
