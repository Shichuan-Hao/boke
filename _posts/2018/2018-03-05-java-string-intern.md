---
title: String#intern
author:
  name: superhsc
  link: https://github.com/Shichuan-hao
date: 2018-02-24 21:11:22 +0800
categories: [博客]
tags:  [值传递]
math: true
mermaid: true
---

在 JAVA 语言中有 8 种基本类型和一种较为特殊的类型 `String`。为了使这些类型在运行过程种速度更快，更节省内存，都提供了一种 **常量池（类似一个 JAVA 系统级别提供的缓存）**的概念。

8 种基本类型的常量池都是系统协调的，`String` 类型的常量池比较特殊。它的主要使用方法有两种：
- 直接使用双引号声明出来的 `String` 对象会直接存储在常量池中。
- 如果不是用双引号声明的 `String` 对象，可以使用 `String` 提供的 `intern` 方法。`intern` 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。

## `String#intern` 源代码

`String#intern` 的源代码如下：

```java
/**
 * Returns a canonical representation for the string object.
 * <p>
 * A pool of strings, initially empty, is maintained privately by the
 * class {@code String}.
 * <p>
 * When the intern method is invoked, if the pool already contains a
 * string equal to this {@code String} object as determined by
 * the {@link #equals(Object)} method, then the string from the pool is
 * returned. Otherwise, this {@code String} object is added to the
 * pool and a reference to this {@code String} object is returned.
 * <p>
 * It follows that for any two strings {@code s} and {@code t},
 * {@code s.intern() == t.intern()} is {@code true}
 * if and only if {@code s.equals(t)} is {@code true}.
 * <p>
 * All literal strings and string-valued constant expressions are
 * interned. String literals are defined in section 3.10.5 of the
 * <cite>The Java&trade; Language Specification</cite>.
 *
 * @return  a string that has the same contents as this string, but is
 *          guaranteed to be from a pool of unique strings.
 * @jls 3.10.5 String Literals
 */
public native String intern();
```

从上面 `String#intern` 方法代码中看到，这个方法是一个 native 方法，但是注释写的非常明了：“如果常量池中存在当前字符串，就会直接返回当前字符串。如果常量池中没有此字符串，会将此字符串放入常量池中后，再返回”。


## `native` 源代码

在 JDK 7 之后，oracle 接管 JAVA 源代码后就不对外开放了，根据 JDK 的主要开发人员声明 openJDK7 和 JDK7 使用的是同一份主代码，只是分支代码会有些许的变动。所以可以直接跟踪 openJDK7 的源码来探究 intern 的实现。


#### native 实现代码
`jdk/src/java.base/share/native/libjava/String.c`：

```c++
Java_java_lang_String_intern(JNIEnv *env, jobject this) {
    return JVM_InternString(env, this)
}
```

`jdk/src/hotspot/share/include/jvm.h`: 


```c++
/*
 * java.lang.String
 */
JNIEXPORT jstring JNICALL
JVM_InternString(JNIEnv *env, jstring str);
```

`jdk/src/hotspot/share/prims/jvm.cpp`：
```c++
// String support ///////////////////////////////////////////////////////////////////////////

JVM_ENTRY(jstring, JVM_InternString(JNIEnv *env, jstring str))
  JvmtiVMObjectAllocEventCollector oam;
  if (str == nullptr) return nullptr;
  oop string = JNIHandles::resolve_non_null(str);
  oop result = StringTable::intern(string, CHECK_NULL);
  return (jstring) JNIHandles::make_local(THREAD, result);
JVM_END
```


`jdk/src/hotspot/share/classfile/stringTable.cpp`：

