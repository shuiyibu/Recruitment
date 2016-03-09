Java集合工具包位于Java.util包下，包含了很多常用的数据结构，如数组、链表、栈、队列、集合、哈希表等。学习Java集合框架下大致可以分为如下五个部分：List列表、Set集合、Map映射、迭代器（Iterator、Enumeration）、工具类（Arrays、Collections）。

Java集合类的整体框架如下：

![Framework](images/2016/03/Screen Shot 2016-03-02 at 23.24.55.png)

从上图中可以看出，集合类主要分为两大类：`Collection`和
`Map`。

`Collection`是List、Set等集合高度抽象出来的接口，它包含了这些集合的基本操作，它主要又分为两大部分：List和Set。

- `List`接口通常表示一个列表（数组、队列、链表、栈等），其中的元素可以重复，常用实现类为`ArrayList`和`LinkedList`，另外还有不常用的`Vector`。另外，**LinkedList还实现了Queue接口，因此也可以作为队列使用**。

- `Set`接口通常表示一个集合，其中的元素不允许重复（通过hashcode和equals函数保证），常用实现类有`HashSet`和`TreeSet`，
 - `HashSet`是通过Map中的`HashMap`实现的
 - `TreeSet`是通过Map中的`TreeMap`实现的。另外，TreeSet还实现了`SortedSet`接口，因此是有序的集合（集合中的元素要实现`Comparable`接口，并覆写`Compartor`函数才行）。

抽象类`AbstractCollection`、`AbstractList`和`AbstractSet`分别实现了`Collection`、`List`和`Set`接口，这就是在Java集合框架中用的很多的适配器设计模式，用这些抽象类去实现接口，在抽象类中实现接口中的若干或全部方法，这样下面的一些类只需直接继承该抽象类，并实现自己需要的方法即可，而不用实现接口中的全部抽象方法。

`Map`是一个映射接口，其中的每个元素都是一个key-value键值对，同样抽象类AbstractMap通过适配器模式实现了Map接口中的大部分函数，TreeMap、HashMap、WeakHashMap等实现类都通过继承AbstractMap来实现，另外，**不常用的HashTable直接实现了Map接口**，它和Vector都是JDK1.0就引入的集合类。

`Iterator`是遍历集合的迭代器（不能遍历Map，**只用来遍历Collection**），Collection的实现类都实现了iterator()函数，它返回一个Iterator对象，用来遍历集合，ListIterator则专门用来遍历List。而**Enumeration则是JDK1.0时引入的，作用与Iterator相同，但它的功能比Iterator要少，它只能再Hashtable、Vector和Stack中使用**。

`Arrays`和`Collections`是用来操作数组、集合的两个工具类，例如在ArrayList和Vector中大量调用了Arrays.Copyof()方法，而Collections中有很多静态方法可以返回各集合类的synchronized版本，即线程安全的版本，当然了，如果要用线程安全的结合类，首选首选Concurrent并发包下的对应的集合类。

------

# ArrayList源码剖析
## ArrayList简介
ArrayList是基于数组实现的，是一个动态数组，其容量能自动增长，类似于C语言中的动态申请内存，动态增长内存。

ArrayList**不是线程安全**的，只能用在单线程环境下，多线程环境下可以考虑用`Collections.synchronizedList(List l`)函数返回一个线程安全的ArrayList类，也可以使用concurrent并发包下的`CopyOnWriteArrayList`类。

ArrayList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了RandomAccess接口，支持快速随机访问，实际上就是通过下标序号进行快速访问，实现了Cloneable接口，能被克隆。

