- Start Date: 2019-09-28
- Relevant Team(s): Ember Data
- RFC PR:
- Tracking:

# ember-data | pushPayload

## Summary

Deprecate `store.pushPayload` and `store.normalize`.

## Motivation

As detailed in this ember-data [issue](https://github.com/emberjs/rfcs/issues/357), `store.pushPayload` is a convenience wrapper for `store.push` that hides looking up the model class and your serializer logic to deserialize a non `json-api` payload.  In order to facilitate simplifying `ember-data`, moving the community to implement `store.push` with a composable, functional pattern to normalize an undocumented payload to a JSON API compliant document will help users avoid a some of pitfalls currently present with `pushPayload`.

// Also, future versions of `ember-data` may not require serializers in their current form, so separating push logic from .

## Detailed design
`store.pushPayload` is a convenience wrapper for `store.push`.  Developers can use `pushPayload` method to deserialize payloads if a corresponding Serializer also implements a `pushPayload` method.  As with other contracts in the EmberJS ecosystem, convenience metthods to provide ease in implementation also have costs.  First, lets make sure we understand what both of these methods do:

#### store.push
A user can call `store.push` with an existing `json-api` compliant [payload](https://jsonapi.org) to notify Ember Data's store of new or updated records that exist on the backend.  These may be full or partial records.  Generally, a non compliant `json-api` response can be normalized and returned in [`normalizeResponse`](https://api.emberjs.com/ember-data/release/classes/JSONAPISerializer/methods/normalizeResponse?anchor=normalizeResponse).  This will create or update existing records.  However, many use cases present themselves to use `store.push` directly, including ad-hoc requests or web socket channels.  A response from fastboot can be grabbed from the shoebox to hydrate the store like so.

```js
initialize(appInstance) {
  ...
  let store = appInstance.lookup('service:store');
  let payload = shoebox.retrieve(config.shoeboxNames.data);
  if (store && payload) {
    store.push(payload);
  }
}
```

#### store.pushPayload
`pushPayload` can also be used to push a `json-api` or non `json-api` compliant response into the store. Here is what approximately `ember-data` uses to deserialize data into a `json-api` compliant format before pushing into the store.

```js
function pushPayload(store, modelName, rawPayload) {
  const ModelClass = store.modelFor(modelName);
  const serializer = store.serializerFor(modelName);
  const jsonApiPayload = serializer.normalizeResponse(store, ModelClass, rawPayload, null, 'query');

  return store.push(jsonApiPayload);
}
```

As you can see, this necessarily requires a serializer with a `normalizeResponse` hook where your intimate knowledge of the undocumented format will be converted into a compliant format.  It should be said that there is no scenario where `ember-data` can infer how to deserialize your data. As a result, although `pushPayload` may be useful for some users, it is also easy and more clear for users to implement their own normalization process.

Effectively, `ember-data` is providing an abstraction that leads to a few problems:

1.
2.

### Implementation

We will first issue deprecation warnings on `store.pushPayload` and `serializer.pushPayload`. This will include a reference link to an example on how to implement.

// Also https://gist.github.com/runspired/96618af26fb1c687a74eb30bf15e58b6/

## How we teach this

An example utility method will be provided in the documentation.  By advocating for a composable, functional approach, users will have a clear functional approach to normalizing their non standard response payload.

## Drawbacks

- Users who currently implement `store.pushPayload` or `serializer.pushPayload` will have to migrate their codebases
  to use `store.push`.
- A variety of use cases may not be currently covered by `ember-data` with a functional/composable pattern including singularizing, dasherizing and camelizing types.  However, future versions of `ember-data` will likely eliminate the need to singularize or dasherize types.

## Alternatives

## Unresolved questions

None
