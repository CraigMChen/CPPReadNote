# 第16章 string类和标准模板库

## 1.string类

表16.1列出了string类的几个构造函数。其中NBTS(null-terminated string)表示以空字符结束的传统C-风格字符串。size_type是一个依赖于实现的整型，是在头文件string中定义的。<font color = #FF0000>string类将string::npos定义为字符串的最大长度，通常为unsigned int的最大值</font>。

<center>表16.1 string类的构造函数</center>

| 构造函数                                                     |                             描述                             |
| ------------------------------------------------------------ | :----------------------------------------------------------: |
| string (const char * s)                                      |                将string对象初始化为指向的NBTS                |
| string(size_type n, char c)                                  | 创建一个包含n个元素的string对象，其中每个元素都被初始化为字符c |
| string(const string & str)                                   |    将一个string对象初始化为string对象str（复制构造函数）     |
| string()                                                     |              创建一个默认的string对象，长度为0               |
| string(const char * s, size_type n)                          | 将string对象初始化为s指向的NBTS的前n个字符，即使超过了NBTS的结尾 |
| template\<class Iter> string(Iter begin, Iter end)           | 将string对象初始化为区间[begin, end)内的字符，其中begin和end是迭代器 |
| string(const string & str, string size_type pos = 0, size_type n = npos) | 将一个string对象初始化为对象str从位置pos开始到结尾的字符，或从位置pos开始的n个字符 |
| string(string && str) noexcept                               | 这是C++11新增的，它将一个string对象初始化为string对象str，并可能修改str（移动构造函数） |
| string(initializer_list\<char> il)                           | 这是C++11新增的，它将一个string初始化为初始化列表il中的字符  |

对于C-风格字符串，有3种输入方式：

```C++
char info[100];
cin >> info;					//读入一个单词
cin.getline(info, 100);			//读入一行，丢弃‘\n’
cin.get(info, 100);				//读入一行，将‘\n’保留在队列中
```

string对象有两种输入方式：

```C++
string stuff;
cin >> stuff;			//读入一个单词
getline(cin, stuff);	//读入一行，丢弃‘\n’
```

两个版本的getline()都有一个可选参数，用于指定使用哪个字符来确定输入的边界：

```C++
cin.getline(info, 100, ‘:’);		//读到‘:’，丢弃‘:’
getline(stuff, ‘:’);				//读到‘:’，丢弃‘:’
```

<font color = #FF0000>注意，在指定分界字符后，换行符将被视为常规字符。</font>

二者的主要区别是，<font color = #FF0000>string版本的getline()将自动调整目标string对象的大小，使之刚好能够存储输入的字符</font>。另一个区别是，读取C-风格字符串的函数是istream类的方法，而string版本是独立的函数。

string版本的getline()函数从输入中读取字符，并将其存储到目标string中，直到发生下列三种情况之一：

* 到达文件尾，在这种情况下，输入流的eofbit将被设置，这意味着方法fail()和eof()都将返回true；

* 遇到分界字符（默认为\n），在这种情况下，将把分界字符从输入流中删除，但不存储它；

* 读取的字符数达到最大允许值（string::npos和可供分配的内存字节数中较小的哪一个），在这种情况下，将设置输入流的failbie，这意味着方法fail()将返回true。

string类对全部6个关系运算符都进行了重载。可以直接对字符串进行比较。对于每个关系运算符，都以三种方式被重载，以便能够将string对象与另一个string对象或C-风格字符串进行比较，并能够将C-风格字符串与string对象进行比较。

可以以多种不同的方式在字符串中搜索给定的自字符串或字符。表16.2简要地描述了find()方法的4个版本。

<center>表16.2 重载的find()方法</center>

| 方法原型                                                     |                             描述                             |
| :----------------------------------------------------------- | :----------------------------------------------------------: |
| size_type find(const string & str, size_type pos = 0) const  | 从字符串的pos位置开始，查找子字符串str。如果找到，则返回该子字符串首次出现时其首字符的索引；否则，返回string::npos |
| size_type find(const char * s, size_type pos = 0) const      | 从字符串的pos位置开始，查找子字符串s。如果找到，则返回该子字符串首次出现时其首字符的索引；否则，返回string::npos |
| size_type find(const char * s, size_type pos = 0, size_type n) | 从字符串的pos位置开始，查找s的前n个字符组成的子字符串。如果找到，则返回该子字符串首次出现时其首字符的索引；否则，返回string::npos |
| size_type find(char ch, size_type pos = 0) const             | 从字符串的pos位置开始，查找字符ch。如果找到，则返回该字符首次出现的位置；否则，返回string::npos |

string库还提供了相关的方法，它们的重载函数特征标都与find()方法相同：

* rfind()方法查找子字符串或字符最后一次出现的位置

* find_first_of()方法在字符串中查找参数中任何一个字符首次出现的位置

* find_last_of()方法在字符串中查找参数中任何一个字符最后一次出现的位置

* find_first_not_of()方法在字符串中查找第一个不包含在参数中的字符

* find_last_not_of()方法在字符串中查找最后一个不包含在参数中的字符

方法capacity()返回当前分配给字符串的内存块大小。方法reserve()能够请求内存块的最小长度。方法c_str()返回一个指向C-风格的指针，该C-风格字符串的内容与用于调用c_str()方法的string对象相同。



## 2.智能指针模板类

<font color = #FF0000>智能指针模板auto_ptr、unique_ptr、和shared_ptr都定义了类似指针的对象，可以将new获得（直接或间接）的地址赋给这种对象。当智能指针过期时，其析构函数将使用delete来释放内存</font>。因此，如果将new返回的地址赋给这些对象，将无需记住稍后将释放这些内存：在智能指针过期时，这些内存将被自动释放。

