# 瓜娃 Guava
---
Guava is the open-sourced version of Google's core Java libraries: the core utilities that Googlers use every day in their code. 

Guava has staggering numbers of unit tests: as of July 2012, the guava-tests package includes over 286,000 individual test cases. Most of these are automatically generated, not written by hand, but Guava's test coverage is extremely thorough, especially for `com.google.common.collect`.
## tablets
Strings, Collections, Concurrency, Basic Utilities


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
| [[Multiset|NewCollectionTypesExplained#multiset]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultiset.html'><code>ImmutableMultiset</code></a> |
| `SortedMultiset` | Guava         | <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/collect/ImmutableSortedMultiset.html'><code>ImmutableSortedMultiset</code></a> |
| [[Multimap|NewCollectionTypesExplained#multimap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableMultimap.html'><code>ImmutableMultimap</code></a> |
| `ListMultimap` | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableListMultimap.html'><code>ImmutableListMultimap</code></a> |
| `SetMultimap` | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableSetMultimap.html'><code>ImmutableSetMultimap</code></a> |
| [[BiMap|NewCollectionTypesExplained#bimap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableBiMap.html'><code>ImmutableBiMap</code></a> |
| [[ClassToInstanceMap|NewCollectionTypesExplained#classtoinstancemap]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableClassToInstanceMap.html'><code>ImmutableClassToInstanceMap</code></a> |
| [[Table|NewCollectionTypesExplained#table]] | Guava         | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/collect/ImmutableTable.html'><code>ImmutableTable</code></a> |





## References
[【guava wiki】https://github.com/google/guava/wiki][idx1]  
[idx1]: https://github.com/google/guava/wiki
