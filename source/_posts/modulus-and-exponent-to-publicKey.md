---
title: 将modules和exponent转换为公钥
date: 2020-05-22 15:01:29
tags: [加解密,openssl]
categories: [加解密,openssl]
---
&emsp;&emsp;在使用加解密时，有时候我们拿到的是公钥中的modulus和exponent字段，当需要通过使用openssl工具进行验签或者加密的时候，就需要将这些信息转换成公钥了。本文介绍如何将modules和exponent转换为公钥。
<!-- more -->
## 公钥文件分析
使用如下命令dump出公钥的信息：
```
$ openssl rsa -pubin -in  pub.pem -noout -text
Public-Key: (2048 bit)
Modulus:
    00:a0:10:a8:32:0b:cb:75:db:7d:8e:ba:a5:df:56:
    b2:fb:9c:e9:34:b3:ae:7b:0b:07:a6:13:40:28:13:
    be:78:5f:69:09:c1:bb:5e:52:62:aa:a3:f5:38:5d:
    70:c2:3c:95:05:71:e8:87:57:10:a0:5d:7b:8a:f7:
    dd:a3:e4:85:f5:e1:e7:ba:e2:34:4c:18:9f:46:55:
    15:49:b5:48:92:22:ed:84:24:17:86:bf:69:10:55:
    42:76:4b:55:bf:d3:ce:a4:2a:bd:5e:a2:e5:29:84:
    4f:77:08:3a:22:04:5c:00:de:8b:9b:ee:7f:68:b0:
    b0:9b:e8:bb:b1:99:e6:8a:cf:9e:b9:85:d2:86:d3:
    5e:c2:a4:56:8d:f8:87:1d:6c:b3:73:83:3e:7b:bc:
    5d:c6:e2:63:fd:e1:4d:18:df:53:0f:4e:13:6f:fd:
    a6:b7:8c:e9:26:a3:4e:2b:fd:61:3d:d1:b5:f4:2f:
    ad:55:12:57:2a:20:ed:30:e7:64:3c:87:26:47:ef:
    97:48:30:df:44:f2:ed:c6:54:ed:5e:4f:dd:d9:3d:
    64:51:02:f9:7b:56:77:44:63:b0:95:1a:5e:83:4b:
    82:ce:59:f2:9e:e1:fc:16:1c:fc:f6:9c:55:37:73:
    ae:6a:e9:76:0c:9e:3a:b1:fc:cf:76:20:6b:17:d4:
    87:c3
Exponent: 65537 (0x10001)
```
从上面的信息可以看到`exponent`的值为`65537`。上面的信息中发现我们的public key的长度是2048位的，但是modulus有257个字节，合计2056（257*8）位,经过验证发现`modulus`中的第一位`00`是不需要的。后面根据[网上资料](https://stackoverflow.com/questions/5403808/private-key-length-bytes)发现rsa格式的密钥，它的密钥长度跟`modulus`的长度是一致的，第一位`00`不影响`modulus`的大小，也就不会影响公钥的长度了。  

### 导出hex格式的modulus
通过openssl asn1parse可以查看密钥的格式,关于pem格式的密钥的介绍请看[这里](https://www.cnblogs.com/adylee/p/9366518.html)。
```
$ openssl asn1parse -in pub.pem  -i
    0:d=0  hl=4 l= 290 cons: SEQUENCE          
    4:d=1  hl=2 l=  13 cons:  SEQUENCE          
    6:d=2  hl=2 l=   9 prim:   OBJECT            :rsaEncryption
   17:d=2  hl=2 l=   0 prim:   NULL              
   19:d=1  hl=4 l= 271 prim:  BIT STRING
```
其中`BIT STRING`就是modulus，可以看到它的`offset`为19,通过如下命令导出hex格式的modulus：
```
$ openssl asn1parse -in pub.pem  -i -dump -strparse 19
    0:d=0  hl=4 l= 266 cons: SEQUENCE          
    4:d=1  hl=4 l= 257 prim: INTEGER           :A010A8320BCB75DB7D8EBAA5DF56B2FB9CE934B3AE7B0B07A613402813BE785F6909C1BB5E5262AAA3F5385D70C23C950571E8875710A05D7B8AF7DDA3E485F5E1E7BAE2344C189F46551549B5489222ED84241786BF69105542764B55BFD3CEA42ABD5EA2E529844F77083A22045C00DE8B9BEE7F68B0B09BE8BBB199E68ACF9EB985D286D35EC2A4568DF8871D6CB373833E7BBC5DC6E263FDE14D18DF530F4E136FFDA6B78CE926A34E2BFD613DD1B5F42FAD5512572A20ED30E7643C872647EF974830DF44F2EDC654ED5E4FDDD93D645102F97B56774463B0951A5E834B82CE59F29EE1FC161CFCF69C553773AE6AE9760C9E3AB1FCCF76206B17D487C3
  265:d=1  hl=2 l=   3 prim: INTEGER           :010001
```
可以看到其中的modulus跟上小节中使用`openssl rsa`命令导出的一样，但是去掉了开头的`00`。

## 提取公钥
这里对公钥的提取需要使用到Python的[pycrypto](https://github.com/dlitz/pycrypto)库。代码如下:
```python
from Crypto.PublicKey.RSA import construct
e = long(65537) ## Exponent
n = int('A010A8320BCB75DB7D8EBAA5DF56B2FB9CE934B3AE7B0B07A613402813BE785F6909C1BB5E5262AAA3F5385D70C23C950571E8875710A05D7B8AF7DDA3E485F5E1E7BAE2344C189F46551549B5489222ED84241786BF69105542764B55BFD3CEA42ABD5EA2E529844F77083A22045C00DE8B9BEE7F68B0B09BE8BBB199E68ACF9EB985D286D35EC2A4568DF8871D6CB373833E7BBC5DC6E263FDE14D18DF530F4E136FFDA6B78CE926A34E2BFD613DD1B5F42FAD5512572A20ED30E7643C872647EF974830DF44F2EDC654ED5E4FDDD93D645102F97B56774463B0951A5E834B82CE59F29EE1FC161CFCF69C553773AE6AE9760C9E3AB1FCCF76206B17D487C3', 16) ## modulus
pubkey = construct((n, e))
strkey = pubkey.exportKey()
print strkey
```
需要注意的是e、n都需要是long类型的，不然会报错，最终获取到如下的key。
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAoBCoMgvLddt9jrql31ay
+5zpNLOuewsHphNAKBO+eF9pCcG7XlJiqqP1OF1wwjyVBXHoh1cQoF17ivfdo+SF
9eHnuuI0TBifRlUVSbVIkiLthCQXhr9pEFVCdktVv9POpCq9XqLlKYRPdwg6IgRc
AN6Lm+5/aLCwm+i7sZnmis+euYXShtNewqRWjfiHHWyzc4M+e7xdxuJj/eFNGN9T
D04Tb/2mt4zpJqNOK/1hPdG19C+tVRJXKiDtMOdkPIcmR++XSDDfRPLtxlTtXk/d
2T1kUQL5e1Z3RGOwlRpeg0uCzlnynuH8Fhz89pxVN3Ouaul2DJ46sfzPdiBrF9SH
wwIDAQAB
-----END PUBLIC KEY-----
```

### 关于Crypto.PublicKey.RSA.construct
下面是Crypto.PublicKey.RSA.construct方法的介绍:
```
        """Construct an RSA key from a tuple of valid RSA components.
        The modulus **n** must be the product of two primes.
        The public exponent **e** must be odd and larger than 1.
        In case of a private key, the following equations must apply:
        - e != 1
        - p*q = n
        - e*d = 1 mod (p-1)(q-1)
        - p*u = 1 mod q
        :Parameters:
         tup : tuple
                    A tuple of long integers, with at least 2 and no
                    more than 6 items. The items come in the following order:
                    1. RSA modulus (n).
                    2. Public exponent (e).
                    3. Private exponent (d). Only required if the key is private.
                    4. First factor of n (p). Optional.
                    5. Second factor of n (q). Optional.
                    6. CRT coefficient, (1/p) mod q (u). Optional.
        
        :Return: An RSA key object (`_RSAobj`).
        """
```
想要了解这堆说明，需要先了解下[`RSA`算法的原理](https://zhuanlan.zhihu.com/p/44185847)。根据这篇文章我们可以知道如下几点：
1、 模数n = p*q,其中q与q互质
2、 e是一个随机数，它不等于1，并且与(q-1)*(p-1)互质。（算法上m = (q-1)*(p-1)）。
3、 d是一个整数，(e*d)%m = 1, 这里得到的d是私钥的一部分。
公钥：(n, e)
私钥：(n, d)

#### 加密过程
假设明文为a，那么密文b = a^e % n，可以看到我们用到了`(n, e)`，所以加密的时候是使用公钥进行加密的。其中需要注意的是，要想使用公钥（n，e) 加密，要求被加密的数字必须小于n。

#### 解密过程
解密的时候用到私钥`(n, d)`，假设接收到的密文b,那么想要得到明文a,则需要通过如下计算：
a = b^d %n。

### 关于验证过程中的注意事项
验证的时候保证明文的一致性，如明文存在文件中，由于windows 、linux文件格式不一样，可能看上去字符串一样，但是文件的hash是不一样的。