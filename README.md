okuyama-client-java
===================

Java client library for Okuyama KVS

## Feature
* Basic set and get operation
* Compare And Set value updating
* Tagging
* Value compression ability with customizable compression strategy
* Socket pooling with adaptive expansion
* Adaptive value serialization form. Encoding base64 for serializable object.  

## Usage
For detail, see [Javadoc](http://mitonize.github.io/okuyama-client-java/site/apidocs/index.html)

At first, create OkuyamaClientFactory and specify endpoints and pool size.
OkuyamaClientFactory can be held through application life time.

```java
String[] endpoints = {"127.0.0.1:8888", "127.0.0.1:8889"};
int poolSize = 10;

OkuyamaClientFactory factory = new OkuyamaClientFactoryImpl(endpoints, poolSize);

// Enable compression with default compression strategy, compress if size of value is over 32 bytes
factory.setCompressionMode(true);

// Or set your own compression strategy
factory.setCompressionStrategy(new YourCompressionStrategy());
```

When you use client instance, call factory.createClient(). This instance is supposed to retain short time as in function.

```java
OkuyamaClient client = factory.createClient();
```
## Compatibility 
###
互換モードを指定すると、オリジナルのOkuyamaClientからでも読み出し可能な形式で格納するように下の設定でクライアントを生成する。

* キーを必ずbase64エンコードする
* 文字列を格納する場合にもJavaのシリアライズを行ってからBase64エンコードする
* 圧縮しない（圧縮戦略を設定すると例外を発生させる）

## Compression
### Implementing CompressionStrategy
```java
public class YourCompressionStrategy implements CompressionStrategy {
	private static final int MINIMUM_LENGTH_TO_COMPRESS = 32;

	private Compressor compressorDefault;
	private Compressor compressorFast;

	public YourCompressionStrategy() {
		compressorDefault = Compressor.getCompressor(JdkDeflaterCompressor.COMPRESSOR_ID);
		compressorFast = Compressor.getCompressor(LZFCompressor.COMPRESSOR_ID);
	}

	@Override
	public Compressor getSuitableCompressor(String key, int valueLength) {
		if (valueLength < MINIMUM_LENGTH_TO_COMPRESS) {
			return null;
		}
		if (key.startsWith("session:")) {
			return compressorFast;
		}
		if (key.startsWith("count:")) {
			return null;
		}
		return compressorDefault;
	}
}
```
