# Node.js Crypto模块(未完成)
crypto是Node.js 提供的加密模块，包括各种的可逆以及不可逆的加密
> crypto 模块提供了加密功能，包含对 OpenSSL 的哈希、HMAC、加密、解密、签名、以及验证功能的一整套封装。

简单使用
``` js
const crypto = require('crypto');
const secret = 'abcdefg';
const hash = crypto.createHmac('sha256', secret).update('I love cupcakes').digest('hex');
console.log(hash);
```

## API
- Class: **Cipher**
> Cipher类的实例用于加密数据。

这个类可以用在以下两种方法中的一种:

1. 作为`stream`，既可读又可写，未加密数据的编写是为了在可读的方面生成加密的数据.
2. 使用`cipher.update()`和`cipher.final()`方法产生加密的数据



