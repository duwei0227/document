---
layout: post
title: PyCryptodome使用笔记
categories: [Python]
description: PyCryptodome使用笔记
permalink: python/pycryptodome.html
tags: PyCryptodome
---
### 一、安装

```shell
pip install pycryptodome
```

### 二、包模块介绍

| 包名                 | 描述                               |
| -------------------- | ---------------------------------- |
| `Crypto.Cipher`    | 数据的加密、解密模块，例如 `AES` |
| `Crypto.Signature` | 数据的签名、验签                   |
| `Crypto.Hash`      | 消息摘要                           |
| `Crypto.PublicKey` | 用于公钥的生成、导出和导入         |
| `Crypto.Random`    | 用于生成随机数据                   |
| `Crypto.Util`      | 工具类，例如数据填充               |

#### 1、Crypto.PublicKey

用于公钥、私钥证书的生成以及证书格式转换(字符串)

`pycryptodome`支持的Key类型：

* [RSA keys](https://www.pycryptodome.org/en/latest/src/public_key/rsa.html)
* [DSA keys](https://www.pycryptodome.org/en/latest/src/public_key/dsa.html)
* [Elliptic Curve keys](https://www.pycryptodome.org/en/latest/src/public_key/ecc.html)

在导入公钥、私钥时，需要进行格式填充

`RSA`私钥文件结构：

```
-----BEGIN RSA PRIVATE KEY-----
私钥正文
-----END RSA PRIVATE KEY-----
```

`RSA`公钥文件：

```
-----BEGIN PUBLIC KEY-----
公钥正文
-----END PUBLIC KEY-----
```

```
# 生成私钥、公钥
# bits指定生成的私钥文件大小
# passphrase指定私钥文件加密密码
key = RSA.generate(bits=2048)
with open('private_key.pem', 'wb') as fp:
    fp.write(key.export_key(format='PEM', passphrase='123456'))

with open('public_key.pem', 'wb') as fp:
    fp.write(key.publickey().export_key())
    
# 导入公钥、私钥文件
private_key_start = '-----BEGIN RSA PRIVATE KEY-----'
private_key_end = '-----END RSA PRIVATE KEY-----'
public_key_start = r'-----BEGIN PUBLIC KEY-----'
public_key_end = r'-----END PUBLIC KEY-----'

with open('private_key.pem', 'rb') as fp:
    pk = fp.read().decode('utf-8')
    if not pk.startswith(private_key_start):
        pk = private_key_start + pk
        if not pk.endswith(private_key_end):
            pk += private_key_end
            private_key = RSA.import_key(pk, passphrase='123456')
```

#### 2、Crypto.Cipher

`Cipher`模块用于数据的加密和解密

* 对称算法：双方使用相同的密钥进行数据的加密和解密。对称加密的速度通常比较快，代表有 `AES`,`DES3`
* 非对称算法

  发送方和接收方使用不同的密钥。发送方使用公钥（非机密）加密，而接收方使用私钥（机密）解密。非对称密钥通常很慢。代表有 `PKCS#1 OAEP (RSA)`

`pycryptodome`支持的算法：

* [Single DES](https://www.pycryptodome.org/en/latest/src/cipher/des.html) and [Triple DES](https://www.pycryptodome.org/en/latest/src/cipher/des3.html) (block ciphers)
* [RC2](https://www.pycryptodome.org/en/latest/src/cipher/arc2.html) (block cipher)
* [ARC4](https://www.pycryptodome.org/en/latest/src/cipher/arc4.html) (stream cipher)
* [Blowfish](https://www.pycryptodome.org/en/latest/src/cipher/blowfish.html) (block cipher)
* [CAST-128](https://www.pycryptodome.org/en/latest/src/cipher/cast.html) (block cipher)
* [PKCS#1 v1.5 encryption (RSA)](https://www.pycryptodome.org/en/latest/src/cipher/pkcs1_v1_5.html) (asymmetric cipher)

##### AES(Advanced Encryption Standard)

`AES`具有16字节的固定数据块大小。它的密钥可以是128、192或256位(bit)长。

* 使用ECB(Electronic Code Book)进行数据的加密解密：

    `ECB`将加密的数据分成若干组，每组的大小跟加密密钥长度相同，然后每组都用相同的密钥进行加密。所以在加密的时候需要对明文数据进行填充保证数据的长度是密码长度的倍数。

```python
# 数据加密
plain_text = b'hello'
key = get_random_bytes(16)
cipher = AES.new(key, AES.MODE_ECB)
ct_bytes = cipher.encrypt(pad(plain_text, AES.block_size))
enc_data = base64.b64encode(ct_bytes).decode('utf-8')


#数据解密
enc_data = '/zgUiJHvUyu1h+wr9evHCA=='
key = 'dhcvKhdF3kwoXUjGTmK3Ww=='
cipher = AES.new(base64.b64decode(key), AES.MODE_ECB)
pad_plain_data = cipher.decrypt(base64.b64decode(enc_data))
plain_data = unpad(pad_plain_data, AES.block_size)
print(plain_data.decode('utf-8'))
```

* `CBC`模式的加密首先也是将明文分成固定长度的块，然后将前面一个加密块输出的密文与下一个要加密的明文块进行异或操作，将计算结果再用密钥进行加密得到密文。第一明文块加密的时候，因为前面没有加密的密文，所以需要一个初始化向量。

```python
# 数据加密
plain_text = b'hello'
key = get_random_bytes(16)
# 初始偏移量可以不指定，如果不指定非随机生成
iv = get_random_bytes(16)
cipher = AES.new(key, AES.MODE_CBC, iv=iv)
ct_bytes = cipher.encrypt(pad(plain_text, AES.block_size))
enc_data = base64.b64encode(ct_bytes).decode('utf-8')
print('加密密文==', enc_data)

# 数据解密
cipher = AES.new(key, AES.MODE_CBC, iv=iv)
pad_plain_data = cipher.decrypt(base64.b64decode(enc_data))
plain_text = unpad(pad_plain_data, AES.block_size).decode('utf-8')
print('解密后明文==', plain_text)
```

##### RSA

`RSA`加解密一般是公钥加密，私钥解密;`PyCryptodome`当前建议使用 `PKCS1_OAEP`算法；如果加解密双方为 `Python`与 `Java`且 `Java`端采用 `RSA/ECB/PKCS1Padding`时，`Python`端需要采用 `PKCS1_v1_5`

**加解密模块为：**`Crypto.Cipher`模块

```
import base64
from Crypto.Cipher import PKCS1_OAEP, PKCS1_v1_5
from Crypto.PublicKey import RSA

# 数据加密--公钥
data = "I met aliens in UFO. Here is the map.".encode("utf-8")
with open('public_key.pem', mode='rb') as fp:
    # 如果是读取字符串公钥需要补齐字符串格式
    public_key = RSA.import_key(fp.read())
    cipher_rsa = PKCS1_v1_5.new(public_key)
    #cipher_rsa = PKCS1_OAEP.new(public_key)
    enc_data = cipher_rsa.encrypt(data)
    print(base64.b64encode(enc_data).decode('utf-8'))
    
# 数据解密--私钥
enc_data = 'rDrJC9aaAmL5szCKzPy1peJ/Yx8noKuBEDFsUGSjDQgxRId3jgG9fAVuXpJnqRtHL+RXncOLuqRtKVowyt5H3Ag/KZm+7RJDeY7nT3AvgUiy0xfQgZChOW63BDoIMFIDZqXv5yPwqoMDT2Tz3Yj7FWYb5/0zbbFbJqwyA3knY2vUOFOp/YyqpZ7gzzyKjgAgi8pWgEDs2bXMirGIWA1McHFVdDfHF7Xi2s2Ob2oHylW7rJT9SXfQh0CPvWRxTstv6e8GWRQo4qYK/qGxqGQfLexSw2/o10DIAsYOxN9xubVKPLA+jXQchSH7Z/Js2g9t3WSI8027JXg//HTB5AO74A=='
with open('private_key.pem', mode='rb') as fp:
    # 如果是读取字符串公钥需要补齐字符串格式
    pk = RSA.import_key(fp.read(), passphrase='123456')
    cipher = PKCS1_v1_5.new(pk)
    #sentinel:当发生错误时返回对象
    plain_data_byte = cipher.decrypt(base64.b64decode(enc_data), sentinel='no')
    # PKCS1_OAEP解密
    #cipher = PKCS1_OAEP.new(pk)
    #plain_data_byte = cipher.decrypt(base64.b64decode(enc_data))
    print(plain_data_byte.decode('utf-8'))

```

#### 3、Crypto.Signature

`RSA`使用私钥签名，公钥验签；签名验签在 `Crypto.Signature`模块包提供

`pycryptodome`支持的算法：

* [PKCS#1 v1.5 (RSA)](https://www.pycryptodome.org/en/latest/src/signature/pkcs1_v1_5.html)
* [PKCS#1 PSS (RSA)](https://www.pycryptodome.org/en/latest/src/signature/pkcs1_pss.html)
* [Digital Signature Algorithm (DSA and ECDSA)](https://www.pycryptodome.org/en/latest/src/signature/dsa.html)

```
import base64
from Crypto.PublicKey import RSA
from Crypto.Signature import PKCS1_v1_5

plain_data = "Hello, World!!"
with open('private_key.pem', mode='rb') as fp:
    # 如果是读取字符串公钥需要补齐字符串格式
    private_key = RSA.import_key(fp.read(), passphrase='123456')
    hash_data = SHA256.new(plain_data.encode('utf-8'))
    sign_data = PKCS1_v1_5.new(private_key).sign(hash_data)
    print(base64.b64encode(sign_data).decode('utf-8'))
    
# 验签
plain_data = "Hello, World!!"
sign_data = 'itVttV7vTzB8OoglNtuPn2PgpRzpZ3ILOnLWQDAqiOuQig9Whs0a6ht5P3oc1baUZ6PfVksSNzuPd8sNaR9eQeKkaSiuiJP11BJ8cugPuSyn'
with open('public_key.pem', mode='rb') as fp:
    hash_data = SHA256.new(plain_data.encode('utf-8'))
    public_key = RSA.import_key(fp.read())
    try:
        PKCS1_v1_5.new(public_key).verify(hash_data, signature=base64.b64decode(sign_data))
    except Exception as e:
        print('验签失败', e)
```

#### 4、Crypto.Hash

`Hash`模块主要用于消息摘要

`pycryptodome`支持的算法：

* SHA-2 family (FIPS 180-4)

  * [SHA-224](https://www.pycryptodome.org/en/latest/src/hash/sha224.html)
  * [SHA-256](https://www.pycryptodome.org/en/latest/src/hash/sha256.html)
  * [SHA-384](https://www.pycryptodome.org/en/latest/src/hash/sha384.html)
  * [SHA-512, SHA-512/224, SHA-512/256](https://www.pycryptodome.org/en/latest/src/hash/sha512.html)
* SHA-3 family (FIPS 202)

  * [SHA3-224](https://www.pycryptodome.org/en/latest/src/hash/sha3_224.html)
  * [SHA3-256](https://www.pycryptodome.org/en/latest/src/hash/sha3_256.html)
  * [SHA3-384](https://www.pycryptodome.org/en/latest/src/hash/sha3_384.html)
  * [SHA3-512](https://www.pycryptodome.org/en/latest/src/hash/sha3_512.html)
  * [TupleHash128](https://www.pycryptodome.org/en/latest/src/hash/tuplehash128.html)
  * [TupleHash256](https://www.pycryptodome.org/en/latest/src/hash/tuplehash256.html)
* BLAKE2

  * [BLAKE2s](https://www.pycryptodome.org/en/latest/src/hash/blake2s.html)
  * [BLAKE2b](https://www.pycryptodome.org/en/latest/src/hash/blake2b.html)

示例：

```python
def cal_hash256(data: str) -> str:
    """
    sha256计算

    :param data: 需要计算hash值的文本字符串

    :return 十六进制hash值
    """
    return SHA256.new(data.encode('utf-8')).hexdigest()
```

### 三、参考

[https://www.pycryptodome.org/en/latest/](https://www.pycryptodome.org/en/latest/)