<font color = #FF0000>头文件memory定义了智能指针模板类，位于名称空间std中。</font>

看下面的赋值语句：

```C++
auto_ptr<string> ps (new string(“I reigned lonely as a cloud.”);
auto_ptr<string> vocation;
vocation = ps;
```

如果ps和vocation是常规指针，则两个指针将指向同一个string对象。这是不能接受的，因为程序将师徒删除同一个对象两次——一次是ps过期时，另一次是vocation过期时。要避免这种问题，方法有多种：

* 定义赋值运算符，使之执行深复制。这样两个指针将指向不同的对象，其中的一个对象是另一个对象的副本。

* <font color = #FF0000>建立所有权概念，对于特定的对象，只能有一个只能指针可拥有它，这样只有拥有对象的智能指针的析构函数会删除该对象。然后，让赋值操作转让所有权。这就是auto_ptr和unique_ptr的策略，但unique_ptr更严格。</font>

* <font color = #FF0000>创建智能更高的指针，跟踪引用特定对象的智能指针数。这称为引用计数</font>。例如，赋值时，计数将加1，而指针过期时，计数将减1。仅当最后一个指针过期时，才调用delete。<font color = #FF0000>这是shared_ptr采用的策略</font>。

下面是auto_ptr和unique_ptr的区别：

```C++
auto_ptr<string> p1(new stirng(“auto”));	//#1
auto_ptr<string> p2;						//#2
p2 = p1;									//#3
```

在语句#3中，p2接管string对象的所有权后，p1的所有权将被剥夺。这导致p1将是一个空指针。如果程序试图使用p1，会导致运行阶段错误。

下面来看使用unique_ptr的情况：

```C++
unique_ptr<string> p3(new string(“auto”));	//#4
unique_ptr<string> p4;						//#5
p4 = p3;									//#6
```

编译器认为语句#6非法，即出现编译阶段错误。避免了p3不在指向有效数据的问题。因此，unique_ptr比auto_ptr更安全。

但有时候，将一个智能指针赋给另一个并不会留下危险的悬挂指针：

```C++
unique_ptr<string> demo(const char * s)
{
	unique_ptr<string> temp(new string(s));
	return temp;
}

unique_ptr<string> ps;
ps = demo(“Uniquely special”);
```

总之，程序将一个unique_ptr赋给另一个时，<font color = #FF0000>如果源unique_ptr是一个临时右值，编译器允许这样做；如果源unique_ptr将存在一段时间，编译器将禁止这样做</font>。

相比于auto_ptr，unique_ptr还有另一个优点。必须将delete和new配对，将delete[]与new[]配对。而模板auto_ptr使用delete而不是delete[]，因此只能与new一起用，而不能与new[]一起使用。<font color = #FF0000>但unique_ptr有使用new[]和delete[]的版本</font>：

```C++
std::unique_ptr<double []> pda(new double(5));
```

那么，如何选择智能指针呢？<font color = #FF0000>如果程序要使用多个指向同一个对象的指针，应选择shared_ptr</font>。这样的情况包括：有一个指针数组，并使用一些辅助指针来标识特定的元素，如最大的元素和最小的元素；两个对象包含都指向第三个对象的指针；STL容器包含指针。<font color = #FF0000>如果程序不需要多个指向同一个对象的指针，则可使用unique_ptr。</font>

在unique_ptr为右值时，可将其赋给shared_ptr。

```C++
unique_ptr<int> make_int(int n)
{
	return unique_ptr<int>(new int(n));
}
unique_ptr<int> pup(make_int(rand() % 1000);		//合法
shared_ptr<int> spp(pup);							//非法
shared_ptr<int> spr(make_int(rand() % 1000);		//合法
```

模板shared_ptr包含一个显式构造函数，可用于将右值unique_ptr转换为shared_ptr。shared_ptr将接管原来归unique_ptr所有的对象。



## 3.标准模板库（STL）

STL提供了一组表示容器、迭代器、函数对象和算法的模板、STL使得能够构造各种容器和执行各种操作。下面介绍STL中的vector类。

vector类定义在头文件vector中，并放在名称空间std中。要创建vector对象，可使用通常的\<type>表示法来指出要使用的类型。vector模板使用动态内存分配，可以用初始化参数来指出需要多少矢量：

```C++
vector<int> ratings(5);
```

所有的STL容器都提供了一些基本方法，其中包括size()——返回容器中元素数目、swap()——交换两个容器中的内容、begin()——返回一个指向容器中第一个元素的迭代器、end()——返回一个表示超过容器尾的迭代器。

<font color = #FF0000>迭代器是一个广义指针。它可以是指针，也可以是一个可对其执行类似指针的操作的对象，如解除引用和递增</font>。每个容器类都声明了一个合适的迭代器，该迭代器的类型是一个名为iterator的typedef，其作用域为整个类。例如，要为vector的double类型规范声明一个迭代器：

```C++
vector<double>::iterator pd;
```

则可以对pd执行这样的操作：

```C++
vector<double> scores;
pd = scores.begin();
*pd = 22.3;
++pd;
```

vector模板类也包含一些只有某些STL容器才有的方法。

push_back()方法将元素添加到矢量末尾：

```C++
scores.push_back(95.0);
```

erase()方法删除矢量中给定区间的元素，它接受两个迭代器参数，这些参数定义了要删除的区间:

```C++
scores.erase(scores.begin(), scores.begin() + 2);
```

insert()方法的功能与erase()方法相反，它接受3个迭代器参数，第一个参数指定了新元素的插入位置，第二个和第三个迭代器参数定义了被插入区间，该区间通常是另一个容器对象的一部分：