## ArrayList源码剖析
```

package java.util;  

public class ArrayList<E> extends AbstractList<E>  
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
{  
    // 序列版本号  
    private static final long serialVersionUID = 8683452581122892189L;  

    // ArrayList基于该数组实现，用该数组保存数据
    private transient Object[] elementData;  

    // ArrayList中实际数据的数量  
    private int size;  

    // ArrayList带容量大小的构造函数。  
    public ArrayList(int initialCapacity) {  
        super();  
        if (initialCapacity < 0)  
            throw new IllegalArgumentException("Illegal Capacity: "+  
                                               initialCapacity);  
        // 新建一个数组  
        this.elementData = new Object[initialCapacity];  
    }  

    // ArrayList无参构造函数。默认容量是10。  
    public ArrayList() {  
        this(10);  
    }  

    // 创建一个包含collection的ArrayList  
    public ArrayList(Collection<? extends E> c) {  
        elementData = c.toArray();  
        size = elementData.length;  
        if (elementData.getClass() != Object[].class)  
            elementData = Arrays.copyOf(elementData, size, Object[].class);  
    }  


    // 将当前容量值设为实际元素个数  
    public void trimToSize() {  
        modCount++;  
        int oldCapacity = elementData.length;  
        if (size < oldCapacity) {  
            elementData = Arrays.copyOf(elementData, size);  
        }  
    }  


    // 确定ArrarList的容量。  
    // 若ArrayList的容量不足以容纳当前的全部元素，设置 新的容量=“(原始容量x3)/2 + 1”  
    public void ensureCapacity(int minCapacity) {  
        // 将“修改统计数”+1，该变量主要是用来实现fail-fast机制的  
        modCount++;  
        int oldCapacity = elementData.length;  
        // 若当前容量不足以容纳当前的元素个数，设置 新的容量=“(原始容量x3)/2 + 1”  
        if (minCapacity > oldCapacity) {  
            Object oldData[] = elementData;  
            int newCapacity = (oldCapacity * 3)/2 + 1;  
			//如果还不够，则直接将minCapacity设置为当前容量
            if (newCapacity < minCapacity)  
                newCapacity = minCapacity;  
            elementData = Arrays.copyOf(elementData, newCapacity);  
        }  
    }  

    // 添加元素e  
    public boolean add(E e) {  
        // 确定ArrayList的容量大小  
        ensureCapacity(size + 1);  // Increments modCount!!  
        // 添加e到ArrayList中  
        elementData[size++] = e;  
        return true;  
    }  

    // 返回ArrayList的实际大小  
    public int size() {  
        return size;  
    }  

    // ArrayList是否包含Object(o)  
    public boolean contains(Object o) {  
        return indexOf(o) >= 0;  
    }  

    //返回ArrayList是否为空  
    public boolean isEmpty() {  
        return size == 0;  
    }  

    // 正向查找，返回元素的索引值  
    public int indexOf(Object o) {  
        if (o == null) {  
            for (int i = 0; i < size; i++)  
                if (elementData[i]==null)  
                      return i;  
        } else {  
            for (int i = 0; i < size; i++)  
                if (o.equals(elementData[i]))  
                    return i;  
        }

        return -1;  
    }  



    // 反向查找(从数组末尾向开始查找)，返回元素(o)的索引值  
    public int lastIndexOf(Object o) {  
        if (o == null) {  
            for (int i = size-1; i >= 0; i--)  
              if (elementData[i]==null)  
                return i;  
        } else {  
            for (int i = size-1; i >= 0; i--)  
              if (o.equals(elementData[i]))  
                return i;  
        }  
        return -1;  
    }  


    // 返回ArrayList的Object数组  
    public Object[] toArray() {  
        return Arrays.copyOf(elementData, size);  
    }  

    // 返回ArrayList元素组成的数组
    public <T> T[] toArray(T[] a) {  
        // 若数组a的大小 < ArrayList的元素个数；  
        // 则新建一个T[]数组，数组大小是“ArrayList的元素个数”，并将“ArrayList”全部拷贝到新数组中  
        if (a.length < size)  
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());  

        // 若数组a的大小 >= ArrayList的元素个数；  
        // 则将ArrayList的全部元素都拷贝到数组a中。  
        System.arraycopy(elementData, 0, a, 0, size);  
        if (a.length > size)  
            a[size] = null;  
        return a;  
    }  

    // 获取index位置的元素值  
    public E get(int index) {  
        RangeCheck(index);  

        return (E) elementData[index];  
    }  

    // 设置index位置的值为element  
    public E set(int index, E element) {  
        RangeCheck(index);  

        E oldValue = (E) elementData[index];  
        elementData[index] = element;  
        return oldValue;  
    }  

    // 将e添加到ArrayList中  
    public boolean add(E e) {  
        ensureCapacity(size + 1);  // Increments modCount!!  
        elementData[size++] = e;  
        return true;  
    }  

    // 将e添加到ArrayList的指定位置  
    public void add(int index, E element) {  
        if (index > size || index < 0)  
            throw new IndexOutOfBoundsException(  
            "Index: "+index+", Size: "+size);  

        ensureCapacity(size+1);  // Increments modCount!!  
        System.arraycopy(elementData, index, elementData, index + 1,  
             size - index);  
        elementData[index] = element;  
        size++;  
    }  

    // 删除ArrayList指定位置的元素  
    public E remove(int index) {  
        RangeCheck(index);  

        modCount++;  
        E oldValue = (E) elementData[index];  

        int numMoved = size - index - 1;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index+1, elementData, index,  
                 numMoved);  
        elementData[--size] = null; // Let gc do its work  

        return oldValue;  
    }  

    // 删除ArrayList的指定元素  
    public boolean remove(Object o) {  
        if (o == null) {  
                for (int index = 0; index < size; index++)  
            if (elementData[index] == null) {  
                fastRemove(index);  
                return true;  
            }  
        } else {  
            for (int index = 0; index < size; index++)  
            if (o.equals(elementData[index])) {  
                fastRemove(index);  
                return true;  
            }  
        }  
        return false;  
    }  


    // 快速删除第index个元素  
    private void fastRemove(int index) {  
        modCount++;  
        int numMoved = size - index - 1;  
        // 从"index+1"开始，用后面的元素替换前面的元素。  
        if (numMoved > 0)  
            System.arraycopy(elementData, index+1, elementData, index,  
                             numMoved);  
        // 将最后一个元素设为null  
        elementData[--size] = null; // Let gc do its work  
    }  

    // 删除元素  
    public boolean remove(Object o) {  
        if (o == null) {  
            for (int index = 0; index < size; index++)  
            if (elementData[index] == null) {  
                fastRemove(index);  
            return true;  
            }  
        } else {  
            // 便利ArrayList，找到“元素o”，则删除，并返回true。  
            for (int index = 0; index < size; index++)  
            if (o.equals(elementData[index])) {  
                fastRemove(index);  
            return true;  
            }  
        }  
        return false;  
    }  

    // 清空ArrayList，将全部的元素设为null  
    public void clear() {  
        modCount++;  

        for (int i = 0; i < size; i++)  
            elementData[i] = null;  

        size = 0;  
    }  

    // 将集合c追加到ArrayList中  
    public boolean addAll(Collection<? extends E> c) {  
        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacity(size + numNew);  // Increments modCount  
        System.arraycopy(a, 0, elementData, size, numNew);  
        size += numNew;  
        return numNew != 0;  
    }  

    // 从index位置开始，将集合c添加到ArrayList  
    public boolean addAll(int index, Collection<? extends E> c) {  
        if (index > size || index < 0)  
            throw new IndexOutOfBoundsException(  
            "Index: " + index + ", Size: " + size);  

        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacity(size + numNew);  // Increments modCount  

        int numMoved = size - index;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index, elementData, index + numNew,  
                 numMoved);  

        System.arraycopy(a, 0, elementData, index, numNew);  
        size += numNew;  
        return numNew != 0;  
    }  

    // 删除fromIndex到toIndex之间的全部元素。  
    protected void removeRange(int fromIndex, int toIndex) {  
    modCount++;  
    int numMoved = size - toIndex;  
        System.arraycopy(elementData, toIndex, elementData, fromIndex,  
                         numMoved);  

    // Let gc do its work  
    int newSize = size - (toIndex-fromIndex);  
    while (size != newSize)  
        elementData[--size] = null;  
    }  

    private void RangeCheck(int index) {  
    if (index >= size)  
        throw new IndexOutOfBoundsException(  
        "Index: "+index+", Size: "+size);  
    }  


    // 克隆函数  
    public Object clone() {  
        try {  
            ArrayList<E> v = (ArrayList<E>) super.clone();  
            // 将当前ArrayList的全部元素拷贝到v中  
            v.elementData = Arrays.copyOf(elementData, size);  
            v.modCount = 0;  
            return v;  
        } catch (CloneNotSupportedException e) {  
            // this shouldn't happen, since we are Cloneable  
            throw new InternalError();  
        }  
    }  


    // java.io.Serializable的写入函数  
    // 将ArrayList的“容量，所有的元素值”都写入到输出流中  
    private void writeObject(java.io.ObjectOutputStream s)  
        throws java.io.IOException{  
    // Write out element count, and any hidden stuff  
    int expectedModCount = modCount;  
    s.defaultWriteObject();  

        // 写入“数组的容量”  
        s.writeInt(elementData.length);  

    // 写入“数组的每一个元素”  
    for (int i=0; i<size; i++)  
            s.writeObject(elementData[i]);  

    if (modCount != expectedModCount) {  
            throw new ConcurrentModificationException();  
        }  

    }  


    // java.io.Serializable的读取函数：根据写入方式读出  
    // 先将ArrayList的“容量”读出，然后将“所有的元素值”读出  
    private void readObject(java.io.ObjectInputStream s)  
        throws java.io.IOException, ClassNotFoundException {  
        // Read in size, and any hidden stuff  
        s.defaultReadObject();  

        // 从输入流中读取ArrayList的“容量”  
        int arrayLength = s.readInt();  
        Object[] a = elementData = new Object[arrayLength];  

        // 从输入流中将“所有的元素值”读出  
        for (int i=0; i<size; i++)  
            a[i] = s.readObject();  
    }  
}
```
## 几点总结

