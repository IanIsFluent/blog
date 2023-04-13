---
tags:
  - typescript
  - jest
---

# Testing Async Exceptions in Jest

There are two ways to test that our async code throws an exception:

1. expect.rejects
2. try + assert / catch + expect

## expect.rejects

The first, and simpler way lets us check our exception type\* and message - but not any other properties of the error.

```ts
it('should throw error matching message', async () => {
  await expect(sut.doSomething()).rejects.toThrow(new Error('message'));
});
```

We can also move the async function inside the expect like this:

```ts
it('should throw error matching message', () => {
  expect(async () => await sut.doSomething()).rejects.toThrow(
    new Error('message')
  );
});
```

But as most of our tests which run this method will be `async`, the first way will be more consistent.

The problem with this approach is that we can't test any properties on the error apart from the type and message. To test other properties of the error, we need to use the second approach.

\* The type will always pass if we use `new Error('message')` - even if we are actually throwing a custom error type.

## try + assert / catch + expect

The second approach lets us test any properties of the error, but it is more verbose.

```ts
it('should throw error with code 400', async () => {
  try {
    await sut.doSomething();
    assert.fail('should throw');
  } catch (e) {
    expect(e.code).toEqual(400);
  }
});
```

We need to use `assert.fail()` to make the test fail if the expected exception isn't thrown. In the catch block we can then test any properties of the exception are as expected. However, we can not use .toEqual({code: 400}) because the error is not an object.