```C++
vector<int> old_v;
vector<int> new_v;
//...
old_v.insert(old_v.begin(), new_v.begin() + 1, new_v.end());
```

下面来看3个具有代表性的STL函数：for_each()、random_shuffle()和sort()。

for_each()函数可用于很多容器类，它接受3个参数。前两个是定义容器中区间的迭代器，最后一个是指向函数的指针（函数对象）。for_each()函数将被指向的函数应用于容器区间中的各个元素。被指向的函数不能修改容器元素的值。假设定义了一个Review结构，一个ShowReview函数，用于显示Review对象的内容，那么可以这样利用for_each()函数：

```C++
vector<Review> books;
for_each(books.begin(), books.end(), ShowReview);
```

random_shuffle()函数接受两个指定区间的迭代器参数，并随机排列该区间中的元素。该函数要求容器类允许随机访问。例如，下面的语句随机排列books矢量中的所有元素：

```C++
random_shffle(books.begin(), books.end());
```

sort()函数也要求容器支持随机访问。该函数有两个版本，第一个版本接受两个定义区间的迭代器参数，并使用为存储在容器中的类型元素定义的<运算符，对区间中的元素进行操作。例如，下面的语句按升序对coolstuff的内容进行排序，排序时使用内置的<运算符值进行比较：

```C++
vector<int> coolstuff;
//...
sort(coolstuff.begin(), coolstuff.end());
```

另一种版本的sort()接受3个参数，前两个参数也是指定区间的迭代器，最后一个参数是指向要使用的函数的指针（函数对象），而不是用于比较的operator<()。返回值可转换为bool，false表示两个参数的顺序不正确。

有一种循环是为STL而设计的，这种循环是<font color = #FF0000>基于范围的for循环</font>。在这种for循环中，括号内的代码表明一个类型与容器存储的内容相同的变量，然后指出了容器的名称。接下来，循环体使用指定的变量一次访问容器的每个元素：

```C++
for (auto x : books) ShowReview(x);
```

不同于for_each，基于范围的for循环可以修改容器的内容，办法是指定一个引用参数，假设InflateReview函数是一个修改Review对象内容的函数：

```C++
void InflateReview(Review & x);
for(auto & x : books) InflateReview(x);
```



## 4.迭代器

STL为使算法能够适用于具体情况，应定义能够满足算法需求的迭代器，并把要求加到容器设计上。即基于算法的要求，设计基本迭代器的特征和容器特征。STL定义了5种迭代器：

* 输入迭代器

术语“输入”是从程序的角度说的，即来自容器的信息被视为输入。对输入迭代器解除引用将使程序能够读取容器中的值，但不一定能让程序修改值。因此，<font color = #FF0000>需要输入迭代器的算法将不会修改容器中的值</font>。输入迭代器能够访问容器中所有的值，这是通过支持++运算符（前缀格式和后缀格式）来实现的。<font color = #FF0000>不能保证输入迭代器第二次遍历容器时，顺序不变。另外，输入迭代器被递增后，也不能保证其先前的值仍然可以被解除引用</font>。基于输入迭代器的任何算法都应当是单通行的，不依赖于前一次遍历时的迭代器值，也不依赖于本次遍历中前面的迭代器值。注意，<font color = #FF0000>输入迭代器是单向迭代器，可以递增，但不能倒退</font>。

* 输出迭代器

术语“输出”指用于将信息从程序传输给容器。输出迭代器与输入迭代器相似，只是解除引用让程序能修改容器值，而不能读取。简而言之，<font color = #FF0000>对于单通行、只读算法，可以使用输入迭代器；而对于单通行、指写算法，则可以使用输出迭代器</font>。

* 正向迭代器

与输入迭代器和输出迭代器相似，正向迭代器只使用++运算符来遍历容器。然而，与输入和输出迭代器不同的是，它<font color = #FF0000>总是按相同的顺序遍历一系列值。另外，将正向迭代器递增后，仍然可以对前面的迭代器值解除引用（如果保存了它），并可以得到相同的值。正向迭代器使得能够读取和修改数据，也使得只能读取数据</font>。

* 双向迭代器

<font color = #FF0000>双向迭代器具有正向迭代器的所有特性，并同时支持两种（前缀和后缀）的递减运算符。</font>

* 随机访问迭代器

<font color = #FF0000>随机访问迭代器具有双向迭代器的所有特性，同时添加了支持随机访问的操作和用于对元素进行排序的关系运算符</font>。表16.3列出了除双向迭代器的操作外，随机访问迭代器还支持的操作。其中，a和b都是迭代器值，n为整数，r为随机迭代器变量或引用。

<center>表16.3 随机访问迭代器操作</center>

| 表达式 |              描述              |
| :----: | :----------------------------: |
| a + n  | 指向a所指向的元素后的第n个元素 |
| n + a  |              同上              |
| a - n  | 指向a所指向的元素前的第n个元素 |
| r += n |        等价于r = r + n         |
| r -= n |        等价于r = r - n         |
|  a[n]  |         等价于*(a + n)         |
| b - a  |  结果为这样的值，即b = a + n   |
| a < b  |     如果b - a > 0，则为真      |
| a > b  |       如果b < a，则为真        |
| a >= b |      如果!(a < b)，则为真      |
| a <= b |      如果!(a > b)，则为真      |

像a+n这样的表达式仅当a和a+n都位于容器区间（包括超尾）内时才合法。

迭代器类型形成了一个层次结构。正向迭代器具有输入迭代器和输出迭代器的全部功能，同时还有自己的功能；双向迭代器具有正向迭代器的全部功能，同时还有自己的功能；随机访问迭代器具有正向迭代器的全部功能，同时还具有自己的功能。表16.4总结了主要的迭代器功能。其中i为迭代器，n为整数。