关于ArrayList的源码，给出几点比较重要的总结：

1. 注意其三个不同的构造方法。无参构造方法构造的ArrayList的容量默认为10，带有Collection参数的构造方法，将Collection转化为数组赋给ArrayList的实现数组elementData。
2. 注意扩充容量的方法ensureCapacity。ArrayList在每次增加元素（可能是1个，也可能是一组）时，都要调用该方法来确保足够的容量。当容量不足以容纳当前的元素个数时，就**设置新的容量为旧的容量的1.5倍加1**，如果设置后的新容量还不够，则直接新容量设置为传入的参数（也就是所需的容量），而后用Arrays.copyof()方法将元素拷贝到新的数组（详见下面的第3点）。从中可以看出，当容量不够时，每次增加元素，都要将原来的元素拷贝到一个新的数组中，非常之耗时，也因此**建议在事先能确定元素数量的情况下，才使用ArrayList，否则建议使用LinkedList**。
3. ArrayList的实现中大量地调用了`Arrays.copyof()`和`System.arraycopy()`方法。我们有必要对这两个方法的实现做下深入的了解。
 - 首先来看`Arrays.copyof()`方法。它有很多个重载的方法，但实现思路都是一样的，我们来看泛型版本的源码：
 ```
 public static <T> T[] copyOf(T[] original, int newLength)
 {
      return (T[]) copyOf(original, newLength, original.getClass());
 }
 ```
 很明显调用了另一个copyof方法，该方法有三个参数，最后一个参数指明要转换的数据的类型，其源码如下：
 ```
 public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType)
 {
   T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
  System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
  return copy;
  }
 ```
 这里可以很明显地看出，该方法实际上是在其内部又创建了一个长度为newlength的数组，调用System.arraycopy()方法，将原来数组中的元素复制到了新的数组中。<br>
 下面来看System.arraycopy()方法。该方法被标记了native，调用了系统的C/C++代码，在JDK中是看不到的，但在openJDK中可以看到其源码。该函数实际上最终调用了C语言的memmove()函数，因此它可以保证同一个数组内元素的正确复制和移动，比一般的复制方法的实现效率要高很多，很适合用来批量处理数组。**Java强烈推荐在复制大量数组元素时用该方法，以取得更高的效率**。
4. 注意ArrayList的两个转化为静态数组的toArray方法。
 - 第一个，`Object[] toArray()`方法。该方法有可能会抛出`java.lang.ClassCastException`异常，**如果直接用向下转型的方法，将整个ArrayList集合转变为指定类型的Array数组，便会抛出该异常，而如果转化为Array数组时不向下转型，而是将每个元素向下转型，则不会抛出该异常**，显然对数组中的元素一个个进行向下转型，效率不高，且不太方便。
 - 第二个，`<T> T[] toArray(T[] a)`方法。该方法可以直接将ArrayList转换得到的Array进行整体向下转型（转型其实是在该方法的源码中实现的），且从该方法的源码中可以看出，参数a的大小不足时，内部会调用Arrays.copyOf方法，该方法内部创建一个新的数组返回，因此对该方法的常用形式如下：
 ```
    public static Integer[] vectorToArray2(ArrayList<Integer> v) {  
        Integer[] newText = (Integer[])v.toArray(new Integer[0]);  
        return newText;  
    }  
 ```
5. ArrayList基于数组实现，可以通过下标索引直接查找到指定位置的元素，因此查找效率高，但每次插入或删除元素，就要大量地移动元素，插入删除元素的效率低。
6. 在查找给定元素索引值等的方法中，源码都将该元素的值分为null和不为null两种情况处理，ArrayList中允许元素为null。

--------------

# LinkedList源码剖析

## LinkedList简介
LinkedList是基于双向循环链表（从源码中可以很容易看出）实现的，除了可以当做链表来操作外，它还可以当做`栈`、`队列`和`双端队列`来使用。

LinkedList同样是**非线程安全**的，只在单线程下适合使用。

LinkedList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了Cloneable接口，能被克隆。


