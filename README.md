# study_jvm

一、  JVM 内存模型

    1. JVM 内存模型

        ----------------------------------------------------------------------------------------
        |        线程共享区                 |                线程私有区                           |
        |                                  |                                                   |
        |    -----------------             |        ------------------------------             |
        |    |               |             |        |                            |             |
        |    |   堆（Heap）   |             |        |         虚拟机栈            |             |
        |    |               |             |        |                            |             |
        |    -----------------             |        -------------------------------            |
        |                                  |                                                   |
        |    -----------------             |    -----------------           -----------------  |
        |    |               |             |    |               |           |               |  |
        |    |   方法区       |             |    |    程序计数器  |           |    本地方法栈   |  |
        |    |               |             |    |               |           |               |  |
        |    -----------------             |    -----------------           -----------------  |
        |                                  |                                                   |
        |                                  |                                                   |
        ----------------------------------------------------------------------------------------

        线程共享区表示可在多线程并发时多个线程可以访问当前区域的对象。因此，这个区域中的对象存在并发同步的问题。 线程私有区，只能是当前线程才能访问的对象

        堆（Heap）: 存放Java中所有类的实例对象的内存区域，也是唯一一块内存区域。 同时Heap也称为GCHeap, 它是GC重点关顾的区域。 Java虚拟机的Heap的大小可以
                   通过虚拟机参数  -XX:InitialHeapSize=[]与 -XX:MaxHeapSize=[]设置，当Heap无法在存放实例对象，并且Heap无法在拓宽内存是，将抛出OutOfMemoryException.


                  TLAB(Thread Local Allocation Buffer): 本地内存分配缓存区
                        类的实例对象都保存在Heap中， 由于Heap是能够支持多线程并发， 因此在每个线程创建类的实例对象时都需要向Heap申请。
                      Heap 在分配内存的时候，要保证内存分配的唯一性，在分配内存的时候需要进行同步。 在这种情况下， Heap分配内存在效率比较低，吞吐率底。
                      因此JVM引入了TLAB。 TLAB在线程创建的时候会向Heap申请一块内存，线程中所有的类对象的内存申请都有TLAB分配。 只有当TLAB内存不够时，
                      才会向Heap申请内存，Heap这时才同步一次。


        方法区： 方法区同Heap都属于线程共享区并且不需要内存连续。方法区存放着类信息， 常量， 静态变量，编译后的代码数据

            运行时常量池： 方法区的一部分，存放编译期的字面量与引用符号。 字面量在Java源码期对应这字符串常量，final修饰的常量等。而引用符号只待编译后的类的全限定名、描述
                         字段名称与描述与方法全限定名与描述三部分。

                         运行时常量池也是属于方法区的，因此在无法存放新的对象时，也会抛出OutofMemoryExecption

             回收常量池中的对象： 运行时常量池中的对象也是GC 照顾的对象之一， 只是GC要回收运行时常量池中的对象需要对象满足一下条件：
                            a. 运行时常量池的Class已经没有对应的实例对象
                            b. 加载Class的ClassLoader 已经被回收
                            c. Class 没有被反射



        虚拟机栈： JVM 执行方法调用的内存模型的体现。 方法调用时会将方法的相关信息生成一个方法栈针压入虚拟机方法栈中，当方法执行完成后，会将方法栈针弹出虚拟机方法栈。
                  栈针对象存放方法的局部变量表， 操作符， 动态链接以及出口信息。


        本地方法区： Java Native 对应的方法管理内存模型


        程序计数器： 记录当前线程正执行到字节码指令的地址。 由于现行的CPU大部分采用时间片轮训机制分配CPU的使用， 每个线程都会在失去CPU的占用是从执行态转到等待态。 再当线程获得CPU的
                   使用时会恢复到之前执行到的字节码执行的位置，才能够继续执行线程中的指令从而不会出错。


    2. JVM 对象创建流程
        在 new 一个对象时， JVM会经过一下几个步骤创建与初始化对象
            a. 检测虚拟机中是否已经load 类对一个的类信息，如果没有则调用ClassLoader加载
            b. Class加载完成，为对象分配内存空间

                内存分配的方法：

                    指针碰撞：如果Java堆中内存绝对规整，所有用过的内存放在一边，空闲内存放在另一边，中间一个指针作为分界点的指示器，那分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相同的距离。

                    空闲列表：如果Java堆中内存并不规整，那么虚拟机就需要维护一个列表，记录哪些内存块是可用的，以便在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录。

            c. 对内存空间进行初始化，初始化完成后，对象创建完成
            d. 调用对象的init（）

    参考： https://blog.csdn.net/justloveyou_/article/details/71189093

    3. 造成OutofMemoryExecption 有那些
        a. Heap 空间不足， 碎片化严重
        b. 线程数量过多
        c. 文件句柄打开过多
        d. 资源连接池打开过多

    4. JVM中访问定位
        JVM 创建对象就是为了访问对象，Java程序通过栈上的本地变量表的reference数据定位对象。 reference数据对象只是指针对象的引用，并不会保存使用什么方式定位，引用对象的具体位置。 现行流行的访问方法定位采用
        指针定位与句柄定位

            指针定位： reference 存储对象的地址

            句柄定位： JVM堆中会划分一个句柄池， 栈中的reference引用句柄池中的一个句柄， 句柄保存这对象的实例数据和对象数据类型信息各自的地址信息。

            总的来说，这两种对象访问定位方式各有千秋。使用句柄访问的最大好处就是reference中存储的是稳定的句柄地址，对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针，
            reference本身不需要修改；而使用直接指针访问的最大好处就是速度快，节省了一次指针定位的时间开销。


