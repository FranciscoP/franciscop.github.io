---
layout: post.hbs
title: Async/Await are awesome
description: How these ES7 features can improve and simplify asynchronous Javascript greatly.
date: 2017-04-06T00:00+09:00
---

It [just became apparent to me](https://twitter.com/FPresencia/status/844859374778535936) that an async function that is called returns a Promise and that is really great. The reason is that there are few libraries that work great with it, including [server for Node.js](https://serverjs.io/) and Jest.

> Good news: from Node.js 7.6 you have native async/await support!



## Server middleware

By building [Comments Network](https://comments.network/) I learned a couple of tricks. If we were to do it naively we could implement it like this with server:

```js
// Promise-based requests
server(get('/story/:id', ctx => {
  request(`API CALL${ctx.req.params.id}`).then(api => {
    request(`HN${api.url}${ctx.req.params.id}`).then(hn => {
      // Do some work with `api` and `hn` here
      ctx.res.send(workedout);
    });
  });
}));
```

```js
// Async/Await based requests
server(get('/story/:id', async ctx => {
  const api = await request(`API CALL${ctx.req.params.id}`);
  const hn = await request(`HN${api.url}${ctx.req.params.id}`);
  // Do some work with `api` and `hn` here
  return workedout;
}));
```

You can use it now and forget about messy Promises and Callback Hell. Of course you will have to use libraries that are based on Promises as well, since some of the popular Node.js libraries such as `fs` and `request` are callback-based (check out `fs-promise` and `request-promises`).



## Testing

Now let’s see about Jest. By default, Jest and other testing frameworks accept two ways of doing asynchronous tests. Through a function that accepts a `done` parameter or through a function that returns a Promise. But thanks to `async` the third option is a no-brainer ([see it live](https://github.com/franciscop/drive-db)):

```js
// Callback based
it('can be loaded', done => {
  drive.load().then(db => {
    expect(db.data).toEqual(jasmine.any(Array));
    expect(db.data.length).toBeGreaterThan(0);
    done();
  }).catch(done);
});

// Promise based
it('[promise] can be loaded', () => {
  return drive.load().then(db => {
    expect(db.data).toEqual(jasmine.any(Array));
    expect(db.data.length).toBeGreaterThan(0);
  });
});

// Async/Await based
it('[async] can be loaded', async () => {
  const db = await drive.load();
  expect(db.data).toEqual(jasmine.any(Array));
  expect(db.data.length).toBeGreaterThan(0);
});
```

That’s it. You don’t even need a return as functions return when finished so they will resolve then as expected.

**HOWEVER** not everything is as great as it looks. Testing for a case where an error is thrown is quite more difficult with `async`:

```js
// Callback based
it('can be loaded', done => {
  drive.load().catch(err => done());
});

// Promise based
it('[promise] can be loaded', () => {
  let passed = false;
  return drive.load().then(db => {
    passed = true;
  }).catch().then(() => {
    if (passed) throw new Error('Did not throw an error');
  });
});

// Async/Await based
it('[async] can be loaded', async () => {
  try {
    const db = await drive.load();
  } catch (err) {
    return;
  }
  throw new Error('Did not throw an error');
});
```

But the good news is that with a bit of code you can fix this:

```js
throws = cb => async () => {
  try {
    const res = await cb();
  } catch(err) {
    return;
  }
  throw new Error('No error was thrown');
};
```

Then you can use it like this with async/await; back to the good bits:

```js
it('[async] can be loaded', async () => {
  await throws(() => drive.load());
});
```

The future for Javascript is looking quite good even though there’s a long, fragmented road to get there. I’m building [server for Node.js](https://serverjs.io/) to get there faster, [**subscribe**](https://francisco.us9.list-manage.com/subscribe?u=8f634b4eeb27e1bb51438f11f&id=7b067d6820) or [**follow me**](https://twitter.com/fpresencia) to get updates!

[*Originally posted on Medium*](https://medium.com/server-for-node-js/async-await-are-awesome-c0834cc09ab)
