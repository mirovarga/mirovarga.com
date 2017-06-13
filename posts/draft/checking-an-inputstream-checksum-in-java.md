# Checking an InputStream Checksum in Java

*Jun 18, 2015*

*Java*

This week I was coding some Java and needed a way to check an InputStream's
checksum. In addition to that, if the checksum was OK I then wanted to read from
the stream.

The solution didn't first look obvious to me, but after some googling I was able
to put together a small utility class.

> I created a [Maven project](https://github.com/mirovarga/checksum) on GitHub
so it's easier to use.

```java
package com.mirovarga.util.checksum;

import java.io.InputStream;
import java.security.MessageDigest;

import org.apache.commons.codec.binary.Hex;
import org.apache.commons.codec.digest.DigestUtils;

/**
 * A utility class for checking an InputStream's checksum.
 */
public final class Checksum {
	private InputStream stream;
	private String algorithm;
	private String hash;

	/* You don't need to instantiate this class. Use the static check method. */
	private Checksum(InputStream stream, String algorithm, String hash) {
		this.stream = stream;
		this.algorithm = algorithm;
		this.hash = hash;
		validateArguments();
	}

	/**
	 * Calculates an InputStream's checksum using the provided algorithm and
	 * compares it with the provided hash.
	 *
	 * It tries to reset the stream after the check but not all streams support
	 * marking/resetting. If you cannot read from the stream after the check
	 * wrap it in a BufferedInputStream and use that instead of the stream itself
	 * (both as the argument to this method and as the stream you read later from).
	 *
	 * @param stream the InputStream whose checksum to check
	 * @param algorithm the algorithm to use for calculating the checksum
	 * @param hash the hash to compare the calculated checksum with
	 *
	 * @returns true if the calculated checksum and the provided hash are equal,
	 *   false otherwise
	 */
	public static boolean check(InputStream stream, String algorithm, String hash) {
		return new Checksum(stream, algorithm, hash).check();
	}

	public boolean check() {
		try {
			if (stream.markSupported())
				stream.mark(Integer.MAX_VALUE);
			return calculateChecksum().equalsIgnoreCase(hash);
		} catch (Exception e) {
			return false;
		} finally {
			try {
				stream.reset();
			} catch (Exception e) {
			}
		}
	}

	private void validateArguments() {
		if (stream == null)
			throw new IllegalArgumentException("stream is required");
		if (algorithm == null || "".equals(algorithm))
			throw new IllegalArgumentException("algorithm is required");
		if (hash == null || "".equals(hash))
			throw new IllegalArgumentException("hash is required");
	}

	private String calculateChecksum() throws Exception {
		MessageDigest digest = DigestUtils.getDigest(algorithm);
		DigestUtils.updateDigest(digest, stream);
		return Hex.encodeHexString(digest.digest());
	}
}
```

## Usage

To check the checksum of an InputStream, use it like that:

```java
InputStream stream = ...;
boolean ok = Checksum.check(stream, "MD5", "8D777F385D3DFEC8815D20F7496026DC");

```

The method tries to reset the stream so you can read from it later. But because
not all streams support resetting it cannot reset all streams. In that case use
it like that:

```java
InputStream stream = ...;

// Wrap the original stream to be sure resetting is supported
InputStream wrap = new BufferedInputStream(stream);
boolean ok = Checksum.check(wrap, "MD5", "8D777F385D3DFEC8815D20F7496026DC");

// Read from the wrapped stream
int byte = wrap.read();
```