<center>表16.4 迭代器性能</center>

|    迭代器功能    | 输入 | 输出 | 正向 | 双向 | 随机访问 |
| :--------------: | :--: | :--: | :--: | :--: | :------: |
|   解除引用读取   |  有  |  无  |  有  |  有  |    有    |
|   解除引用写入   |  无  |  有  |  有  |  有  |    有    |
| 固定和可重复写入 |  无  |  无  |  有  |  有  |    有    |
|     ++i, i++     |  有  |  有  |  有  |  有  |    有    |
|     --i, i--     |  无  |  无  |  无  |  有  |    有    |
|       i[n]       |  无  |  无  |  无  |  无  |    有    |
|      i + n       |  无  |  无  |  无  |  无  |    有    |
|      i - n       |  无  |  无  |  无  |  无  |    有    |
|      i += n      |  无  |  无  |  无  |  无  |    有    |
|      i -= n      |  无  |  无  |  无  |  无  |    有    |

STL使用术语<font color = #FF0000>概念</font>来描述一系列的要求。用术语改进来表示概念上的继承。概念的具体实现被称为模型。

算法<font color = #FF0000>copy()</font>可以将数据从一个容器复制到另一个容器中。这种算法是以迭代器方式实现的，所以它可以从一种容器到另一种容器进行复制，甚至可以在数组之间复制。因为可以将指向数组的指针用作迭代器。下面的代码将一个数组复制到一个矢量中：

```C++
int casts[10] = {6, 7, 2, 9, 4, 11, 8, 7, 10, 5};
vector<int> dice[10];
copy(casts, casts + 10, dice.begin());
```

copy()函数将覆盖目标容器中已有的数据，同时目标容器必须足够大，以便能够容纳被复制的元素。

STL提供了<font color = #FF0000>ostream_iterator</font>模板，该模板是输出迭代器的一个模型。可以通过包含头文件iterator并作下面的声明来创建这种迭代器：

```C++
#include <iterator>
//...
ostream_iterator<int, char> out_iter(cout, “ ”);
```

out_iter现在是一个接口，能使用cout来显式信息。<font color = #FF0000>第一个模板参数指出了被发送给输出流的数据类型；第二个模板参数指出了输出流使用的字符类型（另一个可能的值是wchar_t）。构造函数的第一个参数指出了要使用的输出流，它也可以是用于文件输出的流；构造函数的第二个参数指出了在发送给输出流的每个数据项后显式的分隔符</font>。

可以这样使用迭代器：

```C++
*out_iter++ = 15;
```

这意味着将15和由空格组成的字符串发送到cout管理的输出流中，并为下一个输出做好了准备。可以将copy()用于该迭代器：

```C++
copy(dice.begin(), dice.end(), out_iter);
```

这意味着将dice容器的整个区间复制到输出流中，即显式容器的内容。

iterator头文件还定义了一个istream_iterator模板，使istream输入可用作迭代器接口。它是一个输入迭代器概念的模型，可以使用两个isrteam_iterator对象来定义copy()的输入范围：

```C++
copy(istream_iterator<int, char>(cin), 
     							istream_iterator<int, char>(), dice.begin());
```

<font color = #FF0000>第一个模板参数指出要读取的数据类型，第二个模板参数指出输入流使用的字符类型。使用构造函数参数cin意味着使用cin管理的输入流，省略构造函数参数表示输入失败，因此上述代码从输入流中读取，直到文件结尾、类型不匹配或出现其他输入的故障为止。</font>

除了ostream_iterator和istream_iterator之外，头文件iterator还提供了其他一些专用的预定义迭代器类型，它们是reverse_iterator、back_insert_iterator、front_insert_iterator和insert_iterator。

<font color = #FF0000>reverse_iterator是反向迭代器。对reverse_iterator执行递增操作将导致它被递减。</font>vector类有一个名为rbegin()的成员函数和一个名为rend()的成员函数，前者返回一个指向超尾的反向迭代器，后者返回一个指向第一个元素的反向迭代器。可以使用下面的语句来反向显示内容：

```C++
copy(dice.rbegin(), dice.rend(), out_iter);
```

假设rp是一个被初始化为dice.rbegin()的反向迭代器。<font color = #FF0000>\*rp将在\*rp的当前值之前对迭代器执行解除引用</。也就是说，如果rp指向位置6，则*rp将是位置5的值，以此类推。

<font color = #FF0000>back_insert_iterator将元素插入到容器尾部，而front_insert_iterator将元素插入到容器的前端。insert_iterator将元素插入到insert_iterator构造函数的参数指定的位置前面。</font>

这些迭代器将容器类型作为模板参数，将实际的容器标识符作为构造函数参数。也就是说，要为名为dice的vector<int>容器创建一个back_insert_iterator，可这样做：

```C++
back_insert_iterator<vector<int> > back_iter(dice);
```



## 5.容器

STL具有容器概念和容器类型。概念是具有名称的通用类别；容器类型是可用于创建具体容器对象的模板。以前的11个容器类型分别是<font color = #FF0000>deque、list、queue、priority_queue、stack、vector、map、multimap、set、multiset、bitset</font><font color = #FF0000>。C++新增了forward_list、unordered_map、unorder_multimap、unordered_set和unordered_multiset</font>。

<font color = #FF0000>容器是存储其他对象的对象。被存储的对象必须是同一种类型的，它们可以是OOP意义上的对象，也可以是内置类型值</font>。存储在容器中的数据为容器所有，这意味着当容器过期时，存储在容器中的数据也将过期（然而，如果数据是指针的话，则它指向的数据并不一定过期）。

