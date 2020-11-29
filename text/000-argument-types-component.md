- Start Date: 2020-11-28
- Relevant Team(s): Ember.js
- RFC PR:
- Tracking:

# Custom arg types for Glimmer Component arguments

## Summary

This RFC proposes added a low level, debug only arg types checking argument to [`precompileTemplate`](https://github.com/glimmerjs/glimmer.js/blob/master/packages/babel-plugins/@glimmer/babel-plugin-strict-template-precompile/index.js).

## Motivation

Type checking can help catch a variety of bugs. Ember users can write TypeScript for improved autocomplete, refactoring and bug catching when defining Components or constructing your logic.

Components are the primary unit of composition for our applications. We often create Components and wire them together with data. However, imagine the troubles you may bring yourself if you pass a string to a Component that is expecting a number!  This RFC proposes to alleviate these class of issues. Built in arg types checking can help reduce bugs when building dependent trees of Components or for library authors to solidify and provide important information to users about their Components.

## Detailed design

A simple API addition to `precompileTemplate` should suit as a low level primitive for type checking properties passed to Glimmer Components.

```js
import { precompileTemplate } from '@ember/template-compilation';

precompileTemplate(
  'Hello, {{@name}}!',
  [MyComponent],
  {
    argTypes: {
        name: (value) => typeof value === 'string',
        age: (value) => typeof value === 'number',
        confirmed: (value) => typeof value === 'boolean',
    }
  }
});
```

If you fail any of the arg type checks, you will receive runtime warnings in DEBUG mode.  Production builds will not utilize and run your arg type checks, thus avoiding any negative performance implications that might arise.

## How we teach this

Ample documentation of this new API should suffice in helping developers understand and implement arg type checking.

## Alternatives

- Do not add arg types API to components.

## Open Questions

## Related links and RFCs

-
