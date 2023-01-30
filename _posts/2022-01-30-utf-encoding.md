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
const decoded = Buffer.from(body, 'utf16le').toString('utf8');
```

### UTF Byte Order Marks

But if you try to decode a UTF-8 buffer as UTF-16 it won't work, so we need to check the encoding before decoding it. We can do this by checking the Byte Order Mark (BOM) (we currently only care about UTF-8 or 16!).

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
async function detectEncodingFromBom(
  url: string
): Promise<'utf8' | 'utf16le' | 'utf16be' | undefined> {
  const res = await fetch(url);
  const body = await res.body.getReader().read();
  if (body.value.length >= 3) {
    const hexString = firstThreeHexChars(body.value);
    if (hexString === 'efbbbf') {
      return 'utf8';
    } else if (hexString.startsWith('fffe')) {
      return 'utf16le';
    } else if (hexString.startsWith('feff')) {
      return 'utf16be';
    }
  }
}
```

### Decode the body using the correct encoding

And finally decode the body using the correct encoding:

```ts
async function decodeBody(url: string) {
  const encoding = await detectEncodingFromBom(url);
  const res = await fetch(url);
  const body = await res.text();
  return Buffer.from(body, encoding).toString('utf8');
}
```