二、 Java GC 机制

    1、 GC的作用与意义
        GC 是垃圾收集器的简称， GC是一个守护进程，负责定期回收JVM 中那些已没有引用的实例对象。
        在C语言等底层语言中，对象的内存分配需要程序员手动申请（构造函数申请内存），对象的内存也需要手动释放内存(析构函数释放内存)，程序员需要手动维护整个过程。整个过程往往存在忘记内存释放，造成
        OutofMemoryExecption。 因此Java语言引入了GC来释放程序员的双手，提高程序员的开发效率。

    2、 垃圾对象的甄别

        2.1 引用计数算法
            对象被引用，引用计数器加一； 对象引用被释放，引用计数器减一
            缺点： 相互依赖后的对象无法回收
                class A {
                    public B b;
                }

                class B {
                    public A a;
                }

                static void main(String[] args) {
                    A a = new A();
                    B b = new B();

                    //对象a, b 相互依赖， 计数器+1，计数值为2
                    a.b = b;
                    b.a = a;

                    //释放a,b 的应用， 计数器-1,  计数值为1
                    a = null;
                    b = null;

                    // 程序执行完成， A、 B 对象的计数值不为零， JVM 判定为不可回收垃圾，从而造成内存泄露
                }


        2.2 可达性分析算法
            从GCRoot开始向下逐层扫描内存中的对象的引用，整个扫描过程会生成一条对象之间的引用链，被称作对象引用链。 如果在扫描中
            发现对象的引用链无法到达GCRoot， JVM会判定当前对象无垃圾对象。

            2.2.1 在JVM中可以作为GCRoot的对象包含
                a. 虚拟机栈中的引用用对象（Thread， ClassLoader）
                b. 方法区中的常量引用的对象
                c. 方法区中的静态变量引用的对象
                d. 本地方法栈引用的对象(将Java对象最为JNI native方法参数)

    3、 垃圾回收算法

        3.1 标记-清除
            a. 标记非垃圾对象
            b. 回收垃圾对象

            缺点： 碎片化严重

        3.2 复制算法
            a. 内存对半分为内存分配区与拷贝区
            b. 将非垃圾对象拷贝到拷贝区， 清空分配区中的所有内存
            c. 将拷贝区与分配区调换

            缺点： 总内存的使用空间减半， 利用率下降

        3.3 标记-整理
            a. 标记非垃圾对象
            b. 从内存的跟节点开始，将第一非垃圾对象移动到内存根节点的下一个内存地址，依次移动移动非垃圾对象。 垃圾对象远离根节点。 （整个过程有点像冒泡排序算法， 非垃圾对象前移， 垃圾对象后移）
            c. 清除垃圾对象

            缺点： 效率底

        3.4 分代算法
            ----------------------------------------------------------
            |                                  |                     |
            |   |    Eden     | sur0 | sur1    |        Old          |
            |                                  |                     |
            ----------------------------------------------------------
            内存分为新生代区与老年代区, 新生代区又分为Eden区与survivor1与survivor2（内存比例8:1:1）。 JVM会先将实例对象存放在Eden区， 当Eden区无法存放下新对象申请的内存时，触发一次YongGC
            YongGC 将Eden与sur0 中的非垃圾对象移动到sur1 中，sur1 与sur2 交换角色，所有的非垃圾对象的分代年龄+1. 并且判断sur区中的对象的总内存是否大于sur区的一半或者是否有分代年龄大于15的对象
            ，满足以上条件的对象将从新生代区移动到老年代区(动态年龄判断)。 这时老年代区会判断是否能够存放从新生代区移动过来的对象， 如果无法存放， 触发一次FullGC 回收

            新生代区存放的朝生夕死的对象

            Eden与sur采用复制清除算法回收内存， Old去采用标记整理回收内存

            3.4.1 Eden区的那些对象会移动到Old区
                a. 分代年龄大于15
                b. 大对象
                c. 动态年龄判断
                    发生一次YongGC, 计算统计从分代年龄为1的对象开始计算对象的内存，当内存待遇sur 区的内存一半时，将大于等于一半内存分代年龄的对象移动到Old区
                    例如： 统计到分代年龄为3时的对象内存已经大于sur的一半， 那么将>= 3的所有对象移动到Old区

            3.4.2 JVM参数设置新生代区的内存的大小
                -XX:NewSize                 新生代初始化内存大小
                -XX:MaxNewSize              新生代的最大内存大小
                -XX:PretenureSizeThreshold  新生代最大对象阀值

            3.4.3 分代算法的优势
                减少stop-the-world的频率，提升线程的执行效率。
                分代算法将回收分为YongGC与FullGC, YongGC 的时候只是将Eden区中的对象移动到survivor区，或者新生代的对象移动到Old区，这个过程并不会触发stop-the-world.
                只有在Old区内存不足时，才会触发stop-the-world.

                stop-the-world: 直意停止整个世界，指JVM在GC时将挂起所有线程，整个JVM虚拟机只有GC进程运行。 stop-the-world这种现象发生时，程序被挂起无法正常运行，在Android中的
                                表现就是卡顿。因此， 因此要尽量减少FullGC的回收频率


    4、 垃圾收集器的类型

        CMS： 串行

        G1： 并行

    参考： https://blog.csdn.net/justloveyou_/article/details/71216049


