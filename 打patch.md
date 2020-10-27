```
我们的patch 一个文件对应一个.git 目录

因为是git diff 出来的 有些兄弟是git format-patch -N 出来的 但是都不会影响 哪怕是diff -ruN 整个目录 那和git diff出来又多大的区别呢

只要不要几个.git的patch放到同一个文件就可以了

所以

我们的patch用起来就是那几个步骤 不用去手动打

1、找到.git 管理目录 比如 /kernel （kernel的代码就是一个.git管理的）

2、cp xxx.patch .git管理目录

3、 patch -p1 < xxx.patch

一共就三个步骤

patch是一个非常强大的工具，比git am 和git applay 都强，你可以试试，哪怕是改了行号，一样可以跳过去打patch

打patch最怕的就是两个

1、内容改了

2、行号改了

比如我在文件头部加一个 #define hehe 1 那么整个文件其它行号都变了，这个时候，你用git am 和git applay 是打不了patch的

只有patch这个指令才打的了 它可以"跳行"

还有一个，假如打patch失败了一部分，你可以查看打patch失败生成的xxx.orig 和 xxx.rej 去自己手动将没有打上的 剩下的段需要改的部分打上

再有一个，如果想回到原来的代码 可以patch -Rp1 < xxx.patch 去掉刚刚打的patch 或者是git stash 就Ok

patch非常重要，因为sdk太多，客户拿的sdk很多没有repo，所以经常改的功能以patch的方法打上，但是经常不会合出各种问题


git format-patch -1 commit（git format-patch -3 commit是生成3个）
git am patch(一个一个打比较好)
git am *.patch(一起打也行)
```
