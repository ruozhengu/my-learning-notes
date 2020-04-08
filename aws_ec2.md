### AWS 培训课



##### 第二节课：EC2

![image-20200325162927236](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325162927236.png)

![image-20200325163002589](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325163002589.png)

![image-20200325163023269](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325163023269.png)

每一个区域有不同的AMI编号。

虚拟化类型：全虚拟化，半虚拟化，物理虚拟

启动许可：自己可见，其余developer可见，public可见

![image-20200325163314350](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325163314350.png)

AWS预构建：一个pool，存已经建好的popular的AMI image。可以直接用。安全可靠干净

AWS MarketPlace：第三方。其余企业安装一个好的软件，然后打包出售。比如杀毒软体

自行创建：快速上线，可靠

社区：开源，开放自己的系统软件。 

![image-20200325165253366](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325165253366.png)

黄金影像已经包含了我们需要的东西，之后即可复制

![image-20200325165537465](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325165537465.png)

如何使用image builder 完成部署

1. 准备原始镜像，开启image builder
2. 我们在镜像中准备了很多需要安装的软件，image builder会收集这些信息然后开始自定义部署
3. 完成安全和漏洞扫描检测
4. 部署给所有使用者

第二部：实例类型

![image-20200325171004031](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171004031.png)

存储优化：适合mapper reducer 大数据

加速计算：使用gpu的运算能力比如深度学习

![image-20200325171325392](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171325392.png)

在cloudwatch观察实例运行情况。也可以使用http_load，webbench，ab，siege等工具进行pressure和performance testing

![image-20200325171540413](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171540413.png)

![image-20200325171638045](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171638045.png)

![image-20200325171720835](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171720835.png)

![image-20200325171903532](/Users/gabriel/Library/Application Support/typora-user-images/image-20200325171903532.png)

预留：签订合约，比如三年拿到最多30%优惠

Spot：每个机房为预留需求，会reserve 30% capacity。可以通过竞价购买，但是一旦request超载，随时可能被关闭

---

### 模块3： 存储服务

