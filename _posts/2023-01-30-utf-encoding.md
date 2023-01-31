---
tags:
  - typescript
  - text
  - encoding
---

## UTF Encoding

Text in software is complicated. We were struggling with an HTTP response body which didn't include a charset header - and appeared garbled when requested using Postman.

The answer was that it was UTF-16 encoded, so to decode this we needed to use:

```js
const decoded = Buffer.from(body, 'utf-16le').toString('utf-8');
```

### UTF Byte Order Marks

But if you try to decode a UTF-8 buffer as UTF-16 it won't work, so we need to check the encoding before decoding it. We can do this by checking the [Byte Order Mark (BOM)](https://en.wikipedia.org/wiki/Byte_order_mark) (we currently only care about UTF-8 or 16!).

| Encoding    | BOM      |
| ----------- | -------- |
| UTF-8       | EF BB BF |
| UTF-16 (BE) | FE FF    |
| UTF-16 (LE) | FF FE    |

So we needed to get the first 2-3 hex characters from the response body, and then check if they matched any of the BOMs.

### Get first three hex characters (the BOM) from byte array

```ts
function firstThreeHexChars(byteArray: Uint8Array) {
  const uint8Array = byteArray.slice(0, 3);
  const hexString = Array.prototype.map
    .call(uint8Array, (x) => ('00' + x.toString(16)).slice(-2))
    .join('');
  return hexString;
}
```

### Check the BOM

Now we can check the encoding using those characters:

```ts
function detectEncodingFromBom(
  byteArray: Uint8Array
): 'utf-8' | 'utf-16le' | 'utf-16be' | undefined {
  if (byteArray.length >= 3) {
    const hexString = firstThreeHexChars(byteArray);
    if (hexString === 'efbbbf') {
      return 'utf-8';
    } else if (hexString.startsWith('fffe')) {
      return 'utf-16le';
    } else if (hexString.startsWith('feff')) {
      return 'utf-16be';
    }
  }
}
```

### Decode the body using the correct encoding

So to get a byte array from a url, detect its encoding, and then decode it:

```ts
async function getBodyTextWithDetectedEncoding(url: string) {
  const res = await fetch(url);
  const byteArray = await res.arrayBuffer();
  const encoding = detectEncodingFromBom(new Uint8Array(byteArray));

  const dec = new TextDecoder(encoding);
  const text = dec.decode(byteArray);
  return text;
}
```
