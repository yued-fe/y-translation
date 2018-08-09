>原文地址：https://developers.google.com/web/fundamentals/push-notifications/web-push-protocol

>译文地址：

>译者：张卓

>校对者：

# Web 推送协议


我们已经看到了如何使用一个库来触发消息推送，但这些库究竟做了什么呢？

嗯，他们发送了网络请求，同时确保这些请求符合[Web 推送协议](https://tools.ietf.org/html/draft-ietf-webpush-protocol)的规范。

![Diagram of sending a push message from your server to a push
service.](./images/svgs/server-to-push-service.svg)

这一部分大致会描述服务器如何使用应用程序服务器密钥（Application server keys）来识别自身，以及如何发送签名的有效负载（payload）和关联数据。

这不是 Web 推送容易理解的一个方面，而我（作者）也不是加密专家，但还是让我们看看每一部分，因为它让我们更容易理解这些库的原理。

## 应用程序服务器密钥

当我们订阅一个用户时，我们会传入一个`applicationServerKey`。这个 key 会被传递给推送服务（push service），并被用于检查订阅用户的和推送消息的应用程序是不是同一个。

当我们触发推送消息时，我们将会发送一组 headers 来用于让推送服务进行验证。（通过 [VAPID 规范](https://tools.ietf.org/html/draft-thomson-webpush-vapid)）

这一切究竟意味着什么以及究竟发生了什么？ 下面是应用程序服务器身份验证所采取的步骤：

1. 应用程序服务器使用它的 **私有应用程序密钥（私钥）** 来对一些 JSON 信息进行签名。
2. 这些签名的信息作为 POST 请求中的 header 发送给推送服务。
3. 推送服务将之前保存下来的公钥（用户在调用`pushManager.subscribe()`进行订阅推送服务时，推送服务会将传递的公钥进行保存）来校验接收到的消息是由与之匹配的私钥进行签名的。*注意*: 公钥是传递给subscribe 方法的 `applicationServerKey`。
4. 如果签名的信息合法，则推送服务将推送消息发送给用户。

下面是这种信息流的一个例子。（请注意左下角的图例表示公钥和私钥。）

![Illustration of how the private application server key is used when sending a
message.](./images/svgs/application-server-key-send.svg)

添加到请求头的“签名信息”是 JSON Web 令牌（JSON web token）。

### JSON Web 令牌

[JSON Web 令牌](https://jwt.io/) (缩写为 JWT) 是一种向第三方发送消息的方式，接收方可以验证谁发送了它。

当第三方收到消息时，他们需要获取发送者的公钥并使用它来验证 JWT 的签名。如果签名有效的话，那 JWT 一定是使用了匹配的私钥进行签名，一定是来自预期的发送者。

[https://jwt.io/](https://jwt.io/) 上有许多库可以用来签名，并且我（作者）也建议你使用这些库。但是为了完整起见，我们来看看如何手动创建签名的 JWT。

### Web 推送和签名的 JWT

一个签名的 JWT 只是一串字符串，也可以被认为是由点号连接的三个字符串。

![A illustration of the strings in a JSON Web
Token](./images/svgs/authorization-jwt-diagram-header.svg)

第一个和第二个字符串（JWT Info 和 JWT Data）是已经使用 base64 进行编码的 JSON 片段，这意味着它是公开可读的。

第一个字符串是有关 JWT 本身的信息，指出使用了哪一种算法来创建签名。

Web 推送的 JWT Info 必须包含以下信息：


    {  
      "typ": "JWT",  
      "alg": "ES256"  
    }

第二个字符串是 JWT Data。 它提供了有关 JWT 的发送者，目的人以及有效期等信息。

对于 Web 推送，数据将具有以下字段：


    {  
      "aud": "https://some-push-service.org",
      "exp": "1469618703",
      "sub": "mailto:example@web-push-book.org"  
    }


`aud` 代表 “观众（audience）”，即JWT的用户。对于网络推送，“观众”是推送服务，因此我们将其设置为推送服务的源。

`exp` 代表 JWT 的期限，这可以防止黑客在拦截后重新使用 JWT。 到期时间是以秒为单位的时间戳，必须不大于24小时。

在 Node.js 中，使用以下命令设置到期时间：

    Math.floor(Date.now() / 1000) + (12 * 60 * 60)

这里用了12小时而不是24小时，来避免发送应用程序和推送服务之间的时钟差异产生的任何问题。

最后， `sub` 必须是 URL 或 `mailto` 邮件地址。这样，如果推送服务需要联系发件人，它可以从 JWT 找到联系信息。（这也是网络推送库需要一个电子邮件地址的原因）。

就像 JWT Info 一样，JWT Data 被编码为URL安全的 base64 字符串。

第三个字符串签名，是取前两个字符串的结果（JWT Info和JWT Data）并用点号连接（称为“未签名的令牌”），然后签名生成的。

签名过程需要使用 ES256 加密 “未签名的令牌”。根据 [JWT 规范](https://tools.ietf.org/html/rfc7519)，ES256 是“椭圆曲线数字签名算法（ECDSA）使用 P-256 曲线和 SHA-256 哈希算法”的缩写。使用 web 加密技术，你可以像这样创建签名：

    // 将 UTF-8 编码的 string 转化为 ArrayBuffer 的工具库
    const utf8Encoder = new TextEncoder('utf-8');

    // “未签名的令牌”是由 URL 安全的 base64 算法进行编码的 header 和 body 的组合。
    const unsignedToken = .....;

    // 使用 ES256 (SHA-256 over ECDSA) 签名 |unsignedToken|
    const key = {
      kty: 'EC',
      crv: 'P-256',
      x: window.uint8ArrayToBase64Url(
        applicationServerKeys.publicKey.subarray(1, 33)),
      y: window.uint8ArrayToBase64Url(
        applicationServerKeys.publicKey.subarray(33, 65)),
      d: window.uint8ArrayToBase64Url(applicationServerKeys.privateKey),
    };

    // 使用服务器的私钥签名 |unsignedToken|，来生成签名
    return crypto.subtle.importKey('jwk', key, {
      name: 'ECDSA', namedCurve: 'P-256',
    }, true, ['sign'])
    .then((key) => {
      return crypto.subtle.sign({
        name: 'ECDSA',
        hash: {
          name: 'SHA-256',
        },
      }, key, utf8Encoder.encode(unsignedToken));
    })
    .then((signature) => {
      console.log('Signature: ', signature);
    });


推送服务可以使用公共应用程序服务器密钥验证 JWT 以解密签名，并确保解密的字符串与“未签名的令牌”（即JWT中的前两个字符串）相同。

签名的 JWT（即通过点连接的所有三个字符串）将在前面拼接上 WebPush 作为 header 中 Authorization 的值发送给 Web 推送服务，如下所示：

    Authorization: 'WebPush <JWT Info>.<JWT Data>.<Signature>'

Web 推送协议还规定公共应用程序服务器密钥必须在 header Crypto-key 中一起发送。密钥需要使用 URL 安全的 base64算法进行编码，并加上 `p256ecdsa=` 的前缀。

    Crypto-Key: p256ecdsa=<URL Safe Base64 Public Application Server Key>

## 有效负载加密

接下来让我们看一下如何使用推送消息发送有效负载，以便当我们的Web应用程序收到推送消息时，它可以访问接收的数据。

任何使用过其他推送服务的人都会提出一个相同的问题，那就是为什么网络推送有效负载需要加密？使用原生应用，推送消息可以以纯文本形式发送数据。

Web推送的一部分优点在于，因为所有推送服务都使用相同的 API（Web 推送协议），所以开发人员不必关心推送服务是谁。我们只要以正确的格式发出请求，就可以发送推送消息。这样做的缺点是，开发人员可能会将消息发送到不值得信任的推送服务。通过加密有效负载，推送服务无法读取发送的数据，只有浏览器才能解密信息，这可以保护用户的数据。

有效负载的加密在[消息加密规范](https://tools.ietf.org/html/draft-ietf-webpush-encryption)中进行了定义。

在我们查看加密推送消息有效负载的具体步骤之前，我们应该先介绍一些在加密过程中将使用的技术。（感谢 Mat Scales 非常优秀的关于推送加密的文章）

### ECDH 和 HKDF

ECDH 和 HKDF 都在整个加密过程中使用，为加密信息提供了很多好处。

#### ECDH: 椭圆曲线迪菲-赫尔曼金钥交换

想象一下，你有两个想要分享信息的人，Alice 和 Bob，他们都有自己的公钥和私钥，并且互相分享他们的公钥。

使用 ECDH 生成的密钥的有用之处是，Alice 可以使用她的私钥和 Bob 的公钥来创建一个秘密值“X”，Bob 也可以这样做，利用他的私钥和 Alice 的公钥独立创建相同的值'X'，这使得'X'成为共享秘密。而 Alice 和 Bob 只需要共享他们的公钥，他们就可以使用'X'来加密和解密他们之间的消息。

据我所知，ECDH 定义了曲线的属性，它保证了可以同时生成一个相同的共享秘密“X”。

这是对 ECDH 的一个抽象的解释，如果想了解更多，我建议观看此[视频](https://www.youtube.com/watch?v=F3zzNa42-tQ)。

在代码方面，大多数语言/平台都带有库来轻松的生成这些密钥。

在 node 中，我们执行以下操作：

    const keyCurve = crypto.createECDH('prime256v1');
    keyCurve.generateKeys();

    const publicKey = keyCurve.getPublicKey();
    const privateKey = keyCurve.getPrivateKey();

#### HKDF: 基于 HMAC 的密钥推导函数

维基百科对 [HKDF](https://tools.ietf.org/html/rfc5869) 有一个简洁的描述：

> HKDF 是一种基于 HMAC 的密钥派生功能，可将任何弱密钥内容转换为强加密密钥内容。例如，它可以用于将 Diffie Hellman 交换的共享秘密转换为适用于加密，完整性检查或认证的密钥内容。
>
> -- [维基百科](https://en.wikipedia.org/wiki/HKDF)

从本质上讲，HKDF 将不是特别安全的输入变得更安全。

定义此加密的规范要求使用 SHA-256 作为我们的哈希算法，并且 Web 推送中 HKDF 的结果密钥不应超过256位（32字节）。

在 node 中，可以像这样实现：

    // 简化的 HKDF，返回32个字节长度的密钥
    function hkdf(salt, ikm, info, length) {
      // 提取
      const keyHmac = crypto.createHmac('sha256', salt);
      keyHmac.update(ikm);
      const key = keyHmac.digest();

      // 扩展
      const infoHmac = crypto.createHmac('sha256', key);
      infoHmac.update(info);

      // 一个只有 0x01 的一个字节长的缓冲区
      const ONE_BUFFER = new Buffer(1).fill(1);
      infoHmac.update(ONE_BUFFER);

      return infoHmac.digest().slice(0, length);
    }

对于此示例代码，请参阅 [Mat Scale 的文章](https://developers.google.com/web/updates/2016/03/web-push-encryption)。

这一部分涵盖了 [ECDH](https://en.wikipedia.org/wiki/Elliptic_curve_Diffie%E2%80%93Hellman) 和 [HKDF](https://tools.ietf.org/html/rfc5869)。

ECDH 是一种共享公钥并生成共享密钥的安全方式，HKDF是一种采用不安全的来源并使其安全的方法。

这些将在加密我们的有效负载期间使用，接下来让我们看看我们采取什么作为输入以及如何加密。

### 输入（Inputs）

当我们想要通过有效负载向用户发送推送消息时，我们需要三个输入：

1. 有效负载自身。
2. 来自 `PushSubscription` 的 `auth` secret。
3. 来自 `PushSubscription` 的 `p256dh` 密钥。

我们已经看到 `auth` 和 `p256dh` 值是从 `PushSubscription` 中返回的，但是再次提醒，给定一个订阅，我们将从中获得这些值：

    subscription.joJSON().keys.auth
    subscription.joJSON().keys.p256dh

    subscription.getKey('auth')
    subscription.getKey('p256dh')

`auth` 值应视为机密，不在应用程序外部共享。

`p256dh` 密钥是公钥，有时也称为客户端公钥。这里我们将`p256dh`称为订阅公钥。订阅公钥由浏览器生成，浏览器将保密私钥并将其用于解密有效负载。

需要 `auth`，`p256dh` 和 `payload` 这三个值作为输入进行加密，其结果就是有效负载，而 salt 和公钥仅用于加密数据。

**盐（Salt）**

Salt 需要16字节的随机数据，在 NodeJS 中，我们将执行以下操作来创建 salt：

    const salt = crypto.randomBytes(16);

**公/私钥**

公钥和私钥应该使用 P-256 椭圆曲线生成，我们在 Node 中这样做：

    const localKeysCurve = crypto.createECDH('prime256v1');
    localKeysCurve.generateKeys();

    const localPublicKey = localKeysCurve.getPublicKey();
    const localPrivateKey = localKeysCurve.getPrivateKey();

我们将这些密钥称为“本地密钥”，它们*仅*用于加密，与应用程序服务器密钥*无关*。

使用有效负载，auth secret 和订阅公钥作为输入以及新生成的 salt 和本地密钥，我们准备好了来做一些真正的加密。。

### 共享的 secret

第一步是使用订阅公钥和我们的新私钥创建共享密钥（还记得 ECDH 中关于 Alice 和 Bob 的解释吗？就像那样）。

    const sharedSecret = localKeysCurve.computeSecret(
      subscription.keys.p256dh, 'base64');

这用于下一步计算伪随机密钥（PRK）。

### 伪随机密钥

伪随机密钥（PRK）是推送订阅的 auth secret 和我们刚刚创建的共享秘密的组合。

    const authEncBuff = new Buffer('Content-Encoding: auth\0', 'utf8');
    const prk = hkdf(subscription.keys.auth, sharedSecret, authEncBuff, 32);

你可能想知道 `Content-Encoding: auth\0` 的用途。简而言之，它没有明确的目的，即便如此浏览器可以解密传入的消息并寻找预期的内容编码。`\0`是在缓冲区的末尾添加一个值为0的字节，这是浏览器在解密消息时所期盼的，在内容编码的许多字节之后跟着一个值0，再跟着的是加密的数据。

我们的伪随机密钥只是通过 HKDF 运行 auth，shared secret 和一段编码信息（即使其加密更强）。

### Context

“Context”是一组字节，用于稍后在加密浏览器中计算两个值，它本质上是一个包含订阅公钥和本地公钥的字节数组。

    const keyLabel = new Buffer('P-256\0', 'utf8');

    // 将订阅公钥转换为 buffer
    const subscriptionPubKey = new Buffer(subscription.keys.p256dh, 'base64');

    const subscriptionPubKeyLength = new Uint8Array(2);
    subscriptionPubKeyLength[0] = 0;
    subscriptionPubKeyLength[1] = subscriptionPubKey.length;

    const localPublicKeyLength = new Uint8Array(2);
    subscriptionPubKeyLength[0] = 0;
    subscriptionPubKeyLength[1] = localPublicKey.length;

    const contextBuffer = Buffer.concat([
      keyLabel,
      subscriptionPubKeyLength.buffer,
      subscriptionPubKey,
      localPublicKeyLength.buffer,
      localPublicKey,
    ]);

最终的 context buffer 是一个数组，包括订阅公钥中的字节数及密钥本身，以及本地公钥的字节数和密钥本身。

我们可以使用 context 值来创建随机数和内容加密密钥 (CEK)。

### 内容加密密钥和 nonce

[Nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) 是一个可以防止重放攻击的值，因为它只能使用一次。

内容加密密钥（CEK）是最终用于加密我们的有效负载的密钥。

首先，我们需要为 nonce 和 CEK 创建数据字节，这只是一个内容编码字符串，后跟我们刚刚计算的 context buffer：

    const nonceEncBuffer = new Buffer('Content-Encoding: nonce\0', 'utf8');
    const nonceInfo = Buffer.concat([nonceEncBuffer, contextBuffer]);

    const cekEncBuffer = new Buffer('Content-Encoding: aesgcm\0');
    const cekInfo = Buffer.concat([cekEncBuffer, contextBuffer]);

此信息通过 HKDF 将 salt 和 PRK 与 nonceInfo 和 cekInfo 结合使用：

    // Nonce 应该是12个字节长
    const nonce = hkdf(salt, prk, nonceInfo, 12);

    // CEK 应该是16个字节长
    const contentEncryptionKey = hkdf(salt, prk, cekInfo, 16);

这为我们提供了 nonce 和 content 加密密钥。

### 执行加密

现在我们有了内容加密密钥，我们可以加密有效负载了。

我们使用内容加密密钥作为密钥创建了 AES128 密码，nonce 是其初始化向量。

在 Node 中这样完成：

    const cipher = crypto.createCipheriv(
      'id-aes128-GCM', contentEncryptionKey, nonce);

在我们加密有效负载之前，我们需要定义我们希望添加到有效负载的填充量，之所以添加填充，是以为它可以防止窃听者根据有效负载的大小来确定消息“类型”。

必须添加两个填充字节以指示任何额外的填充长度。

例如，假如你没有添加填充，你也得有两个字节值为0，即没有填充存在，在这两个字节后将读取有效负载。 如果添加了5个字节的填充，前两个字节的值为5，那么消费者将再读取5个字节，然后再开始读取有效负载。

    const padding = new Buffer(2 + paddingLength);
    // 除长度外，Buffer 必须为0
    padding.fill(0);
    padding.writeUInt16BE(paddingLength, 0);

然后我们通过这个密码运行填充和有效负载。

    const result = cipher.update(Buffer.concat(padding, payload));
    cipher.final();

    // 添加 auth tag 到 result 的后面 -
    // https://nodejs.org/api/crypto.html#crypto_cipher_getauthtag
    const encryptedPayload = Buffer.concat([result, cipher.getAuthTag()]);

我们现在有加密的有效负载了，️耶！

剩下的就是确定如何将此有效负载发送到推送服务。

### 加密有效负载的 headers 和 body

要将此加密的有效负载发送到推送服务，我们需要在 POST 请求中定义几个不同的 header。

#### Encryption header

Encryption header 必须包含用于加密有效负载的 salt。

Salt 应该是16字节的，并且经过 base64 URL 安全编码后添加到 Encryption header 中，如下所示：

    Encryption: salt=<URL Safe Base64 Encoded Salt>

#### Crypto-Key header

在“应用程序服务器密钥”章节中，我们已经知道如何使用 `Crypto-Key` header 来包含公共应用程序服务器密钥。

此 header 还用作共享用于加密有效负载的本地公钥。

header 的结果如下所示：

    Crypto-Key: dh=<URL Safe Base64 Encoded Local Public Key String>; p256ecdsa=<URL Safe Base64
    Encoded Public Application Server Key>

#### Content type, length & encoding headers

`Content-Length` header 是加密有效负载中的字节数。 “Content-Type”和“Content-Encoding”标头是固定值，如下所示。

    Content-Length: <Number of Bytes in Encrypted Payload>
    Content-Type: 'application/octet-stream'
    Content-Encoding: 'aesgcm'

设置这些 header 后，我们需要将加密的有效负载作为请求的 body 发送。 请注意将 `Content-Type` 设置为`application/octet-stream`，这是因为加密的有效负载必须作为字节流发送。

在 NodeJS 中我们会这样做：

    const pushRequest = https.request(httpsOptions, function(pushResponse) {
    pushRequest.write(encryptedPayload);
    pushRequest.end();

## 更多的 header？

我们已经介绍了用于 JWT / 应用程序服务器密钥的 headers（即如何使用推送服务认证应用程序），也介绍了用于发送加密有效负载的 headers。

推送服务还有其他的 headers 来用于改变发送消息的行为。其中一些 headers 是必需的，一些是可选的。

### TTL header

**必选**

“TTL”（time to live，生存时间）是一个整数，指定推送消息在推送服务发布之前存活的时间。当 “TTL” 到期时，该消息将从推送服务队列中删除，并且不会被传递。

    TTL: <Time to live in seconds>

如果将 “TTL” 设置为 0，则推送服务将尝试立即传递消息，**但是**如果无法访问设备，则会立即从推送服务队列中删除该消息。

从技术上讲，推送服务可以在需要时减少推送消息的 “TTL”。你可以通过检查推送服务响应中的“TTL”标头来判断是否发生了这种情况。

##### 主题（Topic）

**可选**

当一条的消息与待处理的消息有相同的主题（字符串形式）时，新的消息就会替换旧的。

这在设备离线时发送多条消息的情况下非常有用，用户在设备打开时只会收到最新消息。

##### 紧急度（Urgency）

**可选**

紧急度向推送服务说明消息对用户的重要性。推送服务可以使用此字段来帮助节省用户设备的电量，当电池电量低的时候，只有当重要消息来临的时候才进行唤醒。

Header 的值定义如下。 默认值是 `normal`.

    Urgency: <very-low | low | normal | high>

### 将一切整合在一起

如果你对这一切的工作方式有进一步的疑问，可以随时在 [web-push-libs](https://github.com/web-push-libs) 上查看一些库如何触发推送消息。

一旦你有了加密的有效负载和上面提到的 header，你只需要在 `PushSubscription` 中向 `endpoint` 发出POST请求。

那么我们如何处理 POST 请求的响应呢？

### 推送服务的响应

一旦向推送服务发出请求，你需要检查响应的状态码，它会告诉请求是否成功。

<table>
  <tr>
    <th>状态码</th>
    <th>描述</th>
  </tr>
  <tr>
    <td>201</td>
    <td>创建。收到并接受发送推送消息的请求。
    </td>
  </tr>
  <tr>
    <td>429</td>
    <td>请求过多。意味着应用程序服务器已经达到了推送服务的速率限制。推送服务会包括 “Retry-After” 标头，来指示在下一个请求发出之前等多长时间。</td>
  </tr>
  <tr>
    <td>400</td>
    <td>无效的请求。这通常意味着存在无效的 header 或格式不正确。</td>
  </tr>
  <tr>
    <td>404</td>
    <td>未找到。这表示订阅已过期且无法使用。在这种情况下，你应该删除 `PushSubscription` 并等待客户端重新订阅用户。</td>
  </tr>
  <tr>
    <td>410</td>
    <td>被移除。订阅不再有效，应从应用程序服务器中删除。可以通过在 `PushSubscription` 上调用 `unsubscribe()` 来重现。</td>
  </tr>
  <tr>
    <td>413</td>
    <td>有效负载过大。一个推送服务支持的最小的有效负载大小是 <a
href="https://tools.ietf.org/html/draft-ietf-webpush-protocol-10#section-7.2">4096 bytes</a>
（或者 4kb）。</td>
  </tr>
</table>
