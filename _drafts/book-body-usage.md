 /proc/cpuinfo

每一段信息表示一个逻辑CPU

- processor 表示这一逻辑处理器的唯一标识符
- physical id 表示每个物理CPU的唯一标识符
- core id 表示每个CPU Core的唯一标识符
- siblings 表示相同物理CPU中的逻辑处理器的数量
- cpu cores 表示相同物理CPU中的内核数量

例如，机器配置是2个cpu，每个cpu采用4核，每核心支持2个线程。

相当于physical id是有个2个值的(对应2个cpu)，每个physical id对应的cpu cores是4(对应4核心)，对应的siblings是8(对应每CPU8线程)，processor共计16个，对于每一个相同的 core id 和 physical id，对应的processor有2个(对应2个线程)。

## 物理CPU个数

```
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

## 逻辑CPU个数

```
cat /proc/cpuinfo | grep "processor" | wc -l
```
 
