# String StringBuffer StringBuilder 区别

#### 1. String：字符串常量，字符串长度不可变。
#### 2. StringBuffer：字符串变量（Synchronized，即线程安全）。
#### 3. StringBuilder：字符串变量（非线程安全）。

## 一、String - 不可变 char 数组
    // The associated character storage is managed by the runtime. We only
    // keep track of the length here.
    //关键点final private
    // private final char value[];
## 二、StringBuffer（线程安全） - extends AbstractStringBuilder， 可变 char 数组
    abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;

## 三、StringBuilder（非线程安全） - extends AbstractStringBuilder， 可变 char 数组
    abstract class AbstractStringBuilder implements Appendable, CharSequence {
    /**
     * The value is used for character storage.
     */
    char[] value;

    /**
     * The count is the number of characters used.
     */
    int count;
    
## 四、三者区别
#### 1. String 类型和StringBuffer的主要性能区别：String是不可变的对象, 因此在每次对String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象，所以经常改变内容的字符串最好不要用 String ，因为每次生成对象都会对系统性能产生影响，特别当内存中无引用对象多了以后， JVM 的 GC 就会开始工作，性能就会降低。

#### 2. 使用 StringBuffer 类时，每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用。所以多数情况下推荐使用 StringBuffer ，特别是字符串对象经常改变的情况下。
#### 3. 基本原则：如果要操作少量的数据，用String ；单线程操作大量数据，用StringBuilder ；多线程操作大量数据，用StringBuffer。

## 五、特殊情况：String 对象的字符串拼接其实是被 Java Compiler 编译成了 StringBuffer 对象的拼接，所以这些时候 String 对象的速度并不会比 StringBuffer 对象慢，例如：
    String s1 = “This is only a” + “ simple” + “ test”;
    StringBuffer Sb = new StringBuilder(“This is only a”).append(“ simple”).append(“ test”);
## 生成 String s1对象的速度并不比 StringBuffer慢。其实在Java Compiler里，自动做了如下转换：
## Java Compiler直接把上述第一条语句编译为：
    String s1 = “This is only a simple test”;
## 所以速度很快。但要注意的是，如果拼接的字符串来自另外的String对象的话，Java Compiler就不会自动转换了，速度也就没那么快了，例如：
    String s2 = “This is only a”;
    String s3 = “ simple”;
    String s4 = “ test”;
    String s1 = s2 + s3 + s4;
## 这时候，Java Compiler会规规矩矩的按照原来的方式去做，String的concatenation（即+）操作利用了StringBuilder（或StringBuffer）的append方法实现，此时，对于上述情况，若s2，s3，s4采用String定义，拼接时需要额外创建一个StringBuffer（或StringBuilder），之后将StringBuffer转换为String；若采用StringBuffer（或StringBuilder），则不需额外创建StringBuffer。
