---
title: openssl初探
date: 2020-05-21 14:57:26
tags: openssl
---
## 非对称加密算法概述
&emsp;&emsp;非对称加密算法也称公开密钥算法，其解决了对称加密算法密钥分配的问题，非对称加密算法基本特点如下：
1、加密密钥和解密密钥不同
2、密钥对中的一个密钥可以公开
3、根据公开密钥很难推算出私人密钥
&emsp;&emsp;根据非对称加密算法的特点，可用户数字签名、密钥交换、数据加密。但是由于非对称加密算法较对称加密算法加密速度慢很多，故最常用的用途是数字签名和密钥交换。
&emsp;&emsp;目前常用的非对称加密算法有RSA, DH和DSA三种，但并非都可以用于密钥交换和数字签名。而是RSA可用于数字签名和密钥交换，DH算法可用于密钥交换，而DSA算法专门用户数字签名。
&emsp;&emsp;openssl支持以上三种算法，并为三种算法提供了丰富的指令集，目前使用最多的算法是rsa加密算法，本文介绍OpenSSL在rsa加密算法上的使用。
<!--more-->

## openssl rsa算法相关指令与用法
&emsp;&emsp;RSA虽然可以数字签名、密钥交换和数据加密，但是RSA加密数据速度慢，通常不使用RSA加密数据。对于在实际应用中是使用RSA算法进行签名还是进行加密，可以通过公钥与私钥的使用进行区别：
&emsp;&emsp;公钥加密：用途是密钥交换，用户A使用用户B的公钥将少量数据加密发送给B，B用自己的私钥解密数据。
&emsp;&emsp;私钥签名：用途是数字签名，用户A使用自己的私钥将数据的摘要信息加密一并发送给B，B用A的公钥解密摘要信息并验证。

### openssl genrsa命令介绍
openssl genras 命令用于生成秘钥对，其用法如下：
```bash
xxx@cmos:~$ openssl genrsa -
usage: genrsa [args] [numbits]                                                     //密钥位数，建议1024及以上
 -des            encrypt the generated key with DES in cbc mode                    //生成的密钥使用des方式进行加密
 -des3           encrypt the generated key with DES in ede cbc mode (168 bit key)  //生成的密钥使用des3方式进行加密
 -seed
                 encrypt PEM output with cbc seed                                  //生成的密钥还是要seed方式进行
 -aes128, -aes192, -aes256
                 encrypt PEM output with cbc aes                                   //生成的密钥使用aes方式进行加密
 -camellia128, -camellia192, –camellia256 
                 encrypt PEM output with cbc camellia                              //生成的密钥使用camellia方式进行加密
 -out file       output the key to 'file                                           //生成的密钥文件，可从中提取公钥
 -passout arg    output file pass phrase source                                    //指定密钥文件的加密口令，可从文件、环境变量、终端等输入
 -f4             use F4 (0x10001) for the E value                                  //选择指数e的值，默认指定该项，e值为65537 -3              use 3 for the E value                                             //选择指数e的值，默认值为65537，使用该选项则指数指定为3
 -engine e       use engine e, possibly a hardware device.                         //指定三方加密库或者硬件
 -rand file:file:...
                 load the file (or the files in the directory) into                //产生随机数的种子文件
                 the random number generator
```
&emsp;&emsp;通过上面的使用介绍我们来创建一个私钥，我们指定私钥的加密算法为aes128，创建密码为123456（在使用私钥时需要输入密码，如果设置了创建密码），密钥位数为2048位。
```
openssl genrsa -aes128 -out rsa_2048.pem -passout pass:123456 2048
```
&emsp;&emsp;在创建rsa私钥的时候是必须设置口令的，此密码用于加密私钥文件。以后在使用openssl提供的命令或者api再操作此私钥文件时需要输入口令。如果觉得输入口令不方便，也可以通过如下命令将口令去除：
```
openssl rsa -in rsa_2048.pem -out rsa_2048.pem
```

