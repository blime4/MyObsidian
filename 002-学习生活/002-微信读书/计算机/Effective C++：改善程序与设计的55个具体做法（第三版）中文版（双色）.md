---
doc_type: weread-highlights-reviews
bookId: "33381027"
author: 斯考特·梅耶
cover: https://cdn.weread.qq.com/weread/cover/9/YueWen_33381027/t7_YueWen_33381027.jpg
reviewCount: 2
noteCount: 13
readingStatus: 在读
progress: 96%
totalReadDay: 22
readingTime: 9小时58分钟
readingDate: 2021-11-03
isbn: 9787121123320
lastReadDate: 2023-12-20

---
# 元数据
> [!abstract] Effective C++：改善程序与设计的55个具体做法（第三版）中文版（双色）
> - ![ Effective C++：改善程序与设计的55个具体做法（第三版）中文版（双色）|200](https://cdn.weread.qq.com/weread/cover/9/YueWen_33381027/t7_YueWen_33381027.jpg)
> - 书名： Effective C++：改善程序与设计的55个具体做法（第三版）中文版（双色）
> - 作者： 斯考特·梅耶
> - 简介： 有人说C++程序员可以分为两类，读过Effective C++的和没读过的。世界顶级C++大师Scott Meyers成名之作的第三版的确当得起这样的评价。当您读过这本书之后，就获得了迅速提升自己C++功力的一个契机。在国际上，本书所引起的反响，波及整个计算机技术的出版领域，余音至今未绝。几乎在所有C++书籍的推荐名单上，本书都会位于前三名。作者高超的技术把握力、独特的视角、诙谐轻松的写作风格、独具匠心的内容组织，都受到极大的推崇和仿效。这种奇特的现象，只能解释为人们对这本书衷心的赞美和推崇。这本书不是读完一遍就可以束之高阁的快餐读物，也不是用以解决手边问题的参考手册，而是需要您去反复阅读体会的，C++是真正程序员的语言，背后后精神的思想与无以伦比的表达能力，这使得它具有类似宗教般的魅力。希望这本书能够帮您跨越C++的重重险阻，领略高处才有的壮美风光，做一个成功而快乐的C++程序员。
> - 出版时间 2011-01-01 00:00:00
> - ISBN： 9787121123320
> - 分类： 计算机-编程设计
> - 出版社： 电子工业出版社
> - PC地址：https://weread.qq.com/web/reader/48032b4071fd5aa34809fb5

# 高亮划线

### 条款02：尽量以const，enum，inline替换＃define Prefer consts，enums，and inlines to＃defines.

> 📌 请把这个式子放进一个实现文件而非头文件。由于 class 常量已在声明时获得初值（例如先前声明NumTurns时为它设初值5），因此定义时不可以再设初值。 
> ⏱ 2023-11-27 16:58:16 ^33381027-12-2913-2990

> 📌 这时候万一你的编译器（错误地）不允许“static整数型class常量”完成“in class初值设定”，可改用所谓的 "the enum hack" 补偿做法。其理论基础是：“一个属于枚举类型（enumerated type）的数值可权充ints被使用”，于是GamePlayer可定义如下：
￼
基于数个理由enum hack值得我们认识。第一，enum hack的行为某方面说比较像＃define而不像const，有时候这正是你想要的。例如取一个const的地址是合法的，但取一个enum的地址就不合法，而取一个＃define的地址通常也不合法。如果你不想让别人获得一个pointer或reference指向你的某个整数常量，enum可以帮助你实现这个约束。 
> ⏱ 2023-07-28 16:48:16 ^33381027-12-3620-4149

> 📌 认识enum hack的第二个理由纯粹是为了实用主义。许多代码用了它，所以看到它时你必须认识它。事实上 "enum hack" 是template metaprogramming（模板元编程，见条款48）的基础技术。 
> ⏱ 2023-08-05 13:01:27 ^33381027-12-4316-4424

### 条款03：尽可能使用const Use const whenever possible.

> 📌 编译器会强制实施这项约束 
> ⏱ 2023-08-17 11:38:42 ^33381027-13-521-533

> 📌 请记住■ 将某些东西声明为 const可帮助编译器侦测出错误用法。const可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。 
> ⏱ 2021-11-18 03:06:19 ^33381027-13-9408-9497

> 📌 ■ 当 const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复。 
> ⏱ 2021-11-18 03:06:41 ^33381027-13-9593-9656

### 条款07：为多态基类声明virtual析构函数 Declare destructors virtual in polymorphic base classes.

> 📌 给base class一个virtual析构函数 
> ⏱ 2023-12-20 15:37:02 ^33381027-18-2083-2107

### 条款23：宁以non-member、non-friend替换member函数 Prefer non-member non-friend functions to member functions.

> 📌  让我们从封装开始讨论。如果某些东西被封装，它就不再可见。愈多东西被封装，愈少人可以看到它。而愈少人看到它，我们就有愈大的弹性去变化它，因为我们的改变仅仅直接影响看到改变的那些人事物。因此，愈多东西被封装，我们改变那些东西的能力也就愈大。这就是我们首先推崇封装的原因：它使我们能够改变事物而只影响有限客户。 ^33381027-36-1766-1918
- 💭 精髓呀 - ⏱ 2021-11-25 17:16:32 

### 条款24：若所有参数皆需类型转换，请为此采用non-member函数 Declare non-member functions when type conversions should apply to all parameters.

> 📌 这导出一个重要的观察：member函数的反面是non-member函数，不是friend函数。 
> ⏱ 2023-12-20 16:10:02 ^33381027-37-4476-4523

> 📌 不能够只因函数不该成为member，就自动让它成为friend。 
> ⏱ 2023-12-20 16:10:58 ^33381027-37-4715-4747

### 条款39：明智而审慎地使用private继承 Use private inheritance judiciously.

> 📌 如果你让class D以private形式继承class B，你的用意是为了采用class B内已经备妥的某些特性，不是因为B对象和D对象存在有任何观念上的关系。 
> ⏱ 2023-12-11 12:39:47 ^33381027-54-1295-1376

> 📌 “阻止derived classes重新定义virtual函数”的能力 
> ⏱ 2023-12-11 14:18:07 ^33381027-54-4270-4305

> 📌 private继承主要用于“当一个意欲成为derived class者想访问一个意欲成为base class者的protected成分，或为了重新定义一或多个virtual函数” 
> ⏱ 2023-12-11 14:19:07 ^33381027-54-4653-4742

# 读书笔记

## 条款03：尽可能使用const Use const whenever possible.

### 划线评论
> 📌 ■ 编译器强制实施bitwise constness，但你编写程序时应该使用“概念上的常量性”（conceptual constness）。  ^10284435-7uToTepih
    - 💭 看不懂
    - ⏱ 2021-11-18 03:06:33
   
## 条款23：宁以non-member、non-friend替换member函数 Prefer non-member non-friend functions to member functions.

### 划线评论
> 📌 让我们从封装开始讨论。如果某些东西被封装，它就不再可见。愈多东西被封装，愈少人可以看到它。而愈少人看到它，我们就有愈大的弹性去变化它，因为我们的改变仅仅直接影响看到改变的那些人事物。因此，愈多东西被封装，我们改变那些东西的能力也就愈大。这就是我们首先推崇封装的原因：它使我们能够改变事物而只影响有限客户。  ^10284435-7v4WJrhCG
    - 💭 精髓呀
    - ⏱ 2021-11-25 17:16:40
   
# 本书评论
