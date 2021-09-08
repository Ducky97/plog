# （PLDI'20）BlankIt Library Debloating Getting What You Want Instead of Cutting What You Don’t Chris

在本篇论文中，我们提出了一个新的方法来处理应用程序中的库函数，我们的原则是“拿到你需要的”，这与“删掉你不想要的”有很大的不同，我们的方法侧重于按需activate或deactivate库函数，以达到缩小攻击面的目的。关键思想是只加载**将**在运行时应用程序中的每个库调用站点上使用的库函数集。这种需求驱动的加载方法依赖于输入感知的oracle，它预测在执行过程中给定调用位置所需的一组近乎精确的库函数。预测函数及时加载，返回时**卸载**。

## 背景

## 设计

在运行的时候，BlankIt在program-start的时候初始化，当库第一次加载到内存中的时候，BlankIt会立刻对每个function做如下动作：插入一个probe在函数的入口点；当probe在一个安全的只读内存中之后，复制function的代码；然后删除插入probe后的所有代码。在运行时，BlankIt会按需加载代码。

在运行时，经过训练后的决策树会根据目前加载的代码，预测即将加载的代码。它表明，在正常运行时A中，可以利用**溢出漏洞跳到glibc中的任意地址**，但在运行时B中使用BlankIt，**由于库函数被清除**，跳到任意地址将导致故障。预测机制减少了可通过漏洞达到的功能集。

![image-20210908200437910](C:\Users\wwy\AppData\Roaming\Typora\typora-user-images\image-20210908200437910.png)

## BlankIt的运行时

BlankIt会擦除库中的所有代码，并在需要的时候加载它们。

### BlankIt设计

BlankIt利用了Intel Pin，它的设计继承了Pin的probe 模式和编程模型。在Initial time，BlankIt会便利所有的可执行的共享的对象，对于对象中的每个function，它用trampoline覆盖前几个字节，用非法或无效指令空白剩余字节。并把可信的函数加入到信任的缓存中。在运行时，trampolines跳转执行到通用的handler、或到Pin-speak、或一个探针。The BlankIt探针会将函数字节返回给原有的地方，并返回一个函数。



如下图所示，当app调用malloc时，会首先沿着1到libc的malloc，但是已经呗trampolines覆盖了，并接着沿着2到probe-cooy函数，在第6-9行时，一个对map m的循环会赋值和记录所有出现在预测集里面的函数，当malloc和mmaps被需要时，会它会拷贝回来。在执行probe_copy函数之后，当malloc调用mmap64时，会沿着箭头4,5,6,7执行。

![image-20210908204254248](C:\Users\wwy\AppData\Roaming\Typora\typora-user-images\image-20210908204254248.png)



第二类probe(图4)用于**消隐**和**预测**。在BlankIt允许的应用中，预测调用发生一个库被调用之前。如下图，在调用free之前，在编译时，一个对blankIt_predict(ID)的调用被插入到调用之前，并将blankit_predict替换为probe_blankit_predict，probe_blankit_predict会将要调用的函数转为空白，并修正ID为预测的子集合，并返回给它自己invoke的库函数。在这种情况下，任何错误的预测都会产生警报，并返回给审计机构。

![s](C:\Users\wwy\AppData\Roaming\Typora\typora-user-images\image-20210908205650877.png)