### openssl rsa 命令介绍
rsa指令用户管理生成的密钥，其用法如下:
```bash
xxx@cmos:~$ openssl rsa -
unknown option -
rsa [options] <infile >outfile     
where options are
 -inform arg     input format - one of DER NET PEM                      //输入文件格式，默认pem格式
 -outform arg    output format - one of DER NET PEM                     //输入文件格式，默认pem格式
 -in arg         input file                                             //输入文件
 -sgckey         Use IIS SGC key format                                 //指定SGC编码格式，兼容老版本，不应再使用
 -passin arg     input file pass phrase source                          //指定输入文件的加密口令，可来自文件、终端、环境变量等
 -out arg        output file                                            //输出文件
 -passout arg    output file pass phrase source                         //指定输出文件的加密口令，可来自文件、终端、环境变量等
 -des            encrypt PEM output with cbc des                        //使用des加密输出的文件
 -des3           encrypt PEM output with ede cbc des using 168 bit key  //使用des3加密输出的文件
 -seed           encrypt PEM output with cbc seed                       //使用seed加密输出的文件
 -aes128, -aes192, -aes256
                 encrypt PEM output with cbc aes                        //使用aes加密输出的文件
 -camellia128, -camellia192, -camellia256
                 encrypt PEM output with cbc camellia                   //使用camellia加密输出的文件呢
 -text           print the key in text                                  //以明文形式输出各个参数值
 -noout          don't print key out                                    //不输出密钥到任何文件
 -modulus        print the RSA key modulus                              //输出模数指
 -check          verify key consistency                                 //检查输入密钥的正确性和一致性
 -pubin          expect a public key in input file                      //指定输入文件是公钥
 -pubout         output a public key                                    //指定输出文件是公钥
 -engine e       use engine e, possibly a hardware device.              //指定三方加密库或者硬件
xlzh@cmos:~$
```
1. rsa添加和去除密钥的保护口令
```bash
/*生成不加密的RSA密钥*/
xxx@cmos:~/test$ openssl genrsa -out RSA.pem
Generating RSA private key, 512 bit long modulus
..............++++++++++++
.....++++++++++++
e is 65537 (0x10001)
/*为RSA密钥增加口令保护*/
xxx@cmos:~/test$ openssl rsa -in RSA.pem -des3 -passout pass:123456 -out E_RSA.pem
writing RSA key
/*为RSA密钥去除口令保护*/
xxx@cmos:~/test$ openssl rsa -in E_RSA.pem -passin pass:123456 -out P_RSA.pem
writing RSA key
/*比较原始后的RSA密钥和去除口令后的RSA密钥，是一样*/
xxx@cmos:~/test$ diff RSA.pem P_RSA.pem
```

2. 修改密钥的保护口令与算法
```bash
/*生成RSA密钥*/
xxx@cmos:~/test$ openssl genrsa -des3 -passout pass:123456 -out RSA.pem
Generating RSA private key, 512 bit long modulus
..................++++++++++++
......................++++++++++++
e is 65537 (0x10001)
/*修改加密算法为aes128，口令是123456*/
xxx@cmos:~/test$ openssl rsa -in RSA.pem -passin pass:123456 -aes128 -passout pass:123456 -out E_RSA.pem
writing RSA key
```

3. 查看密钥对中的各个参数
```bash
xxx@cmos:~/test$ openssl rsa -in RSA.pem -des -passin pass:123456 -text -noout
```

4. 提取密钥中的公钥并打印模数值(生成私钥后，一般第一步就是导出公钥)
```bash
/*提取公钥，用pubout参数指定输出为公钥*/
xxx@cmos:~/test$ openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem
writing RSA key
/*打印公钥中模数值*/
xxx@cmos:~/test$ openssl rsa -in pub.pem -pubin -modulus -noout
Modulus=C35E0B54041D78466EAE7DE67C1DA4D26575BC1608CE6A199012E11D10ED36E2F7C651D4D8B40D93691D901E2CF4E21687E912B77DCCE069373A7F6585E946EF
```

5. 转换密钥的格式
```bash
/*把pem格式转化成der格式，使用outform指定der格式*/
xxx@cmos:~/test$ openssl rsa -in RSA.pem -passin pass:123456 -des -passout pass:123456 -outform der -out rsa.der
writing RSA key
/*把der格式转化成pem格式，使用inform指定der格式*/
xxx@cmos:~/test$ openssl rsa -in rsa.der -inform der -passin pass:123456 -out rsa.pem
```

