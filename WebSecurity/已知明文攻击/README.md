## 密码学攻击之已知明文攻击   

### 1. 已知明文攻击  

密码分析中，已知明文攻击是一种攻击模式，指攻击者掌握了明文X及对应的密文Y，选择明文攻击也是一种攻击模式，指攻击者能够构造任意明文X，并获取到对应的密文Y。  

### 2. 易受影响的攻击场景  

现在密码学，对称加密推荐至少使用AES-256-CBC，DES、3DES已经禁止使用，其中ECB模式也不再推荐使用，推荐更安全的GCM模式  

易受攻击的场景，主要集中在block cipher，当采用ECB模式、或固定IV的CBC模式时：  

1. ECB模式加密  
2. IV固定的CBC模式加密
3. IV可预测的CBC模式，且攻击者可控制the first block  

其核心风险为：相同的明文总是加密为相同的密文  

### 3. 验证攻击  

这是开源的一个验证Demo，共参考学习：https://github.com/EiNSTeiN-/chosen-plaintext  
直接跑，会报错，修正验证了2个文件，python3环境，安装加密

### 4. 安全使用密码学算法  

1. 针对IV随机的场景下，AES CBC模式具备抵御已知明文攻击的能力，据网上查询信息，AES等更安全加密算法具备抵御该风险的能力  
2. 在使用对称加密算法时，要随机化IV向量，避免相同明文总是加密为相同的密文  
3. 暴力破解的方式破解密钥，基本不现实，物理上的论点，认为128位对称密钥在计算上可以防止暴力攻击，如果破解256位密钥，从理论上讲，五十台超级计算机每秒检查10亿个，则需要3x10(51次方)年才能耗尽密钥空间  

### 5. GCM模式 
原文摘自：https://blog.csdn.net/T0mato_/article/details/53160772  

GCM ( Galois/Counter Mode) 指的是该对称加密采用Counter模式，并带有GMAC消息认证码。  

术语：  

1. Ek	使用秘钥k对输入做对称加密运算
2. XOR	异或运算
3. Mh	将输入与秘钥h在有限域GF(2^128)上做乘法

#### EBC（电子密码本）
当我们有一段明文，需要对其进行AES加密时，需要对明文进行分组，分组长度可为128，256，或512bits，ECB不能提供对密文的完整性校验。采用ECB模式的分组密码算法加密过程如下图：  
![](https://github.com/shadow-horse/Learning-resource/blob/master/WebSecurity/%E5%B7%B2%E7%9F%A5%E6%98%8E%E6%96%87%E6%94%BB%E5%87%BB/img/ecb.png)

#### CTR ( CounTeR 计数器模式)
在计数器模式下，我们不再对密文进行加密，而是对一个逐次累加的计数器进行加密，用加密后的比特序列与明文分组进行 XOR得到密文。  

计数器模式下，每次与明文分组进行XOR的比特序列是不同的，因此，计数器模式解决了ECB模式中，相同的明文会得到相同的密文的问题。CBC，CFB，OFB模式都能解决这个问题，但CTR的另两个优点是：1）支持加解密并行计算，可事先进行加解密准备；2）错误密文中的对应比特只会影响明文中的对应比特等优点。但CTR仍然不能提供密文消息完整性校验的功能。

过程如下图：
![](https://github.com/shadow-horse/Learning-resource/blob/master/WebSecurity/%E5%B7%B2%E7%9F%A5%E6%98%8E%E6%96%87%E6%94%BB%E5%87%BB/img/ctr.png)

#### MAC  ( Message Authentication Code, 消息验证码)
想要校验消息的完整性，必须引入另一个概念：消息验证码。消息验证码是一种与秘钥相关的单项散列函数。  
密文的收发双发需要提前共享一个秘钥。密文发送者将密文的MAC值随密文一起发送，密文接收者通过共享秘钥计算收到密文的MAC值，这样就可以对收到的密文做完整性校验。当篡改者篡改密文后，没有共享秘钥，就无法计算出篡改后的密文的MAC值。
如果生成密文的加密模式是CTR，或者是其他有初始IV的加密模式，别忘了将初始的计时器或初始向量的值作为附加消息与密文一起计算MAC。
![](https://github.com/shadow-horse/Learning-resource/blob/master/WebSecurity/%E5%B7%B2%E7%9F%A5%E6%98%8E%E6%96%87%E6%94%BB%E5%87%BB/img/mac.png)

#### GMAC ( Galois message authentication code mode, 伽罗瓦消息验证码 )

对应到上图中的消息认证码，GMAC就是利用伽罗华域(Galois Field，GF，有限域)乘法运算来计算消息的MAC值。假设秘钥长度为128bits, 当密文大于128bits时，需要将密文按128bits进行分组。应用流程如下图：
![](https://github.com/shadow-horse/Learning-resource/blob/master/WebSecurity/%E5%B7%B2%E7%9F%A5%E6%98%8E%E6%96%87%E6%94%BB%E5%87%BB/img/gmac.png)

#### GCM（ Galois/Counter Mode ) 
GCM中的G就是指GMAC，C就是指CTR。
GCM可以提供对消息的加密和完整性校验，另外，它还可以提供附加消息的完整性校验。在实际应用场景中，有些信息是我们不需要保密，但信息的接收者需要确认它的真实性的，例如源IP，源端口，目的IP，IV，等等。因此，我们可以将这一部分作为附加消息加入到MAC值的计算当中。下图的Ek表示用对称秘钥k对输入做AES运算。最后，密文接收者会收到密文、IV（计数器CTR的初始值）、MAC值。
![](https://github.com/shadow-horse/Learning-resource/blob/master/WebSecurity/%E5%B7%B2%E7%9F%A5%E6%98%8E%E6%96%87%E6%94%BB%E5%87%BB/img/gcm.png)