基本容器不能保证其元素都按特定的顺序存储，也不能保证元素的顺序不变，但对概念进行改进后，则可以增加这样的保证。所有的容器都提供某些特征和操作。表16.5对一些通用特征进行了总结。其中，X表示容器类型；T表示存储在容器中的对象类型；a和b表示类型为X的值；r表示类型为X&的值；u表示类型为X的标识符。

<center>表16.5 一些基本的容器特征</center>

|     表达式     |     返回类型      |                             说明                             |  复杂度  |
| :------------: | :---------------: | :----------------------------------------------------------: | :------: |
|  X::iterator;  | 指向T的迭代器类型 |                满足正向迭代器要求的任何迭代器                | 编译时间 |
| X::value_type; |         T         |                           T的类型                            | 编译时间 |
|      X u;      |                   |                    创建一个名为u的空容器                     |   固定   |
|      X();      |                   |                     创建一个匿名的空容器                     |   固定   |
|    X u(a);     |                   |                   调用复制构造函数后u == a                   |   线性   |
|    X u = a;    |                   |                        作用同X u(a);                         |   线性   |
|     r = a;     |        X&         |                    调用赋值运算符后r == a                    |   线性   |
|   (&a)->~X()   |       void        |                 对容器中每个元素调用析构函数                 |   线性   |
|   a.begin()    |      迭代器       |                返回指向容器第一个元素的迭代器                |   固定   |
|    a.end()     |      迭代器       |                       返回超尾值迭代器                       |   固定   |
|    a.size()    |    无符号整型     |           返回元素个数，等价于a.end() - a.begin()            |   固定   |
|   a.swap(b)    |       void        |                        交换a和b的内容                        |   固定   |
|     a == b     |   可转换为bool    | 如果a和b的长度相同，且a中每个元素都等于b中相应的元素，则为真 |   线性   |
|     a != b     |   可转换为bool    |                        返回!(a == b)                         |   线性   |

表16.6列出了C++新增的通用容器要求。其中，rv表示类型为X的非常量右值，如函数的返回值。另外，在表16.5中，要求X::iterator满足正向迭代器的要求，而以前只要求它不是输出迭代器。

<center>表16.6 C++11新增的基本容器要求</center>

|   表达式   |    返回类型    |                    说明                     | 复杂度 |
| :--------: | :------------: | :-----------------------------------------: | :----: |
|  X u(rv);  |                |  调用移动构造函数后，u的值与rv的原始值相同  |  线性  |
| X u = rv;  |                |                作用同X(tv);                 |        |
|  a = rv;   |       X&       | 调用移动赋值运算符后，u的值与rv的原始值相同 |  线性  |
| a.cbegin() | const_iterator |     返回指向容器第一个元素的const迭代器     |  固定  |
|  a.cend()  | const_iterator |            返回超尾值const迭代器            |  固定  |

复制构造和复制赋值以及移动构造和移动赋值之间的差别在于，复制操作保留源对象，而移动操作可修改源对象，还可能转让所有权，而不做任何复制。如果源对象是临时的，移动操作的效率将高于常规复制。

可以通过添加要求来改进基本的容器概念。<font color = #FF0000>序列</font>是一种重要的改进，因为7种STL容器类型（deque、forward_list、list、queue、priority_queue、stack和vector）都是序列。序列概念增加了迭代器至少是正向迭代器这样的要求，这保证了元素将按特定顺序排列，不会在两次迭代之间发生变化。序列还要求其元素按严格的线性顺序排列。

因为序列中的元素具有确定的顺序，因此可以执行诸如将值插入到特定位置、删除特定区间等操作。表16.7列出了这些操作以及序列必须完成的其他操作。其中，t表示类型为T的值，n表示整数，p、q、i和j表示迭代器。

<center>表16.7 序列的要求</center>

|      表达式       | 返回类型 |                        说明                         |
| :---------------: | :------: | :-------------------------------------------------: |
|     X a(n, t)     |          |           声明一个名为a的由n个t组成的序列           |
|      X(n, t)      |          |           创建一个有n个t值组成的匿名序列            |
|     X a(i, j)     |          | 声明一个名为a的序列，并将其初始化为区间[i, j)的内容 |
|      X(i, j)      |          |  创建一个匿名序列，并将其初始化为区间[i, j)的内容   |
|  a.insert(p, t)   |  迭代器  |                  将t插入到p的前面                   |
| a.insert(p, n, t) |   void   |                 将n个t插入到p的前面                 |
| a.insert(p, i, j) |   void   |          将区间[i, j)中的元素插入到p的前面          |
|    a.erase(p)     |  迭代器  |                   删除p指向的元素                   |
|   a.erase(p, q)   |  迭代器  |               删除区间[p, q)中的元素                |
|     a.clear()     |   void   |             等价于erase(begin(), end())             |

除了上述操作之外，一些序列还有一些可选操作。表16.8列出了其他操作。

<center>表16.8 序列的可选要求</center>

| 表达式          | 返回类型 | 含义                   | 容器                |
| --------------- | -------- | ---------------------- | ------------------- |
| a.front()       | T&       | *a.begin()             | vector, list, deuqe |
| a.back          | T&       | *--a.end()             | vector, list, deque |
| a.push_front(t) | void     | a.insert(a.begin(), t) | list, deque         |
| a.push_back(t)  | void     | a.insert(a.end(), t)   | vector, list, deque |
| a.pop_front(t)  | void     | a.erase(a.begin())     | list, deque         |
| a.pop_back(t)   | void     | a.erase(--a.end())     | vector, list, deque |
| a[n]            | T&       | *(a.begin() + n)       | vector, deque       |
| a.at(t)         | T&       | *(a.begin() + n)       | vector, deque       |