三、 GC日志分析

        4.1 JVM 参数
            -XX:InitialHeapSize : 初始堆大小
            -XX:MaxHeapSize : 最大堆大小
            -XX:NewSize : 初始新生代大小
            -XX:MaxNewSize : 最大新生代大小
            -XX:PretenureSizeThreshold=10485760 : 指定了大对象阈值是10MB。
            -XX:+PrintGCDetils：打印详细的gc日志
            -XX:+PrintGCTimeStamps：这个参数可以打印出来每次GC发生的时间
            -Xloggc:gc.log：这个参数可以设置将gc日志写入一个磁盘文件

            示例：
                -XX:NewSize=5242880 -XX:MaxNewSize=5242880 -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760
                -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC
                -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log

            参考：https://www.cnblogs.com/klvchen/articles/11841337.html


四、 ClassLoader

    1. 双亲委派机制

    2. 类加载的这个过程
        2.1 加载
        2.2 验证
        2.3 准备
        2.4 解析
        2.5 初始化
        2.6 使用
        2.7 卸载


面试题： https://juejin.cn/post/6844903885115490311



五、 Dalvik 虚拟机初探

    1. Dalvik 虚拟机与Java虚拟机的区别
        a. 运行的字节码不同， Dalvik 运行的是dex 字节码， JVM运行的是class 字节码
        b. 架构不同， Dalvik 是基于寄存器占用空间下，运行速度快。JVM 是基于栈
        c. 一个应用对应一个Dalvik虚拟机， 而JVM只是运行的环境

    2. Dalvik 与 ART
        ART (Android Runtime) 是Android5.0 版本加入的虚拟机架构， 采用ahead-of-time(AOT) 技术将dex字节码文件在安装时全部转化成机器能运行的机器码文件adex 文件。
        而Dalvik 虚拟机采用JIT技术在程序运行时，遍运行遍编译机器能识别的机器码文件。

        2.1 ART 优势
            a. 程序冷启动速度更快
            b. 设备性能更好，更流畅， 耗电更低
            c. 支持更低的赢家

        2.2 ART 缺点
            a. 程序第一次安装时间更长
            b. 占用的文件空间比较大


    3. dex 字节码与class 字节码
        .java 文件通过javac 命令编译成.class 文件。 根据Java规范文档可知， .class文件通过ClassFile 的一个C文件表示整个文件的数据格式信息

            ClassFile {
                majic                                      魔鬼数字
                minor_version
                major_version
                constant_pool_count                         常量池数量
                constant_pools[constant_pool_count - 1]     常量池
                access_flag                                 Class 修饰.取值已ACC_开头的值， 例如ACC_PUBLIC, ACC_INTERFACE
                this_class
                super_class
                interfaces_count                            实现接口数量
                interfaces[interfaces_count - 1]            实现接口的具体信息数组
                fields_count                                变量数量
                fields[fields_count - 1]                    变量信息数组
                methods_count                               方法数量(包好static， final修饰的数量)
                methods[methods_count - 1]                  方法信息数组
                attributs_count                             属性
                attributs[attributs_count - 1]
            }

            从ClassFile 的class 数据结构中可以看到Java语言中的定义的元素信息。 可以通过javap -v xxx.class 查看class文件的结构信息

        dex 字节码文件通过dx 命令将多个.class 文件编译成.dex（一个dex 文件的方法不能超过65525 个方法）。 因此， dex 将每个class 文件中的常量池合并成一个dex常量池,减少了文件的大小。

    4. 命令集
        4.1 .java -> .class
            javac <options> <source files> 编译成<source files>.class
        4.2 .class -> .dex
            dx --dex [--debug] [--verbose] [--positions=<style>] [--no-locals]
              [--no-optimize] [--statistics] [--[no-]optimize-list=<file>] [--no-strict]
              [--keep-classes] [--output=<file>] [--dump-to=<file>] [--dump-width=<n>]
              [--dump-method=<name>[*]] [--verbose-dump] [--no-files] [--core-library]
              [--num-threads=<n>] [--incremental] [--force-jumbo] [--no-warning]
              [--multi-dex [--main-dex-list=<file> [--minimal-main-dex]]
              [--input-list=<file>]
              [<file>.class | <file>.{zip,jar,apk} | <directory>] ...

            例如：
                dx --dex --output=helle.dex helle.class

            dx 命令在SDK目录下的buildtools中

        4.3 查看class文件信息
            javap -v xx.class

        4.4 查看dex 文件信息
            dexdump -d xx.dex


    参考：https://www.infoq.cn/article/android-in-depth-dalvik

