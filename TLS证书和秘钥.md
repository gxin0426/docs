## TLS/CFSSL证书生成工具的使用

```
公钥基础设施（PKI）/CFSSL 证书生成工具的使用

- RSA即非对称加密算法，用其中一个加密的数据只能用另一个密码解开
- PEM 内容为Base64编码的ASCII码文件
- CSR 他是向CA机构申请数字证书时使用的请求文件。在生成请求文件前，我们需要准备一对对称密钥。私钥信息自己保存，请求中会附上公钥信息以及国家，城市，域名，Email等信息，CSR中还会附上签名信息。当我们准备好CSR文件后就可以提交给CA机构，等待他们给我们签名，签好名后我们会收到crt文件，即证书。
- 数字签名：”非对称加密+摘要算法“ 其核心思想是比如A要给B发送数据，A先用摘要算法得到数据的指纹，然后用A的私钥加密指纹 加密后的指纹就是A的签名 B收到数据和A的签名后 也同样使用摘要算法计算指纹， 然后用A的公钥解密签名，比较两个指纹，如果相同 说明数据没有篡改  
- 摘要算法 包括 MD5 SHA1 SHA256
- 数字证书与公钥： 数字证书是由CA对证书申请者真是身份验证之后，用CA的根证书对申请人的一些基本信息以及申请人的公钥进行签名。实际上，数字证书就是经过CA认证过的公钥。除了公钥，还有Email 国家 城市 域名等信息

文章链接

https://blog.51cto.com/liuzhengwei521/2120535
```

- 安装cfssl

```she
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
```

- `cfssl`工具，子命令介绍：

  - `bundle`: 创建包含客户端证书的证书包

  - `genkey`: 生成一个key(私钥)和CSR(证书签名请求)

  - `scan`: 扫描主机问题

  - `revoke`: 吊销证书

  - `certinfo`: 输出给定证书的证书信息， 跟cfssl-certinfo 工具作用一样

  - `gencrl`: 生成新的证书吊销列表

  - `selfsign`: 生成一个新的自签名密钥和 签名证书

  - `print-defaults`: 打印默认配置，这个默认配置可以用作模板

  - `serve`: 启动一个HTTP API服务

  - `gencert`: 生成新的key(密钥)和签名证书

  - - -ca：指明ca的证书
    - -ca-key：指明ca的私钥文件
    - -config：指明请求证书的json文件
    - -profile：与-config中的profile对应，是指根据config中的profile段来生成证书的相关信息

  生成CA证书和CA私钥和CSR(证书签名请求): 

```
# cfssl gencert -initca ca-csr.json | cfssljson -bare ca  ## 初始化ca
# ls ca*
ca.csr  ca-csr.json  ca-key.pem  ca.pem
```

该命令会生成运行CA所必需的文件`ca-key.pem`（私钥）和`ca.pem`（证书），还会生成`ca.csr`（证书签名请求），用于交叉签名或重新签名。 





- 证书：.cer .crt

- 私钥： .key

- 证书签名请求： .csr

- x509证书一般会用到三类文，key,csr,crt

  key是私用密钥openssl格式，通常是rsa算法

  csr是证书请求文件，用于申请证书 在制作csr文件时 必须使用自己的私钥来签署申请 还可以设一个密钥

  crt是CA认证后的证书文 签署人用自己的key给你签署的凭证

