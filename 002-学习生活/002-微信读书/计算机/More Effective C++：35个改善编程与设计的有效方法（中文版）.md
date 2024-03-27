---
doc_type: weread-highlights-reviews
bookId: "33381011"
author: 斯考特·梅耶
cover: https://cdn.weread.qq.com/weread/cover/72/YueWen_33381011/t7_YueWen_33381011.jpg
reviewCount: 0
noteCount: 10
readingStatus: 未标记
progress: 11%
totalReadDay: 2
readingTime: 0小时18分钟
readingDate: 2022-11-13
isbn: 9787121125706
lastReadDate: 2022-11-13

---
# 元数据
> [!abstract] More Effective C++：35个改善编程与设计的有效方法（中文版）
> - ![ More Effective C++：35个改善编程与设计的有效方法（中文版）|200](https://cdn.weread.qq.com/weread/cover/72/YueWen_33381011/t7_YueWen_33381011.jpg)
> - 书名： More Effective C++：35个改善编程与设计的有效方法（中文版）
> - 作者： 斯考特·梅耶
> - 简介： 继Effective C++之后，Scott Meyers于1996推出这本“续集”。条款变得比较少，页数倒是多了一些，原因是这次选材比“第一集”更高阶，尤其是第5章。Meyers将此章命名为技术（techniques），并明白告诉你，其中都是一些patterns，例如virtual constructors，smart pointers，reference counting，proxy classes，double dispatching……这一章的每个条款篇幅都达15~30 页之多，实在让人有“山重水复疑无路，柳暗花明又一村”之叹。虽然出版年代稍嫌久远，但本书并没有第2版，原因是当其出版之时（1996），C++ Standard已经几乎定案，本书即依当时的标准草案而写，其与现今的C++ 标准规范几乎相同。而且可能变化的几个弹性之处，Meyers也都有所说明与提示。读者可以登录作者提供的网址，看看上下两集的勘误与讨论（数量之多，令人惊恐。幸好多是技术讨论或文字斟酌，并没有什么重大误失）。
> - 出版时间 2011-01-01 00:00:00
> - ISBN： 9787121125706
> - 分类： 计算机-编程设计
> - 出版社： 电子工业出版社
> - PC地址：https://weread.qq.com/web/reader/a2a32db071fd5a93a2a736a

# 高亮划线

## 其 他

> 📌 只有深刻了解 C++编译器如何解释你的代码，你才有可能用 C++语言写出健壮的软件。 
> ⏱ 2022-11-13 01:37:43 ^33381011-4-534-576

## 译序

> 📌 C++是一门难学易用的语言！ 
> ⏱ 2022-11-13 01:38:39 ^33381011-5-430-444

### 条款 2：最好使用 C++转型操作符

> 📌 最后一个转型操作符是 reinterpret_cast。这个操作符的转换结果几乎总是与编译平台息息相关。所以 reinterpret_casts 不具移植性。 
> ⏱ 2022-11-13 14:20:48 ^33381011-9-3618-3697

> 📌 reinterpret_cast 的最常用用途是转换“函数指针”类型。假设有一个数组，存储的都是函数指针，有特定的类型： 
> ⏱ 2022-11-13 14:21:13 ^33381011-9-3710-3770

### 条款 3：绝对不要以多态（polymorphically）方式处理数组

> 📌 简单地说，多态（polymorphism）和指针算术不能混用。数组对象几乎总是会涉及指针的算术运算，所以数组和多态不要混用。 
> ⏱ 2022-11-13 14:25:43 ^33381011-10-3839-3901

> 📌 具体类不要继承自另一个具体类 
> ⏱ 2022-11-13 14:26:01 ^33381011-10-4005-4019

### 条款 4：非必要不提供 default constructor

> 📌 非必要不提供 default constructor 
> ⏱ 2022-11-13 14:27:48 ^33381011-11-420-446

> 📌 Classes 如果缺乏 default constructors，带来的第二个缺点是：它们将不适用于许多 template-based container classes。对那些 templates 而言，被实例化（instantiated）的“目标类型”必须得有一个 default constructors。 
> ⏱ 2022-11-13 14:32:50 ^33381011-11-4536-4693

> 📌 Virtual base classes 如果缺乏 default constructors，与之合作将是一种刑罚。 
> ⏱ 2022-11-13 14:41:16 ^33381011-11-5434-5492

> 📌 因为 virtual base class constructors 的自变量必须由欲产生的对象的派生层次最深（所谓 most derived）的 class 提供。于 
> ⏱ 2022-11-13 14:41:30 ^33381011-11-5492-5576

# 读书笔记

# 本书评论
