探讨HashMap占用内存大小

# Java各数据类型占用内存

基本数据类型：

1. 整型：byte 1B, short 2B, int 4B, long 8B
    
2. 浮点型：float 4B, double 8B
    
3. 字符型：char 2B
    
4. 布尔型：boolean 可能占用1bit 也可能是1B加速访问
    
5. 引用：32位JVM为4B，64位JVM为8B（开启压缩指针后为4B）
    
6. 对象：对象头12B + 实例数据大小 + 内存对齐padding
    

# HashMap内存分布

1. 底层结构：
    
    ```
    public class HashMap{
    	// 底层数组
      Node<K, V>[] table;
      private class Node{
      	final int hash;
        final K key;		// 底层存储为Integer对象
        V value;
        Node<K, V> next;
      }
    }
    ```
    
2. HashMap默认负载因子是0.75，默认初始大小16，扩容方式为**2倍扩容**
    
3. 理论计算HashMap占用内存应为：HashMap对象占用内存 + table对象占用内存 + 所有Node对象占用内存 + 所有存放实例对象占用内存 + 所有实例数据占用内存（基本数据类型）
    
    1. 注意：java中每个对象会有一个对象头，32位JVM和64位JVM开启压缩指针情况下为12B，64位JVM不开启压缩指针情况下为16B
        
    2. 对象分配的空间：对象头 + 实例数据（包括基本数据类型和对象引用） + 对齐padding
        

# 实践

1. 问题假设一个HashMap中存放了1024对int键值对
    
2. jmap通过以下指令获取heap dump
    
    ```
    jmap -dump:live,format=b,file=log.hprof PID
    ```
    
3. visualVM中查看该文件可以看到HashMap内部结构：
    
    ![[Pasted image 20240425162933.png]]
    
    1. 可以看到分配给HashMap的大小为48B，对象头12B + 4个基本数据类型4B + 3个4B引用 + table引用4B + 对齐padding 4B = 48B
        
    2. 剩余为table占用内存：73744B
        
4. table内部结构：
    
    ![[Pasted image 20240425162942.png]]
    
    ![[Pasted image 20240425162954.png]]
    
    1. 可以看到table中有2048个元素，每个元素为一个引用位，则内存分配为：对象头12B + 2048个4B引用 + 对齐padding 4B = 8208B
        
    2. 剩余为所有Node占用内存：65536B
        
5. Node内部结构：
    
    ![[Pasted image 20240425163001.png]]
    
    1. Node中主要有1个hash(int)，两个对象引用，因此内存分配为：对象头12B + hash值4B + key, value 4B引用 + next 4B引用 + 对齐padding 4B = 32B
        
    2. 剩余为两个Integer对象占用内存：32B
        
6. Integer占用内存：对象头12B + int 4B = 16B
    
7. 由上总结：一个存放了1024对int键值对的HashMap总占用内存为：48B + 8208B + (32 + 32) * 1024 = 73792B，符合visualVM展示的数据