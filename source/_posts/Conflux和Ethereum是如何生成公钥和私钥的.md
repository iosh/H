---
title: Conflux和Ethereum是如何生成公钥和私钥的
date: 2023-10-04 20:47:39
tags: web3
---

在区块链的世界中，公钥和私钥是用户身份的唯一标识和证明，Conflux 分为两个不同的 `space`: **Core Space** 和 **eSpace**，其中 eSpace 与 Ethereum 是完全兼容的，所以下面主要了解 Conflux Core Space 和 Ethereum 是如何生成极为重要的私钥和公钥，以及如何确定用户地址的。

# 私钥和公钥是什么？

Conflux 和 Ethereum 中私钥都是由64个hex字符组成的字符串。

在区块链中，私钥产生公钥，并且私钥需要对消息进行签名，以保证消息的真实性，在理论上，如果不知道私钥那么是无法伪造签名的。区块链的每一笔交易都需要使用私钥进行数字签名。

私钥是一组`随机`获取的数字，这里的随机需要是真随机，而不能是伪随机，因为伪随机使用的种子是已知的，所以结果可以被预测，通过私钥实现的所有权和控制权是对账户控制的根本，如果私钥被盗，那么账户将会被别人完全控制，并且如果私钥丢失，将丧失对账户的所有权，账户中的内容将永远无法被使用，所以务必保护好自己的私钥。

通过系统提供的随机数生成器或者其他真随机生成器，获得一个256位的值，注意这个生成的过程必须是离线的，任何在线的生成都是不安全的，而使用伪随机生成的或者自己随便写的太容易重复，是很不安全的。

将得到的随机数，然后使用256的hash算法，例如 Keccak-256或者sha-256之后就会产生一个256比特的二进制数字，然后将二进制数字转化为16进制hex字符，就得到了一个 64 位的hex格式私钥。

公钥是通过私钥生成的，使用[**椭圆曲线数字签名算法**](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm),这个方法是不可逆的，也就是说通过使用私钥通过算法计算出公钥，但是是无法通过公钥还原出私钥的。一般使用的椭圆曲线库是 secp256k1 。

在区块链中，消息都是使用私钥进行签名的，区块链上的任何人都可以使用公钥来验证签名的有效性。

# Conflux eSpace 和 Ethereum 账户地址生成

因为 Conflux eSpace 和 Ethereum 是完全兼容的，所以两者可以视为一致，都是下面的流程，同一个私钥得到的地址也是一样的。

```text

     ┌───────────────────────────────────┐
     │         1. Private key            │
     │                                   │
     │                                   │
     │ 64 hexadecimal charachter string  │
     │                                   │
     │                                   │
     └────────────────┬──────────────────┘
                      │
                      │ ECDSA
                      │
                      │
     ┌────────────────▼──────────────────┐
     │        2. ECDSA Publish key       │
     │                                   │
     │                                   │
     │    128 hexadecimal character      │
     │                                   │
     │                                   │
     └────────────────┬──────────────────┘
                      │
                      │ Kecca-256
                      │
     ┌────────────────▼──────────────────┐
     │        3. Hashed public key       │
     │                                   │
     │    64 hexadecimal character       │
     │                                   │
     │                                   │
     └────────────────┬──────────────────┘
                      │
                      │  Pruning
                      │
     ┌────────────────▼──────────────────┐
     │         4. Account address        │
     │                                   │
     │     40 hexadecimal character      │
     │                                   │
     └───────────────────────────────────┘
```

1.  生成私钥

私钥的生成除了上面的办法得到一个64位hex格式字符串以外，还可以使用钱包的助记词生成私钥，通过助记词生成私钥过程稍后下面会记录

2.  使用ECDSA椭圆曲线数字签名算法生成公钥

获取到私钥之后就可以使用椭圆曲线数字签名算法来生成公钥，之后就可以得到一个128位16进制的hex字符串，如果加上两位前缀，04 标识未压缩，那么就是130位。

3. 对公钥进行hash处理