```c++
oop StringTable::intern(Handle string_or_null_h, const jchar* name, int len, TRAPS) {
  // shared table always uses java_lang_String::hash_code
  // 共享表始终使用 java_lang_String::hash_code
  unsigned int hash = java_lang_String::hash_code(name, len);
  oop found_string = lookup_shared(name, len, hash);
  if (found_string != nullptr) {
    return found_string;
  }
  if (_alt_hash) {
    hash = hash_string(name, len, true);
  }
  found_string = do_lookup(name, len, hash);
  if (found_string != nullptr) {
    return found_string;
  }
  return do_intern(string_or_null_h, name, len, hash, THREAD);
}

oop StringTable::lookup_shared(const jchar* name, int len, unsigned int hash) {
  assert(hash == java_lang_String::hash_code(name, len), "hash must be computed using java_lang_String::hash_code");
  return _shared_table.lookup(name, hash, len);
}

unsigned int hash_string(const jchar* s, int len, bool useAlt) {
  return  useAlt ? AltHashing::halfsiphash_32(_alt_hash_seed, s, len) : java_lang_String::hash_code(s, len);
}

oop StringTable::do_lookup(const jchar* name, int len, uintx hash) {
  Thread* thread = Thread::current();
  StringTableLookupJchar lookup(thread, hash, name, len);
  StringTableGet stg(thread);
  bool rehash_warning;
  _local_table->get(thread, lookup, stg, &rehash_warning);
  update_needs_rehash(rehash_warning);
  return stg.get_res_oop();
}

oop StringTable::do_intern(Handle string_or_null_h, const jchar* name,
                           int len, uintx hash, TRAPS) {
  HandleMark hm(THREAD);  // cleanup strings created
  Handle string_h;

  if (!string_or_null_h.is_null()) {
    string_h = string_or_null_h;
  } else {
    string_h = java_lang_String::create_from_unicode(name, len, CHECK_NULL);
  }

  assert(java_lang_String::equals(string_h(), name, len),
         "string must be properly initialized");
  assert(len == java_lang_String::length(string_h()), "Must be same length");

  // Notify deduplication support that the string is being interned.  A string
  // must never be deduplicated after it has been interned.  Doing so interferes
  // with compiler optimizations done on e.g. interned string literals.
  if (StringDedup::is_enabled()) {
    StringDedup::notify_intern(string_h());
  }

  StringTableLookupOop lookup(THREAD, hash, string_h);
  StringTableGet stg(THREAD);

  bool rehash_warning;
  do {
    // Callers have already looked up the String using the jchar* name, so just go to add.
    WeakHandle wh(_oop_storage, string_h);
    // The hash table takes ownership of the WeakHandle, even if it's not inserted.
    if (_local_table->insert(THREAD, lookup, wh, &rehash_warning)) {
      update_needs_rehash(rehash_warning);
      return wh.resolve();
    }
    // In case another thread did a concurrent add, return value already in the table.
    // This could fail if the String got gc'ed concurrently, so loop back until success.
    if (_local_table->get(THREAD, lookup, stg, &rehash_warning)) {
      update_needs_rehash(rehash_warning);
      return stg.get_res_oop();
    }
  } while(true);
}
```

它的大体实现结构就是：JAVA 使用 jni 调用 C++ 实现的 `StringTable` 的 `intern` 方法，`StringTable` 的 `intern` 方法跟 Java 中的 `HashMap` 的实现类似，只是不能自动扩容，默认大小是 1009。

> String 的 String Pool 是一个固定大小的 `HashTable`，默认值大小是 1009，如果放进 String Pool 的 String 非常多，就会造成 Hash 冲突严重，进而导致链表会很长，而链表长了后造成的直接影响就是调用 `String.intern` 时性能会大幅下降（因为要一个一个找）。
{: .prompt-tip }

在 JDK6 中 `StringTable` 是固定的，就是 1009 的长度，所以如果常量池中的字符串过多就会导致效率下降很快。在 JDK7 中，`StringTable` 的长度可以通过 `-XX:StringTableSize=99991`  参数指定。

类似 `String s = new String("abc")` 这个语句创建了几个对象的题目主要就是为了考察对字符串对象的常量池掌握与否。语句 `String s = new String("abc")` 中创建了 2 个对象，第一个对象是 "abc" 字符串存储在常量池中，第二个对象在 JAVA Heap 中的 String 对象。 

例子 1 ：
```java
public static void main( String[] args ) {
    String s = new String("1");
    s.intern();
    String s2 = "1";
    System.out.println(s == s2);


    String s3 = new String("1") + new String("1");
    s3.intern();
    String s4 = "11";
    System.out.println(s3 == s4);
}
```
输出结果：

- jdk6：`false false`
- jdk7: `false true`

例子 2：将 `s.intern();` 放到 `String s2 = "1";` 后面，将 `s3.intern();` 放到 `String s4 = "11";` 后面，即：
```java
public static void main( String[] args ) {
    String s = new String("1");
    String s2 = "1";
    s.intern();
    System.out.println(s == s2);


    String s3 = new String("1") + new String("1");
    String s4 = "11";
    s3.intern();
    System.out.println(s3 == s4);
}
```

输出结果：
- jdk6: `false false`
- jdk7: `false false`


### jdk6 中的解释 

