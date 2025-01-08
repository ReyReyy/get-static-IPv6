# 家庭宽带获取固定 IPv6 （试行）

**前排提醒：我也是慢慢摸索出来的，不保证在你的网络环境下就一定有效！** <br>
**我也只是一只小白，懂的大佬们轻喷！**

### **在折腾之前一定一定一定要记得备份！~~折腾完上不去网就完蛋！~~**

### 你需要：

- 一台**能获取 IPv6 前缀**的路由器
  _路由器需要能修改一些高级设置_
- 一台用来配置路由器的电脑
- 一些些网络知识
- 一颗爱折腾的心

省流：看后记

以下以 OpenWRT 为例：

## 1. 新增/重设 WAN6 接口

- 打开 **网络 -> 接口** 并把 **IPv6-PD** 记下来，一般为 `240n:xxxx:xxxx:xxxx::/64`（没有可以先不用记，后面有）

  > 不同运营商开头四个数字不同 <br>
  > 一般来说： <br>
  > 中国电信 240e <br>
  > 中国联通 2408 <br>
  > 中国移动 2409 <br>
  > 我家是中国联通，所以后续以 2408 为例。

- 把 **WAN6** 接口删掉（没有或删不掉可以不用删）
- 点击 **添加新接口**
  - 名称随意，记得住就行
  - 协议选择 **DHCPv6 客户端**
  - 设备选择 **接口别名: "\@WAN"**（你的 WAN 接口）
    ![添加接口](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/add-interface.png?raw=true)
- 点击 **创建接口**，**保存**，然后 **保存并应用**
- 然后过会儿应该就能看到 IPv6 地址了
  ![接口们](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/show-interfaces-before.png?raw=true)

## 2. 修改 IPv6 前缀

这时，之前记的 IPv6-PD 就派上了用场。

众所周知，一般来说，IPv6 前 48 位为运营商分配的前缀，中间 16 位为子网，它们共同组成了你在路由器界面上看到的 IPv6-PD，而我们现在要修改/固定的就是中间的 16 位**子网**部分。 <br>
![IPv6 拓扑图](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/IPv6-topology.png?raw=true)

回到操作：

- 点击 WAN6 接口右侧的 **编辑** 按钮
  - **常规设置 -> 请求 IPv6 前缀** 改为 **`240n:xxxx:xxxx:abcd::/64`**（最后的部分改成你想要**且合法**的数字，比如 abcd、1234、cafe
    、face 之类的）
    ![常规设置](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/interface-example.png?raw=true)
  - **高级设置 -> 强制链路** 打勾
    ![高级设置](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/advanced-example.png?raw=true)
  - **防火墙设置 -> 创建/分配防火墙区域** 选择 **wan**
    ![防火墙设置](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/firewall-example.png?raw=true)
- 点击 **保存**，**保存并应用**

到这里一般来说就完成了所有操作，🎉 恭喜你已经有了一个专属于自己的 IPv6-PD！
![修改后的接口们](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/show-interfaces-after.png?raw=true)
如果你想在某设备上使用固定 IPv6 地址只需要设置一下后缀，也就是 接口 ID 即可～

## 3. 给路由器加上固定 IPv6（可选）

- 编辑 LAN 口，点击上方 **高级设置** 拉到最下面 **IPv6 后缀**，填写任何你想填的内容，只要合法即可。比如我这里填 `::1`，然后点 **保存**，**保存并应用**
- 你的路由器的后缀就变成了 `::1`，IPv6 地址就是 `240n:xxxx:xxxx:abcd::1`
  ![专属IPv6地址！](https://github.com/ReyReyy/get-static-IPv6/blob/main/images/lan-after.png?raw=true)

🎉 恭喜你，你的漏油器有了一个专属的 IPv6 地址～！

当然你也可以用你的固定前缀给你的服务器、PC、mac，甚至手机都设置固定的 IPv6 地址，这里就不赘述了。

## 后记

折腾的过程并不算长也不算太麻烦，原理也并不复杂，就相当于是把一个 /24 长度的 IPv4 地址固定，大佬估计用不着看我的教程分分钟就能搞定。 <br>
简而言之就是强制获取某个固定的 IPv6-PD ，透过它分配一个固定的 IPv6 地址。

### 关于公网访问

一般来说如果没有开放防火墙，你并不能从别处直接通过你的公网地址访问，这边推荐你看不良林的 [openwrt 软路由配置 ipv6 教程](https://youtu.be/klQ6JbZEeJ0?si=8eCh0UT6-go1HYyf)

### 各家运营商分配前缀长度不同

一般来说，(我遇到的)运营商分配的 IPv6-PD 都是 /64 的，我家是联通，前缀长度是 /64，电信没用过不知道，之前用过移动，它好像会分配 /60，也就是下游路由器可以再多分配一段 IPv6-PD，不过这就要等各位大佬们来研究了 :\(
