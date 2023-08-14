​
在前面章节中，我们详细介绍了公钥的压缩，在比特币网络中，一个私钥可以对应两个地址，一个地址是由未压缩公钥所生成的地址，另一个就是由压缩公钥所创建的地址，从公钥到区块链地址的转换算法，我们在这里给出详细描述和代码实现，本节我们看看私钥的压缩以及相关的WIF数据格式。

搞笑的是私钥”压缩“后，其长度反而比压缩前增加了一个字节。而“压缩”方法也相当简单，就是在私钥末尾增加一个字节01，例如如果私钥的数据为：1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD，
那么对应的“压缩”格式就是：
1E99423A4ED27608A15A2616A2B0E9E52CED330AC530EDCC32C8FFC6A526AEDD01
我们比较一下就可以看出，“压缩”后的私钥就是在末尾增加了字节01。为什么“压缩”私钥呢。前面我们提到过公钥有两种存储方法，压缩格式和非压缩格式，私钥是否“压缩”就对应创建哪种公钥存储模式，如果私钥”压缩“过，那就意味着使用它来创建压缩格式的公钥，如果没有“压缩”，那就使用它创建非压缩格式的公钥。

公钥的压缩是为了能在网络进行传输，通常情况下私钥不需要经常进行网络传输，因为过多的把私钥暴露在网络会增加泄露的几率，一旦私钥泄露，你所有的资产或私有信息将会丢失。然而在某些情况下，私钥也有传输的需要，例如将私钥从一个区块链客户端发送到另一个客户端进行导入时，私钥就需要进行网络传输了，于是我们也就有了对其进行编码的需要，由此私钥对应的编码简称为WIF。

我们看看WIF编码格式的基本步骤：
1，如果私钥对应的是比特币主网络，那么在私钥的开头增加一个字节0x80,如果对应测试网络增增加字节0xef.
2，将其进行大端格式存储
3,如果该私钥要用来创建压缩格式的公钥，那么在步骤2的末尾增加1个字节0x01
4,对步骤3做sha256哈希，然后去结果的前4个字节
5.把步骤3和4的结果首尾相连，然后再做base58编码
我们使用代码实现看看：

privKey = 0x038109007313a5807b2eccc082c8c3fbb988a973cacf1a7df9ce725c31b14776
pubKey = privKey * G

class PrivateKey:
    def __init__(self, secret):
        self.secret = secret

    def wif(self, compressed=True, testnet=False):
        #先将私钥进行大端转换
        secret_bytes = self.secret.to_bytes(32, 'big')
        if testnet:
            #如果是测试网络的私钥则在开头增加字节0xef
            prefix = b'\xef'
        else:
            #如果是主网络则在开头增加字节0x80
            prefix = b'\0x80'
        if compressed:
            #如果要创建压缩格式的公钥，在末尾增加自己0x1
            suffix = b'\0x01'
        else:
            suffix = ''

        return encode_base58_checksum(prefix + secret_bytes + suffix)

private_key = PrivateKey(privKey)
wif_private_key = private_key.wif()
print(f"the wif for give private key is: {wif_private_key}")

上面代码运行后结果为：

the wif for give private key is: 19re3h9z4eEC6WYaziGHvAY8nS8hNddiPcxe4B9a6vA2SbEaSjtqDLYC3SYk
更多内容请在 b 站搜索 coding 迪斯尼

​