a[n]和a.at(n)都返回一个指向容器中第n个元素的引用。它们之间的差别在于，如果n落在有效区间外，则a.at(n)将执行边界检查，并引发out_of_range异常。

<font color = #FF0000>vector是数组的一种类表示，它提供了自动内存管理功能，可以动态地改变vector对象的长度，并随着元素的添加和删除而增大和减小</font>。它提供了对元素的随机访问。在尾部添加和删除元素的时间是固定的，但在头部或中间插入和删除元素的复杂度为线性时间。除序列外，vector还是可反转容器概念的模型。这增加了两个类方法：rbegin()和rend()。vector模板类是最简单的序列类型，除非其他类型的特殊优点能够更好地满足程序的要求，否则应默认使用这种类型。

<font color = #FF0000>deque模板类表示双端队列</font>。其实现类似与vector，支持随机访问。主要区别在于，从deque对象的开始位置插入和删除元素的时间是固定的。所以如果多数操作发生在序列的起始和结尾处，则应考虑使用deque数据结构。

<font color = #FF0000>list模板类表示双向链表</font>。除了第一个和最后一个元素外，每个元素都与前后的元素相链接，这意味着可以双向遍历链表。list也是可反转容器，但不支持数组表示法和随机访问。

除序列和可反转容器的函数外，list模板类还包含了链表专用的成员函数。表16.9列出了其中一些常用函数。通常不用担心Alloc模板参数，因为它有默认值。

<center>表16.9 list成员函数</center>

|                     函数                      |                             说明                             |
| :-------------------------------------------: | :----------------------------------------------------------: |
|       void merge(list<T,   Alloc> & x)        | 将链表x与调用链表合并。两个链表必须已经排序。合并后的链表保存在调用链表中，x为空。这个函数的复杂度为线性时间。 |
|         void remove(const T   & val)          |    从链表中删除val的所有实例。这个函数的复杂度为线性时间     |
|                  void sort()                  |      使用<运算符对链表进行排序；N个元素的复杂度为NlogN       |
| void splice(iterator   pos, list<T, Alloc> x) | 将链表x的内容插入到pos的前面，x将为空。这个函数的复杂度为固定时间 |
|                 void unique()                 |  将连续的相同元素压缩为单个元素。这个函数的复杂度为线性时间  |

注意，unique()只能将相邻的相同值压缩为单个值。

<font color = #FF0000>forward_list实现了单链表</font>。在这种链表中，每个节点都只链接到下一个节点，而没有链接到前一个节点。因此forward_list只需要正向迭代器，而不需要双向迭代器。

<font color = #FF0000>queue模板类表示队列</font>。queue模板的限制比deque更多。它不仅不允许随机访问队列元素，甚至不允许遍历队列。它把使用限制在定义队列的基本操作上，可以将元素添加到队尾、从队首删除元素、查看队首和队尾的值、检查元素数目和测试队列是否为空。表16.10列出了这些操作。

<center>表16.10 queue的操作</center>

| 方法                     | 说明                                    |
| ------------------------ | --------------------------------------- |
| bool empty() const       | 如果队列为空，则返回true；否则返回false |
| size_type size() const   | 返回队列中元素的数目                    |
| T & front()              | 返回指向队首元素的引用                  |
| T & back()               | 返回指向队尾元素的引用                  |
| void push(const T &   x) | 在队尾插入x                             |
| void pop()               | 删除队首元素                            |

<font color = #FF0000>priority_queue表示优先队列</font>。它支持的操作与queue相同。两者之间的主要区别在于，在priority_queue中，最大的元素被移到队首。内部区别在于，默认的底层是vector。可以修改用于确定哪个元素放到队首的比较方式。方法是提供一个可选的构造函数参数：

```C++
priority_queue<int> pq1;
priority_queue<int> pq2(greater<int>);
```

<font color = #FF0000>stack表示栈</font>。stack模板的限制比vector更多。它不仅不允许随机访问栈元素，甚至不允许遍历栈。它把使用限制在定义栈的基本操作上，即可以将压入推到栈顶、从栈顶弹出元素、查看栈顶的值、检查元素数目和测试栈是否为空。表16.11列出了这些操作。

<center>表16.11 stack的操作</center>

|           方法           |                 说明                  |
| :----------------------: | :-----------------------------------: |
|    bool empty() const    | 如果栈为空，则返回true；否则返回false |
|  size_type size() const  |          返回栈中的元素数目           |
|        T & top()         |        返回指向栈顶元素的引用         |
| void push(const T &   x) |             在栈顶部插入x             |
|        void pop()        |             删除栈顶元素              |

array表示数组，它并非STL容器，因为其长度是固定的。因此，array没有定义掉着那个容器大小的操作，但定义了对它来说有意义的成员函数，如operator[]()和at()。可将很多标准STL算法用于array对象。

关联容器是对容器概念的另一个改进。关联容器将值与键关联在一起，并使用键来查找值。STL提供了4中关联容器：set、multiset、map和multimap。前两种在头文件set中定义，后两种在头文件map中定义。

<font color = #FF0000>set是最简单的关联容器。它模拟了多个概念，它可反转，可排序，且键是唯一的</font>，所以不能存储多个相同的值。set使用模板参数来指定要存储的值类型：

```C++
set<string> A;
```

第二个模板参数是可选的，可用于指示用来对键进行排序的比较函数或对象。默认情况下，将使用模板less<>。

STL提供了支持集合的标准操作的算法，它们是通用函数，而不是方法，因此并非只能用于set对象。然而，所有set对象都满足使用这些算法的先决条件，即容器是经过排序的。

<font color = #FF0000>set_union()函数用于求并集</font>，它接受5个迭代器参数。前两个迭代器定义了第一个集合的区间，接下来的两个定义了第二个集合区间，最后一个迭代器是输出迭代器，指出将结果集合复制到什么位置。例如，要显示集合A和B的并集，可以这样做：

