---
layout: post
title:  "前端和 Crypto 那些情事"
date:   2016-07-15 23:00:00
---

> 不知不觉都两个月没写东西了，总感觉成长路上少了点什么，但是又说不出来，直到前天我错失了大好良机才后悔莫及，从今开始要经常鞭策自己去多总结，多深度挖掘才行，机会是不会再等人的。

向来很少关注前端安全，最近学习到 Node 的 Crypto 模块，于是趁此良机好好挖掘一番。

如果经过一番资料收集的话，你会发现 Crypto 已经不知不觉地加入了 Web API 的大家庭，真是润物细无声啊。至于 `window.crypto` 和 `global.crypto` 这对双胞胎谁是老大谁是老幺已经不重要了，反正现在都是前端的菜，我们只关注她们俩和前端的情事嘿嘿。

### window.crypto（Web Crypto API）

这货看起来身材平平，浑身上下貌似没有多少可挖掘的地方，不过却内涵十足，细细品味后别有一番滋味在心头。

首先要注意的是 `Web Crypto API` 依然是一个实验性的 API，也就是说这也算得上是最前瞻的 Web 技术了，玩转 Crypto，你就是一个前端弄潮儿嘿嘿。

`Web Crypto API` 是一个让脚本使用密码学原理来构建系统的一个接口。 她的一个基础特性就是允许 Javascript 来操纵和存储上层的私钥（没有二进制位）。她允许 Javascript 访问以下原语：

* `digest`：用来计算任意数据块的散列，以检测密钥的变化
* `mac`：用来计算消息验证码
* `sign` 和 `verify`：用来给文本进行数字签名以及验证签名
* `encrypt` 和 `decrypt`：用来加密和解密文本
* `import` 和 `export`：用来引入或导出密钥
* `key generation`：用来创建经过加密的安全密钥或密钥对，而不需要使用元密码，不过要用到本地系统的可用熵（数据压缩的最小单位）
* `key wrapping` 和 `unwrapping`：用来传输和接收一个来自第三方的经过另一个密钥加密的密钥 ，而不需要向 Javascript 暴露底层的二进制编码
* `random`：用来生成声音经过加密后的伪随机数列

然而 `Web Crypto API` 没有解决 Web 应用遇到的所有加密问题：

* 没有减轻浏览器同源安全模型的负担，例如当密钥是由多个站点使用一个中央实体来发行的时候
* 无法与专用的硬件进行交互，例如智能芯片、USB 加密狗以及其它的随机数生成器

### 密钥生成算法

##### AES-CBC

<i>AES in Cipher Block Chaining mode</i>（密码块链接模式的 AES 算法）。至于密钥生成，它采用 `PKCS #7` 作为填充方法。

使用这种方法生成的密钥只能通过 `encrypt`、`decrypt`、`wrapKey` 或 `unwrapKey` 来调用，如果使用其它方法来调用这种密钥，脚本将会抛出语法错误而终止。

返回的密钥是一个 `CryptoKey`。

描述 `AES-CBC` 算法的字典必须拥有以下参数：

* `name`：一个包含 `AES-CBC` 的 DOM 字符串
* `length`：一个包含密钥长度（位长）的无符整数，如果这个值不是 128、192 或者 256 的话就会抛出运行错误

##### AES-CTR

<i>AES in Counter mode</i>（计数器模式的 AES 算法）。

主要内容和 `AES-CBC` 大致相同，只是字典参数 `name` 需要包含 `AES-CTR`。

##### AES-GCM

<i>AES in Galois/Counter mode</i>（伽罗瓦运算模式的 AES 算法）。

主要内容和 `AES-CBC` 大致相同，只是字典参数 `name` 需要包含 `AES-GCM`。

##### RSA-OAEP

使用了一个 SHA 哈希函数和一个 `MGF1` 屏蔽生成函数的 `RSAES-OAEP` 算法。