使用[Keccak-256](https://en.wikipedia.org/wiki/SHA-3)对公钥进行hash运算

```text
value = Keccak256(pubkey)
```

得到的结果只保留最后的20位，这样就得到了账户的地址，常常会看到地址以0x开头，这表示地址采用16进制的方式标识。

# Conflux Core 的账户地址生成

Conflux Core 的地址不同于Ethereum，Conflux Core 的地址是base32编码格式的而Ethereum是十六进制的，在初期，Conflux Core 也采用16进制的字符串作为地址，但是由于计算方法不一样，导致一个秘钥在Conflux Core 和 Ethereum 得到的地址并不相同，所以容易对用户产生混淆和转账操作错误，所以为了解决这个问题 Conflux 在[CIP-37](https://github.com/Conflux-Chain/CIPs/blob/master/CIPs/cip-37.md)中引入了一种 base32 格式的编码，新格式是直接对原始的十六进制进行了编码，并且添加了独特的地址前缀例如 cfx: 。

```text

           ┌───────────────────────────────────┐
           │         1. Private key            │
           │                                   │
           │                                   │
           │ 64 hexadecimal charachter string  │
           │                                   │
           │                                   │
           └────────────────┬──────────────────┘
                            │
                            │ ECDSA
                            │
                            │
           ┌────────────────▼──────────────────┐
           │        2. ECDSA Publish key       │
           │                                   │
           │                                   │
           │    128 hexadecimal character      │
           │                                   │
           │                                   │
           └────────────────┬──────────────────┘
                            │
                            │ Kecca-256
                            │
           ┌────────────────▼──────────────────┐
           │        3. Hashed public key       │
           │                                   │
           │    64 hexadecimal character       │
           │                                   │
           │                                   │
           └────────────────┬──────────────────┘
                            │
                            │  Pruning
                            │
           ┌────────────────▼──────────────────┐
           │         4. Hex Address            │
           │                                   │
           │     39 hexadecimal character +    │
           │     1 type indicator              │
           └────────────────┬──────────────────┘
                            │
                            │ base32
                            │
           ┌────────────────▼──────────────────┐
           │          5. Account address       │
           │                                   │
           │    netname+:+payload+checksum     │
           │                                   │
           └───────────────────────────────────┘

```

因为Base32的地址是从16进制的编码地址转换而来的，所以需要了解一下如何计算16进制的地址是怎么计算的的。

Conflux 十六进制的地址是20个字节的16进制字符串，加上0x开头一共42个字符的字符串，hex编码的地址以1(3)字符的类型指示符开头，标识地址类型，分为三类

- (0x)1 代表 EOA 账户地址
- (0x)8 代表这是合约地址
- (0x)0 代表Conflux内部地址

所以普通的EOA地址计算是 4bit(1个字符) 的类型加上，使用Keccakhash计算的公钥的结果的最后面的 156 bit(39个字符)， 这样就得到了40个字符的16进制的字符串

然后将16进制转换为base32的地址：

这个地址有一个网络前缀例如cfx:和一段base32地址，例如 cfx:base32address,又或者是 cfx:type.user:base32address

地址的组成是 `网络前缀:${payload}${checksum}`

网络前缀目前有 `cfx` `cfxtest` `net17`

payload 的计算是使用上面得到的16进制的字符+version-byte，这样得到了一个21为的byte数组, 然后从右到左进行编码。

checksum 的计算是 [5bit的网络前缀,0(:分隔符),payload,0,0,0,0,0,0,0,0] 得到一个byte数组，使用 [checksum](https://github.com/bitcoincashorg/bitcoincash.org/blob/master/spec/cashaddr.md#checksum) 中定义的函数进行计算，然后再和payload类似转为base32

这样就形成了Conflux Core的账户地址

最后一步就是删除130位公钥前面的96位，剩余40位

0x0E6147b9099c3e51e5F590DEe17e4bFd626092C9
0x0E6147b9099c3e51e5F590DEe17e4bFd626092C9
cfx:aatgcv73bgsd6ytf80jr72n8kt80e2ew3e4rn5nj0n
cfx:aatgcv73bgsd6ytf80jr72n8kt80e2ew3e4rn5nj0n
cfxtest:aatgcv73bgsd6ytf80jr72n8kt80e2ew3eug2nrcwb