## LinkedList源码剖析
```

package java.util;  

public class LinkedList<E>  
    extends AbstractSequentialList<E>  
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable  
{  
    // 链表的表头，表头不包含任何数据。Entry是个链表类数据结构。  
    private transient Entry<E> header = new Entry<E>(null, null, null);  

    // LinkedList中元素个数  
    private transient int size = 0;  

    // 默认构造函数：创建一个空的链表  
    public LinkedList() {  
        header.next = header.previous = header;  
    }  

    // 包含“集合”的构造函数:创建一个包含“集合”的LinkedList  
    public LinkedList(Collection<? extends E> c) {  
        this();  
        addAll(c);  
    }  

    // 获取LinkedList的第一个元素  
    public E getFirst() {  
        if (size==0)  
            throw new NoSuchElementException();  

        // 链表的表头header中不包含数据。  
        // 这里返回header所指下一个节点所包含的数据。  
        return header.next.element;  
    }  

    // 获取LinkedList的最后一个元素  
    public E getLast()  {  
        if (size==0)  
            throw new NoSuchElementException();  

        // 由于LinkedList是双向链表；而表头header不包含数据。  
        // 因而，这里返回表头header的前一个节点所包含的数据。  
        return header.previous.element;  
    }  

    // 删除LinkedList的第一个元素  
    public E removeFirst() {  
        return remove(header.next);  
    }  

    // 删除LinkedList的最后一个元素  
    public E removeLast() {  
        return remove(header.previous);  
    }  

    // 将元素添加到LinkedList的起始位置  
    public void addFirst(E e) {  
        addBefore(e, header.next);  
    }  

    // 将元素添加到LinkedList的结束位置  
    public void addLast(E e) {  
        addBefore(e, header);  
    }  

    // 判断LinkedList是否包含元素(o)  
    public boolean contains(Object o) {  
        return indexOf(o) != -1;  
    }  

    // 返回LinkedList的大小  
    public int size() {  
        return size;  
    }  

    // 将元素(E)添加到LinkedList中  
    public boolean add(E e) {  
        // 将节点(节点数据是e)添加到表头(header)之前。  
        // 即，将节点添加到双向链表的末端。  
        addBefore(e, header);  
        return true;  
    }  

    // 从LinkedList中删除元素(o)  
    // 从链表开始查找，如存在元素(o)则删除该元素并返回true；  
    // 否则，返回false。  
    public boolean remove(Object o) {  
        if (o==null) {  
            // 若o为null的删除情况  
            for (Entry<E> e = header.next; e != header; e = e.next) {  
                if (e.element==null) {  
                    remove(e);  
                    return true;  
                }  
            }  
        } else {  
            // 若o不为null的删除情况  
            for (Entry<E> e = header.next; e != header; e = e.next) {  
                if (o.equals(e.element)) {  
                    remove(e);  
                    return true;  
                }  
            }  
        }  
        return false;  
    }  

    // 将“集合(c)”添加到LinkedList中。  
    // 实际上，是从双向链表的末尾开始，将“集合(c)”添加到双向链表中。  
    public boolean addAll(Collection<? extends E> c) {  
        return addAll(size, c);  
    }  

    // 从双向链表的index开始，将“集合(c)”添加到双向链表中。  
    public boolean addAll(int index, Collection<? extends E> c) {  
        if (index < 0 || index > size)  
            throw new IndexOutOfBoundsException("Index: "+index+  
                                                ", Size: "+size);  
        Object[] a = c.toArray();  
        // 获取集合的长度  
        int numNew = a.length;  
        if (numNew==0)  
            return false;  
        modCount++;  

        // 设置“当前要插入节点的后一个节点”  
        Entry<E> successor = (index==size ? header : entry(index));  
        // 设置“当前要插入节点的前一个节点”  
        Entry<E> predecessor = successor.previous;  
        // 将集合(c)全部插入双向链表中  
        for (int i=0; i<numNew; i++) {  
            Entry<E> e = new Entry<E>((E)a[i], successor, predecessor);  
            predecessor.next = e;  
            predecessor = e;  
        }  
        successor.previous = predecessor;  

        // 调整LinkedList的实际大小  
        size += numNew;  
        return true;  
    }  

    // 清空双向链表  
    public void clear() {  
        Entry<E> e = header.next;  
        // 从表头开始，逐个向后遍历；对遍历到的节点执行一下操作：  
        // (01) 设置前一个节点为null   
        // (02) 设置当前节点的内容为null   
        // (03) 设置后一个节点为“新的当前节点”  
        while (e != header) {  
            Entry<E> next = e.next;  
            e.next = e.previous = null;  
            e.element = null;  
            e = next;  
        }  
        header.next = header.previous = header;  
        // 设置大小为0  
        size = 0;  
        modCount++;  
    }  

    // 返回LinkedList指定位置的元素  
    public E get(int index) {  
        return entry(index).element;  
    }  

    // 设置index位置对应的节点的值为element  
    public E set(int index, E element) {  
        Entry<E> e = entry(index);  
        E oldVal = e.element;  
        e.element = element;  
        return oldVal;  
    }  

    // 在index前添加节点，且节点的值为element  
    public void add(int index, E element) {  
        addBefore(element, (index==size ? header : entry(index)));  
    }  

    // 删除index位置的节点  
    public E remove(int index) {  
        return remove(entry(index));  
    }  

    // 获取双向链表中指定位置的节点  
    private Entry<E> entry(int index) {  
        if (index < 0 || index >= size)  
            throw new IndexOutOfBoundsException("Index: "+index+  
                                                ", Size: "+size);  
        Entry<E> e = header;  
        // 获取index处的节点。  
        // 若index < 双向链表长度的1/2,则从前先后查找;  
        // 否则，从后向前查找。  
        if (index < (size >> 1)) {  
            for (int i = 0; i <= index; i++)  
                e = e.next;  
        } else {  
            for (int i = size; i > index; i--)  
                e = e.previous;  
        }  
        return e;  
    }  

    // 从前向后查找，返回“值为对象(o)的节点对应的索引”  
    // 不存在就返回-1  
    public int indexOf(Object o) {  
        int index = 0;  
        if (o==null) {  
            for (Entry e = header.next; e != header; e = e.next) {  
                if (e.element==null)  
                    return index;  
                index++;  
            }  
        } else {  
            for (Entry e = header.next; e != header; e = e.next) {  
                if (o.equals(e.element))  
                    return index;  
                index++;  
            }  
        }  
        return -1;  
    }  

    // 从后向前查找，返回“值为对象(o)的节点对应的索引”  
    // 不存在就返回-1  
    public int lastIndexOf(Object o) {  
        int index = size;  
        if (o==null) {  
            for (Entry e = header.previous; e != header; e = e.previous) {  
                index--;  
                if (e.element==null)  
                    return index;  
            }  
        } else {  
            for (Entry e = header.previous; e != header; e = e.previous) {  
                index--;  
                if (o.equals(e.element))  
                    return index;  
            }  
        }  
        return -1;  
    }  

    // 返回第一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E peek() {  
        if (size==0)  
            return null;  
        return getFirst();  
    }  

    // 返回第一个节点  
    // 若LinkedList的大小为0,则抛出异常  
    public E element() {  
        return getFirst();  
    }  

    // 删除并返回第一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E poll() {  
        if (size==0)  
            return null;  
        return removeFirst();  
    }  

    // 将e添加双向链表末尾  
    public boolean offer(E e) {  
        return add(e);  
    }  

    // 将e添加双向链表开头  
    public boolean offerFirst(E e) {  
        addFirst(e);  
        return true;  
    }  

    // 将e添加双向链表末尾  
    public boolean offerLast(E e) {  
        addLast(e);  
        return true;  
    }  

    // 返回第一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E peekFirst() {  
        if (size==0)  
            return null;  
        return getFirst();  
    }  

    // 返回最后一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E peekLast() {  
        if (size==0)  
            return null;  
        return getLast();  
    }  

    // 删除并返回第一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E pollFirst() {  
        if (size==0)  
            return null;  
        return removeFirst();  
    }  

    // 删除并返回最后一个节点  
    // 若LinkedList的大小为0,则返回null  
    public E pollLast() {  
        if (size==0)  
            return null;  
        return removeLast();  
    }  

    // 将e插入到双向链表开头  
    public void push(E e) {  
        addFirst(e);  
    }  

    // 删除并返回第一个节点  
    public E pop() {  
        return removeFirst();  
    }  

    // 从LinkedList开始向后查找，删除第一个值为元素(o)的节点  
    // 从链表开始查找，如存在节点的值为元素(o)的节点，则删除该节点  
    public boolean removeFirstOccurrence(Object o) {  
        return remove(o);  
    }  

    // 从LinkedList末尾向前查找，删除第一个值为元素(o)的节点  
    // 从链表开始查找，如存在节点的值为元素(o)的节点，则删除该节点  
    public boolean removeLastOccurrence(Object o) {  
        if (o==null) {  
            for (Entry<E> e = header.previous; e != header; e = e.previous) {  
                if (e.element==null) {  
                    remove(e);  
                    return true;  
                }  
            }  
        } else {  
            for (Entry<E> e = header.previous; e != header; e = e.previous) {  
                if (o.equals(e.element)) {  
                    remove(e);  
                    return true;  
                }  
            }  
        }  
        return false;  
    }  

    // 返回“index到末尾的全部节点”对应的ListIterator对象(List迭代器)  
    public ListIterator<E> listIterator(int index) {  
        return new ListItr(index);  
    }  

    // List迭代器  
    private class ListItr implements ListIterator<E> {  
        // 上一次返回的节点  
        private Entry<E> lastReturned = header;  
        // 下一个节点  
        private Entry<E> next;  
        // 下一个节点对应的索引值  
        private int nextIndex;  
        // 期望的改变计数。用来实现fail-fast机制。  
        private int expectedModCount = modCount;  

        // 构造函数。  
        // 从index位置开始进行迭代  
        ListItr(int index) {  
            // index的有效性处理  
            if (index < 0 || index > size)  
                throw new IndexOutOfBoundsException("Index: "+index+ ", Size: "+size);  
            // 若 “index 小于 ‘双向链表长度的一半’”，则从第一个元素开始往后查找；  
            // 否则，从最后一个元素往前查找。  
            if (index < (size >> 1)) {  
                next = header.next;  
                for (nextIndex=0; nextIndex<index; nextIndex++)  
                    next = next.next;  
            } else {  
                next = header;  
                for (nextIndex=size; nextIndex>index; nextIndex--)  
                    next = next.previous;  
            }  
        }  

        // 是否存在下一个元素  
        public boolean hasNext() {  
            // 通过元素索引是否等于“双向链表大小”来判断是否达到最后。  
            return nextIndex != size;  
        }  

        // 获取下一个元素  
        public E next() {  
            checkForComodification();  
            if (nextIndex == size)  
                throw new NoSuchElementException();  

            lastReturned = next;  
            // next指向链表的下一个元素  
            next = next.next;  
            nextIndex++;  
            return lastReturned.element;  
        }  

        // 是否存在上一个元素  
        public boolean hasPrevious() {  
            // 通过元素索引是否等于0，来判断是否达到开头。  
            return nextIndex != 0;  
        }  

        // 获取上一个元素  
        public E previous() {  
            if (nextIndex == 0)  
            throw new NoSuchElementException();  

            // next指向链表的上一个元素  
            lastReturned = next = next.previous;  
            nextIndex--;  
            checkForComodification();  
            return lastReturned.element;  
        }  

        // 获取下一个元素的索引  
        public int nextIndex() {  
            return nextIndex;  
        }  

        // 获取上一个元素的索引  
        public int previousIndex() {  
            return nextIndex-1;  
        }  

        // 删除当前元素。  
        // 删除双向链表中的当前节点  
        public void remove() {  
            checkForComodification();  
            Entry<E> lastNext = lastReturned.next;  
            try {  
                LinkedList.this.remove(lastReturned);  
            } catch (NoSuchElementException e) {  
                throw new IllegalStateException();  
            }  
            if (next==lastReturned)  
                next = lastNext;  
            else
                nextIndex--;  
            lastReturned = header;  
            expectedModCount++;  
        }  

        // 设置当前节点为e  
        public void set(E e) {  
            if (lastReturned == header)  
                throw new IllegalStateException();  
            checkForComodification();  
            lastReturned.element = e;  
        }  

        // 将e添加到当前节点的前面  
        public void add(E e) {  
            checkForComodification();  
            lastReturned = header;  
            addBefore(e, next);  
            nextIndex++;  
            expectedModCount++;  
        }  

        // 判断 “modCount和expectedModCount是否相等”，依次来实现fail-fast机制。  
        final void checkForComodification() {  
            if (modCount != expectedModCount)  
            throw new ConcurrentModificationException();  
        }  
    }  

    // 双向链表的节点所对应的数据结构。  
    // 包含3部分：上一节点，下一节点，当前节点值。  
    private static class Entry<E> {  
        // 当前节点所包含的值  
        E element;  
        // 下一个节点  
        Entry<E> next;  
        // 上一个节点  
        Entry<E> previous;  

        /**  
         * 链表节点的构造函数。  
         * 参数说明：  
         *   element  —— 节点所包含的数据  
         *   next      —— 下一个节点  
         *   previous —— 上一个节点  
         */
        Entry(E element, Entry<E> next, Entry<E> previous) {  
            this.element = element;  
            this.next = next;  
            this.previous = previous;  
        }  
    }  

    // 将节点(节点数据是e)添加到entry节点之前。  
    private Entry<E> addBefore(E e, Entry<E> entry) {  
        // 新建节点newEntry，将newEntry插入到节点e之前；并且设置newEntry的数据是e  
        Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);  
        newEntry.previous.next = newEntry;  
        newEntry.next.previous = newEntry;  
        // 修改LinkedList大小  
        size++;  
        // 修改LinkedList的修改统计数：用来实现fail-fast机制。  
        modCount++;  
        return newEntry;  
    }  

    // 将节点从链表中删除  
    private E remove(Entry<E> e) {  
        if (e == header)  
            throw new NoSuchElementException();  

        E result = e.element;  
        e.previous.next = e.next;  
        e.next.previous = e.previous;  
        e.next = e.previous = null;  
        e.element = null;  
        size--;  
        modCount++;  
        return result;  
    }  

    // 反向迭代器  
    public Iterator<E> descendingIterator() {  
        return new DescendingIterator();  
    }  

    // 反向迭代器实现类。  
    private class DescendingIterator implements Iterator {  
        final ListItr itr = new ListItr(size());  
        // 反向迭代器是否下一个元素。  
        // 实际上是判断双向链表的当前节点是否达到开头  
        public boolean hasNext() {  
            return itr.hasPrevious();  
        }  
        // 反向迭代器获取下一个元素。  
        // 实际上是获取双向链表的前一个节点  
        public E next() {  
            return itr.previous();  
        }  
        // 删除当前节点  
        public void remove() {  
            itr.remove();  
        }  
    }  


    // 返回LinkedList的Object[]数组  
    public Object[] toArray() {  
    // 新建Object[]数组  
    Object[] result = new Object[size];  
        int i = 0;  
        // 将链表中所有节点的数据都添加到Object[]数组中  
        for (Entry<E> e = header.next; e != header; e = e.next)  
            result[i++] = e.element;  
    return result;  
    }  

    // 返回LinkedList的模板数组。所谓模板数组，即可以将T设为任意的数据类型  
    public <T> T[] toArray(T[] a) {  
        // 若数组a的大小 < LinkedList的元素个数(意味着数组a不能容纳LinkedList中全部元素)  
        // 则新建一个T[]数组，T[]的大小为LinkedList大小，并将该T[]赋值给a。  
        if (a.length < size)  
            a = (T[])java.lang.reflect.Array.newInstance(  
                                a.getClass().getComponentType(), size);  
        // 将链表中所有节点的数据都添加到数组a中  
        int i = 0;  
        Object[] result = a;  
        for (Entry<E> e = header.next; e != header; e = e.next)  
            result[i++] = e.element;  

        if (a.length > size)  
            a[size] = null;  

        return a;  
    }  


    // 克隆函数。返回LinkedList的克隆对象。  
    public Object clone() {  
        LinkedList<E> clone = null;  
        // 克隆一个LinkedList克隆对象  
        try {  
            clone = (LinkedList<E>) super.clone();  
        } catch (CloneNotSupportedException e) {  
            throw new InternalError();  
        }  

        // 新建LinkedList表头节点  
        clone.header = new Entry<E>(null, null, null);  
        clone.header.next = clone.header.previous = clone.header;  
        clone.size = 0;  
        clone.modCount = 0;  

        // 将链表中所有节点的数据都添加到克隆对象中  
        for (Entry<E> e = header.next; e != header; e = e.next)  
            clone.add(e.element);  

        return clone;  
    }  

    // java.io.Serializable的写入函数  
    // 将LinkedList的“容量，所有的元素值”都写入到输出流中  
    private void writeObject(java.io.ObjectOutputStream s)  
        throws java.io.IOException {  
        // Write out any hidden serialization magic  
        s.defaultWriteObject();  

        // 写入“容量”  
        s.writeInt(size);  

        // 将链表中所有节点的数据都写入到输出流中  
        for (Entry e = header.next; e != header; e = e.next)  
            s.writeObject(e.element);  
    }  

    // java.io.Serializable的读取函数：根据写入方式反向读出  
    // 先将LinkedList的“容量”读出，然后将“所有的元素值”读出  
    private void readObject(java.io.ObjectInputStream s)  
        throws java.io.IOException, ClassNotFoundException {  
        // Read in any hidden serialization magic  
        s.defaultReadObject();  

        // 从输入流中读取“容量”  
        int size = s.readInt();  

        // 新建链表表头节点  
        header = new Entry<E>(null, null, null);  
        header.next = header.previous = header;  

        // 从输入流中将“所有的元素值”并逐个添加到链表中  
        for (int i=0; i<size; i++)  
            addBefore((E)s.readObject(), header);  
    }  

}
```
## 几点总结
关于LinkedList的源码，给出几点比较重要的总结：
1. 从源码中很明显可以看出，LinkedList的实现是基于双向循环链表的，且头结点中不存放数据，如下图;
![LinkedList](images/2016/03/Screen Shot 2016-03-03 at 22.27.57.png)
2. 注意两个不同的构造方法。无参构造方法直接建立一个仅包含head节点的空链表，包含Collection的构造方法，先调用无参构造方法建立一个空链表，而后将Collection中的数据加入到链表的尾部后面。
3. 在查找和删除某元素时，源码中都划分为该元素为null和不为null两种情况来处理，LinkedList中允许元素为null。
4. LinkedList是基于链表实现的，因此不存在容量不足的问题，所以这里没有扩容的方法。
5. 注意源码中的Entry<E> entry(int index)方法。该方法返回双向链表中指定位置处的节点，而链表中是没有下标索引的，要指定位置出的元素，就要遍历该链表，从源码的实现中，我们看到这里有一个加速动作。源码中先将index与长度size的一半比较，如果index<size/2，就只从位置0往后遍历到位置index处，而如果index>size/2，就只从位置size往前遍历到位置index处。这样可以减少一部分不必要的遍历，从而提高一定的效率（实际上效率还是很低）。
6. 注意链表类对应的数据结构Entry。如下;
```
// 双向链表的节点所对应的数据结构。  
// 包含3部分：上一节点，下一节点，当前节点值。  
private static class Entry<E> {  
    // 当前节点所包含的值  
    E element;  
    // 下一个节点  
    Entry<E> next;  
    // 上一个节点  
    Entry<E> previous;  

    /**  
     * 链表节点的构造函数。  
     * 参数说明：  
     *   element  —— 节点所包含的数据  
     *   next      —— 下一个节点  
     *   previous —— 上一个节点  
     */
    Entry(E element, Entry<E> next, Entry<E> previous) {  
        this.element = element;  
        this.next = next;  
        this.previous = previous;  
    }  
}  
```
7. LinkedList是基于链表实现的，因此插入删除效率高，查找效率低（虽然有一个加速动作）。
8. 要注意源码中还实现了栈和队列的操作方法，因此也可以作为栈、队列和双端队列来使用。