```C++
set_union(A.begin(), A.end(), B.begin(), B.end(),
							ostream_iterator<string, char> out(cout, ‘ ’));
```

函数set_intersection()和set_difference()分别查找交际和获得两个集合的差，它们的接口与set_union()相同。

两个有用的set方法是lower_bound()和upper_bound()。方法lower_bound()将键作为参数，并返回一个迭代器，该迭代器指向集合中第一个不小于键参数的成员。方法upper_bound()将键作为参数，返回一个迭代器，该迭代器指向集合中第一个大于键参数的成员。

<font color = #FF0000>multimap是可反转的、经过排序的关联容器，但键和值的类型不同，且同一个键可能与多个值相关联。</font>

基本的multimap声明使用模板参数指定键的类型和存储的值类型。例如，下面的声明创建一个multimap对象，其中键类型为int，存储的值类型为string：

```C++
multimap<int, string> codes;	
```

第3个模板参数是可选的，指出用于对键进行排序的比较函数或对象。在默认情况下，使用模板less<>。

multimap的成员函数count()接受键作为参数，并返回具有该键的元素数目。成员函数lower_bound()和upper_bound()将键作为参数，工作原理与处理set时相同。成员函数equal_range用键作为参数，且返回两个迭代器，它们表示的区间与该键匹配器，两个迭代器封装在pair中。

无序关联容器是对容器概念的另一种改进。与关联容器一样，无序关联容器也将值与键关联起来，并使用键来查找值。但底层的差别在于，<font color = #FF0000>关联容器是基于数结构的，而无序关联容器是基于数据结构哈希表的</font>。有4中无序关联容器，它们是unordered_set、unordered_multiset、unordered_map和unordered_multimap。



## 6.函数对象

<font color = #FF0000>函数对象也叫函数符。函数符是可以以函数方式与()结合使用的任意对象。这包括函数名、指向函数的指针和重载了()运算符的类对象</font>。STL定义了函数符概念：

* 生成器是不用参数就可以调用的函数符。

* 一元函数是用一个参数可以调用的函数符。

* 二元函数是用两个参数可以调用的函数符。

* 返回bool值的一元函数是谓词。

* 返回bool值的二元函数是二元谓词。

假设已经有了一个接受两个参数的模板函数：

```C++
template <class T>
bool tooBig(const T & val, const T & lim)
{
	return val > lim;
}
```

则可以使用类将它转换为单个参数的函数对象：

```C++
template<class T>
class TooBig2
{
private:
	T cutoff;
public:
	TooBig2(const T & t) : cutoff(t) {}
	bool operator()(const T & v) { return tooBig<T>(v, cutoff); } 
};
```

即可以这样做：

```C++
tooBig2<int> tB100(100);
int x;
cin >> x;
if(tB(x))		//与tooBig(x, 100)相同
//...
```

因此，调用tB100(x)相当于调用tooBig(x, 100)，但两个参数的函数被转换为单单数的函数对象，其中的第二个参数被用于构建函数对象。简而言之，<font color = #FF0000>类函数符TooBig2是一个函数适配器，使函数能够满足不同的接口</font>。

STL定义了多个基本函数符。提供这些对象是为了支持将函数作为参数的STL函数。例如，函数transform()有两个版本。第一个版本接受4个参数，前两个参数是指定容器区间的迭代器，第3个参数是指定将结果复制到哪里的迭代器，最后一个参数是接受单个单数的函数符，它被应用于区间中的每个元素，生成结果中的新元素。例如，下面的代码输出矢量gr8中所有元素的平方根：

```C++
const int LIM = 5;
double arr1[LIM] = {36, 39, 42, 45, 48};
vector<double> gr8(arr1, arr1 + LIM);
ostream_iterator<double, char> out(cout, “ ”);
transform(gr8.begin(), gr8.end(), out, sqrt);
```

上述代码将计算结果发送到输出流。目标迭代器可以位于原始区间中。可以用其他迭代器替换out。

第2种版本使用一个接受两个参数的函数，并将该函数用于两个区间中元素。它用另一个参数（第3个）标识第二个区间的起始位置。例如，如果m8是另一个vector\<double>对象，mean(double, double)返回两个值的平均值，则下面的代码将输出来自gr8和m8的平均值：

```C++
transform(gr8.begin(), gr8.end(), m8.begin(), out, mean);
```

头文件functional定义了多个模板类函数对象。表16.12列出了这些函数符的名称。它们可以用于处理C++内置类型或任何用户定义类型（如果重载了相应的运算符）。

<center>表16.12 运算符和响应的函数符</center>

| 运算符 | 响应的函数符  |
| :----: | :-----------: |
|   +    |     plus      |
|   -    |     minus     |
|   *    |  multiplies   |
|   /    |    divides    |
|   %    |    modulus    |
|   -    |    negate     |
|   ==   |   equal_to    |
|   !=   | not_equal_to  |
|   >    |    greater    |
|   <    |     less      |
|   >=   | greater_equal |
|   <=   |  less_equal   |
|   &&   |  logical_and  |
|  \|\|  |  logical_or   |
|   !    |  logical_not  |

上面列出的函数都是自适应的。实际上STL有5个相关的概念：自适应生成器、自适应一元函数、自适应二元函数、自适应谓词、自适应二元谓词。STL提供了使用这些工具的适配器类。例如，假设要将矢量gr8的每个元素都增加2.5倍，则需要使用接受一个一元函数参数的transform()版本。multiplies()函数符可以执行乘法运算，但它是二元函数。因此需要一个函数适配器，将接受两个参数的函数符转换为一个参数的函数符。前面的TooBig2提供了一种方法。STL使用<font color = #FF0000>binder1st</font>和<font color = #FF0000>binder2nd</font>类自动完成这一过程。

