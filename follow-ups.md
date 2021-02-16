# Follow ups

## DONE

- `[MAJOR]` Deprecated `component.base`

## Backing Node follow ups

- Revisit `prevDom` code path for null placeholders
- Ensure we are always doing `_flags` check instead of vnode.type checks
- Address many TODOs
- Move refs to internal renderCallbacks
- Rewrite rerender loop to operate on internals, not components
- Rewrite commit loop to operate on internals not components
- rewrite hooks to operate on internals?
- Always assign a number to `_original` (not null, use 0 to clear?)

## Child diffing investigations

- Reduce allocations for text nodes
- Investigate skip-index tracking instead of while loop
- Loop multiple times - recursive diff, unmount children, place child
- Combine placeChild and unmounting loop?? Do placeChild backwards? Do
  unmounting first then placeChild loop
- Explore replacing children in `internal._children` as we diff instead of
  building up a new array each time

## TODOs

- Consider further removing `_dom` pointers from non-dom VNodes
- Fix Suspense tests:
  - "should correctly render nested Suspense components without intermediate DOM #2747"
- Fix Suspense hydration tests:
  - "should hydrate lazy components through components using shouldComponentUpdate"
- Rebuild Suspense List to work with backing tree
- Reconsider eagerly allocating `_renderCallbacks` for components
- Reconsider defaulting `c.state` to `{}` for all components
- Move/remove more component properties to Internal (e.g. `_parentDom`, `_dirty`, `_globalContext`)

## Other

- `[MAJOR]` Remove select `<option>` IE11 fix in diffChildren and tell users to
  always specify a value attribute for `<option>`. History:
  https://github.com/preactjs/preact/pull/1838
- One possible implementation for effect queues: Internal nodes can have a local
  queue of effects for that node while a global queue contains the internal
  nodes that have effects.
- Feature: Top-level render handles Fragment root
- Figure out a way to externally support the use case of "start rendering at
  this child of parentDom" (in other words, remove all that code from core &
  recommend folks use
  [this technique](https://gist.github.com/developit/f321a9ef092ad39f54f8d7c8f99eb29a))
- Golf everything! Look for @TODO(golf)
- Look for ways to optimize DOM element diffing, specifically how we diff props
  - Investigate diffing props before or after children
  - Investigate inlining the loops in diffProps to capture special props
    (dangerouslySetInnerHTML, value, checked, multiple)
- Revisit all replaceNode tests
  - Top-level `render()` no longer accepts a `replaceNode` argument, and does not removed unmatched DOM nodes

## Thoughts on Suspense

- Use a special VNode (e.g. Root node) that reparents all children into a new
  DOM node (like Portals). Currently suspense manually does this but I think it
  is buggy in that it doesn't properly reset all dom pointers we currently
  maintain lol.
- Need a way to trigger updates on VNodes that may need mounting or patching.
  I'm thinking backing nodes will help here so when a node suspends, we can
  maintain its suspended state on the backing node and not the component which
  may need to be nulled or remounted (i.e. constructor & lifecycles called
  again) (re: the bug about calling forceUpdate on a suspended hydration
  component)
- Maintain the state (i.e. whether or not a node's last render suspended) as a
  flag on the VNode. This flag would be useful for commit or unmount option
  hooks to discover that that a suspended node has successfully rerendered or
  unmounted. Once that is detected the nearest Suspense node that is waiting can
  be updated as such (no more overriding render or componentWillUnmount!)