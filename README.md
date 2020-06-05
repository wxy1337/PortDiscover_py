#端口扫描器Python开发#
##基本信息##

本程序类似于**nmap**,但只实现了其中的TCP握手扫描端口功能
>自己编写的**PortDiscover**可省略其中扫描到开放端口后的继续分析功能
可以在面对较弱防御主机或老版本漏洞主机时，
能较快发现已有服务漏洞端口，从而进行针对该漏洞特定的攻击

##基本功能及实现原理##
基于_python_中的_socket_建立TCP连接扫描端口
其功能是连接TCP套接字，从服务器上读取_banner_
~~再与已有漏洞的服务器版本比较~~
~~本功能实现需要收集大量信息~~
~~而且扫描后进行比较的意义不大~~
~~如果是已知的常见漏洞端口信息~~
~~可以直接百度服务名称得到漏洞信息~~

##网络##
使用_socket_连接指定IP
然后有两张方法收集返回信息和连接信息
>###方法1: connect()和recv(1024)###
	连接成功后用recv(1024)方法读取套接字中接下来的1024B数据
	将这个服务器的响应结果打印
	例如：connect(("192.168.95.148"),21) //端口21是ftp服务标准端口
	结果将ans=s.recv(1024)变量print输出
	220 FreeFloat Ftp Server (Version 1.00)
>###方法2: connect_ex()###
	设置连接时间s.settimeout(timeout)
	result_code = s.connect_ex((ip, port)) //开放放回0
	这种方法在使用多线程方面较为优秀
	而且返回值对于简单端口开放扫描较好
	一般扫描主机开放端口较少不需要太多返回信息
	

##交互界面##
*模仿SQLmap的命令行文本图形界面*
并给出相关的例子和用法

>--Examples:

	py PortDiscover.py -i 192.168.1.1 -p 1000 -t 100
	py PortDiscover.py -u https://www.bilibili.com -p 1-65535

>--Usage:

    py PortDiscover.py
    [-i IP(0.0.0.0-255.255.255.255)]
    [-p port_scope(1-65535)|default=1000]
    [-t thread_num(default = 10)]

    or

    py PortDiscover.py 
    [-u url(http/https)] 
    [-p [port_scope(1-65535)|default=1000]]
    [-t thread_num(default = 10)]

##基本使用##
>###命令行传入参数
使用sys模块，用于解析命令行参数，sys.argv列表中含有所有的命令行参数。

	
	例如：命令行输入 py scanner.py ip
	其中py(python3)为命令行指令，
	scanner.py为sys.argv[0]即脚本名称
	ip为sys.argv[1]第二个参数，传递被扫描主机ip
	

##编程操作中的功能实现与问题解决##
- ###多线程###
>套接字中连接需要一个时间变量timeout，扫描过程中timeout虽然只需要几秒，但扫描端口较多时就会出
现时间长度超乎想象的成倍增加，所以可以基于多线程，一次同时扫描多个套接字，而不是一个一个扫描
>多个线程同时打印输出，就可能出现乱码和失序，此时就有两种方法。
>本质上两种方法实现原理基本一致
>本程序直接采用第二种

		使用一个信号量，阻止其他线程运行，简单来说就是“上锁”，持有信号量的线程有权输出
		使用sys.stdout.write(OPEN_MSG % port)，在使用join()阻塞线程，输出信息
	

- ###ip和域名转化功能###
>使用socket.gethostname(dmoin)方法

		def get_ip_by_name(self, domain):

        domain = (domain.replace("http://","")).replace("https://","")
        print(domain)
        try:
            return socket.gethostbyname(domain)
        except Exception as e:
            print("%s:%s"%(domain, e))

- ###端口命令行参数###
>端口参数sys.argv[4]解析，得到开始端口和结束端口

		def split(self, args):
        port_temp = args[4].split("-")
        start_port = int(port_temp[0])
        end_port = int(port_temp[1])
        return int(start_port), int(end_port)

- ###用法参数解析###

	>`-h`	输出相关的使用方法和例子

	>`-i`	传入被扫描主机ip地址

	>`-p`	端口号指定，可以使用默认的常用端口号，或指定范围

	>`-t`	指定线程数量，以便根据计算机性能，尽可能更快的扫描

	>`-u`	url参数传入，紧跟域名解析功能，

- ###美化界面###
	>使用[patorjk](http://patorjk.com/software/taag/#p=display&f=Doom&t=PortDiscover)网站可以将字母转换为字符画签名

		______          _  ______ _
		| ___ \        | | |  _  (_)
		| |_/ /__  _ __| |_| | | |_ ___  ___ _____   _____ _ __
		|  __/ _ \| '__| __| | | | / __|/ __/ _ \ \ / / _ \ '__|
		| | | (_) | |  | |_| |/ /| \__ \ (_| (_) \ V /  __/ |
		\_|  \___/|_|   \__|___/ |_|___/\___\___/ \_/ \___|_|  By wangxuyang

##总结评价##
	端口扫描，就是对一段端口或指定的端口进行扫描。通过扫描结果可以知道一台计算机上都提供了哪些服务，然后就可以通过所提供的这些服务的己知漏洞就可进行攻击。其原理是当一个主机向远端一个服务器的某一个端口提出建立一个连接的请求，如果对方有此项服务，就会应答，如果对方未安装此项服务时，即使你向相应的端口发出请求，对方仍无应答，利用这个原理，如果对所有熟知端口或自己选定的某个范围内的熟知端口分别建立连接，并记录下远端服务器所给予的应答，便可知道哪些端口是开放的。
	本程序可以实现较快的扫描TCP全连接端口扫描,扫描本局域网中的另一台主机全部端口，大概需要十二十秒钟，但是功能较为单一，需要进一步升级优化，现进已经存在python-nmap可以被导入，导入现有脚本可以实现其他端口扫描类型，功能较全
	没有扫描后的结果自动保存为文件功能，如果忘记端口数，需要重新执行扫描
	编程能力不强，导致对于各种模块接口类型不了解，花费学习模块用法时间较长，且最终实现功能单一，需要改进


