# Changelog

## 0.7.1 (20/04/2016)

Add `leeway` support in `JwtOptions`

## 0.7.0 (17/03/2016)

Support for Circe 0.3.0

## 0.6.0 (09/03/2016)

Support for Play Framework 2.5.0

## 0.5.1 (05/03/2016)

Fix bug not-escaping quotation mark `"` when stringifying JSON.

## 0.5.0 (31/12/2015)

### Circe support

Thanks to @dwhitney , `JWT Scala` now has support for [Circe](https://github.com/travisbrown/circe). Check out [samples](http://pauldijou.fr/jwt-scala/samples/jwt-circe/) and [Scaladoc](http://pauldijou.fr/jwt-scala/api/latest/jwt-circe/).

### Disable validation

When decoding, `JWT Scala` also performs validation. If you need to decode an invalid token, you can now use a `JwtOptions` as the last argument of any decoding function to disable validation checks like expiration, notBefore and signature. Read the **Options** section of the [core sample](http://pauldijou.fr/jwt-scala/samples/jwt-core/) to know more.

### Fix null session in Play 2.4

Since 2.4, Play assign `null` as default value for some configuration keys which throw a `ConfigException.Null` in TypeSafe config lib. This should be fixed with the new configuration system at some point in the future. In the mean time, all calls reading the configuration will be wrapped in a try/catch to prevent that.

## 0.4.1 (30/09/2015)

Fix tricky bug inside all JSON libs not supporting correctly the `none` algorithm.

## 0.4.0 (24/07/2015)

Thanks a lot to @drbild for helping review the code around security vulnerabilities.

### Now on Maven

All the sub-projects are now released directly on Maven Central. Since Sonatype didn't accept `pdi` as the groupId, I had to change it to `com.pauldijou`. Sorry about that, you will need to quickly update your `build.sbt` (or whatever file contains your dependencies).

### Breaking changes

**Good news** Those changes don't impact the `jwt-play` lib, only low level APIs.

All decoding and validating methods with a `key: String` are now removed for security reasons. Please use their counterpart which now needs a 3rd argument corresponding to the list of algorithms that the token can be signed with. This list cannot mix HMAC and asymetric algorithms (like RSA or ECDSA). This is to prevent a server using RSA with a String key to receive a forged token signed with a HMAC algorithm and the RSA public key to be accepted using the same RSA public key as the HMAC secret key by default. You can learn more by reading [this article](https://www.timmclean.net/2015/03/31/jwt-algorithm-confusion.html).

```scala
// Before
val claim = Jwt.decode(token, key)

// After (knowing that you only expect a HMAC 256)
val claim = Jwt.decode(token, key, Seq(JwtAlgorithm.HS256))
// After (supporting all HMAC algorithms)
val claim = Jwt.decode(token, key, JwtAlgorithm.allHmac)
```

If you are using `SecretKey` or `PublicKey`, the list of algorithms is optional and will be automatically computed (using `JwtAlgorithm.allHmac` and `JwtAlgorithm.allAsymetric` respesctively) but feel free to provide you own list if you want to restrict the possible algorithms. More security never killed any web application.

Why not deprecate them? I considered doing that but I decided to enforce the security fix. I'm pretty sure that most people only use one HMAC algorithm with a String key and it will force them to edit their code but it should be a minor edit since you usually only decode tokens once or twice inside a code base. The fact that the project is still very new and at a `0.x` version played in the decision.

### Fixes

Fix a security vulnerability around timing attacks.

### Features

Add implicit class to convert `JwtHeader` and `JwtClaim` to `JsValue` or `JValue`. See [examples for Play JSON](http://pauldijou.fr/jwt-scala/samples/jwt-play-json/) or [examples for Json4s](http://pauldijou.fr/jwt-scala/samples/jwt-json4s/).

```scala
// Play JSON
JwtHeader(JwtAlgorithm.HS256).toJsValue
JwtClaim().by("me").to("you").about("something").issuedNow.startsNow.expiresIn(15).toJsValue

// Json4s
JwtHeader(JwtAlgorithm.HS256).toJValue
JwtClaim().by("me").to("you").about("something").issuedNow.startsNow.expiresIn(15).toJValue
```

## 0.2.1 (24/07/2015)

Same as `0.4.0` but targeting Play 2.3

## 0.3.0 (08/06/2015)

### Breaking changes

- move exceptions to their own package
- move algorithms to their own package

### Features

- support Play 2.4.0

## 0.2.0 (02/06/2015)

### Breaking changes

- removed all `Option` from API. Now, it's either nothing or a valid key. It shouldn't have a big impact since the majority of users were using valid keys already.
- when decoding a token to a `Tuple3`, the last part representing the signature is now a `String` rather than an `Option[String]`.

### New features

- full support for `SecretKey` for HMAC algorithms
- full support for `PrivateKey` and `PublicKey` for RSA and ECDSA algorithms
- Nearly all API now have 4 possible signatures (note: `JwtAsymetricAlgorithm` is either a RSA or a ECDSA algorithm)
  - `method(...)`
  - `method(..., key: String, algorithm: JwtAlgorithm)`
  - `method(..., key: SecretKey, algorithm: JwtHmacAlgorithm)`
  - `method(..., key: PrivateKey/PublicKey, algorithm: JwtAsymetricAlgorithm)`

Use `PrivateKey` when encoding and `PublicKey` when decoding or verifying.

### Bug fixes

- Some ECDSA algorithms were extending the wrong super-type
- `{"algo":"none"}` header was incorrectly supported

## 0.1.0 (18/05/2015)

No code change from 0.0.6, just more doc and tests.

## 0.0.6 (14/05/2015)

Add support for Json4s (both Native and Jackson implementations)

## 0.0.5 (13/05/2015)

We should be API ready. Just need more tests and scaladoc before production ready.
