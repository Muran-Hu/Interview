# LruCache
LruCache 实现原理
#### LruCache 的核心原理就是对 LinkedHashMap 的有效利用，它的内部存在一个 LinkedHashMap 成员变量。值得我们关注的有四个方法：构造方法、get、put、trimToSize。

###### 1. 构造方法： 在 LruCache 的构造方法中做了两件事，设置了 maxSize、创建了一个 LinkedHashMap。这里值得注意的是 LruCache 将 LinkedHashMap的accessOrder 设置为了 true，accessOrder 就是遍历这个LinkedHashMap 的输出顺序。true 代表按照访问顺序输出，false代表按添加顺序输出，因为通常都是按照添加顺序输出，所以 accessOrder 这个属性默认是 false，但我们的 LruCache 需要按访问顺序输出，所以显式的将 accessOrder 设置为 true。

###### 2. get方法： 本质上是调用 LinkedHashMap 的 get 方法，由于我们将 accessOrder 设置为了 true，所以每调用一次get方法，就会将我们访问的当前元素放置到这个LinkedHashMap的尾部。

###### 3. put方法： 本质上也是调用了 LinkedHashMap 的 put 方法，由于 LinkedHashMap 的特性，每调用一次 put 方法，也会将新加入的元素放置到 LinkedHashMap 的尾部。添加之后会调用 trimToSize 方法来保证添加后的内存不超过 maxSize。

###### 4. trimToSize方法： trimToSize 方法的内部其实是开启了一个 while(true)的死循环，不断的从 LinkedHashMap 的首部删除元素，直到删除之后的内存小于 maxSize 之后使用 break 跳出循环。

#### 其实到这里我们可以总结一下，为什么这个算法叫 最近最少使用 算法呢？原理很简单，我们的每次 put 或者get都可以看做一次访问，由于 LinkedHashMap 的特性，会将每次访问到的元素放置到尾部。当我们的内存达到阈值后，会触发 trimToSize 方法来删除 LinkedHashMap 首部的元素，直到当前内存小于 maxSize。为什么删除首部的元素，原因很明显：我们最近经常访问的元素都会放置到尾部，那首部的元素肯定就是 最近最少使用 的元素了，因此当内存不足时应当优先删除这些元素。
