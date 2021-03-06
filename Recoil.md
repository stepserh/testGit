### [Atoms](https://recoiljs.org/docs/api-reference/core/atom)

An atom represents state in Recoil. The `atom()` function returns a writeable `RecoilState` object.

Recoil manages atom state changes to know when to notify components subscribing to that atom to re-render,
so you should use the hooks listed below to change atom state. If an object stored in an atom was mutated directly
it may bypass this and cause state changes without properly notifying subscribing components.
To help detect bugs Recoil will freeze objects stored in atoms in development mode.

Most often, you'll use the following hooks to interact with atoms:

- `useRecoilState()`: Use this hook when you intend on both reading and writing to the atom. This hook subscribes the component to the atom.
- `useRecoilValue()`: Use this hook when you intend on only reading the atom. This hook subscribes the component to the atom.
- `useSetRecoilState()`: Use this hook when you intend on only writing to the atom.
- `useResetRecoilState()`: Use this hook to reset an atom to its default value.
- `useRecoilCallback()`: For rare cases where you need to read an atom's value without subscribing the component

You can initialize an atom either with a static value or with a `Promise` or a `RecoilValue` representing a value of the same type.
Because the `Promise` may be pending or the default selector may be asynchronous it means that the atom value may also be pending or throw an error when reading.
Note that you cannot currently assign a `Promise` when setting an atom. Please **use selectors for async functions**.

Atoms cannot be used to store `Promise`'s or `RecoilValue`'s directly, but they may be wrapped in an object.
Note that Promise's may be mutable. Atoms can be set to a `function`, as long as it is pure, but to do so
you may need to use the updater form of setters. (e.g. `set(myAtom, () => myFunc);`).

[**Atom Family**](https://recoiljs.org/docs/api-reference/utils/atomFamily)

An `atom` represents a piece of state with Recoil. An atom is created and registered per `<RecoilRoot>` by your app.
But, what if your state isn’t global? What if your state is associated with a particular instance of a control, or with a particular element?
For example, maybe your app is a UI prototyping tool where the user can dynamically add elements and each element has state, such as its position.
Ideally, each element would get its own atom of state. You could implement this yourself via a memoization pattern.
But, Recoil provides this pattern for you with the atomFamily utility. An Atom Family represents a collection of atoms.
When you call atomFamily it will return a function which provides the `RecoilState` atom based on the parameters you pass in.

The `atomFamily` essentially provides a map from the parameter to an atom. You only need to provide a single key for the `atomFamily` and it will
generate a unique key for each underlying atom. These atom keys can be used for persistence, and so must be stable across application executions.
The parameters may also be generated at different callsites and we want equivalent parameters to use the same underlying atom.
Therefore, value-equality is used instead of reference-equality for `atomFamily` parameters.
This imposes restrictions on the types which can be used for the parameter. atomFamily accepts primitive types,
or arrays or objects which can contain arrays, objects, or primitive types.

-----

### [Selectors](https://recoiljs.org/docs/api-reference/core/selector)

A **selector** represents a piece of **derived state**
(the output of passing state to a pure function that modifies the given state in some way).
Derived state is a powerful concept because it lets us build dynamic data that depends on other data.

Selectors can be used as one way to incorporate asynchronous data into the Recoil data-flow graph.
Selectors represent "idempotent" functions: For a given set of inputs they should always produce the same results (at least for the lifetime of the application).
This is important as selector evaluations may be cached, restarted, or executed multiple times.
Because of this, selectors are generally a good way to model read-only DB queries.

From a component's point of view, **selectors** can be read using the same hooks that are used to read atoms.
However it's important to note that certain hooks only work with **writable state** (i.e `useRecoilState()`).
All atoms are writable state, but only some selectors are considered writable state (selectors that have both a `get` and `set` property).

Selectors represent a function, or **derived state** in Recoil. You can think of them as similar to
an "idempotent" or "pure function" without side-effects that always returns the same value for a given set of dependency values.
If only a `get` function is provided, the selector is read-only and returns a `RecoilValueReadOnly` object.
If a `set` is also provided, it returns a writeable `RecoilState` object.

Recoil manages atom and selector state changes to know when to notify components subscribing to that selector to re-render.
If an object value of a selector is mutated directly it may bypass this and avoid properly notifying subscribing components.
To help detect bugs, Recoil will freeze selector value objects in development mode.

[**Selector Family**](https://recoiljs.org/docs/api-reference/utils/selectorFamily)

Returns a function that returns a read-only `RecoilValueReadOnly` or writeable `RecoilState` selector.

A `selectorFamily` is a powerful pattern that is similar to a `selector`, but allows you to pass parameters to the get and set callbacks of a selector.
The `selectorFamily()` utility returns a function which can be called with user-defined parameters and returns a `selector`.
Each unique parameter value will return the same memoized `selector` instance.

The `selectorFamily` essentially provides a map from the parameter to a `selector`.
Because the parameters are often generated at the callsites using the family, and we want equivalent parameters to re-use the same underlying selector,
it uses value-equality by default instead of reference-equality. (There is an unstable API to adjust this behavior).
This imposes restrictions on the types which can be used for the parameter. Please use a primitive type or an object that can be serialized.
Recoil uses a custom serializer that can support objects and arrays, some containers (such as ES6 Sets and Maps),
is invariant of object key ordering, supports Symbols, Iterables, and uses toJSON properties for custom serialization
(such as provided with libraries like Immutable containers). Using functions or mutable objects, such as Promises, in parameters is problematic.

----

### [Asynchronous Data Queries](https://recoiljs.org/docs/guides/asynchronous-data-queries)

Recoil can map state and **derived state** to React components via a data-flow graph. The functions in the graph can also be asynchronous.
So we can use asynchronous functions in synchronous React component render functions.
Recoil allows you to mix synchronous and asynchronous functions in your data-flow graph of **selectors**.
Simply return a Promise to a value instead of the value itself from a **selector** get callback, the interface remains exactly the same.
Because these are just **selectors**, other **selectors** can also depend on them to further transform the data.

Todo:
- check and add sync and async examples from https://recoiljs.org/docs/guides/asynchronous-data-queries/
- explore selectors more: https://recoiljs.org/docs/api-reference/core/selector/
