- Start Date: 2019-09-28
- Relevant Team(s): Ember Data
- RFC PR:
- Tracking:

# EmberData | store.pushPayload and store.normalize deprecation

## Summary

Deprecate `store.pushPayload` and `store.normalize`.

## Motivation

The EmberData team is working to encapsulate features specific to `Network` separate from the `Store` (and other concerns such as `Presentation` and `Caching`).  This encapsulation allows for improved modularity and future improvements of the APIs within `Network`.

The `store.pushPayload` and `store.normalize` methods force the `store` to be aware of the specifics of serializers and normalization, breaking the desired pattern of modularity and encapsulated concerns.

These methods are both redundant (as 1:1 patterns for using public APIs exist for achieving the same behavior) and a common source of confusion and errors due to obscuring the format used by the cache and the indirection of the APIs they involve.

In addition to better respecting separation of concerns, the public API alternatives to these APIs avoid introducing many of the bugs and confusion these APIs have by keeping responsibilities clear and preserving context.

## Normalization

**deprecate `store.normalize`**

`store.normalize` converts a single json payload into a normalized form before passing the data to `store.push`.

It's implementation is approximately implemented as follows.

```js
function normalize(modelName, rawPayload) {
  const serializer = this.serializerFor(modelName);
  const model = this.modelFor(modelName);
  return serializer.normalize(model, rawPayload);
}
```

The first issue with this method is it requires knowledge of `Network` and `Presentation` concerns breaking encapsulation and modularity. // TODO bugs and maintenance

**Alternatives**

The same functionality provided by `store.normalize` can be impelemented a few ways.

- `store.serializerFor(...).normalizeResponse(...)`
// TODO
- functional normalization
// TODO

### Pushing Data

**deprecate `store.pushPayload()`**

`store.pushPayload` can be used to normalize a `json-api` or non `json-api` compliant response and subsequently pushing the data into the store.

Here is what `EmberData` approximately uses to deserialize data and update the `Cache`.

```js
function pushPayload(store, modelName, rawPayload) {
  const ModelClass = store.modelFor(modelName);
  const serializer = store.serializerFor(modelName);
  const jsonApiPayload = serializer.normalizeResponse(store, ModelClass, rawPayload, null, 'query');

  return store.push(jsonApiPayload);
}
```

Similar to `store.normalize`, this necessarily requires a serializer with a `normalizeResponse` hook where your intimate knowledge of the undocumented format will be converted into a compliant format.  It should be said that there is no scenario where `EmberData` can infer how to deserialize your data.

Effectively, `EmberData` is providing an abstraction that leads to a few problems.

1. As noted above, the `Store` should be unaware of `Network` concerns as `EmberData` moves to modernize it's codebase.  Having clear and directional responsibilities for managing your data is preferred to stitching together obligations of various components of `EmberData`.
2. Although abstractions provide ease of use, they also can reduce understandability on the part of the user.  Public APIs already exist to acheive the same level of functionality.
3. More framework code necessarily increases the amount of bugs.  Simplifying and continuing work to reorganize and modularize the `EmberData` codebase will help reduce future bugs.

**Alternatives**

- normalization followed by `store.push`

### Implementation

We will first issue deprecation warnings on `store.pushPayload` and `store.normalize`. This will include a reference link to an example on how to implement.

// Also https://gist.github.com/runspired/96618af26fb1c687a74eb30bf15e58b6/

## How we teach this

An example utility method using existing public APIs will be provided in the documentation.  By advocating for a composable, functional approach, users will have an easy to understand approach to normalizing their non standard response payload.

## Drawbacks

- Users who currently implement `store.pushPayload` and `store.normalize` will have to migrate their codebases
  to use `store.push`.
- A variety of use cases may not be currently covered by `EmberData` with a functional/composable pattern including singularizing, dasherizing and camelizing types.  However, future versions of `EmberData` will likely eliminate the need to singularize or dasherize types.
- More lines of code

## Alternatives

## Unresolved questions

None