### openssl rsautl 命令介绍
&emsp;&emsp;前面介绍的genrsa与rsa命令是生成密钥与对密钥进行管理的命令，rsautl则是真正用于密钥交换和数字签名，实质上就是用公钥或者私钥进行加密。
&emsp;&emsp;而无论是使用公钥加密还是私钥加密，RSA每次能够加密的数据长度不能超过RSA密钥长度，并且根据具体的补齐方式不同输入的加密数据最大长度也不一样，而输出长度则总是跟RSA密钥长度相等。RSA不同的补齐方法对应的输入输入长度如下表：  

|   数据补齐方式     |   输入数据长度           |   输出数据长度  |   参数字符串   |
|   :---------      |      :---------         |   :----------  |  ---------:  |
|  PKCS#1 v1.5      |   少于(密钥长度-11)字节   | 同密钥长度      | -pkcs        |
|  PKCS#1 OAEP      |   少于(密钥长度-11)字节   | 同密钥长度      | -oaep        |
| PKCS#1 for SSLv23 |   少于(密钥长度-11)字节   | 同密钥长度      | -ssl         |
| 不使用补齐         |   同密钥长度             | 同密钥长度       | -raw        |

rsautl指令用法如下:
```bash
xlzh@cmos:~$ openssl rsautl - 
Usage: rsautl [options]                  
-in file        input file                                           //输入文件
-out file       output file                                          //输出文件
-inkey file     input key                                            //输入的密钥
-keyform arg    private key format - default PEM                     //指定密钥格式
-pubin          input is an RSA public                               //指定输入的是RSA公钥
-certin         input is a certificate carrying an RSA public key    //指定输入的是证书文件
-ssl            use SSL v2 padding                                   //使用SSLv23的填充方式
-raw            use no padding                                       //不进行填充
-pkcs           use PKCS#1 v1.5 padding (default)                    //使用V1.5的填充方式
-oaep           use PKCS#1 OAEP                                      //使用OAEP的填充方式
-sign           sign with private key                                //使用私钥做签名
-verify         verify with public key                               //使用公钥认证签名
-encrypt        encrypt with public key                              //使用公钥加密
-decrypt        decrypt with private key                             //使用私钥解密
-hexdump        hex dump output                                      //以16进制dump输出
-engine e       use engine e, possibly a hardware device.            //指定三方库或者硬件设备
-passin arg    pass phrase source                                    //指定输入的密码
```
1. 使用rsautl进行加密与解密操作（公钥加密，私钥解密）
```
/*生成RSA密钥*/
xxx@cmos:~/test$ openssl genrsa -des3 -passout pass:123456 -out RSA.pem 
Generating RSA private key, 512 bit long modulus
............++++++++++++
...++++++++++++
e is 65537 (0x10001)
/*提取公钥*/
xxx@cmos:~/test$ openssl rsa -in RSA.pem -passin pass:123456 -pubout -out pub.pem 
writing RSA key
/*使用RSA作为密钥进行加密，实际上使用其中的公钥进行加密*/
xxx@cmos:~/test$ openssl rsautl -encrypt -in plain.txt -inkey RSA.pem -passin pass:123456 -out enc.txt
/*使用RSA作为密钥进行解密，实际上使用其中的私钥进行解密*/
xxx@cmos:~/test$ openssl rsautl -decrypt -in enc.txt -inkey RSA.pem -passin pass:123456 -out replain.txt
/*比较原始文件和解密后文件*/
xxx@cmos:~/test$ diff plain.txt replain.txt 
/*使用公钥进行加密*/
xxx@cmos:~/test$ openssl rsautl -encrypt -in plain.txt -inkey pub.pem -pubin -out enc1.txt
/*使用RSA作为密钥进行解密，实际上使用其中的私钥进行解密*/
xxx@cmos:~/test$ openssl rsautl -decrypt -in enc1.txt -inkey RSA.pem -passin pass:123456 -out replain1.txt
/*比较原始文件和解密后文件*/
xxx@cmos:~/test$ diff plain.txt replain1.txt
```