调用密钥的方法和 AES 系列密钥一样。

返回的密钥是一个 `CryptoKeyPair`。

描述 `RSA-OAEP` 算法的字典必须拥有以下参数：

* `name`：一个包含 `RSA-OAEP` 的 DOM 字符串
* `hash`：一个 `HashAlgorithmIdentifier`（使用到的哈希算法的识别码）

##### AES-KW

`the key wrapping in AES algorithm`（密钥以 AES 标准来包装的算法）。

与其它的 AES 系列算法不同，它生成的密钥只能通过 `wrapKey` 或 `unwrapKey` 来调用。其它内容大致相同，字典参数 `name` 需要包含 `AES-KW`。

##### HMAC

`the hash-based message authentication method using SHA hash functions`（使用 SHA 哈希函数的基于散列的信息验证方法）。

调用密钥的方法只能为 `sign` 或 `verify`。

要注意虽然使用了 SHA 哈希函数，但是它返回的密钥却是一个 `CryptoKey`。

描述 `HMAC` 算法的字典必须拥有以下参数：

* `name`：一个包含 `HMAC` 的 DOM 字符串
* `hash`：一个 `HashAlgorithmIdentifier`（使用到的哈希算法的识别码）
* `length`（可选）：一个说明密钥大小的整数。如果没有提供，则使用哈希函数块的大小

##### RSASSA-PKCS1-v1_5

使用 SHA 哈希函数的 `RSASSA-PKCS1-v1_5` 算法。

主要内容和 RSA 系列算法大致相同，字典参数 `name` 需要包含 `RSASSA-PKCS1-v1_5`。

除了这些还有尚未完善的 `ECDSA`、`ECDH` 和 `DH` 算法。

`window.crypto` 花样可真多，一般人可真的吃不消，特别是一些新的花样（`CryptoKey`）仅有 Chrome 和 Firefox hold 得住。我们还是乖乖地去找性感温驯可调教的 `global.crypto` MM 吧(●ˇ∀ˇ●)

### global.crypto

`global.crypto` 提供了封装好 `OpenSSL’s hash`、`HMAC`、`cipher`、`sign` 和 `verify` 的加密功能。

把玩她是如此的 easy，先来上一点前戏：

``` javascript
const crypto = require('crypto');

const pwd = 'tay1989';
const hash = crypto.createHmac('sha256', pwd)
               .update('Mars loves Tay')
               .digest('hex');
console.log(hash); // 3d9d297c2c451676b697ae3dc5eaf2929596ef8e03e94dc668ace21c2d68a236
```

短短三两下的前戏，我们就用 `Hmac` 算法创建了一个密钥，是不是瞬间欲罢不能了，那就继续体验她的各种花式吧。

##### CLass:Certificate

`SPKAC` 最初是由 `Netscape` 实现的一个证书签名请求机制，目前正式指定为 HTML5 keygen 元素的一部分。

Crypto 模块提供了一个 `Certificate` 类来处理 `SPKAC` 数据。最常见的用法就是处理 HTML5 keygen 元素生成的数据。为了实现这个类，Node 在内部使用了 [OpenSSL’s SPKAC implementation][spkac]。

##### Class:Cipher

`Cipher` 的实例被用来加密数据，它可以通过两种方式来使用：

* 作为一个读写流来使用，未加密的数据在流中经过加密产生加密的数据在可读的一端

``` javascript
// 将 Cipher 作为流来使用
const crypto = require('crypto');

const cipher = crypto.createCipher('aes192', 'tay1989');
let encrypted = '';

cipher.on('readable', () => {
  let data = cipher.read();
  if (data) encrypted += data.toString('hex');
});

cipher.on('end', () => {
  console.log(encrypted);
});

cipher.write('Mars loves Tay');
cipher.end(); // a04be3610f210ac8f87db28489016ecb
```

