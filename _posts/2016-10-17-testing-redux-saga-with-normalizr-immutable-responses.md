---
layout: post
title: Testing redux-saga With normalizr-immutable Responses
---

I've recently starting using [redux-saga][1] to handle async API calls for [tessera-client][2]. tessera-client uses [immutablejs][3] data structures for the store and [normalizr-immutable][4] to handle API response normalization and converting it into immutable data structures.

I ran into an issue when attempting to test the first saga I created. Here's the code for that saga:

``` javascript
// src/sagas.js
export function* fetchTickets(action) {
  try {
    const response = yield call(api.fetchTickets, {})
    yield put(actions.fetchTicketsSuccess(response))
  } catch (e) {
    yield put(actions.fetchTicketsFailure(e))
  }
}
```

And here's the test I had originally written:

``` javascript
// test/sagas_spec.js
describe('GET: Fetch Tickets', () => {
  it('fetches tickets', () => {
    const generator = sagas.fetchTickets()

    expect(generator.next().value).to.deep.eq(call(api.fetchTickets, {}))

    // fake response
    const tickets = []

    expect(generator.next(tickets).value).to.deep.eq(put(actions.fetchTicketsSuccess(tickets)))
  })
})
```

The problem I ran into was that I couldn't get mocha to realize that the values of the final generator call and the `put` action were equal. I inspected both structures and found that they were equal, and had this stucture:

``` javascript
{ '@@redux-saga/IO': true,
  PUT:
    { channel: null,
      action: { 
        type: 'TICKETS/FETCH_SUCCESS',
        response: Record { "entities": Record {}, "result": List []  
      } 
    } 
  } 
}
```

The test was unable to determine that both structures were equal because the action itself is a js object, but the action response is an immutable data structure.

I ended up using the following workaround to test that they were equal:

``` javascript
const next = generator.next(tickets).value
const expected = put(actions.fetchTicketsSuccess(tickets))

expect(next.PUT.action.type).to.equal(expected.PUT.action.type)
expect(next.PUT.action.response).to.equal(expected.PUT.action.response)
```

This tests that the types of both actions are equal and that the responses of both actions are equal as well.

[1]: https://github.com/yelouafi/redux-saga "redux-saga"
[2]: https://github.com/lionize/tessera-client "tessera-client"
[3]: https://github.com/facebook/immutable-js "immutable-js"
[4]: https://github.com/mschipperheyn/normalizr-immutable "normalizr-immutable"
