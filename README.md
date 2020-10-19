# recycle
# 快速上手
Linux回收站，通过bash shell脚本实现，用于放置系统手残党执行rm命令时造成严重后果，该回收站会将每个用户删除的文件放在不同的文件夹下，以确保内容不会串，并且回收站的权限会严格控制在仅删除用户可见，其它用户无法操作（除root）；第一次执行时会创建/.RECYCLE/.${user}目录作为当前用户的回收站，其中${user}表示当前用户例如root，因为该目录为隐藏目录，所以以"."作为前缀；操作者可以查看这个目录来查看回收站；

若需要恢复某一个文件或目录，执行store 文件/文件夹 ，需要注意的是，这里文件/文件夹是在/.RECYCLE/.${user}中的文件或文件夹，这里暂时没有提供查看列表的方式，用户可以在配置好环境变量是进入到回收站中执行；

当移除一个名称已经在回收站中存在的文件时RO会在移除时为新的文件添加尾缀编号例如（1），而后再添加相同文件时会编号为（2），依此类推；


部署方式：
1. 打开库中的文件夹可以看到所有的命令，其中包括rm和restore命令
2. 将该库中的文件拷贝到指定的目录（自己决定目录的位置）
3. 将/usr/bin下和sbin下rm 命令更名为remove
4. 配置环境变量，使其指向刚才配置的目录（能看到rm和restore文件的目录），修改/etc/profile在尾部添加 export path=$path:/库目录
5. 完成后执行source /etc/profile命令
6. 完成后测试，创建一个文件然后使用rm命令删除，当出现Message:** removed to recycle的消息时，则表示配置成功

# 下面都是执行逻辑
1. 第一次运行时，程序会在/目录创建.RECYCLE文件夹（如果没有），并且同时还会创建一个子文件夹（如果没有），该子文件夹以用户名命名，并且所有权为创建该
目录的用户（首次执行该命令的用户，避免其它用户访问到非自己的回收站）；
2. 除上面的回收站创建以外，同时还会在当前执行用户的家目录下创建.recycleInfo的目录，该目录下记录着回收站的信息，例如下面的.recycleTable.tab文件记录了
所有删除文件/文件夹的名称和原来的位置，当执行restore命令的时候，就会查找该文件中的名称并将符合的文件从回收站移到原来的路径，如果删除的是重复的文件，则回收站并不会将回收站原有的文件覆盖，而是进行编号（文件+后缀编号），所以如果恢复时也需要加编号（用户可以直接去回收站查看，例如root用户可访问/.RECYCLE/root查看回收站的内容）
3. 当然，回收站并不是无穷无尽大的，所以需要使用计划任务进行定时清理，这时可以使用系统rm命令进行清理，不过，在上面我们已经将rm更改为remove了，如果此时执行rm，则相当于把回收站的某个文件再次放到了回收站（可能会更改文件名，）；所以在计划任务中，我们可以在指定的时间将/.RECYCLE/用户名/目录删掉，例如将下面引号中的内容添加到crontab中,表示30天后清空回收站（用户名根据实际用户名替换）
	
"* * */2 * * remove -rf /.RECYCLE/用户名/"
 

【Node】:千万注意高权限的用户往往拥有删除所有用户回收站的权力，所以计划任务的时候千万不能对/.RECYCLE/目录执行remove -rf 操作
【Node】:请不要手动移动文件到回收站目录，手动移入的文件将无法再恢复到原始目录
【Node】:请不要修改用户目录下的.recycleInfo文件夹下的任何文件，那将可能导致你无法恢复回收站中的项目
【Node】:请不要修改回收站目录中的文件/文件夹名称，那将会导致该项目无法恢复到原始位置
【Node】:请不要以任何方式操作上述提到的所有文件，除非你已完整阅读过源代码，并充分理解其中的逻辑
【Node】:若以学习或试图修改源码扩展功能时，请复制程序脚本文件而不是在原文件上修改；新的脚本应该在充分测试之后投入使用