---------------

# Vector源码剖析
## Vector简介
Vector也是基于数组实现的，是一个动态数组，其容量能自动增长。

Vector是JDK1.0引入了，它的很多实现方法都加入了同步语句，因此是**线程安全**的（其实也只是相对安全，有些时候还是要加入同步语句来保证线程的安全），可以用于多线程环境。

Vector实现了Serializable接口，因此它支持序列化，实现了Cloneable接口，能被克隆，实现了RandomAccess接口，支持快速随机访问。

## Vector源码剖析
Vector的源码如下（加入了比较详细的注释）：
```
package java.util;  

public class Vector<E>  
    extends AbstractList<E>  
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable  
{  

    // 保存Vector中数据的数组  
    protected Object[] elementData;  

    // 实际数据的数量  
    protected int elementCount;  

    // 容量增长系数  
    protected int capacityIncrement;  

    // Vector的序列版本号  
    private static final long serialVersionUID = -2767605614048989439L;  

    // Vector构造函数。默认容量是10。  
    public Vector() {  
        this(10);  
    }  

    // 指定Vector容量大小的构造函数  
    public Vector(int initialCapacity) {  
        this(initialCapacity, 0);  
    }  

    // 指定Vector"容量大小"和"增长系数"的构造函数  
    public Vector(int initialCapacity, int capacityIncrement) {  
        super();  
        if (initialCapacity < 0)  
            throw new IllegalArgumentException("Illegal Capacity: "+  
                                               initialCapacity);  
        // 新建一个数组，数组容量是initialCapacity  
        this.elementData = new Object[initialCapacity];  
        // 设置容量增长系数  
        this.capacityIncrement = capacityIncrement;  
    }  

    // 指定集合的Vector构造函数。  
    public Vector(Collection<? extends E> c) {  
        // 获取“集合(c)”的数组，并将其赋值给elementData  
        elementData = c.toArray();  
        // 设置数组长度  
        elementCount = elementData.length;  
        // c.toArray might (incorrectly) not return Object[] (see 6260652)  
        if (elementData.getClass() != Object[].class)  
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);  
    }  

    // 将数组Vector的全部元素都拷贝到数组anArray中  
    public synchronized void copyInto(Object[] anArray) {  
        System.arraycopy(elementData, 0, anArray, 0, elementCount);  
    }  

    // 将当前容量值设为 =实际元素个数  
    public synchronized void trimToSize() {  
        modCount++;  
        int oldCapacity = elementData.length;  
        if (elementCount < oldCapacity) {  
            elementData = Arrays.copyOf(elementData, elementCount);  
        }  
    }  

    // 确认“Vector容量”的帮助函数  
    private void ensureCapacityHelper(int minCapacity) {  
        int oldCapacity = elementData.length;  
        // 当Vector的容量不足以容纳当前的全部元素，增加容量大小。  
        // 若 容量增量系数>0(即capacityIncrement>0)，则将容量增大当capacityIncrement  
        // 否则，将容量增大一倍。  
        if (minCapacity > oldCapacity) {  
            Object[] oldData = elementData;  
            int newCapacity = (capacityIncrement > 0) ?  
                (oldCapacity + capacityIncrement) : (oldCapacity * 2);  
            if (newCapacity < minCapacity) {  
                newCapacity = minCapacity;  
            }  
            elementData = Arrays.copyOf(elementData, newCapacity);  
        }  
    }  

    // 确定Vector的容量。  
    public synchronized void ensureCapacity(int minCapacity) {  
        // 将Vector的改变统计数+1  
        modCount++;  
        ensureCapacityHelper(minCapacity);  
    }  

    // 设置容量值为 newSize  
    public synchronized void setSize(int newSize) {  
        modCount++;  
        if (newSize > elementCount) {  
            // 若 "newSize 大于 Vector容量"，则调整Vector的大小。  
            ensureCapacityHelper(newSize);  
        } else {  
            // 若 "newSize 小于/等于 Vector容量"，则将newSize位置开始的元素都设置为null  
            for (int i = newSize ; i < elementCount ; i++) {  
                elementData[i] = null;  
            }  
        }  
        elementCount = newSize;  
    }  

    // 返回“Vector的总的容量”  
    public synchronized int capacity() {  
        return elementData.length;  
    }  

    // 返回“Vector的实际大小”，即Vector中元素个数  
    public synchronized int size() {  
        return elementCount;  
    }  

    // 判断Vector是否为空  
    public synchronized boolean isEmpty() {  
        return elementCount == 0;  
    }  

    // 返回“Vector中全部元素对应的Enumeration”  
    public Enumeration<E> elements() {  
        // 通过匿名类实现Enumeration  
        return new Enumeration<E>() {  
            int count = 0;  

            // 是否存在下一个元素  
            public boolean hasMoreElements() {  
                return count < elementCount;  
            }  

            // 获取下一个元素  
            public E nextElement() {  
                synchronized (Vector.this) {  
                    if (count < elementCount) {  
                        return (E)elementData[count++];  
                    }  
                }  
                throw new NoSuchElementException("Vector Enumeration");  
            }  
        };  
    }  

    // 返回Vector中是否包含对象(o)  
    public boolean contains(Object o) {  
        return indexOf(o, 0) >= 0;  
    }  


    // 从index位置开始向后查找元素(o)。  
    // 若找到，则返回元素的索引值；否则，返回-1  
    public synchronized int indexOf(Object o, int index) {  
        if (o == null) {  
            // 若查找元素为null，则正向找出null元素，并返回它对应的序号  
            for (int i = index ; i < elementCount ; i++)  
            if (elementData[i]==null)  
                return i;  
        } else {  
            // 若查找元素不为null，则正向找出该元素，并返回它对应的序号  
            for (int i = index ; i < elementCount ; i++)  
            if (o.equals(elementData[i]))  
                return i;  
        }  
        return -1;  
    }  

    // 查找并返回元素(o)在Vector中的索引值  
    public int indexOf(Object o) {  
        return indexOf(o, 0);  
    }  

    // 从后向前查找元素(o)。并返回元素的索引  
    public synchronized int lastIndexOf(Object o) {  
        return lastIndexOf(o, elementCount-1);  
    }  

    // 从后向前查找元素(o)。开始位置是从前向后的第index个数；  
    // 若找到，则返回元素的“索引值”；否则，返回-1。  
    public synchronized int lastIndexOf(Object o, int index) {  
        if (index >= elementCount)  
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);  

        if (o == null) {  
            // 若查找元素为null，则反向找出null元素，并返回它对应的序号  
            for (int i = index; i >= 0; i--)  
            if (elementData[i]==null)  
                return i;  
        } else {  
            // 若查找元素不为null，则反向找出该元素，并返回它对应的序号  
            for (int i = index; i >= 0; i--)  
            if (o.equals(elementData[i]))  
                return i;  
        }  
        return -1;  
    }  

    // 返回Vector中index位置的元素。  
    // 若index月结，则抛出异常  
    public synchronized E elementAt(int index) {  
        if (index >= elementCount) {  
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);  
        }  

        return (E)elementData[index];  
    }  

    // 获取Vector中的第一个元素。  
    // 若失败，则抛出异常！  
    public synchronized E firstElement() {  
        if (elementCount == 0) {  
            throw new NoSuchElementException();  
        }  
        return (E)elementData[0];  
    }  

    // 获取Vector中的最后一个元素。  
    // 若失败，则抛出异常！  
    public synchronized E lastElement() {  
        if (elementCount == 0) {  
            throw new NoSuchElementException();  
        }  
        return (E)elementData[elementCount - 1];  
    }  

    // 设置index位置的元素值为obj  
    public synchronized void setElementAt(E obj, int index) {  
        if (index >= elementCount) {  
            throw new ArrayIndexOutOfBoundsException(index + " >= " +  
                                 elementCount);  
        }  
        elementData[index] = obj;  
    }  

    // 删除index位置的元素  
    public synchronized void removeElementAt(int index) {  
        modCount++;  
        if (index >= elementCount) {  
            throw new ArrayIndexOutOfBoundsException(index + " >= " +  
                                 elementCount);  
        } else if (index < 0) {  
            throw new ArrayIndexOutOfBoundsException(index);  
        }  

        int j = elementCount - index - 1;  
        if (j > 0) {  
            System.arraycopy(elementData, index + 1, elementData, index, j);  
        }  
        elementCount--;  
        elementData[elementCount] = null; /* to let gc do its work */
    }  

    // 在index位置处插入元素(obj)  
    public synchronized void insertElementAt(E obj, int index) {  
        modCount++;  
        if (index > elementCount) {  
            throw new ArrayIndexOutOfBoundsException(index  
                                 + " > " + elementCount);  
        }  
        ensureCapacityHelper(elementCount + 1);  
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);  
        elementData[index] = obj;  
        elementCount++;  
    }  

    // 将“元素obj”添加到Vector末尾  
    public synchronized void addElement(E obj) {  
        modCount++;  
        ensureCapacityHelper(elementCount + 1);  
        elementData[elementCount++] = obj;  
    }  

    // 在Vector中查找并删除元素obj。  
    // 成功的话，返回true；否则，返回false。  
    public synchronized boolean removeElement(Object obj) {  
        modCount++;  
        int i = indexOf(obj);  
        if (i >= 0) {  
            removeElementAt(i);  
            return true;  
        }  
        return false;  
    }  

    // 删除Vector中的全部元素  
    public synchronized void removeAllElements() {  
        modCount++;  
        // 将Vector中的全部元素设为null  
        for (int i = 0; i < elementCount; i++)  
            elementData[i] = null;  

        elementCount = 0;  
    }  

    // 克隆函数  
    public synchronized Object clone() {  
        try {  
            Vector<E> v = (Vector<E>) super.clone();  
            // 将当前Vector的全部元素拷贝到v中  
            v.elementData = Arrays.copyOf(elementData, elementCount);  
            v.modCount = 0;  
            return v;  
        } catch (CloneNotSupportedException e) {  
            // this shouldn't happen, since we are Cloneable  
            throw new InternalError();  
        }  
    }  

    // 返回Object数组  
    public synchronized Object[] toArray() {  
        return Arrays.copyOf(elementData, elementCount);  
    }  

    // 返回Vector的模板数组。所谓模板数组，即可以将T设为任意的数据类型  
    public synchronized <T> T[] toArray(T[] a) {  
        // 若数组a的大小 < Vector的元素个数；  
        // 则新建一个T[]数组，数组大小是“Vector的元素个数”，并将“Vector”全部拷贝到新数组中  
        if (a.length < elementCount)  
            return (T[]) Arrays.copyOf(elementData, elementCount, a.getClass());  

        // 若数组a的大小 >= Vector的元素个数；  
        // 则将Vector的全部元素都拷贝到数组a中。  
    System.arraycopy(elementData, 0, a, 0, elementCount);  

        if (a.length > elementCount)  
            a[elementCount] = null;  

        return a;  
    }  

    // 获取index位置的元素  
    public synchronized E get(int index) {  
        if (index >= elementCount)  
            throw new ArrayIndexOutOfBoundsException(index);  

        return (E)elementData[index];  
    }  

    // 设置index位置的值为element。并返回index位置的原始值  
    public synchronized E set(int index, E element) {  
        if (index >= elementCount)  
            throw new ArrayIndexOutOfBoundsException(index);  

        Object oldValue = elementData[index];  
        elementData[index] = element;  
        return (E)oldValue;  
    }  

    // 将“元素e”添加到Vector最后。  
    public synchronized boolean add(E e) {  
        modCount++;  
        ensureCapacityHelper(elementCount + 1);  
        elementData[elementCount++] = e;  
        return true;  
    }  

    // 删除Vector中的元素o  
    public boolean remove(Object o) {  
        return removeElement(o);  
    }  

    // 在index位置添加元素element  
    public void add(int index, E element) {  
        insertElementAt(element, index);  
    }  

    // 删除index位置的元素，并返回index位置的原始值  
    public synchronized E remove(int index) {  
        modCount++;  
        if (index >= elementCount)  
            throw new ArrayIndexOutOfBoundsException(index);  
        Object oldValue = elementData[index];  

        int numMoved = elementCount - index - 1;  
        if (numMoved > 0)  
            System.arraycopy(elementData, index+1, elementData, index,  
                     numMoved);  
        elementData[--elementCount] = null; // Let gc do its work  

        return (E)oldValue;  
    }  

    // 清空Vector  
    public void clear() {  
        removeAllElements();  
    }  

    // 返回Vector是否包含集合c  
    public synchronized boolean containsAll(Collection<?> c) {  
        return super.containsAll(c);  
    }  

    // 将集合c添加到Vector中  
    public synchronized boolean addAll(Collection<? extends E> c) {  
        modCount++;  
        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacityHelper(elementCount + numNew);  
        // 将集合c的全部元素拷贝到数组elementData中  
        System.arraycopy(a, 0, elementData, elementCount, numNew);  
        elementCount += numNew;  
        return numNew != 0;  
    }  

    // 删除集合c的全部元素  
    public synchronized boolean removeAll(Collection<?> c) {  
        return super.removeAll(c);  
    }  

    // 删除“非集合c中的元素”  
    public synchronized boolean retainAll(Collection<?> c)  {  
        return super.retainAll(c);  
    }  

    // 从index位置开始，将集合c添加到Vector中  
    public synchronized boolean addAll(int index, Collection<? extends E> c) {  
        modCount++;  
        if (index < 0 || index > elementCount)  
            throw new ArrayIndexOutOfBoundsException(index);  

        Object[] a = c.toArray();  
        int numNew = a.length;  
        ensureCapacityHelper(elementCount + numNew);  

        int numMoved = elementCount - index;  
        if (numMoved > 0)  
        System.arraycopy(elementData, index, elementData, index + numNew, numMoved);  

        System.arraycopy(a, 0, elementData, index, numNew);  
        elementCount += numNew;  
        return numNew != 0;  
    }  

    // 返回两个对象是否相等  
    public synchronized boolean equals(Object o) {  
        return super.equals(o);  
    }  

    // 计算哈希值  
    public synchronized int hashCode() {  
        return super.hashCode();  
    }  

    // 调用父类的toString()  
    public synchronized String toString() {  
        return super.toString();  
    }  

    // 获取Vector中fromIndex(包括)到toIndex(不包括)的子集  
    public synchronized List<E> subList(int fromIndex, int toIndex) {  
        return Collections.synchronizedList(super.subList(fromIndex, toIndex), this);  
    }  

    // 删除Vector中fromIndex到toIndex的元素  
    protected synchronized void removeRange(int fromIndex, int toIndex) {  
        modCount++;  
        int numMoved = elementCount - toIndex;  
        System.arraycopy(elementData, toIndex, elementData, fromIndex,  
                         numMoved);  

        // Let gc do its work  
        int newElementCount = elementCount - (toIndex-fromIndex);  
        while (elementCount != newElementCount)  
            elementData[--elementCount] = null;  
    }  

    // java.io.Serializable的写入函数  
    private synchronized void writeObject(java.io.ObjectOutputStream s)  
        throws java.io.IOException {  
        s.defaultWriteObject();  
    }  
}

```
