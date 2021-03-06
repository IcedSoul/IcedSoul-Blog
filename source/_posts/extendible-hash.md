title: 可扩展哈希 ExtendibleHash Java实现
tags:
  - 扩展哈希
  - ExtendibleHash
  - 算法
categories: 常见算法
---
# 可扩展哈希 ExtendibleHash
## 定义
扩展哈希是动态哈希的一种，以下定义摘自维基百科[4] 。
> Extendible hashing is a type of hash system which treats a hash as a bit string and uses a trie for bucket lookup.[1] Because of the hierarchical nature of the system, re-hashing is an incremental operation (done one bucket at a time, as needed). This means that time-sensitive applications are less affected by table growth than by standard full-table rehashes.
Extendible hashing was described by Ronald Fagin in 1979. Practically all modern filesystems use either extendible hashing or B-trees. In particular, the Global File System, ZFS, and the SpadFS filesystem use extendible hashing


## 理解
单单读这里定义其实并不是能够很好的理解扩展哈希，需要对哈希表本身的实现方式并且理解参考内容[2]中的多哈希表的实现方式有所理解才能看懂扩展哈希的做法，如果不知道多哈希表是什么请先看参考内容[2]。  
扩展哈希的思想是引入目录数组，通过翻倍目录数组的方式来代替桶的翻倍。它和线性哈希的都属于动态哈希。和线性哈希相比，动态哈希多了一个目录数组，目录数组里面存储指向桶的指针(在Java里即地址，以下统称指针)，其扩展方式也和线性哈希不太一样。  
在扩展哈希算法中，基本的数据结构有目录项和桶。目录项数组用来存储指向桶的指针，桶是实际存放数据的地方，每个桶可以存放一定数量的数据。除此之外，每张哈希表还拥有一个全部深度globalDepth，每个桶还有一个局部深度depth，这两个属性与扩展哈希的动态扩展有关。  
在多哈希表中，因为数据过多单个哈希表冲突过多，影响插入和查询的性能，所以采取使用多张哈希表来存储数据，多个哈希表共用同一个哈希函数，目录项分别指向不同的哈希表。这种做法虽然实现简单，但是有空间利用率太低，占用内存过多。而扩展哈希算法就是在多哈希表的基础上做出了改进:不新建哈希表，通过目录项翻倍的方式来代替桶的翻倍，而实际上桶只有在满了之后才会分裂出一个新的桶，即正常情况下，直接插入数据。在待插入数据的桶已满时，将这个桶分裂为两个，其中的数据尽量平均分到两个桶中。然后更新目录项的索引，使其始终指向正确的桶。在目录项不够时，目录项直接翻倍。  
目录项翻倍和保证指针的正常则借助与全局深度globalDepth和局部深度depth，这两个属性来保持。目录项中始终有2^(globalDepth)项，并且编号分别为二进制(001，010，011...)，通过哈希函数计算得到的hsah(key)的二进制后globalDepth位在目录项中对应的桶即为实际的数据的桶。在插入数据的过程中，如果桶满了会进行桶的分裂操作，并且动态得维护指针，使在插入过程中目录项的指针不会指向错误的桶。  
## 例子
维基百科也有例子，但是我这边经过理解之后也举出了自己的例子。  
如图所示，我们设置hask(key) = key.hashCode()，整数的hashCode()为其本身，每个桶最多存储4个数据，目录项最多有100项，然后依次向扩展哈希表中插入:  
10，17，3，29，4，5，18，6，22  
首先，依次插入10，17，3，29，因为globalDepth为0，所以直接插入到其对应的桶中:  