2. 使用rsautl进行签名和验证操作
```
/*提取PCKS8格式的私钥*/
xxx@cmos:~/test$ openssl pkcs8 -topk8 -in RSA.pem -passin pass:123456 -out pri.pem -nocrypt
/*使用RSA密钥进行签名，实际上使用私钥进行加密*/
xxx@cmos:~/test$ openssl rsautl -sign -in plain.txt -inkey RSA.pem -passin pass:123456 -out sign.txt
/*使用RSA密钥进行验证，实际上使用公钥进行解密*/
xxx@cmos:~/test$ openssl rsautl -verify -in sign.txt -inkey RSA.pem -passin pass:123456 -out replain.txt
/*对比原始文件和签名解密后的文件*/
xxx@cmos:~/test$ diff plain.txt replain.txt 
/*使用私钥进行签名*/
xxx@cmos:~/test$ openssl rsautl -sign -in plain.txt -inkey pri.pem  -out sign1.txt
/*使用公钥进行验证*/
xxx@cmos:~/test$ openssl rsautl -verify -in sign1.txt -inkey pub.pem -pubin -out replain1.txt
/*对比原始文件和签名解密后的文件*/
xxx@cmos:~/test$ cat plain replain1.txt
```
&emsp;&emsp;这种使用方法其实还是对原文件的加密与解密操作，因为真正的签名与验签过程是需要给源文件进行摘要提取，然后对摘要进行签名，主要通过openssl dgst命令进行。

## openssl dgst签名与验签
dgst指令用法介绍如下：
```
xxx@cmos:~/test$ openssl dgst -
unknown option '-'
options are
-c              to output the digest with separating colons        //输出的摘要信息以分号隔离，和-hex同时使用
-r              to output the digest in coreutils format           //指定输出的格式
-d              to output debug info                               //输出BIO调试信息
-hex            output as hex dump                                 //以16进制打印输出结果
-binary         output in binary form                              //输出二进制结果
-hmac arg       set the HMAC key to arg                            //指定hmac的key
-non-fips-allow allow use of non FIPS digest                       //允许使用不符合fips标准的摘要算法
-sign   file    sign digest using private key in file              //执行签名操作，后面指定私钥文件
-verify file    verify a signature using public key in file        //执行验证操作，后面指定公钥文件，与prverfify不能同时使用
-prverify file  verify a signature using private key in file       //执行验证操作，后面指定密钥文件，与verfify不能同时使用
-keyform arg    key file format (PEM or ENGINE)                    //指定密钥文件格式，pem或者engine
-out filename   output to filename rather than stdout              //指定输出文件，默认标准输出
-signature file signature to verify                                //指定签名文件，在验证签名时使用
-sigopt nm:v    signature parameter                                //签名参数
-hmac key       create hashed MAC with key                         //制作一个hmac 使用key
-mac algorithm  create MAC (not neccessarily HMAC)                 //制作一个mac
-macopt nm:v    MAC algorithm parameters or key                    //mac算法参数或者key
-engine e       use engine e, possibly a hardware device.          //使用硬件或者三方加密库
-md4            to use the md4 message digest algorithm            //摘要算法使用md4
-md5            to use the md5 message digest algorithm            //摘要算法使用md5
-ripemd160      to use the ripemd160 message digest algorithm      //摘要算法使用ripemd160
-sha            to use the sha message digest algorithm            //摘要算法使用sha
-sha1           to use the sha1 message digest algorithm           //摘要算法使用sha1
-sha224         to use the sha224 message digest algorithm         //摘要算法使用sha223
-sha256         to use the sha256 message digest algorithm         //摘要算法使用sha256
-sha384         to use the sha384 message digest algorithm         //摘要算法使用sha384
-sha512         to use the sha512 message digest algorithm         //摘要算法使用sha512
-whirlpool      to use the whirlpool message digest algorithm      //摘要算法使用whirlpool
```
使用RSA密钥进行签名验证操作:
```bash
openssl dgst -sha256 -sign rsa_2048.pem -out stroptr.c.sign stroptr.c  # 生成摘要签名
openssl  dgst  -verify pub.pem -sha256 -signature stroptr.c.sign  stroptr.c # 验证签名
```