假设有一个自适应二元函数对象f2()，则可以创建一个binder1st对象，该对象与一个将被用作f2()的第一个参数的特定值（val）相关联：

```C++
binder1st(f2, val) f1;
```

这样，使用单个参数调用f1时，返回的值与将val作为第一参数、将f1()的参数作为第二参数调用f2()返回的值相同。即f1(x)等价于f2(val, x)，只是前者是一元函数。f2()函数被适配。

STL还提供了了函数<font color = #FF0000>bind1st()</font>，以简化binder1st类的使用。可以向其提供用于构建binder1st对象的函数名称和值，它将返回一个这种类型的对象。例如，要将二元函数multiplies()转换为将参数乘以2.5的一元函数，则可以这样做：

```C++
bind1st(multiplies<double>(), 2.5);
```

因此，将gr8中的每个元素与2.5相乘，并显式结果的代码如下：

```C++
transform(gr8.begin(), gr8.end(), out,  bind1st(multiplies<double>(), 2.5));
```

binder2nd类与此类似，只是将常数赋给第二个参数，而不是第一个参数。它有一个名为bind2nd的助手函数，该函数的工作方式类似与bind1st。



## 7.算法

STL将算法库分成4组：

* 非修改式序列操作；

* 修改式序列操作；

* 排序和相关操作；

* 通用数字运算。

前3组在头文件algorithm中描述，第4组是专用于数值数据的，有自己的头文件，称为numeric。

<font color = #FF0000>非修改式序列操作对区间中的每个元素进行操作</font>。这些操作不修改容器的内容。例如find()和for_each()就属于这一类。

<font color = #FF0000>修改式序列操作也对区间中的每个元素进行操作</font>。它们可以修改容器的内容。可以修改值，也可以修改值的排列顺序。transform()、random_shuffle()和copy()属于这一类。

<font color = #FF0000>排序和相关操作包括多个排序函数</font>（包括sort()和其他各种函数，包括集合操作。

<font color = #FF0000>数字操作包括将区间的内容累积、计算两个容器的内部乘积、计算小计、计算相邻对象差的函数。</font>

对算法进行分类的方式之一是按结果防止的位置进行分类。有些算法是就地完成工作，有些则是创建拷贝。就地完成工作的算法称为<font color = #FF0000>就地算法</font>，如sort()；创建拷贝的算法称为<font color = #FF0000>复制算法</font>，如copy()。有些算法有两个版本：就地版本和复制版本。STL的约定是，复制版本的名称将以_copy结尾。复制版本将接受一个额外的输出迭代器参数，该参数指定结果的放置位置。复制算法返回一个迭代器，该迭代器指向复制的最后一个值的后面的一个位置。

有些函数有这样的版本，即根据将函数应用于容器元素得到的结果来执行操作，这些版本的名称通常以if结尾。如replace_if()。

<font color = #FF0000>有时可以选择使用STL方法或STL函数。通常方法是更好的选择</font>。首先，它更适合与特定的容器；其次，作为成员函数，它可以使用模板类的内存管理工具，从而在需要时调整容器的长度。例如，假设有一个由数字组成的链表，并要删除链表中某个特定的值（如4）的所有实例。如果la是一个list<int>对象，则可以使用链表的remove()方法：

```C++
la.remove(4);
```

调用该方法后，链表中所有值为4 的元素都将被删除，同时链表的长度将被自动调成。

还有一个名为remove()的STL算法。如果lb是一个list<int>对象，则调用该函数的代码如下：

```C++
remove(lb.begin(), lb.end(), 4);
```

然而，由于该remove()函数不是成员，因此不能调整链表的长度。它将没被删除的元素放在链表的开始位置，并返回一个指向新的超尾值的迭代器。



## 8.其他库

C++提供了三个数组模板：vector、valarray和array。vactor模板类是一个容器类和算法系统的一部分，它支持面向容器的操作。而valarray类模板是面向数值计算的，不是STL的一部分，它为很多数学运算提供了一个简单、直观的接口。array是为替代内置数组而设计的，它通过提供更好、更安全的接口，让数组更紧凑，效率更高。

<font color = #FF0000>slice对象可用作数组索引，在这种情况下，它表示的不是一个值而是一组值。slice对象被初始化为三个整数值：起始索引、索引数和跨距</font>。起始索引是第一个被选中的元素的索引，索引数指出要选择多少个元素，跨距表示元素之间的间隔。例如，slice(1, 4, 3)创建的对象表示选择4个元素，它们的索引分别是1、4、7和10。如果varint是一个varry<int>对象，则下面的语句把第1、4、7和10个元素都设置为10：

```C++
varint[slice(1, 4, 3)] = 10;
```

模板<font color = #FF0000>initializer_lis</font>t是C++11新增的，表示初始化列表。可以使用初始化列表将STL容器初始化为一系列值：

```C++
std::vector<double> payments {45.99, 39.23, 19.95, 89.01};
```

这将创建一个包含4个元素的容器，并使用列表中的4个值来初始化这些元素。这之所以可行，是因为容器类现在包含将initializer_list<T>作为参数的构造函数。例如，vector<double>包含一个将initializer_list<double>作为参数的构造函数，因此上述声明与下面的代码等价：

```C++
std::vector<double> payments({45.99, 39.23, 19.95, 89.01});
```

这里显式地将列表指定为构造函数参数。

要使用initializer_list，必须包含头文件initializer_list。这个模板类包含成员函数begin()和end()。还包含成员函数size()，返回元素数。