``` javascript
// 将 Cipher 作为管道流来使用
const crypto = require('crypto');
const fs = require('fs');

const cipher = crypto.createCipher('aes192', 'tay1989');
const input = fs.createReadStream('test.js');
const output = fs.createWriteStream('test.enc');

input.pipe(cipher).pipe(output);
```

* 或者通过使用 `cipher.update()` 和 `cipher.final()` 方法来生成加密数据

``` javascript
const crypto = require('crypto');

const cipher = crypto.createCipher('aes192', 'tay1989');
let encrypted = cipher.update('Mars loves Tay', 'utf8', 'hex');

encrypted += cipher.final('hex');
console.log(encrypted); // a04be3610f210ac8f87db28489016ecb
```

##### Class:Decipher

`Decipher` 的实例被用来解密数据，和 `Cipher` 一样，我们也可以通过两种方法来使用：

* 通过流来使用

``` javascript
// 将 Decipher 作为流来使用
const crypto = require('crypto');

const encrypted = 'a04be3610f210ac8f87db28489016ecb';
const decipher = crypto.createDecipher('aes192', 'tay1989');
let decrypted = '';

decipher.on('readable', () => {
  let data = decipher.read();
  if (data) decrypted += data.toString('utf8');
});

decipher.on('end', () => {
  console.log(decrypted);
});

decipher.write(encrypted, 'hex');
decipher.end(); // Mars loves Tay
```

``` javascript
const crypto = require('crypto');
const fs = require('fs');

const decipher = crypto.createDecipher('aes192', 'tay1989');
const input = fs.createReadStream('test.enc');
const output = fs.createWriteStream('test.js');

input.pipe(decipher).pipe(output);
```

* 通过 `decipher.update()` 和 `decipher.final()` 来使用

``` javascript
const crypto = require('crypto');

const decipher = crypto.createDecipher('aes192', 'tay1989');
const encrypted = 'a04be3610f210ac8f87db28489016ecb';
let decrypted = decipher.update(encrypted, 'hex', 'utf8');

decrypted += decipher.final('utf8');
console.log(decrypted); // Mars loves Tay
```

##### Class:DiffieHellman

`DiffieHellman` 类是一个用于创建 `Diffie-Hellman` 密钥交换的工具。

这里不再有流了：

``` javascript
const crypto = require('crypto');
const assert = require('assert');

// 生成 Tay 的密钥
const tay = crypto.createDiffieHellman(520);
const tay_key = tay.generateKeys();

// 生成 Mars 的密钥
const mars = crypto.createDiffieHellman(tay.getPrime(), tay.getGenerator());
const mars_key = mars.generateKeys();

// 交换密钥并生成最终的密钥
const tay_secret = tay.computeSecret(mars_key);
const mars_secret = mars.computerSecret(tay_key);

// 剧终
assert.equal(tay_secret.toString('hex'), mars_secret.toString('hex'));
```

##### Class:Sign

`Sign` 类是一个用来生成数字签名的工具，它同样可以通过两种方式来使用：

* 通过流来使用

``` javascript
const crypto = require('crypto');

const sign = crypto.createSign('RSA-SHA256');
const private_key = getPrivateKeySomehow();

sign.write('Mars loves Tay');
sign.end();

console.log(sign.sign(private_key, 'hex'));
```

* 通过 `sign.update()` 和 `sign.sign()` 的方法来使用

``` javascript
const crypto = require('crypto');

const sign = crypto.createSign('RSA-SHA256');
const private_key = getPrivateKeySomehow();

sign.update('Mars loves Tay');

console.log(sign.sign(private_key, 'hex'));
```

诸如这些算法还有 N 多个，都大同小异。

前端和 Crypto 的情事就此画上句号了。或许光是看这些 API 很无趣，不过如果有心去研究当中的原理，还是可以保持新鲜感的。当中大多算法都是精品，如果再深度开发，则又是一大片世外桃源。

[spkac]: https://www.openssl.org/docs/man1.0.2/apps/spkac.html