![jdk6 代码图](https://shichuan-hao.github.io/images/static/java/java-string-intern-jdk6.svg)

注：图中绿色线条代表 string 对象的内容指向。黑色线条代表地址指向。

首先说下 jdk6 中的情况，在 jdk6 中上述的所有打印都是 false 的。如上图，因为 jdk6 中的常量池是放在 Perm 区中的，Perm 区和正常的 JAVA Heap 区域是完全分开的。上面说过如果是使用引号声明的字符串都是会直接在字符串常量中生成，而 new 出来的 String 对象是放在 JAVA Heap 区。所以拿一个 JAVA Heap 区域的对象地址和字符串常量池对象地址进行比较肯定是不相同的，即使调用 `String.intern` 方法也是没有任何关系的。


### jdk7 中的解释

再说说 jdk7 中的情况。 需要明确一点：

> 在 jdk6 以及以前的版本中，字符串的常量池是放在堆的 Perm 区的，Perm 区是一个类静态的区域，主要存放一些加载类的信息，常量池，方法片段等内容，默认大小只有 4m，一旦常量池中大量使用 intern 是会直接产生 java.lang.OutOfMemoryError: PermGen space 错误的。基于这个原因，在 jdk7 的版本中，字符串常量池已经从 Perm 区移到正常的 Java Heap 区域了。为什么要移动，Perm 区域太小是一个主要原因，当然 jdk8 已经直接取消了 Perm 区域，而新建了一个元区域。应该是 jdk 开发者认为 Perm 区域已经不适合现在 JAVA 的发展了。
{: .prompt-tip }

正是因为字符串常量池移动到 JAVA Heap 区域后，出现上述输出结果就很容易明白了。


对于案例 1：

![jdk7 案例 1](https://shichuan-hao.github.io/images/static/java/java-string-intern-jdk7-1.svg)

1. 先看 s3 和 s4 字符串。`String s3 = new String("1") + new String("1");`，这句代码中现在生成了 2 个最终对象，是字符串常量池中的 “1” 和 JAVA Heap 中的 s3 引用指向的对象。中间还有 2 个匿名的 `new String("1")` 暂且不讨论。此时 s3 引用对象内容是 “11”，但此时常量池中是没有 “11” 对象的。
2. 接下来 `s3.intern();` 这一句代码，是将 s3 中的 “11” 字符串放入 String 常量池中，因为此时常量池中不存在 “11” 字符串，因此常规做法是跟 jdk6 图中表示的那样，在常量池中生成一个 “11” 的对象，关键点是 jdk7 中常量池不在 Perm 区域了，这块做了调整。常量池不需要再存储一份对象了，可以直接存储堆中的引用，这份引用指向 s3 引用的对象。也就是说 引用地址是相同的。
3. 最后 `String s4 = "11";`，这句代码中 “11” 是显示声明的，因此会直接区常量池中创建，创建的时候发现已经有这个对象了，此时也就是指向 s3 引用对象的一个引用。所以 s4 引用就指向和 s3 一样了。因此最后的比较  `s3 == s4` 是 true。
4. 再看 s 和 s2 对象。`String s = new String("1");` 第一句代码，生成了 2 个对象。常量池中的 “1” 和 JAVA Heap 中的字符串对象。`s.intern();` 这一句是 s 对象去常量池中寻找后发现 “1” 已经在常量池里了。
5. 接下来 `String s2 = "1"`, 这句代码是生成一个 s2 的引用指向常量池中的 “1” 对象。结果就是 s 和 s2 的引用地址明显不同。图中画的很清晰。


对于案例 2：

![jdk7 案例 2](https://shichuan-hao.github.io/images/static/java/java-string-intern-jdk7-2.svg)

1. 从上图观察。案例 1 和 案例 2 的改变就是 `s3.intern();` 的顺序是放在 `String s4 = "11";` 后了。这样，首先执行 `String s4 = "11";`，声明 s4 的时候常量池是不存在 “11” 对象的，执行完毕后，“11” 对象是 s4 声明产生的新对象。然后再执行  `s3.intern();` 时，常量池中 "11" 对象已经存在了，因此 s3 和 s4 的引用是不同的。
2. 案例 2 中的 s 和 s2 代码中，`s.intern()` 这一句代码往后放也不会有什么影响了，因为对象池中在执行第一句代码 `String s = new String("1");` 的时候已经生成 “1” 对象了。下边的 s2 声明都是直接从常量池中取地址引用的，所以 s 和 s2 的引用地址是不会相等的。

### 小结

从上述例子代码中可以看出 jdk7 版本对 intern 操作和常量池都做了一定的修改。主要包括 2 点：
1. 将 String 常量池从 Perm 区移动到了 Java Heap 区
2. `String#intern` 方法时，如果存在堆中的对象，会直接保存对象的引用，而不会重新创建对象。


## intern 正确使用例子

一个比较常见的使用 `String#intern` 方法的例子：

代码如下：

``
```



