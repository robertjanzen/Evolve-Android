# Gotchas

Things about this codebase's framework stack (Vue 2, Buefy and Bulma, LESS) that aren't obvious,
and that came up more than once while building the mobile UI.

## CSS rules can lose even when they look right

There are three separate tie-break rules for which CSS rule wins, checked in this order.

1. A rule marked `!important` always beats one that isn't, no matter which is more specific.
2. Among rules that agree on `!important`, the more specific selector wins, no matter which one
   appears later in the file.
3. Only when two rules are equally specific and agree on `!important` does the one written later
   win.

A mobile override that looks correct, and sits later in the file, can still lose to an older, more
specific desktop rule. Check both rules' specificity before assuming a typo.

## LESS nesting raises specificity without warning

Writing `.main { .content { .action { span.on { ... } } } }` in LESS compiles to the selector
`.main .content .action span.on`, not just `.action span.on`. A flat mobile rule targeting
`.action span.on` can lose to this compiled rule even though it looks like it targets the same
element.

## Inline styles usually beat `!important` stylesheet rules, but not always

Vue's `v-show` directive, and jQuery's `.css('display', 'none')`, both set a plain inline style (not
marked important). Any `!important` stylesheet rule for that same property will silently win over it
anyway, which breaks the intended show or hide.

If a mobile style needs to lose to one of these, drop `!important` from that one property and raise
the selector's specificity instead. If a hide genuinely needs to always win (see
`.action a.button span.count` in `evolve.less`), set the inline style from JS with
`element.style.setProperty('display', 'none', 'important')` instead of jQuery's `.css()`, which
can't express `!important` at all. An inline `!important` style beats every stylesheet rule,
important or not.

## Bulma's own rules often beat our custom classes

Rules like `.content ul`, `.content li + li`, and `.columns.is-mobile` combine a class with one or
two element selectors, which beats a single custom class on specificity alone. The result is a
margin, padding, or wrap rule that looks correct in our stylesheet but never actually applies. Check
the element's computed style and its full list of matching rules before assuming a typo. The fix is
almost always to add `!important` or match the framework's own specificity, not to rewrite our rule.

## `.columns.is-mobile` never wraps on its own

Bulma only adds `flex-wrap: wrap` through a separate `.is-multiline` modifier. `.is-mobile` on its
own just forces a row layout at every screen width. A `.columns.is-mobile` row with more content
than fits will overlap instead of wrapping, unless `flex-wrap: wrap` is added by hand.

## Tap-and-hold can't rely on `mouseover` and `event.isTrusted`

A natural way to give a button both "tap to run it" and "hold to see its description" is to show the
description on `mouseover`, and use `event.isTrusted` to tell a real hover apart from the
touch-compatibility mouseover every browser fires after a tap. That check isn't reliable across
every device and WebView.

The fix, in `popover()` in `functions.js`: on a touch device, don't listen for the real `mouseover`
event at all. Listen for a made-up event called `evolve:popover-show` instead (the constant
`MANUAL_POPOVER_TRIGGER`). Only a deliberate press-and-hold timer fires that event (see `setAction()`
in `actions.js` and `bindHoldToShowInfo()` in `functions.js`). The browser's own compatibility
mouseover, trusted or not, has nothing to trigger.

## `<b-tabs>` is empty until Vue mounts it

Code that needs to touch the real rendered `nav`, `ul`, and `li` elements of a tab bar has to run
after the matching `new Vue({el, ...})` or `vBind()` call finishes. Before that, `<b-tabs>` is just a
placeholder tag with none of its real content built yet.

## Passive touch listeners can't stop scrolling

A custom drag gesture, like the pull-up drawer in `index.js`, needs its `touchmove` listener
registered with `{ passive: false }`, and needs to call `preventDefault()` while the drag is active.
A passive listener can watch a touch but can never stop the browser's own scrolling. If the same area
also needs to allow scrolling in another direction, wait for a few pixels of movement first and
decide whether the drag is vertical or horizontal before doing anything.

## Two CSS variables track real bar heights

`--topbar-height` and `--bottomnav-height` are set by a `ResizeObserver` in `main.js`, not
hardcoded, because the top and bottom bars can be a different height depending on the theme, the
font, and whether the top bar has wrapped to two lines. Anything that needs to clear one of these
bars should read the variable instead of guessing a fixed height.

## One row layout to copy for label-and-stepper rows

`.fuelItem` and `.factory` in `evolve.less` are the pattern to copy for any row that has a label on
the left and a minus, number, plus stepper on the right. The label gets a fixed width, and the
number box between the buttons grows to fill the rest of the row, which pushes the whole stepper
flush against the row's right edge. New rows shaped like this should reuse this pattern instead of
inventing a new one.

## Installing a new Capacitor plugin means two `npm install`s

Capacitor plugins need to be installed in the root `package.json` so `npx cap sync` can register
them on the native Android side, and separately in `game/package.json` so esbuild can resolve the
`import` when it bundles `game/src/index.js`. Installing it in only one place will build fine and
then fail at runtime, or fail to bundle at all.
