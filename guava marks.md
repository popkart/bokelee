<link rel="stylesheet" href="http://yandex.st/highlightjs/6.2/styles/googlecode.min.css">
 
<script src="http://code.jquery.com/jquery-1.7.2.min.js"></script>
<script src="http://yandex.st/highlightjs/6.2/highlight.min.js"></script>
 
<script>hljs.initHighlightingOnLoad();</script>
<script type="text/javascript">
 $(document).ready(function(){
      $("h2,h3,h4,h5,h6").each(function(i,item){
        var tag = $(item).get(0).localName;
        $(item).attr("id","wow"+i);
        $("#category").append('<a class="new'+tag+'" href="#wow'+i+'">'+$(this).text()+'</a></br>');
        $(".newh2").css("margin-left",0);
        $(".newh3").css("margin-left",20);
        $(".newh4").css("margin-left",40);
        $(".newh5").css("margin-left",60);
        $(".newh6").css("margin-left",80);
      });
 });
</script>
<div id="category"></div>
# 瓜娃 Guava
---
Guava is the open-sourced version of Google's core Java libraries: the core utilities that Googlers use every day in their code. 

Guava has staggering numbers of unit tests: as of July 2012, the guava-tests package includes over 286,000 individual test cases. Most of these are automatically generated, not written by hand, but Guava's test coverage is extremely thorough, especially for `com.google.common.collect`.
## tablets
* [Strings](#Strings)
*  [Collections](#Collections)
*  [Hashing](#Hashing)
*  [Concurrency](#Concurrency)
*  [Basic Utilities](#Basic Utilities)


## Strings
优势：责任链式配置。

	String r = Joiner.on(";").withKeyValueSeparator(":").join(map);//出map
	Map<String, String> m  = Splitter.on(";").withKeyValueSeparator(":").split(r);//入map

	Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"

	Splitter.on(',')
       .trimResults()//trim结果数组中每个字符串
       .omitEmptyStrings()//忽略空字符串
       .splitToList("foo,bar,,   qux");

**CharMatcher**  
以往StringUtils对某一类**字符**的做法是，匹配上某个模式，然后对其操作。瓜娃用这个类来简化操作。直接上例子：

```
String noControl = CharMatcher.JAVA_ISO_CONTROL.removeFrom(string); // remove control characters
String theDigits = CharMatcher.DIGIT.retainFrom(string); // only the digits
String spaced = CharMatcher.WHITESPACE.trimAndCollapseFrom(string, ' ');
  // trim whitespace at ends, and replace/collapse whitespace into single spaces
String noDigits = CharMatcher.JAVA_DIGIT.replaceFrom(string, "*"); // star out all digits
String lowerAndDigit = CharMatcher.JAVA_DIGIT.or(CharMatcher.JAVA_LOWER_CASE).retainFrom(string);
  // eliminate all characters that aren't digits or lowercase
```

注意：是仅仅对匹配上的一系列**字符**进行操作，不是字符串！

还有`CharSets`这个类定义了6个常用的`CharSet`，jdk1.7之后建议使用`StandardCharsets`来指定编码集。

## Collections

### immutable collection
Guava provides simple, easy-to-use immutable versions of each standard  Collection  type, including Guava's own  Collection  variations.  
JDK也提供了自己的`Collections.unmodifiable*`类，但是它既笨重效率还低还不太安全(￣▽￣") 。如果你想要一个不变的集合，那么`defensively copy`它到immutable collection吧！注意，瓜娃的immutable collection不支持null。想用null请用JDK自带的那个。    
An `ImmutableXXX` collection 可通过下面方法创建:
  * using the `copyOf` method, `ImmutableSet.copyOf(set)`
  * using the `of` method, for example, `ImmutableSet.of("a", "b", "c")` or `ImmutableMap.of("a", 1, "b", 2)`
  * using a `Builder`,
```
public static final ImmutableSet<Color> GOOGLE_COLORS =
       ImmutableSet.<Color>builder()//这个语法
           .addAll(WEBSAFE_COLORS)
           .add(new Color(0, 191, 255))
           .build();
```

在创建集合时顺序已经确定：
```
ImmutableSet.of("a", "b", "c", "a", "d", "b")
```
will iterate over its elements in the order "a", "b", "c", "d".

### 瓜娃的Collection列表 

| Interface | JDK or Guava? | Immutable Version |
|:----------|:--------------|:------------------|
| `Collection` | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableCollection.html'><code>ImmutableCollection</code></a> |
| `List`    | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableList.html'><code>ImmutableList</code></a> |
| `Set`     | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSet.html'><code>ImmutableSet</code></a> |
| `SortedSet`/`NavigableSet` | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedSet.html'><code>ImmutableSortedSet</code></a> |
| `Map`     | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMap.html'><code>ImmutableMap</code></a> |
| `SortedMap` | JDK           | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSortedMap.html'><code>ImmutableSortedMap</code></a> |
| [[Multiset]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html'><code>ImmutableMultiset</code></a> |
| `SortedMultiset` | Guava         | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/ImmutableSortedMultiset.html'><code>ImmutableSortedMultiset</code></a> |
| [[Multimap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html'><code>ImmutableMultimap</code></a> |
| `ListMultimap` | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html'><code>ImmutableListMultimap</code></a> |
| `SetMultimap` | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html'><code>ImmutableSetMultimap</code></a> |
| [[BiMap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html'><code>ImmutableBiMap</code></a> |
| [[ClassToInstanceMap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html'><code>ImmutableClassToInstanceMap</code></a> |
| [[Table]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a> |

## Hashing
java的hash比较简单快速，但是冲突比较高。在一些需要低冲突率的场景下，比如`BloomFilter`，我们有`com.google.common.hash`。  

### HashFunction接口
把序列（bits，string）映射到bits序列（其实就是hash了）。他代表了一个hash的方法，如MD5、sha、crc32等。  

| <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#md5()'><code>md5()</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#murmur3_128()'><code>murmur3_128()</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#murmur3_32()'><code>murmur3_32()</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#sha1()'><code>sha1()</code></a> |
|:----------------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------|
| <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#sha256()'><code>sha256()</code></a> | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#sha512()'><code>sha512()</code></a>           | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/hash/Hashing.html#goodFastHash(int)'><code>goodFastHash(int bits)</code></a> |


以下以MD5为例。`Hashing`可产生它。它是无状态的。它可以产生有状态的`Hasher`。当然HashFunction也可以直接产生hash

	HashFunction hf = Hashing.md5();
	String hs = hf.hashString("dfdfdferf", Charset.defaultCharset()).toString();
###　Hasher接口
它拥有流式方法来添加任意的内容到当前hash上下文（bits，数组，字符串，`对象`等等都可以），然后得出其hash。
	
	Hasher her = hf.newHasher();
	HashCode hc = her.putLong(123)
		   		   	.putString("popkart", Charsets.UTF_8)
				     .hash();
怎么添加对象？添加对象的时候要指定怎么计算这个对象的hash，就是`Funnel`干的事。

### Funnel接口
比如我们添加一个对象`Person`，定义如下：

```
class Person {
  final int id;
  final String firstName;
  final String lastName;
  final int birthYear;
}

```
我们的`Funnel`需要定义如下：

```
Funnel<Person> personFunnel = new Funnel<Person>() {
  @Override
  public void funnel(Person person, PrimitiveSink into) {
    into
      .putInt(person.id)
      .putString(person.firstName, Charsets.UTF_8)
      .putString(person.lastName, Charsets.UTF_8)
      .putInt(birthYear);
  }
};
//然后就能这样写了
 hc.put(new Person(),personFunnel);
```
这个做法类似对象的`toString()`方法。  
_Note:_ `putString("abc", Charsets.UTF_8).putString("def", Charsets.UTF_8)` is fully equivalent to `putString("ab", Charsets.UTF_8).putString("cdef", Charsets.UTF_8)`, because they produce the same byte sequence.  This can cause unintended hash collisions.  Adding separators of some kind can help eliminate unintended hash collisions.最终都是变成字节流，然后进行处理。`PrimitiveSink`代表那个字节流对象。  
guava由`Funnels`类来提供默认的`Funnel`（int/long/string/byte[]等），如：

	Funnel<CharSequence> fl = Funnels.stringFunnel(Charsets.UTF_8);




### HashCode
`HashCode`可由`Hasher`调用`hash()`方法得出，注意该方法只能被调用一次，也就是一个`Hasher`调用一次`hash()`之后就废了。`HashCode`代表了之前那个字节序列的哈希值，它有一些方法来表示这个哈希值，如`asBytes()`（128bits，16字节）、`asInt()`（前4字节转换成的int值）、`toString()`（32个字符长度的16进制表示）等。HashCode还提供了`fromBytes()`，`fromString()`，`fromLong()`的功能，可以用于哈希值表示形式的转换。

### BloomFilter
guava提供了一个内置`BloomFilter`，我们只用指定一个`Funnel`来解析要进行`BloomFilter过滤`的对象即可。You can obtain a fresh [BloomFilter&lt;T&gt;](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html) with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#create(com.google.common.hash.Funnel, int, double)'><code>create(Funnel funnel, int expectedInsertions, double falsePositiveProbability)</code></a>, or just accept the default false probability of 3%. `BloomFilter<T>` offers the methods `boolean mightContain(T)` and `void put(T)`。  
以字符串为例：

```
		Funnel<CharSequence> fl = Funnels.stringFunnel(Charsets.UTF_8);
		//1亿占用100多M内存。1亿根据公式大概需要10亿位，10^9/8≈125M
		BloomFilter<String> blf = BloomFilter.create(fl, 100000000, 0.01);
		blf.put("abc");
		System.out.println(blf.mightContain("abc"));
```

## References
[【guava wiki】https://github.com/google/guava/wiki][idx1]  
[idx1]: https://github.com/google/guava/wiki
