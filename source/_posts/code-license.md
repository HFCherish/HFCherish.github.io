---
title: code license
toc: true
date: 2018-07-26 09:47:13

---

license 是软件的授权许可。

对于开源软件来说，虽然别人可以用，但是用的时候希望别人遵循一些要求，比如，使用时必须标明原作者是谁、可以做怎样的修改、软件被用作不正规用途原作者是否要负责……这些其实就是一个协议。

对于作者来说，自己为开源代码写符合法律条规的繁冗的 license 太麻烦，所以就可以采用广为流传的开源协议（eg. MIT, CC…），在 license 文件中标明 "Licnse under the MIT license"

# 快速选择

详细的协议选择可以从 github [choose license](https://choosealicense.com/) 项目中选。下边列些常用的（参见 [如何为你的代码选择一个开源协议](https://www.cnblogs.com/Wayou/p/how_to_choose_a_license.html)）

## MIT 协议

宽松但覆盖一般要点。此协议允许别人以任何方式使用你的代码同时署名原作者，但原作者不承担代码使用后的风险，当然也没有技术支持的义务。jQuery和Rails就是MIT协议

## apache 协议

作品涉及到专利，可以考虑这个。也比较宽松，但考虑了专利，简单指明了作品归属者对用户专利上的一些授权（我的理解是软件作品中含有专利，但它授权你可以免费使用）。Apache服务器，SVN还有NuGet等是使用的Apache协议。

## GPL

对作品的传播和修改有约束的，可以使用这个。GPL（[V2](http://choosealicense.com/licenses/gpl-v2)或[V3](http://choosealicense.com/licenses/gpl-v3)）是一种版本自由的协议（可以参照copy right来理解，后者是版本保留，那copyleft便是版权自由，或者无版权，但无版权不代表你可以不遵守软件中声明的协议）。此协议要求代码分发者或者以此代码为基础开发出来的衍生作品需要以同样的协议来发布。此协议的版本3与版本2相近，只是多3中加了条对于不支持修改后代码运行的硬件的限制（没太明白此句话的内涵）。