![ExtendibleHash-1](http://img.icedsoul.cn/img/blog/extendibleHash/1.png)

此时，插入4时，第一个桶已满，则会进行分裂，一个桶分裂成两个桶，桶深度depth增大1，变为1，并且根据hash(key)二进制的后depth位进行区分，为0的留在这个桶中，为1的进入新的桶中如10的二进制最后一位为0，则保留，17，3，29二进制最后一位为1，则进入新的桶。然后因为局部深度depth大于全局深度globalDepth，所以globalDepth也变为1，目录进行翻倍，然后维护目录指针，使新的目录指向正确的桶。分裂完成后，再插入4，此时globalDepth为1了，所以要看hash(key)的二进制的后一位，因为4的二进制最后一位为0，所以插入第一个桶。  

![ExtendibleHash-2](http://img.icedsoul.cn/img/blog/extendibleHash/2.png)

然后继续插入5，此时globalDepth为1，hash(5)的二进制后一位为1，第二个桶仍然有空间，所以插入成功。然后插入18和6，hash(18)和hash(6)的二进制后一位都为0，第一个桶仍有两个位置，所以都插入成功:  

![ExtendibleHash-3](http://img.icedsoul.cn/img/blog/extendibleHash/3.png)

然后插入22，hash(22)的二进制最后一位为0，本应插入第一个桶，但是第一个桶已经满了，所以需要进行分裂，depth增大1，然后根据hash(key)的二进制数的倒数第depth位即倒数第2位进行区分，为0的保留，为1的进入新的桶，其中4的二进制后2位为00，所以保留，而10，18，6的二进制后两位为10，进入新的桶，之后因为局部深度大于全局深度，所以全局深度更新为2，左边目录项翻倍，更新目录项指针，新分裂出来的最后两位是10，所谓目录的第三项指向新分裂出来的桶，而第四项11暂时没有对应桶，但是不能空着，因为最后一位为1是数次数仍在第二个桶里面，所以需要把它的指针指向第二个桶。如下图所示，从上向下数第三个桶即为新分裂出来的桶:  

![ExtendibleHash-4](http://img.icedsoul.cn/img/blog/extendibleHash/4.png)

后面一直重复这个过程即可。  
从例子可以看出，扩展哈希巧妙地利用了key的hashCode的二进制位，用全局深度来表示当前有效的倒数globalDepth位二进制位，根据这个码可以快速定位数据。而在桶的分裂时，则根据局部深度来判定当前桶有效的查询位，分裂时深度加一，也要对数据进行区分，区分开的数据往往比较平均。虽然右边桶的局部深度可能不一样，但是通过简单地维护索引都可以进行方便地查询，比如上图中，第二个桶的局部深度只有1，小于全局深度，虽然查询目录时是根据全局深度来的，但是我们通过维护目录数组，让01和11都指向了depth为1的桶，这样查询依然准确，而且在第二个桶进行分裂时只需要修改11的指针仍然能够保证正确。  

## 代码实现
ExtendibleHash.java
```java
package extendibleHash;

import java.util.HashMap;
import java.util.Map;

public class ExtendibleHash<T, V> {
    private Integer maxValueNumber = 20;
    private Integer maxBucketNumber = 100000;
    private Integer globalDepth;

    private Bucket<T, V> buckets[];

    public ExtendibleHash(){
        this.globalDepth = 0;
        this.buckets = new Bucket[this.maxBucketNumber];
        this.buckets[0] = new Bucket<>();
    }

    public ExtendibleHash(Integer maxBucketNumber, Integer maxValueNumber){
        this.maxBucketNumber = maxBucketNumber;
        this.maxValueNumber = maxValueNumber;
        this.globalDepth = 0;
        this.buckets = new Bucket[this.maxBucketNumber];
        this.buckets[0] = new Bucket<>();
    }

    public void insert(T value, V key){
        //根据全局深度计算出应该插入的桶的编号
        Integer bucketNumber = hash(key);

        //判断当前桶是否装满
        //如果装满了,进行桶扩展
        if(buckets[bucketNumber].isFull()){
            Bucket bucket = buckets[bucketNumber];
            //桶局部深度增加
            bucket.depth++;
            //新建桶,并且对当前桶进行拆分
            Bucket<T, V> tempBucket = bucket.split();

            //如果局部深度大于全部深度,则更新全局深度,
            if(bucket.depth > this.globalDepth){
                this.globalDepth = bucket.depth;
            }

            //更新目录索引
            for(int i = 1 << (this.globalDepth - 1); i < 1 << this.globalDepth; i++){
                if(tempBucket.hashKey.equals(i))
                    this.buckets[i] = tempBucket;
                else if(this.buckets[i] == null)
                    this.buckets[i] = this.buckets[i - (1 << (this.globalDepth - 1))];
            }


            this.insert(value, key);
        }
        //如果没有装满,直接进行插入
        else{
            buckets[bucketNumber].insert(value, key);
        }
    }

    public T find(V key){
        return this.buckets[hash(key)].find(key);
    }


    /**
     * 根据全局深度计算当前key所在的桶的编号
     * @param key
     * @return
     */
    public Integer hash(V key){
        return key.hashCode() % (1 << this.globalDepth);
    }

    /**
     * 定义桶的数据结构
     * @param <T> 插入的数据的类型
     * @param <V> 索引的键的类型
     */
    class Bucket<T, V>{
        //用数组复制起来比较麻烦,所以这里用map来存储
        private Map<V, T> values;
        private Integer number;
        private Integer depth;
        private Integer hashKey;

        public Bucket(){
            this.values = new HashMap<V, T>();
            this.number = 0;
            this.depth = 0;
            this.hashKey = 0;
        }

        public boolean isFull(){
            return this.number >= maxValueNumber;
        }

        public void insert(T value, V key){
            this.values.put(key, value);
            this.number++;
        }

        public T find(V key){
            return this.values.get(key);
        }

        /**
         * 进行桶分裂
         * @return
         */
        public Bucket<T, V> split(){
            //新建桶,设置桶局部深度
            Bucket<T, V> bucket = new Bucket<>();
            bucket.depth = this.depth;

            //计算分裂出的桶的hashKey
            bucket.hashKey = (1 << (bucket.depth - 1)) + this.hashKey;

            Map<V, T> tempMap = new HashMap<>();
            Integer tempNumber = 0;
            for(V key1 : this.values.keySet()){
                if(this.hash(key1).equals(bucket.hashKey)) {
                    bucket.values.put(key1, this.values.get(key1));
                    bucket.number++;
                }
                else {
                    tempMap.put(key1, this.values.get(key1));
                    tempNumber++;
                }
            }
            this.values = tempMap;
            this.number = tempNumber;
            return bucket;
        }

        /**
         * 根据局部深度计算当前key所在的桶的编号
         * @param key
         * @return
         */
        public Integer hash(V key){
            return key.hashCode() % (1 << this.depth);
        }
    }
}

```
以下代码用于测试：
Product.java
```java
package extendibleHash;

public class Product {
    private Integer id;
    private String name;
    private Double price;

    public Product(Integer id, String name, Double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }
}

```
Test.java
```java
package extendibleHash;

import Product;

import java.util.Scanner;

public class Test {
    public static void main(String[] args){
        Scanner input = new Scanner(System.in);
        ExtendibleHash<Product, Integer> b = new ExtendibleHash<>(10000000, 1);

        long time1 = System.nanoTime();

        for (int i = 0; i < 20000; i++) {
            Product p = new Product(i,"test",i * 0.1);
            b.insert(p, p.getId());
        }

        long time2 = System.nanoTime();

        Product p = b.find(12441);

        long time3 = System.nanoTime();

        System.out.println("插入耗时: " + (time2 - time1));
        System.out.println("查询耗时: " + (time3 - time2));
    }
}

```
## 参考内容
1. 扩展哈希的Java实现：https://github.com/peter-otoole/Extendible-Hashing/blob/master/src/extendiblehashing/Functions.java
2. 动态哈希方法：https://www.cnblogs.com/kegeyang/archive/2012/04/05/2432608.html
3. 内存数据库中的索引技术：https://blog.csdn.net/wallwind/article/details/48106765
4. 维基百科 Extendible_hashing：https://en.wikipedia.org/wiki/Extendible_hashing
