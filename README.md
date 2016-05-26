##使用者文档
#ubuntu效果
<img src=https://github.com/xanarry/LanTrans-desktop/blob/master/screenshut_linux.png/>
#windows效果
<img src=https://github.com/xanarry/LanTrans-desktop/blob/master/sreenshut_windows.PNG/>

主程序, Lantans_desktop.py
运行时所需环境： python3.4, PyQT5

#接收文件
使用:
打开软件->选择 接受 ->选择保存路径->点击 等待接收, 然后等待局域网中的发送方即可.
注意:
如果长时间没后得到传输文件的文件, 请确保文件的发送者在线

#发送文件
使用:
  打开软件->点Add file选择要发送的文件(可以是多个文件)->点击搜索主机->等待状态栏提示红色->点 开始 开始传输

#注意事项:
最佳的传输方案接收方先打开客户端, 等待接收, 然后发送方再发送
如果是发送方先开始活动, 发送方会首先向局域网中广播10秒钟的信息, 如果10秒钟内任然没有发现接收方, 请确定接收方在局域网内, 或者网络连通性, 可以不断尝试多次搜索接受者

##已知问题:
传输过程中速度时快时慢， 不知道为什么
文件名中不要有"`"符号, 否则保存文件的文件名会错乱, 严重会导致程序崩溃

##开发者文档:
项目使用python3.4 + PyQt5开发, linux windows跨平台使用

#原理概述:
如何确定接收者?
确定两端用户使用UDP连接, 当用户要发送文件时, (client)会向整个局域网中广播信息(发送的信息是本机的网络主机名), 持续广播10秒, 每隔2秒广播一次, 如果在广播的过程中发现了文件接收者, 那么接收者(server)在收到client发送的信息后, 将传输文件将使用的[端口号]发送给文件发送者(client), 接着接收方就打开TCP服务端等待发送方client发起TCP连接请求, 建立TCP连接后, 即可开始相互传输文件

如果传输文件?
在发送方(TCP client)与接收方(TCP server)建立了TCP连接之后:

#发送方
将本次要发送的文件的描述信息发给接收方. 用于接收方通过信息生成GUI传输列表信息描述信息格式要求:同一个文件的文件名和长度用~隔开, 文件之间使用|隔开.
格式如下:

    字符串<文件1名~文件1长度|文件2名~文件2长度| ...>
    
java代码描述:

    for (File f in files) {
        msg += f.getname() + "~" + file.length + "`";
    }
    
等待接收方的回复, 接收方会将上述的文件描述回发给发送方
进入循环开始发送文件:
a.在发每一个文件之前, 发送方会本次发送文件的描述, 格式如上, 字符串<文件名~文件长度>
b.等待接收方的确认
c.开始文件字节发送, 直到该文件发送完
d.等待接收方发送确认信息, 确保文件完成发送, 进行下一个文件的传输

#接收方
1.接收发送方发的文件(可能是多个)描述, 然后根据文件描述生成GUI文件传输列表
2.进入循环等待发送方发送文件
a.接收本次要接收的文件描述
b.发送确认信息
c.开始接收文件, 直到接收到的字节数和文件描述中的一致
d.发送确认信息, 进行下一个文件的接收

##注意:
1.每次发文件之前都发送文件描述是为了防止连续穿两个文件导致的字节粘滞, 比如说文件1大小为1000字节, 文件2大小为24字节, 程序一次读取1024字节, 如果没有确认信息的话, 发送方发完1000字节后接着发24字节, 然后接收方一次接收了1024字节, 最终导致, 第一个文件错误, 第二个文件字节流丢失

2.如何确认端口号?
程序启动时端口设为65500端口, 发送方和接收方的角色每转换(程序未关闭的情况下发送者变为接收者, 接收者变为发送者)一次, 将端口号减一, 以避免地址已被占用的情况, 因为, 程序放弃一个TCP连接时, 操作系统并不会立即释放这个连接 


