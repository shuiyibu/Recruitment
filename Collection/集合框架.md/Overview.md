![集合框架的接口](/images/2016/03/Framework Interface.png)

集合两个基本接口：`Collection` 和 `Map`

>`RandomAccess`接口可以用于检测一个特定的集合是否支持高效的随机访问

```
if(c instanceof RandomAccess)
{
	use random access algorithm
}
else
{
	use sequential access algorithm
}
```

![集合框架中的类](/images/2016/03/FrameWork Class.png)

# 视图与包装器
## 视图
### 轻量级集包装器
Arrays类的静态方法
>Returns a fixed-size list backed by the specified array. The returned list is serializable and implements RandomAccess.

```
String[] names = new String[20];
List<String> nameList=Arrays.asList(names);
nameList.add("Felix");
```
改变数组的所有方法（如：add或remove）都会抛出以下异常
```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.AbstractList.add(AbstractList.java:148)
	at java.util.AbstractList.add(AbstractList.java:108)
	at collection.framework.FrameworkTest.main(FrameworkTest.java:24)
```
This method also provides a convenient way to create a fixed-size list initialized to contain several elements:
```
List<String> stooges = Arrays.asList("Larry", "Moe", "Curly");
```
#### `collections.ncopy(n, nObject)`
```
List<String> list=Collections.nCopies(100, "default");
for (String string : list)
{
	System.out.println(string);
}
```
#### `Collections.singleton(anObject)`
```
ArrayList<String> list2=new ArrayList<>();
list2.add("Carl");
list2.add("Bounge");
list2.add("Miller");
list2.add("Carl");

Set<ArrayList<String>> set=Collections.singleton(list2);
```

### 子范围
`subList`

### 不可修改的视图
`unmodifiableCollection`
### 同步视图
>使用视图机制来确保常规集合的线程安全

```
Map<String, Employee> map=Collections.synchronizedMap(new HashMap<String,Employee>());
```
### 检查视图

### 可选操作

# 批操作
