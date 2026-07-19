## Recipes & Fixes

> Problem → solution threads extracted from #need_help Discord.
> Organized by topic. Each entry is a complete problem → solution.

### Styling

### Row with `fillMaxSize()` does not center content horizontally

**Problem:** A `Row` with `fillMaxSize()` does not center its child `Column` horizontally. The Column stays left-aligned instead of centered. ``` Row( modifier = Modifier .alignContent(AlignContent.Center) .fillMaxSize()) { Column( horizontalAlignment = Alignment.CenterHorizontally, modifier = Modifier .fillMaxHeight() .width...

**Solution:** Use `horizontalArrangement = Arrangement.Center` on the `Row` instead of `alignContent`. The `alignContent` property distributes space along the cross axis in a flex container, not the main axis. For horizontal centering in a `Row`, use `horizontalArrangement`. Also ensure the Row has a fixed or constrained width; `fillMaxSize()` without a parent constraint may not provide a reference width. Debug by temporarily setting a fixed width like `width(800.px)`.

```

Row(    modifier = Modifier        .alignContent(AlignContent.Center)        .fillMaxSize()) {    Column(        horizontalAlignment = Alignment.CenterHorizontally,        modifier = Modifier            .fillMaxHeight()            .width(600.px)            .textAlign(TextAlign.Center)            .backgroundColor(Color.red),    ) {        Header()        content()        Spacer()        Footer()    }}
```

---

### Column and Row nesting does not center on both axes

**Problem:** A `Column` with `horizontalAlignment = Alignment.CenterHorizontally` wrapping a `Row` with `verticalAlignment = Alignment.CenterVertically` does not center content on both axes. ``` Column(Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally) { Row(Modifier.fillMaxHeight(), verticalAlignment = Alignment.CenterVertically) { content() } } ```

**Solution:** Replace `Column` + `Row` with a `Box`. `Box` centers on both axes by default with `contentAlignment = Alignment.Center`, avoiding the nested flex-direction conflict.

```

Column(Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally) {         
   Row(Modifier.fillMaxHeight(), verticalAlignment = Alignment.CenterVertically) {     
       content()    
   }
}
```

---

### Learning curve: combining HTML/CSS with Compose

**Problem:** A developer familiar with HTML/CSS is struggling to translate styling concepts into Kobweb's Compose-based approach.

**Solution:** Kobweb uses JetBrains Compose HTML, which replaces CSS selectors with `Modifier` chains and `ComponentStyle` definitions. Map CSS concepts directly:
- CSS classes → `ComponentStyle { base { Modifier... } }`
- CSS pseudo-classes → `hover`, `focus`, `active` blocks inside `ComponentStyle`
- CSS flexbox → `Row` / `Column` composables with `Arrangement` and `Alignment`
- CSS `@media` → `Breakpoint.MD { Modifier... }` blocks
Study `kobweb create app` scaffold output for a working example.

---

### Page found but Kobweb returns 404

**Problem:** The page loads but Kobweb returns a 404 error for the requested route.

**Solution:** Reproduce with `kobweb run --env prod --layout static` locally. This issue typically occurs when routes are not properly configured or when the export layout doesn't match the deployment target. Check that:
1. All `@Page` annotated files are in the correct `pages/` directory
2. Route overrides in `@Page(routeOverride = "...")` don't conflict
3. For static export, verify `conf.yaml` has the correct `layout: static`

---

### Horizontally scrollable Row

**Problem:** How to create a horizontally scrollable Row with flex items that don't shrink.

**Solution:** Use `flexShrink(0)` on child items to prevent them from shrinking, and `overflowX(Overflow.Auto)` on the parent Row. `Row` already uses flex display, so `display(DisplayStyle.Flex)` is redundant.

```kotlin
Row(
                Modifier
                    .height(60.px)
                    .fillMaxWidth()
                    .display(DisplayStyle.Flex)
                    .overflow(Overflow.Auto, Overflow.Hidden),
                horizontalArrangement = Arrangement.Center
            ) {
                tags.forEach { tag ->
                    Row(
                        Modifier
                            .backgroundColor(Color.gray)
                            .padding(4.px)
                            .borderRadius(20.px)
                            .flexShrink(0),
                        verticalAlignment = Alignment.CenterVertically
                    ) {
                        SpanText(tag)
                    }
                }
            }
```

---

### FlexWrap.Nowrap with JustifyContent.Start for scrollable row

**Problem:** Whether `flexWrap(FlexWrap.Nowrap)` works with `justifyContent(JustifyContent.Start)` in a scrollable Row.

**Solution:** Yes. Use `overflowX(Overflow.Auto)` with `flexWrap(FlexWrap.Nowrap)` and `justifyContent(JustifyContent.Start)` on a `Row` to create a horizontally scrollable set of items.

```

Row(
        Modifier
            .fillMaxWidth()
            .overflowX(Overflow.Auto)
            .flexWrap(FlexWrap.Nowrap)
    ) {
        for (i in 1..10) {
            Button(onClick = {}, Modifier.margin(5.px).minWidth(100.px)) {
                Text("Button $i")
            }
        }
    }
```

---

### touchmove event does not cancel long-press detection

**Problem:** The `touchmove` event listener does not fire during touch interactions, preventing cancellation of long-press detection.

**Solution:** Register `touchstart`, `touchend`, and `touchmove` via `attrsModifier { addEventListener(...) }`. Use `delay` in a coroutine to detect long press, and cancel the job on `touchend` / `touchmove`.

```

PageLayout("Welcome to Kobweb!") {
        val scope = rememberCoroutineScope()
        Row(Modifier.width(100.px).height(30.px).backgroundColor(Colors.AliceBlue).attrsModifier {
            addEventListener("click") {
                scope.launch {
                    delay(800L)
                    println("CLICKED")
                }
            }
        }) {

        }
```

```kotlin
@Composable
fun WordRow(
    word: WordModel,
    breakpoint: Breakpoint,
    front: @Composable () -> Unit = {},
    end: @Composable () -> Unit = {},
    mobileContext: List<ContextMenuItem> = emptyList(),
    onContext: ((SyntheticMouseEvent, WordModel) -> Unit)? = null
) {
    val scope = rememberCoroutineScope()
    var job: Job? = null
    var isOpen by remember { mutableStateOf(false) }
    Column(Modifier.width(if (breakpoint.isMobile()) 100.percent else 80.percent)) {
        Row(
            SurfaceContent.toModifier().fillMaxWidth(),
            verticalAlignment = Alignment.CenterVertically
        ) {
            front()
            Row(
                Modifier
                    .justifyContent(JustifyContent.SpaceBetween).padding(4.px)
                    .onClick { window.location.href = "/app/card?id=${word.id}" }
                    .flexGrow(1)
                    .attrsModifier {
                        onContext?.let { fn ->
                            this.onContextMenu {
                                it.preventDefault()
                                if (!breakpoint.isMobile()) fn(it, word)
//                                else isOpen = !isOpen
                            }

                            addEventListener("touchstart") {
                                job = scope.launch {
                                    delay(800L)
                                    isOpen = true
                                }
                            }

                            addEventListener("touchend") {
                                job?.cancel()
                            }
                            addEventListener("touchmove") {
                                job?.cancel()
                            }
                        }
                    }
            ) { content stuff
```

---

### addVariant fails with "Key ... is missing in the map"

**Problem:** Calling `ButtonStyle.addVariant` in a top-level property before using it inside a Composable gives `Key ... is missing in the map`.

**Solution:** Reference the Silk widgets source for the correct variant registration pattern: https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-widgets

---

### Add hashCode() to ComponentVariant and ComponentStyle

**Problem:** The `ComponentVariant` and `ComponentStyle` classes lack `hashCode()` implementations.

**Solution:** If not using variants, the missing `hashCode()` is unlikely to be noticed. Variants are optional; skip them if they add complexity without benefit.

---

### Using ComponentStyle

**Problem:** How to use `ComponentStyle` in Kobweb?

**Solution:** Reference the Button style implementation in Kobweb's source: https://github.com/varabyte/kobweb/blob/a46c08e97dd40cb32eebba493f4ecc40b1ac3983/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L62

---

### CSS descendant selector in ComponentStyle

**Problem:** How to define a style for every `tr` element inside a table using `ComponentStyle` without creating a separate component?

**Solution:** Use `cssRule` within `ComponentStyle` to apply CSS child combinators. Reference: https://github.com/varabyte/kobweb/blob/f82c8fc40b19950fb9679bb8e5575647b3b261c9/frontend/kobweb-compose/src/jsMain/kotlin/com/varabyte/kobweb/compose/style/KobwebComposeStyleSheet.kt#L26. For "every `tr` in a table", use `cssRule("table > tr") { ... }`.

---

### Scoped CSS rules for specific component styles

**Problem:** CSS rules defined in a `ComponentStyle` apply globally to all matching elements. How to scope rules to just one component?

**Solution:** Use a class name string with CSS selectors like `"myClassName > tr"`. Alternatively, use `cssRule` inside `ComponentStyle`:

```
ComponentStyle("test") { cssRule("> tr") { ... } }
```

---

### Column justify-content property overridden by Kobweb CSS

**Problem:** Using `justifyContent(JustifyContent.SpaceEvenly)` in a `ComponentStyle` base block gets overridden by the `.kobweb-col...` CSS class.

**Solution:** Reference the available arrangement options: https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement. Use `Arrangement` parameters directly on the `Column` composable instead of inside `ComponentStyle` when the Kobweb default style conflicts.

---

### Using ComponentStyle with breakpoints

**Problem:** ComponentStyle provides cleaner code for responsive design, but the layout property gets overridden by Kobweb's default column styles.

**Solution:** Define breakpoint-specific modifiers in `ComponentStyle` and apply via `toModifier()`:

```kotlin
val myStyle = ComponentStyle("my-style") {
    base {
        Modifier.justifyContent(JustifyContent.SpaceEvenly)
    }
    Breakpoint.MD {
        Modifier.justifyContent(JustifyContent.Start)
    }
}

@Page
@Composable
fun HomePage() {
    PageLayout("Index") {
        Column(
            myStyle.toModifier()
                .backgroundColor(Colors.LightBlue)
                .width(200.px)
                .height(500.px)
        ) {
            repeat(3) {
                SpanText("Text")
            }
        }
    }
}
```

---

### extraModifiers parameter broken for ComponentStyle.addVariant

**Problem:** The `extraModifiers` parameter of `ComponentStyle.addVariant` / `addVariantBase` works for creating styles but not for variants.

**Solution:** This is a known issue with variant support not being thoroughly tested. The variant system applies `extraModifiers` at the style level, but variants layer on top of the base style and may bypass extra modifiers.

```kotlin
val ExtraVariant = ButtonStyle.addVariantBase(
    "extra-variant-base",
    extraModifiers = Modifier.backgroundColor(Colors.Blue).id("myID-variant-base")
) {
    Modifier.size(200.px)
}

val ExtraVariantBase = ButtonStyle.addVariant(
    "extra-variant",
    extraModifiers = Modifier.backgroundColor(Colors.Blue).id("myID-variant")
) {
    base{ Modifier.size(200.px) }
}

val NormalVariant = ButtonStyle.addVariantBase(
    "normal-variant2",
) {
    Modifier.size(200.px).backgroundColor(Colors.Blue)
}

val NewExtraButtonStyle = ComponentStyle(
    "extra-style",
    extraModifiers = Modifier.backgroundColor(Colors.Blue).id("myID-style")
) {
    base {
        Modifier.size(200.px)
    }
}

@Page
@Composable
fun HomePage() {
    PageLayout("Index") {
        Button(
            { println("clicked") },
            variant = ExtraVariantBase
        ) { Text("Variant Base") }
        Button(
            { println("clicked") },
            variant = ExtraVariant
        ) { Text("Variant") }
        Button(
            { println("clicked") },
            variant = NormalVariant
        ) { Text("Style") }
        Box(NewExtraButtonStyle.toModifier())
    }
}
```

---

### Arrangement.None to avoid setting justify-content

**Problem:** Need a way to avoid setting the `justify-content` CSS property so parent or inline styles can take precedence.

**Solution:** Use `verticalArrangement = Arrangement.None` on `Column` to skip setting `justify-content`. This lets inline `styleModifier` values apply without Kobweb overriding them.

```

".kobweb-col.kobweb-unset" {  }
```

```kotlin
val MyStyle = ComponentStyle("my-style") {
    base {
        Modifier
            .width(200.px)
            .height(200.px)
            .justifyContent(JustifyContent.SpaceEvenly)
    }
    Breakpoint.MD {
        Modifier.justifyContent(JustifyContent.Start)
    }
}

@Page
@Composable
fun HomePage() {
    PageLayout("Index") {
        Column(MyStyle.toModifier().backgroundColor(Colors.LightBlue), horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.None) {
            repeat(3) {
                SpanText("Text")
            }
        }
    }
}
```

---

### Button variant with color-mode-aware modifier across all states

**Problem:** Applying a color-mode-aware modifier to a `ButtonStyle` variant requires overriding each state (hover, focus, active, enabled) separately.

**Solution:** Override all state combinations explicitly since variant styles layer on top of the base and each state:

```kotlin
val variant = ButtonStyle.addVariant("variant") {
    val modifier = Modifier.backgroundColor(colorMode.toSilkPalette().button.default)
    base { modifier }
    (hover + enabled) { modifier }
    (focus + enabled) { modifier }
    (active + enabled) { modifier }
    (focus + active + enabled) { modifier }
}
```

---

### Separate base ButtonStyle from ThemedButton

**Problem:** Button styles require including `enabled` in each state block. A base button without effects and a separate `ThemedButton` that incorporates the palette would be cleaner.

**Solution:** The `enabled` requirement in each state block is a current limitation. The approach of separating a base style from a themed variant is reasonable but needs API work to eliminate the redundant `+ enabled` pattern.

---

### Override ButtonStyle backgroundColor without per-state `+ enabled`

**Problem:** Overriding `backgroundColor` on the default `ButtonStyle` without adding `+ enabled` to every state does not work as expected.

**Solution:** This approach does not reliably suppress hover / focus / active effects. Each state must still be explicitly overridden:

```kotlin
val ButtonStableVariant = ButtonStyle.addVariant("stable") {
    (hover + enabled) { Modifier }
    (focus + enabled) { Modifier }
    (active + enabled) { Modifier }
    (focus + active + enabled) { Modifier }
}
```

---

### Circular button with border-radius 50%

**Problem:** Whether `borderRadius(50.percent)` creates a circular button.

**Solution:** A better approach is to add a single `circle` variant using `clip(50.percent)` so it works across multiple sizes. Users can also apply `clip(50%)` themselves in their modifier chain.

---

### Combining multiple styles with toModifier()

**Problem:** How to combine multiple style modifiers into one?

**Solution:** Use `modifier = listOf(TextStyle, TextHeaderStyle).toModifier()` to combine multiple `ComponentStyle` instances into a single `Modifier`.

---

### Styled UI component library for Kobweb

**Problem:** Suggestion to create a styled UI library to make Kobweb more attractive to users.

**Solution:** Silk's widget library is currently rudimentary but will expand as it receives more development effort.

---

### CSS animation-timing-function in Kobweb

**Problem:** How to write `animation-timing-function: cubic-bezier(0, 1, 1, 0);` in Kobweb?

**Solution:** Use `styleModifier { property("animation-timing-function", "cubic-bezier(0, 1, 1, 0)") }`. Animation properties require an `animation-name` to be meaningful.

---

### Three-state box animation with CSS

**Problem:** Can a box move to three different positions with a button toggling states using CSS?

**Solution:** Use CSS transitions triggered by class or state changes. Simplifying the animation logic using Kotlin state management may be easier than pure CSS keyframe animations.

---

### Clipboard paste data extraction

**Problem:** How to extract clipboard data on paste events using `it.clipboardData?.getData("text/plain")`?

**Solution:** Use `attrsModifier { onPaste { event -> event.clipboardData?.getData("text/plain") } }`.

---

### Using attrsModifier for paste event

**Problem:** How to implement paste event handling in Kobweb without raw JavaScript?

**Solution:** Use `attrsModifier { onPaste { evt -> ... } }` instead of raw JS `addEventListener`:

```kotlin
onPaste { event ->
    event.preventDefault()
    val text = event.clipboardData?.getData("text/plain")
    document.execCommand("insertHtml", false, text!!)
}
```

---

### Hover style on child elements with cssRule

**Problem:** How to apply a hover style to direct children of a component?

**Solution:** Use `(hover + cssRule(" > *"))` inside `ComponentStyle` to target direct children on hover:

```
(hover + cssRule(" > *")) {
        Modifier
            .border(
                width = 1.px,
                style = LineStyle.Solid,
                color = Color(Theme.Primary.color)
            )
            .backgroundColor(Colors.White)
    }
```

---

### Styling CheckboxInput / using @InitSilk

**Problem:** How to apply custom styles to `CheckboxInput`?

**Solution:** Register styles via `@InitSilk` instead of creating a separate `StyleSheet` class:

```kotlin
@InitSilk
fun initSilk(ctx: InitSilkContext) {
  ctx.stylesheet.registerStyle("...") { ... }
}
```

---

### Using Kobweb widgets in browser extensions

**Problem:** Browser extensions only run as `.html` files, making full Kobweb page usage difficult.

**Solution:** Some Kobweb parts (widgets, Modifier APIs) work within extensions. Partial Kobweb usage inside extension popup or options pages is feasible.

---

### Simulating focus border color change on input

**Problem:** How to update the border color of an input field when clicking associated icons to simulate focus?

**Solution:** Use CSS transitions triggered by class toggling. The transition API is being cleaned up: https://github.com/varabyte/kobweb/commit/2a97f17da380f5222cec879e21ee1eea5cdd4f8d

---

### Event handling: preventDefault, stopPropagation, stopPropagationImmediately

**Problem:** What are the differences between the three event handling options?

**Solution:** Three event handling options:
1. `preventDefault()` — prevents the underlying element type from handling the event (e.g., preventing focus on a div)
2. `stopPropagation()` — stops the event from being handled by other elements underneath the current one
3. `stopPropagationImmediately()` — stops the event from all other elements AND any other event handlers registered on the current element

```

Surface(
    modifier = Modifier
        .align(Alignment.CenterEnd)
        .margin(right = 12.px)
        .onMouseDown { it.preventDefault() }
        .onClick { value = "" }
) {
```

---

### Preserving sidebar open/close state across page changes

**Problem:** How to preserve sidebar open/close state when navigating between pages?

**Solution:** Use a `mutableStateOf` value at the app level that persists across page transitions. No `lazy` trick needed unless the value depends on startup configuration (like `colorModeState`).

---

### Preventing recomposition of shared composables across pages

**Problem:** When switching pages, shared composables like `SideNavBar` get recomposed unnecessarily.

**Solution:** If `SideNavBar` is called within `PageLayout`, it persists across page navigations as long as `PageLayout` is called by both pages. DOM shifts caused by element changes can trigger recomposition.

---

### Virtual scroll / LazyColumn in Compose HTML

**Problem:** Does the concept of virtual scrolling (like `LazyColumn`) exist in CSS or Compose HTML?

**Solution:** No built-in `LazyColumn` exists for Compose HTML yet. Community implementations exist but have not been integrated into Kobweb.

---

### Hide scrollbar on overflow column

**Problem:** How to hide the scrollbar on a Column with overflow?

**Solution:** Use CSS `overflow: hidden` on the container: https://www.w3schools.com/css/tryit.asp?filename=trycss_overflow_hidden

---

### Smooth scroll to section on menu click

**Problem:** How to smoothly scroll to a section when a menu item is selected?

**Solution:** Use a `LaunchedEffect` keyed on the selected section ID. Query the target element with `querySelector` and call `scrollIntoView("smooth")`:

```kotlin
var scrollableColumn: Element? by remember { mutableStateOf(null) }

    LaunchedEffect(state.selectedSection) {
        scrollableColumn?.let { scrollableColumn ->
            val section = scrollableColumn.querySelector(state.selectedSection.title)
            section?.scrollIntoView("smooth")
        }
    }
```

Capture the scroll container ref via `attrsModifier { ref { ... } }`:

```kotlin
.attrsModifier {
                        ref {
                            scrollableColumn = it
                            onDispose { scrollableColumn = null }
                        }
                    }
```

Assign IDs to target sections:

```kotlin
UserDetailSection(
                    modifier = Modifier.id(UserDetailSection.USER_INFO.title),
                    state = state,
                    vm = vm,
                    header = {
                        SpanText(UserDetailSection.USER_INFO.title)
                        UserDetailHeaderSection(state)
                    }
                )
```

---

### Smooth scroll without hash fragment

**Problem:** How to implement smooth scrolling using CSS without changing the href or hash?

**Solution:** Use JavaScript's `scrollIntoView("smooth")` or CSS `scroll-behavior: smooth`: https://codepen.io/jasongart/pen/qgmGxB

---

### styleModifier not applying CSS property

**Problem:** Using `styleModifier { property("text-shadow", ...) }` but the property does not appear on the element.

**Solution:** `text-shadow` is not yet exposed as a first-class modifier. Use `styleModifier` for custom properties. The `text-shadow` modifier will be added in a future release.

```

Row(Modifier
            .fontFamily("Koulen")
            .fontStyle(FontStyle.Normal)
            .fontWeight(400)
            .fontSize(64.px)
            .color(rgb(233, 0, 0))
            .styleModifier { property("text-shadow", "3px, 4px, 7px, 0px") }) {
            H1 { Text("KORIPSYON") }
        }
```

---

### Multiple text shadows support

**Problem:** Does Kobweb support multiple text shadow values?

**Solution:** Not yet. Support for multiple text shadows will be added.

---

### TextShadow API design

**Problem:** Why not use a sealed class pattern for the `TextShadow` API?

**Solution:** The proposed sealed class approach:

```

sealed class TextShadow {
   class Keyword : TextShadow
   class Params : TextShadow
}

fun textShadow(vararg params: TextShadow.Params)
fun textShadow(keyword: TextShadow.Keyword)
```

may be inconsistent with the rest of Kobweb's modifier API. A simpler overloaded function approach may be preferred.

---

### Managing ComponentStyle constants in objects

**Problem:** How to organize multiple `ComponentStyle` values in a structured way?

**Solution:** Group styles in an `object`:

```kotlin
object ProjectStyles {
const val container by ComponentStyle {
...
}
const val items by ComponentStyle {
...
}
}
```

**Note:** Style names ("container", "items") are taken from the `by ComponentStyle` delegation. Copy-pasting between objects and forgetting to update the variable name will cause runtime errors because the CSS class name stays the same.

---

### Returning StyleResult from style functions

**Problem:** Can all functions return `StyleResult` for use in stylesheets?

**Solution:** Use `ctx.stylesheet.registerBaseStyle("@font-face")` for font-face rules:

```css
ctx.stylesheet.apply {
        registerBaseStyle("@font-face") {
            Modifier
                .fontFamily("CardTitle")
                .styleModifier {
                    property("src", "url(fonts/CardTitle.ttf")
                }
        }
```

---

### Empty base styles with base { Modifier }

**Problem:** How to handle empty or minimal styles in `ComponentStyle`?

**Solution:** Use `base { Modifier }` for empty base styles, or create a helper method. Verify the font file is at the correct path in `resources/public/font/`.

---

### Multiple @font-face registrations

**Problem:** Registering multiple `@font-face` rules with different font weights using `@InitSilk`.

**Solution:** Copy the generated HTML output to a text editor to inspect the CSS and verify each `@font-face` rule is correctly registered.

```kotlin
@InitSilk
fun updateTheme(ctx: InitSilkContext) {
    ctx.stylesheet.registerStyle("@font-face") {
        base {
            Modifier
                .fontFamily("Centra")
                .fontWeight(700)
                .styleModifier {
                    property("src", "url(font/CentraNo2-Bold.ttf)")
                }
        }
    }
    ctx.stylesheet.registerStyle("@font-face") {
        base {
            Modifier
                .fontFamily("Centra")
                .fontWeight(500)
                .styleModifier {
                    property("src", "url(font/CentraNo2-Medium.ttf)")
                }
        }
    }
    ctx.stylesheet.registerStyle("@font-face") {
        base {
            Modifier
                .fontFamily("Centra")
                .fontWeight(400)
                .styleModifier {
                    property("src", "url(font/CentraNo2-Book.ttf)")
                }
        }
    }
    ctx.stylesheet.registerStyle("body") {
        base {
            Modifier
                .fontFamily(
                    "-apple-system", "BlinkMacSystemFont", "Segoe UI", "Roboto", "Oxygen",
                    "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
                    "sans-serif;"
                )
                .styleModifier {
                    property("-webkit-font-smoothing", "antialiased")
                    property("-moz-osx-font-smoothing", "grayscale")
                }
        }
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <META http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Portfolio Kotlin</title>
    <meta content="Powered by Kobweb" name="description">
    <link href="/favicon.ico" rel="icon">
    <meta content="width=device-width, initial-scale=1" name="viewport">
    <meta content="/banner.png" name="og:image">
    <meta content="Kobweb Portfolio" name="og:site_name">
    <meta content="website" name="og:type">
    <meta content="Kobweb Portfolio" name="twitter:site">
    <meta content="Kobweb Portfolio" name="twitter:title">
    <meta content="A sample portfolio site made from kotlin using kobweb🗿" name="twitter:description">
    <meta content="summary_large_image" name="twitter:card">
    <meta content="/banner.png" name="twitter:image:src">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css" rel="stylesheet">
  </head>
  <body>
    <div id="root"></div>
    <!-- Encoded spinner character is a cobweb -->
    
    <div id="status">
      <span id="warning">❌</span><span id="spinner">&#128376;️</span> <span id="text"></span>
    
      ...KobWeb Style..
      
    </div>
    <script src="/portfolio.js"></script>
  </body>
</html>
```

---

### Integrating JavaScript libraries with callbacks (SweetAlert2)

**Problem:** How to call JavaScript libraries from Kobweb and handle callbacks like `.then()` when using SweetAlert2?

**Solution:** Minimize raw `js("...")` calls. Write Kotlin/JS interop code, cast the result to a `Promise`, and handle it from Kotlin:

```

P(
            attrs = Modifier
                .id("myId")
                .onClick {
                    js("Swal.fire({\n" +
                        "  title: 'Error!',\n" +
                        "  text: 'Do you want to continue',\n" +
                        "  icon: 'error',\n" +
                        "  confirmButtonText: 'Cool'\n" +
                        "});") as Unit
                }
                .toAttrs()
        ) {
            Text("Click Me")
        }
```

```

Swal.fire({
  title: 'Do you want to save the changes?',
  showDenyButton: true,
  showCancelButton: true,
  confirmButtonText: 'Save',
  denyButtonText: `Don't save`,
}).then((result) => {
  /* Read more about isConfirmed, isDenied below */
  if (result.isConfirmed) {
    Swal.fire('Saved!', '', 'success')
  } else if (result.isDenied) {
    Swal.fire('Changes are not saved', '', 'info')
  }
})
```

---

### Conditional styles for selected navigation state

**Problem:** How to apply a selected style when a navigation link is active, alongside existing hover styles?

**Solution:** Reference the Button's state-based style implementation: https://github.com/varabyte/kobweb/blob/369b94fe7349d22dc95a28004747353244641c24/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L51

---

### Sticky header with position: sticky

**Problem:** How to implement a sticky header without z-index issues?

**Solution:** Use CSS `position: sticky` instead of `fixed`. Reference implementation: https://github.com/bitspittle/bitspittle.dev/blob/222310af50d8f17973d733ee9dfee0d49aceb421/site/src/jsMain/kotlin/dev/bitspittle/site/components/sections/NavHeader.kt#L38. Consider `deferRender` for handling stacking context issues.

---

### Layout with fixed header and scrollable content

**Problem:** How to create a layout with a non-scrollable header and a scrollable content area?

**Solution:** Use a `ScrollableColumn` — a `Column` with the appropriate overflow modifier — inside the main `Column`:

```kotlin
Column {
    Header()

    ScrollableColumn {
        Content()
    }
}
```

---

### ScrollableColumn is a pattern, not a built-in

**Problem:** Is `ScrollableColumn` a built-in Kobweb composable?

**Solution:** No, `ScrollableColumn` is a pattern (a `Column` with `overflow: auto` modifier), not a built-in composable.

---

### Storing AttrBuilderContext in a variable

**Problem:** Can you store `AttrBuilderContext` in a variable and reuse it?

**Solution:** Yes, but reusing attribute contexts is a weak area in Kobweb. Use `Modifier.toAttrs` for a cleaner approach:

```kotlin
// I know it's not the right syntax
val attrs = AttrBuilderContext<HTMLVideoElement> {
    attr("width", "100%")
    onMouseEnter { (it.target as HTMLVideoElement).setAttribute("controls", "") }
    onMouseLeave { (it.target as HTMLVideoElement).removeAttribute("controls") }
}

Video(attrs)
```

---

### HTML attributes cannot be set via CSS

**Problem:** HTML attributes like `controls` cannot be manipulated via CSS.

**Solution:** Use `Modifier.toAttrs` to set HTML attributes:

```

Video(attrs = Modifier.toAttrs {
   /* .. video attributes go here .. */
}
```

---

### Combining event handlers and attributes with toAttrs

**Problem:** How to minimize code inside `toAttrs` block while keeping both event handlers and HTML attributes?

**Solution:** Chain event handlers on the `Modifier` itself before calling `toAttrs`:

```kotlin
val attrs =  Modifier.toAttrs<AttrsScope<HTMLVideoElement>> {  
    attr("width", "100%")
    onMouseEnter { (it.target as HTMLVideoElement).setAttribute("controls", "") }
    onMouseLeave { (it.target as HTMLVideoElement).removeAttribute("controls") }
}

Video(attrs)
```

```

Video(attrs = Modifier.toAttrs { ... })
```

```

Modifier
  .onMouseEnter { it.setAttribute("controls", "") }
  .onMouseLeave { it.removeAttribute("controls") }
  .toAttrs { attr("width", "100%") }
```

---

### Modifier.toAttrs generic type resolution

**Problem:** Why does `Modifier.toAttrs` require explicit `.toAttrs<AttrScope<*>>` instead of allowing `.toAttrs<*>`?

**Solution:** The generic type `AttrScope`'s element type cannot be inferred from `*` alone. Use the explicit `<AttrsScope<E>>` syntax or provide a generic helper function.

---

### Adding both inferred and explicit toAttrs overloads

**Problem:** Can both inferred and explicit generic `toAttrs` overloads coexist?

**Solution:** Yes, the compiler selects the correct overload:

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null): AttrsScope<E>.() -> Unit =
    this.toAttrs(finalHandler)
```

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null): AttrsScope<E>.() -> Unit =
    this.toAttrs<AttrsScope<E>>(finalHandler)
```

---

### Div composable with AttrBuilderContext

**Problem:** The `Div` composable takes `attrs: AttrBuilderContext<HTMLDivElement>? = null`. How to use it generically?

**Solution:** Use a helper function for the generic `toAttrs` conversion:

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null) =
    this.toAttrs<AttrsScope<E>>(finalHandler)
```

---

### Styling range inputs and creating a switch widget

**Problem:** Resources for styling cross-browser range inputs and creating a toggle switch.

**Solution:** Reference articles: https://css-tricks.com/styling-cross-browser-compatible-range-inputs-css/ and http://danielstern.ca/range.css/. A Silk switch widget should leverage both CSS transition and animation approaches.

---

### Debugging large error messages truncated by Discord

**Problem:** Error message is too large for Discord and gets auto-converted to a text file.

**Solution:** Look for the part where the layout renders as a grid of features to identify the error location.

---

### SVG Circle vs Kobweb shape

**Problem:** The `Circle` from the SVG library supports SVG-specific properties like `stroke-dasharray`, but Kobweb shapes do not.

**Solution:** Use `LaunchedEffect` to get a coroutine scope for animation timing when working with SVG elements.

---

### Canvas resizing with browser window

**Problem:** How to make a Canvas resize with the browser window?

**Solution:** Set a max size on the buffer to avoid RAM issues on ultra-wide displays. Resizing the canvas style on window resize is fine for performance.

---

### Accessing CompositionLocalProvider values in pages

**Problem:** How to access a `CompositionLocalProvider` value from any page in a Kobweb app?

**Solution:** Wrap `content()` in `CompositionLocalProvider` at the `@App` level:

```kotlin
@App
@Composable
fun MyApp(content: @Composable () -> Unit) {

    var lang by remember { mutableStateOf("french") }
    val LocalLanguage = compositionLocalOf { lang }

    SilkApp {
        Surface(SmoothColorStyle.toModifier().minHeight(100.vh)) {
            CompositionLocalProvider(LocalLanguage provides lang){
                content()
            }
        }
    }
}
```

Reference: https://github.com/varabyte/kobweb/blob/5d1259da76d7e589c4781a93da1f61f478fec557/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/core/AppGlobals.kt#L26

---

### Color-mode-aware colors in ComponentStyle

**Problem:** How to set colors based on `colorMode` inside a `ComponentStyle`?

**Solution:** Four approaches are available depending on desired complexity:

```
// Option #1 - Classic
val MyWidgetStyle by ComponentStyle {
  base { Modifier.color(colorMode -> ...) }
}

// Option #2 - Kobweb support
// No new class for variables, Kobweb plugin finds all 
// ColorComputers and registers them for you
val MyWidgetColorVar by StyleVariable<CSSColorValue>()
val MyWidgetColorComputer = ColorComputer(MyWidgetColorVar) { colorMode -> ... }

val MyWidgetStyle by ComponentStyle {
  base { Modifier.color(MyWidgetColorVar.value()) }
}

// Option #3 - Hybrid
// New class for color variables
val MyWidgetColorVar by SilkColorVariable { colorMode -> ... }

val MyWidgetStyle by ComponentStyle {
  base { Modifier.color(MyWidgetColorVar.value()) }
}

// Option #4 - Explicit
val MyWidgetColorVar by StyleVariable<CSSColorValue>()

val MyWidgetStyle by ComponentStyle {
  base { Modifier.color(MyWidgetColorVar.value()) }
}

@Composable
fun MyApp {
  val root = remember { document.getElementById("root") as HTMLElement }
  val colorMode by rememberColorMode()
  root.setVariable(MyWidgetColorVar, colorMode -> ...)
}
```

---

### Scrollable Column for images

**Problem:** How to create a scrollable Column containing images?

**Solution:** Use `Modifier.overflow(Overflow.Auto)` on the Column. Reference: https://developer.mozilla.org/en-US/docs/Web/CSS/overflow

---

### Screen size / breakpoint awareness

**Problem:** How to make a composable aware of screen size without manually tracking `window.innerWidth`?

**Solution:** Use Kobweb's built-in breakpoint system:

```kotlin
val SideBarColumnStyle by ComponentStyle {
    base { ... }
    Breakpoint.MD{
        Modifier.left((-100).px)
    }
}
```

---

### Multiple box shadows

**Problem:** Can you use multiple box shadows with `Modifier.boxShadow()`?

**Solution:** No, `boxShadow()` currently only supports a single shadow value.

---

### Dotted lines / listStyleType modifier

**Problem:** How to implement dotted lines or use `listStyleType` modifier?

**Solution:** Reference the Compose HTML implementation in Kobweb's Table of Contents component: https://github.com/varabyte/kobweb/blob/47c4e10739bd8983fbf57840b6d295a75794cdfa/frontend/kobweb-silk/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/document/Toc.kt#L69

---

### Hide scrollbar using modifier

**Problem:** How to hide a scrollbar using a Kobweb modifier?

**Solution:** Use a `Breakpoint` block inside a `ComponentStyle` to control overflow behavior, or use `Modifier.styleModifier { property("overflow", "hidden") }`.

---

### Converting CSS scrollbar example to modifier

**Problem:** How to use the CSS `::-webkit-scrollbar { display: none }` pattern in Kobweb's modifier system?

**Solution:** Reference the W3Schools guide: https://www.w3schools.com/howto/howto_css_hide_scrollbars.asp

---

### Hiding scrollbar with ::-webkit-scrollbar in ComponentStyle

**Problem:** How to hide scrollbar using `::-webkit-scrollbar` in Kobweb?

**Solution:** Use `cssRule` inside `ComponentStyle`:

```kotlin
val SectionStyle by ComponentStyle {
    cssRule("::-webkit-scrollbar") {
        Modifier.display(DisplayStyle.None)
    }
}
```

---

### Animate content visibility

**Problem:** How to animate content visibility like `AnimatedVisibility` in Jetpack Compose?

**Solution:** Use `Keyframes` and `toAnimation()`:

```kotlin
val FadeInKeyframes by Keyframes {
  0.percent { Modifier.opacity(0) }
  100.percent { Modifier.opacity(1) }
}
```

Apply with `Modifier.animation(FadeInKeyframes.toAnimation())`.

---

### animateContentSize equivalent

**Problem:** How to animate content size changes like `animateContentSize()` in Compose?

**Solution:** Use a CSS transition on `height`:

```

Modifier
  .transition(CSSTransition("height", 1.s))
```

---

### Animating auto-height elements

**Problem:** Padding stays the same when animating height with `auto`.

**Solution:** Browsers cannot animate `auto` heights due to technical complexity. Use a fixed height or `max-height` for smooth transitions.

---

### Weight / flexGrow for Column and Row children

**Problem:** How to distribute space proportionally among Column or Row children?

**Solution:** Use `Modifier.flexGrow(1)` on child elements. Future `weight` methods on `ColumnScope` / `RowScope` would wrap `flexGrow`.

---

### Recommended CSS units for Kobweb

**Problem:** Which CSS units should be used in Kobweb for sizing?

**Solution:** `cssRem` for general sizing, `percent` within widgets, `px` for small values like `borderRadius`.

---

### hCaptcha integration in Kobweb

**Problem:** How to integrate hCaptcha (or similar JS widgets) in a Kobweb page?

**Solution:** Add the script in the page head and attach the callback function as a `@JsExport`:

```kotlin
@Page
@OptIn(ExperimentalComposeWebApi::class)
@Composable
fun LoginPage() {
    var email by remember { mutableStateOf("") }
    var password  by remember { mutableStateOf("") }
    Form(attrs = Modifier
        .fillMaxWidth(80.percent)
        .position(Position.Absolute)
        .top(50.percent)
        .left(50.percent)
        .transform {
            translate(-50.percent,-50.percent)
        }.toAttrs()
    ) {
        Column(
            Modifier.fillMaxWidth().columnGap(8.px)
        ) {
            Input(InputType.Text, attrs = InputModifier.toAttrs {
                placeholder("Email")
                onInput { e -> email = e.value }
            })
            Input(InputType.Text, attrs = InputModifier.toAttrs {
                placeholder("Password")
                onInput { e -> password= e.value }
            })
            Div(attrs = {
                this.classes("h-captcha")
                this.attr("data-sitekey","10000000-ffff-ffff-ffff-000000000001")
                this.attr("data-callback","onSubmit")
            })
            Button(onClick = {},
                ButtonStyle.toModifier()) {
                SpanText("Login")
            }
        }
    }
}
@OptIn(ExperimentalJsExport::class)
@JsExport
fun onSubmit(token: String) {
    console.log("Your Token is $token")
}
```

```gradle
head.add {
                script {
                    async = true
                    defer = true
                    src = "https://js.hcaptcha.com/1/api.js"
                }
            }
```

---

### Interactive buttons / color-scheme browser support

**Problem:** How to use `color-scheme` for interactive buttons?

**Solution:** `color-scheme: light-only` has limited browser support: https://developer.mozilla.org/en-US/docs/Web/CSS/color-scheme#browser_compatibility. `color-scheme: dark-only` is more widely supported.

---

### Silk dark mode configuration

**Problem:** Why is `color-scheme: light-only` less supported than `dark-only`, and how does Silk handle dark mode?

**Solution:** Silk has its own dark mode system independent of `color-scheme`. Configure initial setup: https://github.com/bitspittle/bitspittle.dev/blob/8637f2af8787aa96ee7f471ad6dd66f79aa68d82/site/src/jsMain/kotlin/dev/bitspittle/site/AppStyles.kt#L36

---

### Hover-to-scale transition with thenIf

**Problem:** How to implement a hover effect that scales a box using CSS transitions and conditional modifiers?

**Solution:** Use `thenIf(condition, modifier)` with `transition(CSSTransition("scale", 1.s))`:

```kotlin
Column(Modifier.gap(1.cssRem)) {
    var showSecondBox by remember { mutableStateOf(false) }
    Box(Modifier.size(200.px).backgroundColor(Colors.Red)
        .onMouseOver { showSecondBox = true }.onMouseOut { showSecondBox = false },
        contentAlignment = Alignment.Center) {
        Text("Hover me!")
    }

    Box(
        Modifier.size(200.px).backgroundColor(Colors.Blue).transition(CSSTransition("scale", 1.s))
            .scale(0).thenIf(showSecondBox, Modifier.scale(1))
    )
}
```

---

### Hover style on specific nested element

**Problem:** How to add a hover style to a specific element inside a nested view?

**Solution:** Use `window.fetch` (stdlib with Kobweb extensions) or `window.http` for HTTP requests: `window.http.get("https://www.google.com/search?q=varabyte+kobweb")`.

---

### Adding Lucide icons to Kobweb

**Problem:** How to add Lucide or other third-party icon libraries to Kobweb?

**Solution:** Reference existing icon implementations:
- Font Awesome: https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-icons-fa
- Material Design: https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-icons-mdi

Follow the same pattern for new icon sets.

---

### CSS complexity of container types

**Problem:** General frustration with CSS container complexity.

**Solution:** CSS container types became complex because they try to be catch-all. Kobweb's `Row` / `Column` / `Box` provide clearer separation of layout concerns.

---

### ComponentStyle with breakpoints, hover, active, disabled

**Problem:** How to handle a `ComponentStyle` that includes breakpoints, hover, active, and disabled states?

**Solution:** Use Kobweb's component style system with state selectors. Users can also use Tailwind alongside Silk or leverage third-party component library code.

---

### Tailwind vs Kobweb ComponentStyles

**Problem:** Tailwind classes seem less readable and reusable compared to Kobweb's `ComponentStyle`.

**Solution:** Kobweb provides routing and multiplatform Kotlin code reuse even without Silk. ComponentStyles offer better readability and reusability than Tailwind utility classes.

---

### Classes vs styleModifier for icons

**Problem:** Is `classes` a `styleModifier`?

**Solution:** No, `classes` is not a `styleModifier`. Use `fontSize` modifier directly or use `rememberBreakpoint` to pass different sizes into icon composables.

---

### Debug console on Android browser

**Problem:** How to open developer tools console on Chrome for Android?

**Solution:** https://stackoverflow.com/questions/37256331/is-it-possible-to-open-developer-tools-console-in-chrome-on-android-phone

---

### backgroundImage URL path

**Problem:** What URL path to use for `backgroundImage` in Kobweb?

**Solution:** If the image is under `jsMain/resources/public`, use `url("/image.png")`.

---

### Making elements focusable / unfocusable

**Problem:** How to make any component focusable or unfocusable in Kobweb?

**Solution:** Use the `tabindex` HTML attribute: https://stackoverflow.com/questions/716235/how-to-make-a-div-unfocusable

---

### Prevent content overflow

**Problem:** How to prevent content from going outside a container?

**Solution:** Use `Modifier.overflow(Overflow.Auto)` or `Modifier.styleModifier { property("overflow", "hidden") }`. Reference: https://developer.mozilla.org/en-US/docs/Web/CSS/overflow

---

### Debugging CSS animations in Modifier

**Problem:** How to debug CSS animations applied via `Modifier`?

**Solution:** Set the animation duration to an absurdly long value (e.g., 10 seconds) to slow it down and inspect the animation behavior.

---

### ComponentStyle filter chain with colorMode and prefix

**Problem:** Adding `backgroundColor` to a `ComponentStyle` with filter modifiers does not produce the expected result.

**Solution:**
1. `colorMode` can be queried inside the style via property access
2. `prefix` is only needed for library styles, not application code

```kt
val ImageTintStyle by ComponentStyle(prefix = "silk") {
    base {
        Modifier
            .filter(invert(86.percent))
            .filter(sepia(15.percent))
            .filter(saturate(306.percent))
            .filter(hueRotate(156.deg))
            .filter(brightness(122.percent))
            .filter(contrast(102.percent))
    }
}
```

```css
.silk-image-tint {
  filter: contrast(102%);
}
```

---

### getColorMode vs rememberColorMode

**Problem:** `rememberColorMode` is confusing because it returns a state object, not the current value.

**Solution:** Use `getColorMode()` (no `remember` needed) to get the current color mode value directly. `rememberColorMode` should have been named `rememberColorModeState`.

---

### Filter causing image to move to top stacking layer

**Problem:** Applying a CSS filter to a background image causes it to render on top of content in a Box overlay.

**Solution:** CSS filters create a new stacking context: https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context. Apply `Isolation.Isolate` on sibling elements to control layering order.

---

### Filter background with isolation to control stacking

**Problem:** Background with grayscale filter renders on top of content instead of behind it.

**Solution:** Apply `Modifier.isolation(Isolation.Isolate)` to the content `Column` to create a new stacking context, keeping the content above the filtered background:

```kt
Box(
                modifier = Modifier
                    .width(100.vw).height(100.vh),
                contentAlignment = Alignment.Center
            ) {
                PatternBackground (
                    Modifier.styleModifier { filter {
                        grayscale(50.percent)
                    } }
                        .fillMaxSize()
                        .objectFit(ObjectFit.Cover)
                )
                Column(
                    modifier = Modifier.isolation(Isolation.Isolate)
                ) {
                    DesktopContent(
                        stateVsEvent = StateVsEvent(field) { field = it },
                        onSearch = {
                            // ctx.router.navigateTo(PageRoutes.FORM_DETAILS)
                            ctx.router.navigateTo("/search/jobs/$field")
                        }
                    )
                }
            }
```

---
### White background on mobile due to viewport overflow

**Problem:** On mobile, the site has a strange white background. The page is wider than 100% because of the reduced mobile dimensions. ```kt @Composable fun markdownLayout(content: @Composable () -> Unit) { Box( Modifier .fillMaxSize() .minHeight(100.percent) .gridTemplateRows { size(1.fr); size(minContent) } ) { Column(Modifier.fillMaxSize())...

**Solution:** The root cause is the page content exceeding 100% viewport width. On mobile with reduced dimensions, the `width(100.vh)` on the inner Column causes overflow. Use `width(100.percent)` instead of `width(100.vh)` for the inner Column, and ensure all child elements respect the viewport width. Add `overflow(Overflow.Hidden)` on the outer `Box` to contain any remaining overflow.

```kt
@Composable
fun markdownLayout(content: @Composable () -> Unit) {
  Box(
    Modifier
      .fillMaxSize()
      .minHeight(100.percent)
      .gridTemplateRows { size(1.fr); size(minContent) }
  ) {
    Column(Modifier.fillMaxSize()) {
      navHeader()

      Box(Modifier.fillMaxSize()) {
        Column(
          Modifier
            .fillMaxHeight()
            .width(100.vh)
            .maxWidth(90.percent)
            .justifySelf(JustifySelf.Center)
            .border(2.px, LineStyle.Solid, Color("#3b3b3b"))
            .borderRadius(40.px)
            .paddingInline(2.vh)
        ) {
          content()
        }
      }
    }
    Footer(Modifier.align(Alignment.Center).gridRow(2, 3))
  }
}
```

---

### Controlling `TextArea` size

**Problem:** Setting width on a `TextArea` via `attrsModifier` does not work. ``` TextArea(attrs = Modifier.attrsModifier { style { this@attrsModifier.width(1000.px) } }.toAttrs(), value = postBody.value) ```

**Solution:** Use the `attrs` parameter directly with a style block instead of chaining through `Modifier.attrsModifier`. The `attrs` lambda accepts an `AttrsBuilderContext` where `width(...)` applies the CSS property directly.

```
TextArea(attrs = Modifier.attrsModifier {
                style {
                    this@attrsModifier.width(1000.px)
                }
            }.toAttrs(), value = postBody.value)
```

---

### Applying gradient as background color

**Problem:** Attempting to apply a gradient background color fails. The CSS `background` property (which supports gradients) differs from `background-color`.

**Solution:** Kobweb exposes CSS properties but does not abstract away all CSS concepts. Use `Modifier.styleModifier` to set `background` as a CSS gradient string directly. For example: `.styleModifier { property("background", "linear-gradient(45deg, red, blue)") }`. This works because CSS `background` accepts gradients while `background-color` does not.

---

### `text-transform: uppercase` not available in Kobweb

**Problem:** `text-transform: uppercase` converts to `.textTransform(TextTransform.Uppercase)` which is not available in Kobweb's modifier API.

**Solution:** If a CSS property is missing from Kobweb's `Modifier` surface, use `.styleModifier { property("text-transform", "uppercase") }` to apply it directly. File an issue on the Kobweb repo to request native `Modifier` support for the missing property.

---

### Adding space between columns in `SimpleGrid`

**Problem:** Using padding on individual column divs to add spacing between columns in `SimpleGrid`. Looking for a cleaner approach.

**Solution:** Use `Modifier.gap` on the grid container instead of padding individual children. `gap` adds spacing between grid/ flex children without affecting the edges, which is the standard CSS approach.

---

### Clarifying `SimpleGrid` column structure

**Problem:** What's in the divs is not relevant.

**Solution:** The approach creates a 3-column div inside a `SimpleGrid`. Confirm the `SimpleGrid` configuration matches the expected column count.

---

### `Link` style precedence over custom `ComponentStyle`

**Problem:** The `Link` component's built-in style overrides the custom `ComponentStyle` applied via modifier.

**Solution:** Use browser dev tools to inspect which CSS rule has higher specificity. Combine breakpoint conditions with the link pseudo-selector: `(Breakpoint.MD + anyLink) { ... }` to increase specificity. Alternatively, use `Modifier.classes(...)` with a custom CSS class that overrides the link defaults via higher CSS specificity.

---

### Using `fillMaxWidth` for centering

**Problem:** Needs `fillMaxWidth` to enable centering.

**Solution:** Apply `fillMaxWidth()` on the outer container and use `horizontalAlignment = Alignment.CenterHorizontally` on the `Column`. Nested content will be centered within the column's width.

---

### Centering a container horizontally

**Problem:** A green container needs to be centered horizontally in the view.

**Solution:** Wrap content in `Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally)`.

```kotlin
Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally) {
   Column(...) {
   }
}
```

---

### Silk-specific `toModifier` and `SpanText` cause error

**Problem:** Using silk-specific `toModifier()` and `SpanText` composable causes a compilation error.

**Solution:** This indicates the Kobweb Gradle plugin is not applied, or only the `silk-foundation` dependency is included without the full Silk widgets module. Ensure `configAsKobwebApplication()` is in the build script and that `kobweb-silk-widgets` (or the appropriate Silk dependency) is declared.

---

### Rendering Markdown with third-party library in Kobweb

**Problem:** Rendering Markdown text using `md-block` library (a custom HTML tag). Using `TagElement` composable from Compose HTML. Wondering if special handling is needed for Kobweb integration.

**Solution:** No special Kobweb integration is required. Kobweb has `compose-html-ext`, a Kobweb-agnostic layer. Use `Modifier.toAttrs()` to adapt the element if needed. Third-party custom HTML elements work directly with `TagElement` in Compose HTML.

---

### Manipulating `cssRule` for a specific child element

**Problem:** How to use `cssRule` to modify only a specific `Box` inside a `Column` without affecting other children.

**Solution:** Tag the target element with an ID or class name using `Modifier.id(...)` or `Modifier.classes(...)`. Then use a CSS selector in `cssRule` that targets that specific element. The child must be a direct or indirect descendant for the selector to match.

---

### Finding the `visibility` CSS property

**Problem:** Looking for the `visibility` CSS property in Kobweb. Also, where to find all existing CSS properties available in Kobweb.

**Solution:** Read https://github.com/varabyte/kobweb#learning-css-through-kobweb for the mapping from CSS to Kobweb modifiers. For properties not exposed as modifiers, use `.styleModifier { property("visibility", "hidden") }`.

---

### Converting `CSSColorValue` to Kobweb `Color`

**Problem:** Need to convert a `CSSColorValue` object to a Kobweb `Color` object.

**Solution:** The `CSSColorValue` class is opaque with no public API to extract r, g, b components, so direct conversion is impossible. Either store the color as a Kobweb `Color` directly, or parse the CSS color string representation manually.

---

### `box-shadow` applied but not visible

**Problem:** `box-shadow` is applied via `Modifier.boxShadow()` and the CSS property shows in dev tools, but the shadow is not visible. ``` .boxShadow(offsetX = 5.px, offsetY = 1.px, blurRadius = 4.px, spreadRadius = 0.px, color = Color.rgba(46, 46, 70, 1f)) .styleModifier { property("-webkit-box-shadow", "5px 1px 4px 0px rgba(46,46,70,1)") property("-moz-box-shadow", "5px 1px 4px 0px rgba(46,46,70,1)") } ```

**Solution:** Use browser dev tools to discover working values first, then translate to Kobweb. The issue is typically z-index stacking context (a parent clipping or another element overlapping) or the element having no explicit size. Check the element's `position` and `z-index`.

---

### Implementing adaptive layout

**Problem:** How to implement adaptive (mobile vs desktop) layout in Kobweb.

**Solution:** Use Kobweb's `Breakpoint` system in `ComponentStyle` blocks. Define different styles per breakpoint: `base { ... }`, `Breakpoint.MD { ... }`, `Breakpoint.LG { ... }`. Reference the `rememberBreakpoint()` composable for dynamic breakpoint values at runtime.

---

### Typo in ComponentStyle documentation

**Problem:** The README shows a typo: `ComponenetStyle` instead of `ComponentStyle`.

**Solution:** Documentation fixes are expected; the actual API uses the correct spelling. The code example is valid:
```kt
val MyDropDownStyle by ComponentStyle {
//...
}
BSDropdown(modifier = MyDropDownStyle.toModifier()
```

---

### Tailwind CSS IntelliJ suggestions in Kotlin files

**Problem:** Tailwind CSS classes work when applied via `Modifier.classes("...")` in Kobweb, but IntelliJ IDEA does not provide autocomplete suggestions in `.kt` files.

**Solution:** IntelliJ Ultimate has Tailwind CSS support, but it may not work in Kotlin files. Generate Kotlin extension functions from Tailwind classes for type-safe access:

```kt
fun Modifier.w24() = this.classes("w-24")
fun Modifier.textLg() = this.classes("text-lg")
...
```

```kt
sealed interface TailwindClass

fun Modifier.classNames(vararg classes: TailwindClass) = this.classNames(*classes.unsafeCast<Array<String>>())

inline val w24 get() = "w-24".unsafeCast<TailwindClass>()
inline val textLg get() = "text-lg".unsafeCast<TailwindClass>()
```

---

### `ColorMode.current` not persisting after being set

**Problem:** `ColorMode.current` does not work (state not updating) unless directly set on the page. ```kt var colorMode = ColorMode.current colorMode = ColorMode.LIGHT```

**Solution:** Verify the initial color mode by printing `ColorMode.current.name` after setting. The issue may be that the color mode change is happening outside a composable context or before the Silk theme is initialized. Set the initial color mode in `@InitSilk` or inside a `LaunchedEffect`.

```kt
var colorMode = ColorMode.current
    colorMode = ColorMode.LIGHT
```

---

### Bootstrap CSS overriding Kobweb styles due to layer priority

**Problem:** Bootstrap's CSS layers are more aggressive than Kobweb's styles, causing Bootstrap to override custom styles.

**Solution:** Import Bootstrap styles into a lower-priority CSS layer: https://github.com/varabyte/kobweb?tab=readme-ov-file#importing-third-party-styles-into-layers

---

### Was `ComponentStyle` removed?

**Problem:** Concerned that `ComponentStyle` was removed from the API.

**Solution:** `ComponentStyle` was the most significant breaking change in the latest version. There are no plans for similarly large changes before 1.0. Check the migration guide in the Kobweb release notes.

---

### Passing data from Markdown files to layouts

**Problem:** With the new layout system, how to pass frontmatter data from Markdown files to `@Layout` composables.

**Solution:** Markdown frontmatter values are accessible via the `MarkdownContext` in the layout. Use `@Layout` annotation on composables that receive `MarkdownContext`.

---

### `attrsModifier` scoping with `Modifier.toAttrs`

**Problem:** The `this` scope inside `attrsModifier` does not match expectations when used with `Modifier.toAttrs`.

**Solution:** For `attrsModifier`, manually cast the `this` scope: `attrsModifier { (this as AttrsScope<*>).attr(...) }`. Alternatively, use `Modifier.toAttrs { ... }` which provides the correct `AttrsScope` directly.

---

### Creating responsive websites with Silk without raw divs

**Problem:** Can an entire responsive website be built using only Silk's `Row`, `Column`, and `Box` composables, resorting to low-level Compose HTML elements only for specific cases?

**Solution:** Yes, generally. Occasionally a raw HTML element is better for CSS-specific cases, but most layouts can be sculpted with high-level Compose concepts. Use `Row`/`Column` for flex layouts, `Box` for stacking, and `SimpleGrid` for grid layouts. Fall back to `Div`, `Span`, `Input` for HTML-specific elements.

---

### Routing

### SEO for single-page apps without SSR

**Problem:** How SEO works for crawlers that don't execute JavaScript. How dynamic pages provide content for indexing.

**Solution:** Kobweb currently generates static HTML snapshots during export for SEO. Full hydration (as done by Next.js) is not currently possible with Compose HTML's architecture. For now, `kobweb export` generates static HTML that search engines can read. This may change as Compose HTML evolves.

---

### Heroku deployment file system watch error

**Problem:** Heroku logs show: `Unable to list file systems to check whether they can be watched. Reason: could not open mount file`. This occurs during export.

**Solution:** Use export mode that doesn't require file system watching: `kobweb export --mode dumb && kobweb run --env prod --mode dumb`.

```
> kobweb export --mode dumb
> kobweb run --env prod --mode dumb
```

---

### `verticalAlignment` in Row with images

**Problem:** `verticalAlignment` parameter in `Row` doesn't work as expected when the row only contains an image.

**Solution:** A `Row` won't be taller than its tallest child (the image). If the `Row` has no explicit height, `verticalAlignment` has no effect because the row and its content have the same height. Set an explicit `height` on the `Row` or use a container with a defined height.

---

### Dockerfile confusion with Kobweb dependencies

**Problem:** Confusion about which `Dockerfile` to use for Kobweb (referencing `kobweb-site/blob/main/Dockerfile`).

**Solution:** The Kobweb site Dockerfile demonstrates a full build pipeline. The issue described (feature A registering feature B which persists after feature A stops) sounds like a dependency/resource management problem in the server code, not a Kobweb issue.

---

### Automatic SPA redirect for GitHub Pages

**Problem:** Automating the SPA redirect script injection for GitHub Pages deployment.

**Solution:** Use the `kobweb { index { head.add { ... } } }` block in `build.gradle.kts` to inject a script that handles SPA redirects for gh-pages.

```css
kobweb {
    index {
        description.set("Powered by Kobweb")
        head.add {
            script(type="text/javascript"){
                consumer.onTagContent(
                    """
                    |
                    |      (function(l) {
                    |        if (l.search[1] === '/' ) {
                    |          var decoded = l.search.slice(1).split('&').map(function(s) { 
                    |            return s.replace(/~and~/g, '&')
                    |          }).join('?');
                    |          window.history.replaceState(null, null,
                    |              l.pathname.slice(0, -1) + decoded + l.hash
                    |          );
                    |        }
                    |      }(window.location))
                    |  """.trimMargin()
                )
            }
            comment("For gh-pages 404 redirects. Credit: https://github.com/rafgraph/spa-github-pages")
        }
    }
}
```

---

### Chrome used for DOM snapshot during static export

**Problem:** What happens during `kobweb export` in terms of browser interaction.

**Solution:** Kobweb uses a headless Chrome instance to load the compiled HTML + JS (a single-page app), visit each route, let the JS engine process the page, and save the resulting DOM as HTML snapshots. This enables static export of Compose-rendered pages.

---

### Documentation for Markdown cross-linking

**Problem:** Documentation for creating links between Markdown files in Kobweb could help new users.

**Solution:** Markdown cross-links work naturally with relative paths in Kobweb. The author plans to allow route customization in frontmatter YAML configuration.

---

### Purpose of `focus + active` combined style

**Problem:** What is the purpose of `focus + active`? Shouldn't `active` override `focus`?

**Solution:** `focus + active` targets the state when both pseudo-classes are simultaneously true (e.g., a button that is focused and being clicked). Previously backed by a `div`, the widget is now backed by an actual `<button>` element, making pseudo-class behavior more precise.

---

### `kobwebStart` fails when run from a submodule

**Problem:** Running `gradlew kobwebStart` from a non-top-level submodule fails to start the Kobweb server. Reproducer: https://github.com/DVDAndroid/kobweb-issue1

**Solution:** The error indicates the `.kobweb` folder cannot be found. Ensure the `.kobweb` directory exists in the project root (not the submodule). Change the working directory to the project root before running, or configure the submodule's working directory in the Gradle task.

---

### Multimodule project gives 404 for pages in non-main module

**Problem:** After converting to multimodule format, pages in a non-main module (without `.kobweb`) return 404 errors. The `META-INF` output is missing page metadata.

**Solution:** Pages must be declared in a module that has the Kobweb application plugin applied. The error `Skipped over @Page fun MyPage. It is defined under package othermodule.pages but must exist under mainmodule.pages` confirms the page package path must match the application module's package. Move pages or restructure module dependencies.

```
Exception in thread "DefaultDispatcher-worker-1" java.lang.NullPointerException: line must not be null
        at com.varabyte.kobweb.common.io.StreamUtilsKt$consumeAsync$1.invokeSuspend(StreamUtils.kt:24)
        at kotlin.coroutines.jvm.internal.BaseContinuationImpl.resumeWith(ContinuationImpl.kt:33)
        at kotlinx.coroutines.DispatchedTask.run(DispatchedTask.kt:106)                                                                                                             
        at kotlinx.coroutines.scheduling.CoroutineScheduler.runSafely(CoroutineScheduler.kt:571)                                                                                    
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.executeTask(CoroutineScheduler.kt:750)                                                                           
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.runWorker(CoroutineScheduler.kt:678)                                                                             
        at kotlinx.coroutines.scheduling.CoroutineScheduler$Worker.run(CoroutineScheduler.kt:665)
```

```
`Skipped over `@Page fun MyPage`. It is defined under package `othermodule.pages` but must exist under `mainmodule.pages`
```

---

### `max-lines` CSS property for text clamping

**Problem:** Looking for `max-lines` (CSS line clamping) support in Kobweb for truncating text to a specific number of lines.

**Solution:** `max-lines` is an experimental CSS property. In Kobweb, use CSS line-clamp via `styleModifier`: `.styleModifier { property("display", "-webkit-box") property("-webkit-line-clamp", "3") property("-webkit-box-orient", "vertical") property("overflow", "hidden") }`.

---

### Allowing `.html` suffix in URLs

**Problem:** Support for paths ending in `.html` in Kobweb routes. Ref: https://github.com/varabyte/kobweb/issues/153

**Solution:** Define a custom error page in `@InitKobweb` and use a route redirect to strip the `.html` suffix:

```kotlin
@Composable
private fun CustomErrorPage(errorCode: Int) {
   println(window.location.href)
   Div { Text("Error code: $errorCode") }
}

@InitKobweb
fun initKobweb(ctx: InitKobwebContext) {
  ctx.router.errorPage = CustomErrorPage
}
```

```
if (window.location.href.endsWith(".html")) {
  val ctx = rememberPageContext()
  ctx.navigateTo(window.location.href.removeSuffix(".html")
}
```

---

### Route interceptor API design feedback

**Problem:** Feedback on the route interceptor API. Suggesting `addRouteInterceptor { if (path == "entry.html") path = "entry" }` pattern instead of returning null for no changes.

**Solution:** The concern with mutable path modification is that query params and fragment order could be unintentionally modified by the interceptor. The current design keeps the path read-only to prevent this.

---

### Making `.html` links work with route interceptors

**Problem:** Making `.html` ending links work using a route interceptor. Reloading sends to an invalid path because the `.html` suffix is lost.

**Solution:** Use `routePrefix` configuration with a route interceptor that preserves the `.html` suffix. If `.html` endings are the convention, Kobweb should always append them. Without the suffix, browser reload causes a 404.

---

### Kobweb template for a data deletion request site

**Problem:** Building a data deletion request site template with Kobweb.

**Solution:** The structure is: one `jsMain` page with a message, account input field, and submit button; one `jvmMain` API route that receives the request, forwards it to the external server, and returns the result. The frontend calls the API via `window.api.post(...)`.

---

### WebSocket connection with `remember`

**Problem:** Using `val socket = remember { Websocket() }` followed by `socket.connect()` in a composable.

**Solution:** Use a sealed interface state machine pattern to manage WebSocket lifecycle:

```kotlin
sealed interface PageState {
  object Connecting : PageState,
  class Connected(websocket: Websocket) : PageState,
  ...
}

@Page
@Composable
fun somePage() {
   var state by remember { mutableStateOf(PageState.Connecting) }

   when (state) {
      is PageState.Connecting {
         LaunchedEffect(Unit) {
            val websocket = Websocket()
            websocket.connect()
            state = PageState.Connected(websocket)
         }
      }
      is PageState.Connected { ... }
   }
}
```

---

### Displaying `.mp4` video in Kobweb

**Problem:** Displaying an `file.mp4` video on a Kobweb page.

**Solution:** Place the `.mp4` file in `resources/public/videos/` (or `resources/public/images/`). Reference it with `src = "/videos/filename.mp4"` using the `Video` composable or via `attrsModifier` on a `<video>` element.

---

### Overriding server error/status pages

**Problem:** Override the default server error pages (404, 500) in Kobweb.

**Solution:** Call `Router#setErrorHandler` in an `@InitKobweb` method:

```kotlin
@InitKobweb
fun initKobweb(ctx: InitKobwebContext) {
  ctx.router.setErrorHandler { errorCode ->
   ...
}
```

---

### Dropdown menu in Kobweb

**Problem:** Implementing a dropdown (pulldown) menu in Kobweb.

**Solution:** A `Pulldown` element is planned for Silk in upcoming versions. For now, implement it manually using a `Box` with state toggling visibility of a child `Column`, styled as a dropdown menu.

**Note:** A pulldown element is planned to be added to silk in the next few versions.

---

### Changing the default page from `Index.kt`

**Problem:** The default page is always `Index.kt`. How to change it to a different page?

**Solution:** Two approaches: 1) Have `Index.kt` delegate to another page composable by calling it directly. 2) Use a route interceptor to redirect from root to the desired route: https://github.com/varabyte/kobweb/blob/fcda655ef9ba10512377c925ccde758517bf12ea/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/Router.kt#L204

---

### Callback pattern for sibling focus via `ref`

**Problem:** Using a callback to focus the next sibling input on Enter key press.

**Solution:** Use `ref` to capture the `HTMLElement`, then `nextSiblingOf<T>()` to find and focus the next element:

```kotlin
private inline fun <reified T> HTMLElement.nextSiblingOf(): T? {
    var next = nextSibling
    while (next != null) {
        if (next is T) {
            return next
        }
        next = next.nextSibling
    }
    return null
}

Input(
    InputType.Text,
    attrs = {
        lateinit var element: HTMLElement
        ref { element = it; onDispose { } }
        placeholder("Input #1")
        onKeyDown { event ->
            if (event.key == "Enter") {
                element.nextSiblingOf<HTMLInputElement>()?.focus()
            }
        }
    }
)
```

---

### Page shows blank/error on exception

**Problem:** Page goes blank or shows error when an exception occurs during rendering.

**Solution:** Check the browser console for errors. If an exception is thrown during composition, route to an error page using `ctx.router.navigateTo(...)` inside a `catch` block or use a `SideEffect` to monitor state.

---

### `cursor` property and button vs div vs span semantics

**Problem:** Confusion about cursor styling and the semantic choice between `<button>`, `<a>`, and `<span>`/`<div>` for interactive elements.

**Solution:** CSS cursor properties can be set via `Modifier.cursor(Cursor.Pointer)`. For semantic correctness: use `<button>` for actions, `<a>` for navigation, and `<span>`/`<div>` only when neither applies. Resources: https://karlgroves.com/links-are-not-buttons-neither-are-divs-and-spans/ and https://javascript.plainenglish.io/stop-using-divs-for-buttons-87a0b3d7945e

---

### Static sites and Markdown pages

**Problem:** Do static sites support Markdown pages?

**Solution:** Markdown support requires Kobweb server logic for processing and live reloading. Extracting this into a reusable library for static-only sites is non-trivial. Currently, Markdown pages work in full-stack mode but not in static-only export without additional setup.

---

### Wrapping React components with `ComponentPreview`

**Problem:** Making React components work in Kobweb without individual wrappers by using `ComponentPreview`.

**Solution:** Define a `CPreview` composable that wraps the React component via `useReactEffect`:

```kotlin
@Composable
fun CPreview(
    code: String = "",
    component: ChildrenBuilder.() -> Unit
) {
    useReactEffect {
        ComponentPreview {
            this.previewChild = {
                component(it)
            }
            this.code = code
        }
    }
}
```

---

### `redirect` vs `rewrite` for SPA fallback

**Problem:** Understanding `redirect` vs `rewrite` in web server config for Kobweb SPA fallback. A rewrite does not change the client-side URL, enabling Kobweb to serve the dynamic route from `index.html`.

**Solution:** Use a rewrite rule (not redirect) for all routes to `index.html`. This preserves the URL in the browser so Kobweb's client-side router can handle it. The error message mentioning Java 20 suggests a JDK version mismatch in the build environment.

---

### Registering and removing event listeners

**Problem:** Registering an event listener and removing it on dispose. In React, the cleanup is done in `useEffect`'s return callback.

**Solution:** Kobweb provides `EventListenerManager` for easier disposal of multiple listeners. Use `addEventListener` inside `attrsModifier` with an `onDispose` callback: `ref { onDispose { /* remove listener */ } }`.

---

### YAML frontmatter parsing for nested lists

**Problem:** Frontmatter YAML with nested lists does not convert to the expected Kotlin structure automatically.

**Solution:** Kobweb's frontmatter parser handles simple key-value pairs. For nested structures, import a YAML parser like `kaml` and parse the frontmatter values manually.

```kotlin
"radix" to listOf("link" to "https://www.radix-ui.com/docs/primitives/components/toast", "api" to "https://www.radix-ui.com/docs/primitives/components/toast#api-reference")
```

---

### Ktor auth plugin with Kobweb backend

**Problem:** Does the Ktor authentication plugin work with Kobweb's backend?

**Solution:** Yes, Kobweb's backend is built on Ktor, so Ktor auth plugins work. For frontend-only auth (email/password login), that is entirely doable on the client side.

---

### Images in non-root route pages

**Problem:** Pages at sub-routes (e.g., `services/web`) do not display images even though they are in the images directory.

**Solution:** Place images in `resources/public/` directory. Access them with absolute paths starting with `/` (e.g., `src = "/image.png"`). The path is relative to the server root, not the page route.

---

### Google Analytics script injection in `<head>`

**Problem:** Adding a Google Analytics script tag after the `<head>` element via `kobweb { index { head.add { ... } } }`.

**Solution:** The `head.add` block in `build.gradle.kts` supports adding `<script>` tags. The author plans to add body-level tag support. For now, inject analytics scripts via `head.add { script { ... } }` or add them directly to `index.html`.

---

### Requesting code for a solution

**Problem:** Can the solution code be posted?

**Solution:** Kobweb renders the page first as a static snapshot, then JavaScript runs and re-renders. This dual-phase rendering can cause issues with code that runs only once. Ensure initialization code handles both phases correctly.

---

### Filing a bug report with limited KSP knowledge

**Problem:** How to file a useful bug report when you have zero knowledge about KSP (Kotlin Symbol Processing)?

**Solution:** Provide a clear bug report with: the full error message, a link to project (or minimal reproducer), and step-by-step reproduction instructions. This is valuable even without deep knowledge of the underlying technology.

---

### Recomposition not happening after state change

**Problem:** A composable does not recompose when fields on `patient` change. ```kotlin var patient: Patient by remember { mutableStateOf(Patient()) } //Default constructor just inits everything with "N/A" LaunchedEffect(patientId) { patient = getPatientData(patientId) //API call where the actual values are downloaded } Column { Row {Text("Name: ${patient.name}")} Row {Text("Surname: ${patient.surname}")} Row {Text("Disease: ${patient.disease}")} } ```

**Solution:** Verify the API call is actually returning successfully by adding logging: `.also { println("Returning patient data: ${it.name}") }`. Check if `Patient` is a data class (correct `equals`/`hashCode`) or a regular class. If a regular class, Compose may not detect the change because it compares by reference. Use `copy()` to create a new instance, or make `Patient` a data class.

```kotlin
suspend fun getPatientData() {
   ... a bunch of stuff ...
   return patientData
     .also { println("Returning patient data: ${it.name}") }
}
```

---

### `CompletableDeferred` pattern for async initialization

**Problem:** Using `CompletableDeferred` to bridge callback-based APIs with coroutine-based code.

**Solution:** Use `CompletableDeferred` to convert callback results into awaitable coroutine values:

```kotlin
val patient = CompletableDeferred<Patient>()
somethingWithCallbacks
  .onSuccess {
      patient.complete(result)
  }
  .onFailure {
      patient.complete(Patient.from(patientUuid))
  }
return patient.await()
```

---

### Navigation state management between pages

**Problem:** Navigating between pages while preserving UI state. E.g., a dropdown open on the Home page should remain open when navigating back from Projects.

**Solution:** Use `LocalStorage` (persists across sessions) or `SessionStorage` (persists until tab close) via `window.localStorage` and `window.sessionStorage`. Save the UI state to storage before navigating, and restore it in `LaunchedEffect` when the page loads.

---

### Site showcase announcement

**Problem:** Finished site announcement and appreciation for the community help.

**Solution:** The site has been pinned in the showcase channel.

---

### Path parameters in Kobweb backend API

**Problem:** `ctx.req.params` returns query parameters but not path parameters (e.g., `/api/user/{id}`).

**Solution:** Path parameters are intentionally not supported on the backend because API routes are triggered by code only, not user-visible URLs. Frontend routes support path parameters for clean URLs. Use query parameters for backend API routes, or parse the path manually from `ctx.req.uri`.

---

### Timing of server startup/reload behavior

**Problem:** Clarifying whether a behavior occurs on initial server start, after code changes, or on page reload.

**Solution:** The described behavior is likely from a dev build. Run `kobweb export && kobweb run --env prod` to test production performance. If the issue persists, it may be a genuine Kobweb issue worth reporting.

---

### `navigateTo` and state management with sidebar

**Problem:** A `SideBar` composable receives an inner sidebar specific to each page. State resets on navigation.

**Solution:** Hoist sidebar state outside the page scope using a global state holder or `AppGlobals`. The navigation callback can trigger side effects like analytics pings: https://github.com/bitspittle/bitspittle.dev/blob/f4e419bf1c017551cc6c1ea7d95b9a1d157c157f/site/src/jsMain/kotlin/dev/bitspittle/site/components/layouts/PageLayout.kt#L56

---

### Case-sensitive route prefix

**Problem:** `routePrefix` setting is case-sensitive. `myexample.com/myroute` is not accessible at `myexample.com/MYROUTE`.

**Solution:** The router matches routes case-sensitively. Use the `Router` source as reference: https://github.com/varabyte/kobweb/blob/38c22650e36eedf103f09dfe7e43739f62a52c01/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/Router.kt#L288. Implement a custom route interceptor that lowercases paths for case-insensitive routing.

---

### Using TypeScript npm library in Kobweb

**Problem:** Using an npm library written in TypeScript requires manual type definitions in Kotlin/JS, even though TS already has types.

**Solution:** Automatic type bridging between Kotlin and TypeScript is not trivial. JetBrains experimented with automatic type generation but pulled back. Manual external declarations are required: declare `@JsModule` and `@JsNonModule` external interfaces for the API need.

---

### Error after deleting Kotlin files during development

**Problem:** After deleting unused Kotlin files, the index page shows `Error code: ...` instead of the page content.

**Solution:** The KSP processor may have gotten confused by the file deletion. Stop the server, run `./gradlew clean`, then restart. This clears the KSP cache and forces regeneration of metadata.

---

### Navigating to the root route with `RoutePrefix`

**Problem:** `ctx.router.navigateTo("/")` navigates to `https://xibalbam.github.io/` instead of `https://xibalbam.github.io/AJTextGameEngine/`.

**Solution:** Use `RoutePrefix.prepend("/")` to generate the correct prefixed route: https://github.com/varabyte/kobweb/blob/da9ea932d01d770cdd6cf2afce50edc9b615fde2/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/RoutePrefix.kt#L25

---

### Live reload not working with `kobwebStart` from root

**Problem:** `./gradlew :site:kobwebStart` does not live-reload. `../gradlew kobwebStart` from within the site folder does.

**Solution:** Live reload requires the Kobweb CLI to be run from the site directory. Use `cd site && ../gradlew kobwebStart` or configure the Gradle task to set the working directory to the site module.

---

### Demo/reference page for all Silk widgets

**Problem:** Is there a page that demos all available Silk widgets?

**Solution:** Clone the Kobweb repo and run the playground:

```bash
$ git clone https://github.com/varabyte/kobweb
$ cd kobweb/playground/site
$ kobweb run
```

---

### Reading a properties file in a static Kobweb site

**Problem:** Recommended way to read a `config.properties` file in a static Kobweb site.

**Solution:** Two approaches: 1) Use `window.fetch("/config.properties").await().text()` and parse manually. 2) If the data does not need to be dynamic or editable by non-developers, use `AppGlobals` to store configuration values at compile time.

---

### Dynamic Open Graph meta tags for social media previews

**Problem:** Displaying a dynamic image thumbnail in Open Graph meta tags when sharing links on social media.

**Solution:** Open Graph meta tags are typically static per page, not personalized per user. If dynamic OG tags are needed, set them via JavaScript in `LaunchedEffect`:

```kotlin
// Update Open Graph meta tags
            document.title = "${companyState.name} - Zakazivanje termina"

            val setMeta = { name: String, content: String ->
                val metaTag = document.head?.querySelector("meta[property='$name']")
                    ?: document.createElement("meta").apply {
                        setAttribute("property", name)
                        document.head?.appendChild(this)
                    }
                metaTag.setAttribute("content", content)
            }

            setMeta("og:title", "${companyState.name} - Zakazivanje termina")
            setMeta("og:description", "Zakazivanje termina za ${companyState.name}")
            setMeta("og:image", companyState.thumbnail)
            setMeta("og:url", window.location.href)
            setMeta("og:type", "website")
            setMeta("og:site_name", "Zakazivanje")
            setMeta("twitter:title", "${companyState.name} - Zakazivanje termina")
            setMeta("twitter:description", "Zakazivanje termina za ${companyState.name}")
            setMeta("twitter:image", companyState.thumbnail)
```

---

### Customizing Open Graph thumbnail per page content

**Problem:** The link preview takes the first image from the page, but needs to be customized based on thumbnail data from the database.

**Solution:** Set OG meta tags dynamically using `document.head` manipulation as shown above. For dynamic OG tags that work when the link is shared (before JS runs), use Kobweb's page-specific metadata: https://kobweb.varabyte.com/docs/concepts/foundation/page-metadata#page-specific-metadata

---

### Broadcasting events via API Streams for resource locking

**Problem:** Implementing a resource locking mechanism using API Streams. The `broadcast` and `sendTo` methods are only available on `LimitedStream`, which is short-lived.

**Solution:** Use the `data` user map on the stream context to store a managing object. Stream ids are stable even though stream instances are short-lived. Save the id and use it to reference the stream across invocations.

---

### Removing generated Markdown `About.kt` to free up route

**Problem:** The generated `About.kt` from a Markdown file blocks using the `/about` route for a custom page.

**Solution:** Delete the `about.md` file that came with the template. The Markdown-to-page generation only processes existing `.md` files.

---

### Dynamic routing for blog posts from a database

**Problem:** Adding dynamic routing so that blog posts stored in MongoDB (served by a backend) have canonical links like `/en/writings/{id}`.

**Solution:** Use Kobweb's dynamic route syntax `@Page(routeOverride = "/en/writings/{id}")` and access the path parameter via `rememberPageContext().route.params["id"]`. The site must be in full-stack mode (not static) for dynamic routes.

---

### `fetch` cache error with Kobweb's `window.fetchBytes`

**Problem:** `TypeError: Failed to execute 'fetch' on 'Window': Failed to read the 'cache' property from 'RequestInit': The provided value 'null' is not a valid enum value of type RequestCache.` when using `window.fetchBytes`.

```kt
window.fetchBytes(
    HttpMethod.POST,
    resource = "https://bla",
    headers = mapOf("Content-Type" to "application/json"),
    body = """
          {
            "form": "demo",
            "email": "$email",
            "content": "User request a demo",
          }
    """.trimIndent().encodeToByteArray(),
    redirect = FetchDefaults.Redirect,
    abortController = null
)
```

**Solution:** The issue is in Kobweb's `Fetch.kt` where `cache` may be set to `null` instead of a valid `RequestCache` enum value. Workaround: use `window.fetch` directly with explicit `RequestInit` parameters.

---

### `fetch` issues related to Kobweb's code

**Problem:** The fetch error originates from Kobweb's code in `Fetch.kt`. Using `window.fetch` directly with explicit `undefined` params works. https://github.com/varabyte/kobweb/blob/174e2597e6c6f12ffa8c3fc44cf5f2b8d42c0d96/frontend/browser-ext/src/jsMain/kotlin/com/varabyte/kobweb/browser/http/Fetch.kt#L115

**Solution:** Work around the Kobweb fetch utility by using `window.fetch` with explicit `RequestInit`:

```kt
window.fetch(
    input = "https://", init = RequestInit(
        method = "POST",
        headers = Headers().apply {
            append("Content-Type", "application/json")
        },
        body = """
        {
          "form": "demo",
          "email": "$email",
          "content": "User request a demo",
        }
    """,
        mode = RequestMode.CORS,
        cache = RequestCache.NO_CACHE,
        credentials = RequestCredentials.SAME_ORIGIN,
        redirect = RequestRedirect.MANUAL,
        referrerPolicy = "origin",
        keepalive = true,
    )
).await()
```

---

### Server

### Getting Kobweb running and contributing

**Problem:** Successfully got Kobweb running and considering contributions.

**Solution:** The Ktor-based server code is the heart of Kobweb. The refactored codebase shares logic across dev server, production server, and static server modes. Review the server module in the Kobweb repo for contribution opportunities.

---

### Moving Kobweb config into `build.gradle.kts`

**Problem:** Moving Kobweb configuration into `build.gradle.kts` for better IDE integration.

**Solution:** Config in `build.gradle.kts` is recommended: `kobweb { site { title = "Buddy" } server { port = 8080 } }`. This approach allows using any build tool while Kobweb remains agnostic.

---

### Static export deployment

**Problem:** Exported static site (`.html` + JS files) deployment. How to serve the files?

**Solution:** After `kobweb export --layout static`, upload the contents of `.kobweb/site` (or `build/distribution`) to any static hosting provider (Firebase, Vercel, Netlify, etc.). Configure SPA fallback to serve `index.html` for all routes if client-side routing is used.

---

### Docker deployment for Kobweb

**Problem:** Learning Docker properly for Kobweb deployment, already fighting with Ktor API deployment.

**Solution:** Kobweb can be deployed via Docker using the standard Ktor Dockerfile pattern with a multi-stage build: build the site, export, then serve with the Kobweb server in production mode. The Apache/NGINX config on the host may need adjustment for reverse proxying.

---

### ETag caching for Kobweb JS files

**Problem:** Browser caching of `kobweb.js` via ETags. The reference to the file cannot be cached.

**Solution:** Kobweb's server supports ETag-based caching. The request flow: first request sends ETag, browser caches, second request sends `If-None-Match`, server returns 304, third request (after update) gets new ETag and new content.

```
First time: browser requests kobweb.js

Server replies: Here you go, ETag: 1234567

Second time: browser requests Kobweb.js, saying it already has copy 1234567

Server replies: you already got it yo

Third time: browser requests Kobweb.js, saying it already has copy 1234567

Server replies: here's a new copy 9876543
```

---

### Using only Kobweb's UI features without the full framework

**Problem:** Interested in Kobweb's UI features (Compose HTML wrappers) but not the routing/server features.

**Solution:** Kobweb can be used modularly. Include only `kobweb-compose` and `kobweb-silk` for UI components without the full application plugin. The README provides guidance on selective feature inclusion.

---

### Running `jvmMain` code without `kobwebGenApi` task

**Problem:** Running code in `jvmMain` module triggers the `kobwebGenApi` task.

**Solution:** If want JVM code that is not consumed by the Kobweb server, place it outside the Kobweb application module. The Kobweb application plugin treats `jvmMain` as the server module and runs code generation on it.

---

### Disabling Kobweb plugin code generation

**Problem:** Adding a `fun main()` in `jvmMain` runs but also generates `ApisFactoryImpl`. Trying to avoid that generation.

**Solution:** The author may add a setting to disable the Kobweb plugin's code generation. For now, put standalone JVM code in a separate module not marked with `configAsKobwebApplication`.

```kotlin
fun main() {
    println("Hello, world!")
}
```

---

### Built-in `IntersectionObserver` support

**Problem:** Is there built-in `IntersectionObserver` support in Kobweb?

**Solution:** Not yet. Filed as issue https://github.com/varabyte/kobweb/issues/223.

---

### Canvas rendering support

**Problem:** Is it possible to add support for Canvas rendering in Kobweb?

**Solution:** Kobweb supports Canvas2D via the Compose HTML Canvas element. Reference: https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D

---

### `includeServer` configuration

**Problem:** What does `includeServer = true/false` do?

**Solution:** Set `includeServer = false` if the JVM/server module is not needed. This skips API code generation and server integration.

---

### Java 17+ compatibility issue

**Problem:** Error occurs when Gradle JVM is set to Java 17 or higher.

**Solution:** This is a known compatibility issue. The author will investigate. For Android projects, newer Android Studio bundles JDK 17, which may cause similar issues.

---

### Multimodule Kobweb project with WebSocket module

**Problem:** Creating a multimodule Kobweb project with WebSocket files in a separate module.

**Solution:** Serialization versions must be consistent across modules. Use a version catalog (`libs.versions.toml`) to keep dependency versions synchronized across all modules.

---

### Data class with `@SerializedName` for JSON parsing

**Problem:** Defining a data class with `@SerializedName` annotations for custom JSON field mapping.

**Solution:** Use `kotlinx.serialization` with `@SerializedName` for custom JSON field mapping. Example payload structure:

```
data class Payload(
    @SerializedName("t")
    val t: String? = null,
    @SerializedName("s")
    val s: Int? = null,
    @SerializedName("op")
    val op: OpCodes? = null,
    @SerializedName("d")
    val d: JsonObject? = null
)
```

---

### Kobweb plugin capabilities

**Problem:** What else does the Kobweb Gradle plugin do?

**Solution:** Issue #252 tracks plugin capabilities. The plugin handles page registration, style registration, API code generation, and Markdown processing.

---

### Custom headers on API calls

**Problem:** How to use custom headers on API calls in Kobweb.

**Solution:** Reference the `ApiFetcher` source for the header API: https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/browser/ApiFetcher.kt#L64

---

### Using `@InitApi` to initialize MongoDB

**Problem:** Should `@InitApi` be used to initialize MongoDB? Currently doing it via Koin and injecting UseCases directly into `@Api` function arguments.

**Solution:** `@InitApi` runs once at server startup and is appropriate for MongoDB initialization. Koin is also a valid approach.

---

### `@JsExport` function requirements

**Problem:** Does the function meet the requirements for `@JsExport`? Ref: https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/-js-export/

**Solution:** Verify the function signature matches `@JsExport` constraints (must be top-level, return/parameter types must be valid JS types). Reference: https://github.com/bitspittle/bitspittle.dev/blob/8637f2af8787aa96ee7f471ad6dd66f79aa68d82/site/src/jsMain/kotlin/dev/bitspittle/site/components/layouts/BlogLayout.kt#L41

---

### Gradle dependency configuration

**Problem:** Configuring Ktor client dependencies in `jvmMain`.

**Solution:** Declare Ktor dependencies in `jvmMain`'s `dependencies` block:

```kotlin
val jvmMain by getting {
            dependencies {
                implementation(libs.kobweb.api)
                val ktorVersion = "2.2.2"
                implementation("io.ktor:ktor-client-core:$ktorVersion")
                implementation("io.ktor:ktor-client-json:$ktorVersion")
                implementation("io.ktor:ktor-client-serialization:$ktorVersion")
                implementation("io.ktor:ktor-client-cio:$ktorVersion")
            }
        }
```

---

### Direct API calls from JS frontend

**Problem:** Can direct API calls be made from the JS frontend without a Kobweb backend?

**Solution:** Yes, using `window.fetch` or `window.http` to call external APIs. For form submissions that send emails, implement rate limiting or CAPTCHA on the backend to prevent spam.

---

### Kobweb directory structure clarification

**Problem:** Clarification about the `.kobweb` directory being created manually.

**Solution:** The `.kobweb` directory is created by the Kobweb CLI. It can be placed in any location. The server guidelines discourage spamming the community.

---

### JVM file generation when not using server

**Problem:** Does Kobweb always generate JVM files even when not using the server?

**Solution:** Code generation only happens for a `jvmMain` target inside a Kobweb application module with `includeServer = true`. If have a non-server JVM target (e.g., `vertxMain`), it will not be affected.

---

### Understanding Kobweb's JVM module role

**Problem:** What exactly does not work? The rule is that `jvmMain` inside a Kobweb application module is treated as the server only.

**Solution:** Non-server JVM code (standalone apps, libraries) should be in a separate module outside the Kobweb application module. The Kobweb Application Plugin treats the tagged module as a full-stack unit.

---

### Removing Kobweb API to use custom JVM main

**Problem:** Using plain `jvmMain` and removing `kobweb-api` dependency to avoid code generation.

**Solution:** The Kobweb Application Plugin treats the module as a full-stack unit. To avoid this, either set `includeServer = false` or move the JVM code to a separate module.

---

### API endpoint returning 404

**Problem:** Function located at `jvmMain/kotlin/my/project/name/api/Echo.kt` returns 404 at `localhost:8080/api/echo?message=hello`.

**Solution:** Verify the build script has `configAsKobwebApplication(includeServer = true)` set. Without this, API routes are not registered.

---

### Static vs dynamic site terminology

**Problem:** Clarifying static vs dynamic site terminology. A portfolio site calls a weather API via Ktor using `LaunchedEffect` - is it still a static site?

**Solution:** Kobweb uses "full stack" (not "dynamic") to describe sites with both frontend and backend logic. A static site exports pre-rendered HTML. If the weather API call happens client-side via `LaunchedEffect`, the exported site is still static - the API call runs in the browser after load.

---

### `kotlinx.serialization` compatibility

**Problem:** Is `kotlinx.serialization` compatible with Kobweb?

**Solution:** Yes. Create the Todo example (`kobweb create examples/todo`) and check its build script for the required serialization plugin configuration.

---

### Ktorfit (Ktor client generator) compatibility

**Problem:** Ktorfit causes `Task with name 'kspKotlinJs' not found` error.

```none
A problem occurred configuring project ':site'.
> Could not create task ':site:kobwebExport'.
> Task with name 'kspKotlinJs' not found in project ':site'.
```

**Solution:** Ktorfit uses KSP, which may conflict with Kobweb's KSP usage. Check both plugin versions for compatibility.

---

### Ktor JS client capturing custom headers

**Problem:** Using Ktor JS client to capture a custom response header. The header appears in Postman but not in the Ktor client.

**Solution:** Verify whether the server is Kobweb or external. If using Kobweb's server, ensure the header is set in the API route response. JS cross-origin restrictions may also hide non-standard headers unless the server includes `Access-Control-Expose-Headers`.

---

### Bundled resources in JS target

**Problem:** JS does not have bundled resources - they must be fetched from a server.

**Solution:** For `get` requests to APIs in the JVM module, use `window.api.get(...)` or `window.http.get(...)`. Resources like JSON files are served from the server's public directory.

---

### Debugging with `println` in `jvmMain`

**Problem:** `println` in `jsMain` can be seen in the browser console. How to check variables from `jvmMain`?

**Solution:** API routes receive a `ctx` parameter with a logger. Use `ctx.logger.info { "value = $value" }`. Logs are written to `.kobweb/server/logs/kobweb-server.log`.

---

### Button calling backend endpoint

**Problem:** Button that calls a backend endpoint using `window.http.post`.

**Solution:** Use `coroutineScope.launch` in the button's `onClick` handler. On the server side, use `ctx.logger` in the API route for debugging.

```
Button(onClick = {
    coroutineScope.launch {
        playing = true
        window.http.post(
            "http://localhost:8080/api/myGame/createGame",
            body = JsonSerializer
                .encodeToString(GameStart(playerOne = "Me", playerTwo = "You"))
                .encodeToByteArray()
        )
    }

}, colorScheme = ColorSchemes.Blue) {
    Text("Start a new Game!")
}
```

---

### Returning non-String primitives from API calls

**Problem:** Using `window.api` suspend function to retrieve a `Double` from MongoDB. Strings work but primitives do not.

**Solution:** Check the serialization format on the server side. `kotlinx.serialization` may wrap primitives in JSON differently than strings. Ensure the API route returns `Double` encoded as a number in JSON, and the frontend deserializes it correctly.

---

### Intermittent exception with retry logic

**Problem:** An odd exception that does not always occur. Repeating the call usually succeeds.

**Solution:** Use retry logic with `CompletableDeferred` and exponential backoff:

```kotlin
var retryCount = 3
val patientDeferred = CompletableDeferred<Patient>()
while (retryCount > 0 && !patient.isCompleted) {
   try {
       patientDeferred.complete(fetchPatientFromServer())
   } catch (ex: WeirdException) {
       --retryCount
       if (retryCount == 0) {
           patientDeferred.completeExceptionally(ex)
       }
   }
}

return patientDeferred.await()
```

---

### Server-side rendering support

**Problem:** How to provide Server-Side Rendering (SSR) in Kobweb?

**Solution:** Kobweb / Compose HTML does not support SSR at this time. For SSR needs, check out https://github.com/kwebio/kweb-core.

---

### Dropdown component

**Problem:** Looking for a dropdown component like a `<select>` element.

**Solution:** Use an `<a>` tag with `preventDefault` to create a custom dropdown. Reference: https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault. A pulldown widget is planned for Silk.

---

### Mono-repo with Compose Multiplatform + Ktor + Kobweb

**Problem:** Building a mono-repo with Compose Multiplatform, Ktor server, and Kobweb. Potential Gradle conflicts.

**Solution:** See https://github.com/varabyte/kobweb#adding-kobweb-to-an-existing-project for guidance on multi-project setups. Kobweb can be added as a module alongside existing modules.

---

### API concept per-method vs per-file

**Problem:** Discussion about changing the API concept to be per-method instead of per-file.

**Solution:** The author is open to supporting this change. Currently, API endpoints are defined per file (one API route per file). Per-method routing would allow multiple endpoints in one file.

---

### Drawing arbitrary lines (Canvas API)

**Problem:** Is there a good API for drawing arbitrary lines programmatically, like Canvas in vanilla JS?

**Solution:** Use Canvas2D support. Generate a clock example: `kobweb create examples/clock`. This demonstrates programmatic drawing with Canvas.

---

### Logging in the API module

**Problem:** How to do logging and print statements within the API portion of Kobweb for debugging failures.

**Solution:** Read https://github.com/varabyte/kobweb?tab=readme-ov-file#kobweb-server-logs. Use `ctx.logger.info { ... }` in API routes.

---

### `ApiStream` documentation

**Problem:** Looking for documentation on what `ApiStream` does.

**Solution:** https://kobweb.varabyte.com/docs/concepts/server/fullstack#define-api-streams

---

### Dynamic loading of Markdown files

**Problem:** Is there dynamic loading of Markdown files at runtime?

**Solution:** Markdown is processed at compile time into Kotlin pages. Runtime loading would require binding JS libraries into Kotlin, which is possible but depends on the library API complexity.

---

### Server components vs client components (Next.js concept)

**Problem:** Does Kobweb have a concept like Next.js's server component vs client component for protecting secrets?

**Solution:** Kobweb uses Kotlin project structure conventions: `jsMain` (frontend bundle), `jvmMain` (backend), and build scripts (compile-time). Secrets belong in `jvmMain` only, exposed via API endpoints. This is more explicit than Next.js's mixing of server and client code.

---

### Deployment

### Static site export with Docker

**Problem:** Developing with Kobweb (web-compose + Silk) and noticing the Todo example export does not generate UI correctly.

**Solution:** The site has a Docker file that downloads Kobweb, clones a repo, and runs export. Use Docker for CI/CD. The Todo export issue should be logged as a bug with repro steps.

---

### Kobweb project roadmap

**Problem:** What are the plans for Kobweb's direction - closer to Jetpack Compose or a different flavor?

**Solution:** Kobweb targets the non-canvas use case for Compose HTML. Canvas rendering (e.g., Compose for Web's Canvas) is a different approach that Kobweb does not focus on. Kobweb is for DOM-based web applications.

---

### Deploying to Heroku

**Problem:** How to deploy a Kobweb app to Heroku?

**Solution:** Try it. Report walls or successes to the author, who plans to document deployment strategies beyond Google Cloud.

### Does dynamic routing work for static sites

**Problem:** Does dynamic routing work for static sites?

**Solution:** That said do need to document it these thoughts down (but at least there's a bug for that https://github.com/varabyte/kobweb/issues/134)

---

### Trying to run cli on github actions, but regardless of mac/linux, it seems "stuck". Any...

**Problem:** Running `kobweb` CLI on GitHub Actions gets stuck regardless of macOS or Linux runner. The command hangs during static site export.

**Solution:** GitHub Actions runners may not have sufficient resources or TTY for the Kobweb CLI. Use the Gradle wrapper directly instead:
```bash
./gradlew kobwebExport -PkobwebLayout=STATIC
```
Or use the `--notty` flag for headless environments:
```bash
kobweb export --layout static --notty
```
For CI/CD, consider building locally and committing the output to a deploy branch, following the pattern described in https://bitspittle.dev/blog/2022/staticdeploy. The Gradle daemon may need additional memory in constrained CI environments — add `org.gradle.jvmargs=-Xmx2g` to `gradle.properties`.

---

### Don't know if you know, but you can use a github actions workflow like this to...

**Problem:** It is. Don't know if you know, but you can use a github actions workflow like this to automatically generate the static files on every commit

**Solution:** Review Kobweb documentation and SKILLS.md for relevant APIs.

---

### I don t understand what is a problem

**Problem:** In non static everything work excellent, I don’t understand what is a problem

**Solution:** That's very strange. This has nothing to do with Kobweb, it should all be JS

---

### Static build instruction: same name is important

**Problem:** During static build, the naming of output files must match expected paths. If the generated file name differs from what the HTML references, the site breaks.

**Solution:** When using static export (`kobweb export --layout static`), ensure the output file names match the references in the generated HTML. The `conf.yaml` project name determines the output JS bundle name. If manually changing file names after export, update all references in the HTML files accordingly. Reference `SKILLS.md` Deployment section for export workflow.

---

### Can the kobweb generated js file be used with the async attribute

**Problem:** Can the kobweb generated js file be used with the async attribute?

**Solution:** Also it's a static file so 'd have to manually update it each time

---

### To export the website as static we use 'kobweb export --layout static', what is gradle task...

**Problem:** To export the website as static we use 'kobweb export --layout static', what is gradle task equivalent to this?

**Solution:** Export is the worst one. One sec as LoP has a workflow that uses it

---

### Can anyone spot out anything wrong with this

**Problem:** Adding variables to a Gradle task but the variables are not being applied. ```kt tasks.register("run-all") { group = "application" val site = project(":site") val export = site.tasks.findByName("kobwebExport")!! export.inputs.prop...

**Solution:** Here's how read them on end: ```kotlin val env = project.findProperty("kobwebEnv") val runLayout = project.findProperty("kobwebRunLayout") ... etc ... ```

```kt
tasks.register("run-all") {
    group = "application"
    val site = project(":site")
    val export = site.tasks.findByName("kobwebExport")!!
    export.inputs.property("kobwebReuseServer", false)
    export.inputs.property("kobwebEnv", "DEV")
    export.inputs.property("kobwebRunLayout", "KOBWEB")
    export.inputs.property("kobwebBuildTarget", "RELEASE")
    export.inputs.property("kobwebExportLayout", "STATIC")
    dependsOn(export)

    val stop = site.tasks.findByName("kobwebStop")
    dependsOn(stop)
}
```

```kotlin
val env = project.findProperty("kobwebEnv")
val runLayout = project.findProperty("kobwebRunLayout")
... etc ...
```

---

### Kobweb task variables via `export.extra[]`

**Problem:** Setting Kobweb task variables via `export.extra[]` ``` export.extra["kobwebReuseServer"] = false export.extra["kobwebEnv"] = "DEV" export.extra["kobwebRunLayout"] = "KOBWEB" export.extra["kobwebBuildTarget"] = "RELEASE" export.extra["kobwebExportLayout"] = "STATIC" ```

**Solution:** Use `export.extra[]` pattern for setting Kobweb Gradle task properties in build scripts.

```
export.extra["kobwebReuseServer"] = false
    export.extra["kobwebEnv"] = "DEV"
    export.extra["kobwebRunLayout"] = "KOBWEB"
    export.extra["kobwebBuildTarget"] = "RELEASE"
    export.extra["kobwebExportLayout"] = "STATIC"
```

---

### Kobweb `export --layout static` limitations and alternatives

**Problem:** Concerns about `kobweb export --layout static` quality and alternative export approaches.

**Solution:** This relates to Kobweb template authoring; can be safely ignored for standard projects.

---

### Which Gradle command produces the production JS script during export

**Problem:** When running `kobweb export --layout static`, what gradle command(s) produce the prod script that's used? I thought it'd be `jsBrowserProductionWebpack` but that gets me a slightly different script

**Solution:** Reference: https://github.com/varabyte/kobweb-cli/blob/897edd9ec5d688089e61e49171090fafc3bdd681/kobweb/src/main/kotlin/com/varabyte/kobweb/cli/common/GradleUtils.kt#L126

---

### Using JS libraries with Kobweb Composable code

**Problem:** Using JavaScript libraries (like SweetAlert2) from Kobweb Composables requires wrapping JS calls.

**Solution:** Use `js(""" ... """.trimIndent())` for cleaner JS interop in Kotlin/JS Composables.

```
LaunchedEffect(Unit) {
        (js("Swal.fire({\n" +
            "  title: 'Are you sure?',\n" +
            "  text: \"You won't be able to revert this!\",\n" +
            "  icon: 'warning',\n" +
            "  showCancelButton: true,\n" +
            "  confirmButtonColor: '#3085d6',\n" +
            "  cancelButtonColor: '#d33',\n" +
            "  confirmButtonText: 'Yes, delete it!'\n" +
            "});") as Promise<*>).then {
            println("Promise Result: $it")
        }.catch {
            println("Promise Error")
        }
    }
```

---

### `localhost` refuses connection

**Problem:** This site can’t be reached localhost refused to connect.

**Solution:** Check if the dev server is running and listening on the correct port. The issue resolved without changes.

---

### `kobwebExport` takes 15+ minutes

**Problem:** `kobwebExport` has been running for about 15 minutes now. Is that normal?

**Solution:** It's downloading and caching a browser.. Depending on people's locations it might get stuck

---

### Is there any specific requirements to host kobweb website or is it possible to host in any...

**Problem:** Is there any specific requirements to host kobweb website or is it possible to host in any platform?

**Solution:** Did happen to see blog posts? <https://bitspittle.dev/blog/2022/staticdeploy> and <https://bitspittle.dev/blog/2023/clouddeploy>

---

### Same-origin API calls with Kobweb on static hosting

**Problem:** In a Next.js project, an API route was added on the server side and called from the client to maintain same-origin requests. With Kobweb static export, there is no server to host the API route, so the site cannot be deployed to static hosts like Vercel.

**Solution:** Kobweb static export (`--layout static`) generates only client-side files. API routes require a server. Options:
1. Use **fullstack layout** (`kobweb export --layout FULLSTACK`) — deploys a Ktor server alongside static files
2. Use an **external API service** (Firebase, Supabase, etc.) and call it from the client with proper CORS configuration
3. Deploy the static site to a platform that supports server-side functions (Vercel, Netlify) and implement API calls as serverless functions
For CORS issues on the Render hosting platform, ensure the server's CORS headers allow requests from the Kobweb site's origin.

---

### Can i do this if site is hosted in a subdomain?

**Problem:** Can i do this if site is hosted in a subdomain?

**Solution:** Start with a small Kobweb project to learn the basics before worrying about future scalability.


---

### Jul 16 09:20:23 PM Dockerfile:46 Jul 16 09:20:23 PM -------------------- Jul 16 09:20:23 PM...

**Problem:** Jul 16 09:20:23 PM Dockerfile:46 Jul 16 09:20:23 PM -------------------- Jul 16 09:20:23 PM 44 | echo "org.gradle.jvmargs=-Xmx256m" >> ~/.gradle/gradle.properties Jul 16 09:20:23 PM 45 | Jul 16 09:20:23 PM 46 | >>> RUN kobweb export --notty Jul 16 09:20:23 PM 47 | Jul 16 09:20:23 PM 48 | #-------...

**Solution:** <https://github.com/bitspittle/kobweb-todo-on-render/blob/a5244fa568b0f358fb670e996235441e77b54ffc/Dockerfile#L5> should probably be set to "site" for most project

---

### Compiling the Kobweb Markdown plugin without cloning the full repo

**Problem:** Compiling the Kobweb Markdown plugin without cloning and compiling the entire Kobweb repo. A specific functionality is needed in the Markdown plugin.

**Solution:** Static export with a custom server to serve the files is a valid approach.

---

### Host my portfolio site on Linode. Should I copy paste my whole project on L

**Problem:** I just executed `kobweb export` command. I want to host my portfolio site on Linode. Should I copy paste my whole project on Linode and then use `kobweb run --env prod`? Or is there any specific folder which is generated which I can upload on Linode and run with prod environment?

**Solution:** The way do it for own site is do an export locally and then run `firebase deploy` which finds the Kobweb directory and uploads it.

---

### After that I explored directory inside folder. There is one directory called which ha

**Problem:** I just executed `kobweb export --layout static` command. After that I explored `build` directory inside `site` folder. There is one directory called `distribution` which has one folder `public` and one file `portfolio.js` Should I upload these on the server if I want to host my site? If yes, when...

**Solution:** Can change the folder to a different location if need to, but if all needed was the files, sounds like 're good to go

---

### > .kobweb/conf.yaml?

**Problem:** > `kobweb.conf` .kobweb/conf.yaml?

**Solution:** Didn't mention `kobweb export` anywhere in there before now. Long overdue, even if this first pass might be a bit shoe-horned in.

---

### What is the purpose of this task exactly

**Problem:** What is the purpose of this task exactly ?

**Solution:** In a pinch can just run `kobweb export --layout static` yourself, files will be in .kobweb/site when 're done

---

### How can I access Kobweb from my phone ? My PC is connected with Wifi to my box, and my...

**Problem:** How can I access Kobweb from my phone ? My PC is connected with Wifi to my box, and my phone also, but when I go to `localhost:8080` I get an `ERR_CONNECTION_REFUSED` error

**Solution:** Have to set up a proxy. don't think can access localhost directly from another machine (but don't quote on that)

---

### Kobweb and Cloudflare edge functions

**Problem:** Distributing a website on Cloudflare. Kobweb does not generate edge functions like some JS frameworks.

**Solution:** Kobweb's static export does not generate edge functions. Create convention plugins to offload shared build logic for Cloudflare deployment.

---

### Build

### Apologies for the sharp corners in advance

**Problem:** One is general modularity, and in my case you can't use the compose and kotlinx serialization gradle plugins in the same module

**Solution:** Thanks for tinkering and for this initial feedback. Apologies for the sharp corners in advance!

---

### How to verify the Kobweb backend is running

**Problem:** After starting the Kobweb backend, how to verify it is running and accepting requests.

**Solution:** They use a gradle feature where they import another project from disk as if it were a dependency

---

### How would I version the kobweb generated js file

**Problem:** How would I version the kobweb generated js file?

**Solution:** Once 've done that, read it like so: https://github.com/bitspittle/morple/blob/40d9484c07178a235dd4d64db0675403fd4b02e8/site/src/jsMain/kotlin/dev/bitspittle/morple/components/sections/Header.kt#L54

```kotlin
link(
                rel = "stylesheet",
                href = "/checkboxes.css?v=${Random.nextInt()}"
            )
```

---

### The only difference is that I make it a job I can cancel and I use attrsModifier...

**Problem:** The only difference is that I make it a job I can cancel and I use attrsModifier instead of asAttributesBuilder

**Solution:** E.g. if do something like ``` val scope = ... blah { scope.launch { ... } } ``` and that works, but later add: ``` val scope = ... blah { anotherblah { scope.launch { ... } } } ``` and `anotherblah` itself has its own "scope" field, will still compile but behavior might change unexpectedly

```kotlin
val scope = ...

blah {
  scope.launch { ... }
}
```

```kotlin
val scope = ...

blah {
  anotherblah {
    scope.launch { ... }
  }
}
```

---

### Simple data fetching with `kotlinx.serialization`

**Problem:** Simple data fetching with `kotlinx.serialization` and `window.fetch`: ```kt @Serializable data class Product( val id: Int, val title: String, val price: Double, val description: String, val category: String, val image: String, val rating: Rating, ) @Serializable data class Rating( val rate: Double, val count: Int, ) suspend fun fetc...

**Solution:** Add the Compose compiler plugin configuration in the build script: ```kotlin import org.jetbrains.kotlin.gradle.dsl.KotlinCompile .... compose { kotlinCompilerPlugin.set("1.4.0") } project.tasks.withType<KotlinCompile<*>>().configureEach { kotlinOptions.apply { freeCompilerArgs = freeCompilerArgs + listOf("-P", "plugin:androidx.compose.compiler.plugins.kotlin:suppressKotlinVersionCompatibilityCheck=1.8.10") } } ```

```kt
@Serializable
data class Product(
    val id: Int,
    val title: String,
    val price: Double,
    val description: String,
    val category: String,
    val image: String,
    val rating: Rating,
)

@Serializable
data class Rating(
    val rate: Double,
    val count: Int,
)

suspend fun fetchApi(): List<Product> {
    val res = window.fetch("https://fakestoreapi.com/products").await().text().await()
    return Json.decodeFromString(res)
}
```

```kotlin
import org.jetbrains.kotlin.gradle.dsl.KotlinCompile
....

compose {
    kotlinCompilerPlugin.set("1.4.0")
}

project.tasks.withType<KotlinCompile<*>>().configureEach {
    kotlinOptions.apply {
        freeCompilerArgs = freeCompilerArgs + listOf("-P", "plugin:androidx.compose.compiler.plugins.kotlin:suppressKotlinVersionCompatibilityCheck=1.8.10")
    }
}
```

---

### Kobweb stressing Gradle / Kotlin daemon

**Problem:** ``` ./gradlew clean && ./gradlew --stop ```

**Solution:** Kobweb's live reloading experience stresses Gradle and the Kotlin compiler daemon.

```
./gradlew clean && ./gradlew --stop
```

---

### Unknown issue

**Problem:** Here is the simplified version:

**Solution:** ``` .toAttrs { onInput { value = it.value } ``` onInput is not resolving for for some reason

```kotlin
@Composable
fun TestView() {
    var backingElement: HTMLInputElement? by remember { mutableStateOf(null) }
    var value by remember { mutableStateOf("") }
    Box {
        TextInput(
            value = value,
            attrs = listOf(TextStyle, EditableTextViewStyle).toModifier()
                .toAttrs {
                    onInput { value = it.value }
                    type(inputType)
                    ref { element ->
                        backingElement = element
                        this.onDispose { backingElement = null }
                    }
                }
        )
        if (value.isNotEmpty()) {
            Surface(
                modifier = Modifier
                    .align(Alignment.CenterEnd)
                    .margin(right = 12.px)
                    .onClick {
                        backingElement?.focus()
                        value = ""
                        it.preventDefault()
                        it.stopPropagation()
                    }
            ) {
                MdiClear(style = IconStyle.OUTLINED)
            }
        }
    }
}

val EditableTextViewStyle = ComponentStyle("editable-text-view-style") {
    base {
        Modifier
            .padding(20.px)
            .height(60.px)
            .backgroundColor(colorMode.toSilkPalette().tooltip.background)
            .borderRadius(14.px)
            .boxSizing(BoxSizing.BorderBox)
            .border(
                width = 3.px,
                style = LineStyle.Solid,
                color = colorMode.toSilkPalette().tooltip.background
            )
            .outline(style = LineStyle.None)
            .transition(CSSTransition("width 0.4s ease-in-out"))
    }

    focus {
        Modifier
            .backgroundColor(colorMode.toSilkPalette().background)
            .border(
                width = 3.px,
                style = LineStyle.Solid,
                color = colorMode.toSilkPalette().color
            )
    }
}
```

---

### Koin import failing with no error message

**Problem:** Importing koin into a Kobweb project fails with no error message.

**Solution:** Ensure a Gradle sync is performed after adding the dependency.

---

### But in my case it really does nothing after loading the project and indexing stuff. As...

**Problem:** But in my case it really does nothing after loading the project and indexing stuff. As you can see in my gradle, it says "Nothing to show". Kinda weird right?. For a newbie like me I felt stuck for a moment haha maybe you can add this in ReadMe. If intelliJ doesn't load your project you can

**Solution:** It's a hard balance. Put everything into the README and no one can find anything not sure can answer general IntelliJ questions in there, that will be endless...

---

### How you open the project

**Problem:** How you open the project. If you open directly the settings.gradle.kts file, it figures out everything automatically. If you open via VCS, it asks you if you want to load it. If you open the folder, sometimes it finds it, sometimes not...

**Solution:** These days pretty much everything is an indeterminate spinner

---

### Or maybe there's a Gradle task for it

**Problem:** Or maybe there's a Gradle task for it? I don't remember

**Solution:** Reference: https://kotlinlang.org/docs/using-packages-from-npm.html seems to imply manual work

---

### I can generate material3 for kobweb

**Problem:** I can generate material3 for kobweb ?

**Solution:** In a magic world with fairies and dragons would just add an `npm` dependency and then just start calling into it with Kotlin

---

### If it consistently gets bad, as a temporary fix you can use gradlew kobwebStart -t and get...

**Problem:** If it consistently gets bad, as a temporary fix you can use gradlew kobwebStart -t and get mostly the same behavior. That fixed things for me when I was having similar issues on windows. Just make sure to run kobwebStop when you're done.

**Solution:** Investigate the performance issue.

---

### Where i can find guide for kobweb? We are still early days of Kobweb so guides are sparse

**Problem:** Where i can find guide for kobweb?

**Solution:** However, fairly quickly 'll see it's a Kotlin version of JavaScript / CSS, so usually if start searching how to do things using CSS, figuring out what to do in Kobweb is usually not too hard

---

### Adding favicon other than the default one

**Problem:** Does kobweb not support adding favicon other than the default one ?

**Solution:** Icon set here: <https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/gradle-plugins/application/src/main/kotlin/com/varabyte/kobweb/gradle/application/extensions/AppBlock.kt#L42>

---

### Sharing Gradle task configuration between Ktor and Kobweb modules

**Problem:** Sharing Gradle task configuration between a Ktor module and a Kobweb module.

**Solution:** The Gradle task configuration to share between Ktor and Kobweb modules includes:

```kotlin
java {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}

application {
    mainClass.set("com.jaq.aas.modules.MainAppModuleKt")
}

tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    kotlinOptions.jvmTarget = JavaVersion.VERSION_11.toString()
    kotlinOptions.javaParameters = true
    kotlinOptions.freeCompilerArgs += "-opt-in=org.mylibrary.OptInAnnotation"
}

tasks.withType<JavaCompile> {
    options.encoding = "UTF-8"
    options.compilerArgs.add("-parameters")
}

tasks.register<Copy>("copyFrontendBuild") {
    from(File(rootProject.rootDir.path, "webapp/build/kobweb/webapp").absolutePath)
    into(layout.projectDirectory.dir("build/resources/main/webapp"))
}

tasks.named("shadowJar") {
    dependsOn("copyFrontendBuild")
}

tasks.named("buildFatJar") {
    dependsOn("copyFrontendBuild")
}

tasks.named("clean") {
    doLast {
        layout.projectDirectory.dir("dist").asFile.deleteRecursively()
    }
}

tasks.register("packForLocal") {
    dependsOn("buildFatJar")
    doLast {
        copy {
            from(layout.projectDirectory.dir("build/libs/backend-all.jar"))
            into(layout.projectDirectory.dir("dist/${rootProject.name}"))
        }
        copy {
            from(
                rootProject.file("docker/prod/Dockerfile")
            )
            into(layout.projectDirectory.dir("dist/${rootProject.name}"))
        }
    }
}
```

---

### Build project Gradle configuration issue

**Problem:** Build project failure due to Gradle configuration.

**Solution:** Gradle settings must be set to project settings. Verify the Gradle configuration in `build.gradle.kts`.

---

### Maybe worth setting explicitly in

**Problem:** Maybe worth setting explicitly in `build.gradle.kts`? ``` tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> { compilerOptions.jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_11) } ```

**Solution:** Play around with Kobweb, see what think! Hope enjoy working with it

```
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    compilerOptions.jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_11)
}
```

---

### In your browser those tags will be empty

**Problem:** If you inspect the site in your browser those tags will be empty. not sure why

**Solution:** At export time, actually turn those invisible style tags visible with this code: <https://github.com/varabyte/kobweb/blob/fcda655ef9ba10512377c925ccde758517bf12ea/gradle-plugins/application/src/main/kotlin/com/varabyte/kobweb/gradle/application/tasks/KobwebExportTask.kt#L76>

---

### What version of kobweb are you using

**Problem:** What version of kobweb are you using?

**Solution:** If can't upgrade, can use the stdlib window.fetch if want but it kind of sucks.

---

### How are you able to generate toc from headings

**Problem:** How are you able to generate toc from headings ? you dont have them in markdown page in yaml section, you don't have set any gradle configuration for that im talking about this line <https://github.com/bitspittle/bitspittle.dev/blob/2f40c1f931e31e0f687547cb7bebdf2cb61eedc2/site/src/jsMain/k...

**Solution:** Reference: https://raw.githubusercontent.com/bitspittle/bitspittle.dev/2f40c1f931e31e0f687547cb7bebdf2cb61eedc2/site/src/jsMain/resources/markdown/blog/2022/KoverBadge.md

```kotlin
mdCtx.frontMatter["toc-min"]?.singleOrNull()?.toIntOrNull() ?: 2,
mdCtx.frontMatter["toc-max"]?.singleOrNull()?.toIntOrNull() ?: 3,
```

---

### Manually providing CompositionLocal

**Problem:** Manually providing a CompositionLocal may introduce new issues.

**Solution:** Check the Kobweb version being used.

---

### Can we do this too in kobweb

**Problem:** ```<!DOCTYPE html> <html lang="en-US"> <head> <meta charset="utf-8" /> <title>TensorFlow.js browser example</title> <!-- Load TensorFlow.js from a script tag --> <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script> </head> <body> <h1>TensorFlow.js example</h...

**Solution:** Spend some time trying to fix it

---

### Adding scripts via `head.add` and calling JS functions in pages

**Problem:** Adding a script in `head.add` and then calling `js("const model...")` in a page.

**Solution:** Add the script inside the `build.gradle.kts` via `head.add`, then run the JS logic inside a composable (e.g., `MyApp`).

---

### NPM dependencies not accessible via `js()` calls

**Problem:** NPM dependencies cannot be used by calling them inside `js()`. The `js()` function only works with packages available through CDN. NPM libraries require external declarations.

**Solution:** Open a browser console and experiment with JavaScript code in the dev tools to test functionality.

---

### Not registering keyframes definition , as only top-level definitions are supported at this time

**Problem:** Not registering keyframes definition `val ShiftRight`, as only top-level definitions are supported at this time. Although fixing this is recommended, you can manually register your keyframes inside an @InitSilk block instead (`ctx.stylesheet.registerKeyframes(ShiftRight)`). Suppress this message

**Solution:** It should be a top level global public var so the Kobweb gradle plugin can find it and register it

---

### What properties are available to configure for the meta tag in the build gradle

**Problem:** What properties are available to configure for the meta tag in the build gradle?

**Solution:** Just in case, would be in code like this (but not using meta): <https://github.com/bitspittle/bitspittle.dev/blob/8037bc0ad7ab182caf4b7d50d26e2b2d50685e43/site/build.gradle.kts#L36>

---

### 11 can have issues sometimes

**Problem:** 11 can have issues sometimes.

**Solution:** Try removing this block and try again: <https://github.com/varabyte/kobweb/blob/db534eff808f585542cae80b975dd21d4efedb07/buildSrc/build.gradle.kts#L32C1-L32C1>

---

### Is the issue when you run kobwebExport or when you build the project

**Problem:** Is the issue when you run kobwebExport or when you build the project?

**Solution:** My guess is Kobweb and <https://plugins.gradle.org/plugin/de.comahe.i18n4k> are interacting in a bad way

---

### Getting this error, need help

**Problem:** I saw https://github.com/varabyte/kobweb/#adding-kobweb-to-an-existing-project and added a site from kobweb to my KotlinMultiPlatform project. I'm getting this error, need help! ----- TestKmm3/build.gradle.kts' at line: 1 Plugin [id: 'com.android.application', version: '8.1.0', apply: false] not

**Solution:** > `org.gradle.api.plugins.UnknownPluginException: Plugin [id: 'com.android.application', version: '8.1.0', apply: false] not found in `one of the following sources:

---

### Did you modify the root gradle or settings file when adding the site module? It's possible...

**Problem:** Did you modify the root gradle or settings file when adding the site module?

**Solution:** It's possible that by becoming multi module are making gradle resolve a plugin in a place 're not expecting (e.g. the site module probably)

---

### Kobweb project creation stuck on Node

**Problem:** Creating a Kobweb project gets stuck on Node.

**Solution:** Run `./gradlew kobwebStart -t` instead of `kobweb run`. This provides a clearer error message.

---

### Curious like how can I achieve this the same using .js file? Let's say I created a .js file...

**Problem:** How can I achieve this the same using .js file? Let's say I created a .js file and pasted the same script. How can I call that function without making it .kt file? Ahh, this is so confusing sometimes..

**Solution:** Put it in public directory, include it as a <script> in build gradle kts file, and then if necessary create external bindings to it

---

### Playground uses symlinks for gradle wrapper which makes it impossible to run on windows or...

**Problem:** Playground uses symlinks for gradle wrapper which makes it impossible to run on windows or maybe i need a plugin for that ?

**Solution:** Sure, although symlinks are amazing and it's a tragedy they weren't adopted on Windows earlier! would highly recommend setting it up at some point, it's not too hard.

---

### Guys, can anyone help me with this

**Problem:** Can anyone help me with this. My website in mobile is loading like the desktop mode first and 1s later move all to the mobile version https://www.carlosgub.dev , I want to fix this

**Solution:** It wasn't something realized when first started working on Kobweb. It can still be useful but should probably document the method with what learned

---

### Build stuck after updating Kobweb to 0.20.0

**Problem:** After updating Kobweb to 0.20.0, the build is stuck. `kobweb run` and `kobweb export` also hang. Kotlin and Compose compiler versions match the latest sample for 0.20.0. Possibly caused by `kobweb-lib`.

**Solution:** If reproducible steps can be provided, this will be investigated as a high priority.

---

### Running , and it does recompile, but the browser doesn't reload Yes that should support...

**Problem:** Idk, I'm running `./gradlew :site:kobwebStart -PkobwebEnv=DEV --continuous`, and it does recompile, but the browser doesn't reload

**Solution:** Yes that should support live reloading.

---

### Unable to reproduce Gradle issue

**Problem:** I'll update you in a few days

**Solution:** It's all good. If can repro it here, that should be enough to dig into it. assume it's a issue and not a Gradle issue.

---

### KMP module structure causes unexpected sourceset requirements

**Problem:** Running into a weird problem I've not run into before. I'm adding Kobweb to the webApp module of the KindredCircl KMP and it's now looking for commonMain, commonTest, jsMain, and jsTest in the :site directory jsMain is obviously declared, and I can add jsTest, but the others aren't neces...

**Solution:** New Gradle sourceset issue

---

### Kobweb CLI fails but Gradle command works

**Problem:** If the gradle command works I'd expect the cli to work too. Do other kobweb commands work?

**Solution:** It's how CLI chooses gradle to run?

---

### Silk

### TextInput loses focus when clear button is clicked

**Problem:** TextInput loses focus when the clear button is clicked. How to override and set focus state manually?

**Solution:** Implement focus management in the button click handler. Reference: https://github.com/varabyte/kobweb/blob/a8910bf5168a3e27be88ab49fce8b0a86322caac/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L72

---

### Can FaIcons and Silk be removed if not used?

**Problem:** Does removing FaIcons, silk break things ? If I'm not using it.

**Solution:** Removing icons is easy, Silk may be harder as it contains a bunch of useful things

---

### Created an OTP field with silk or compose

**Problem:** Has anyone created an OTP field with silk or compose?

**Solution:** Review Kobweb documentation and SKILLS.md for relevant APIs.

---

### Setting a default variant for all Silk Button instances

**Problem:** Need to apply a variant to all Silk Button instances globally without adding the variant parameter to every Button call.

**Solution:** There is no built-in mechanism to set a default variant globally for Silk widgets. The Compose-idiomatic approach is to create a wrapping composable that applies the desired variant internally:
```kotlin
@Composable
fun ThemedButton(
    onClick: () -> Unit,
    content: @Composable () -> Unit
) {
    Button(onClick, variant = MyCustomVariant) { content() }
}
```
Use this wrapper throughout the codebase instead of importing Silk's `Button` directly. This pattern is consistent with Compose's composition-over-configuration philosophy.

---

### Markdown

### Have a markdown editor in my website, anyone ever implement something like that

**Problem:** Have a markdown editor in my website, anyone ever implement something like that?

**Solution:** It was not well documented, and think it was missing yaml extensions, but might want to look at it

---

### I am setting up highliter js for markdown code block

**Problem:** I am setting up highliter js for markdown code block. but getting hljs is not defined error. please let me know what i am mission.

**Solution:** FWIW do plan to move away from highlight.js at some point (want an engine that allows to highlight individual lines), but highlight.js is definitely not a bad solution.

---

### General

### How do i get png showing

**Problem:** An `Img("/logo.png")` composable shows nothing even though the PNG file is in resources.

**Solution:** (The reason for the "/public" prefix is so can put other things in the resource file that don't automatically get exposed to users, but when kobweb runs, it strips it)

---

### My cards are in svg format, can the canvas display svgs though

**Problem:** My cards are in svg format, can the canvas display svgs though?

**Solution:** And there were issues creating them but think that's since been fixed

---

### `kobweb create site` shows no wizard or error

**Problem:** Running `kobweb create site` shows no wizard and no error message.

**Solution:** Test the setup without Kobweb to isolate the issue.

---

### But Tobo asked for centering Row, did he

**Problem:** But Tobo asked for centering Row, did he? what in compose is Row( horizontalArrangement = Arrangement.Center )

**Solution:** Can use that as well, that's true

---

### Thanks for the beer, but whaaaaat does it mean

**Problem:** Thanks for the beer, but whaaaaat does it mean?

**Solution:** Looking at the Compose tutorial, this is what they do: ``` @Composable fun ArtistCard(artist: Artist) { Row(verticalAlignment = Alignment.CenterVertically) { Image(/*...*/) Column { Text(artist.name) Text(artist.lastSeenOnline) } } } ```

```kotlin
var features: [Landmark] {
        landmarks.filter { $0.isFeatured }
    }
```

```kotlin
@Composable
fun ArtistCard(artist: Artist) {
    Row(verticalAlignment = Alignment.CenterVertically) {
        Image(/*...*/)
        Column {
            Text(artist.name)
            Text(artist.lastSeenOnline)
        }
    }
}
```

---

### Putting the image in the pre-existing resources/public folder

**Problem:** Putting the image in the pre-existing resources/public folder?

**Solution:** Cabs not sure if saw but mid move, 're just unloading now

---

### But how's that relevant I found this

**Problem:** But how's that relevant

**Solution:** Found this. Not sure it's exactly right but it talks about the mechanism for how the browser knows if it should get a new copy of the file or not

```kotlin
script {
  src = "/fluense_web.js?v=${Random.nextInt()}"
  async = true
}
```

---

### Unknown issue

**Problem:** I didn't mean to start a riot. I can always just have something run over the generated html files to do that myself. loading the script in header vs body is what I'm invested in though

**Solution:** Can find the js file inside .kobweb/site folder and manually edit it before uploading to vps

---

### Like, the minimum we want to test is ````

**Problem:** 

**Solution:** Like, the minimum want to test is ``` addEventListener("touchstart") { scope.launch { delay(800L) }} ```

---

### Recomposition triggered without visible box changes

**Problem:** `SideEffect { console.log("Variant recomposed ${componentVariant.hashCode()}") }` triggers recomposition without the box changing.

**Solution:** The `equals` and `hashCode` methods on `ComponentVariant` are not being called during recomposition. Investigate the Compose compiler's behavior with the variant state tracking.

---

### Is the class immutable

**Problem:** Is the `ComponentVariant` class immutable?

**Solution:** ``` Stable -- Indicates a type that is mutable, but the Compose runtime will be notified if and when any public properties or method behavior would yield different results from a previous invocation. ``` Yeah definitely don't intend to do that

---

### But kobweb actually works in the subfolder with no changes If it works for you though...

**Problem:** But kobweb actually works in the subfolder with no changes

**Solution:** If it works for though that's great!

---

### More content of the file or something else

**Problem:** Is the overlaying text more content of the file or something else?

**Solution:** Investigate the issue.

---

### Rry to keep bothering you, but are image resources supposed to work from the non main...

**Problem:** Rry to keep bothering you, but are image resources supposed to work from the non main module?

**Solution:** Resources are supposed to work, yes. They should be in the public folder.

---

### Removing formatting when pasting into a contentEditable div

**Problem:** A Div with `contentEditable` enabled retains formatting when pasting content from an IDE. Only plain text is desired.

**Solution:** Use `onPaste { it.clipboardData?.clearData() }` to remove formatting on paste, or implement a custom paste handler that strips formatting.

```kotlin
onPaste { it.clipboardData?.clearData() }
```

---

### Is it possible to share components between two kobweb application modules directly, not...

**Problem:** Is it possible to share components between two kobweb application modules directly, not using a library module?

**Solution:** Kobweb libraries put metadata in jars read by Kobweb applications

---

### Can anyone tell me why my column items are vertically spaced

**Problem:** Why my column items are vertically spaced?

**Solution:** ``` Column { SpanText("hello") SpanText("hello") SpanText("hello") } ```

```
Column {
   SpanText("hello")
   SpanText("hello")
   SpanText("hello")
}
```

---

### Any reason why you're not using

**Problem:** Any reason why you're not using `<p>`? Semantically I believe it would make more sense?

**Solution:** Like, just want to use ``` Column { Button Text TextInput Whatever } ```

```
Column {
   Button
   Text
   TextInput
   Whatever
}
```

---

### Accessibility: choosing between `<a>` and `<button>` elements

**Problem:** Accessibility considerations in the choice between `<a>` and `<button>` elements.

**Solution:** Accessibility must be a primary consideration when choosing between `<a>` (navigation) and `<button>` (action) elements. Use `<a>` for links and `<button>` for actions per ARIA guidelines.

---

### Can we do firebase database integration

**Problem:** Can we do firebase database integration?

**Solution:** But yeah have a WIP web card game (on hold now since Kobweb seems to need more attention at the moment) that is successfully using firebase for auth, database, and very basic analytics

---

### How are you removing embeds

**Problem:** How are you removing embeds!!

**Solution:** See also: https://github.com/varabyte/kobweb/blob/main/COMPATIBILITY.md

---

### How can I "reload" my kobweb site when I update the code

**Problem:** Kobweb site does not reload automatically when code is updated. The auto-reload feature stopped working.

**Solution:** Kobweb's dev server (`kobweb run`) includes live reload. To ensure it triggers reliably:
1. Enable IntelliJ "Auto Save" (File → Settings → Appearance & Behavior → Auto Save → Save files on focus loss)
2. Run `kobweb run` from the `site/` directory (not the root project)
3. Alternatively use `./gradlew kobwebStart -t` from the `site/` directory for continuous Gradle watch mode
4. Verify the browser is pointing to `http://localhost:8080` and the WebSocket connection is active
If live reload still fails, check for Gradle daemon memory issues or try `./gradlew clean` followed by `./gradlew --stop`.

---

### Is there anything similar to runBlocking in multi-platform You can use or to get into...

**Problem:** Is there anything similar to runBlocking in multi-platform

**Solution:** Can use `LaunchedEffect` or `rememberCoroutineScope` to get into suspend fun code on JS

---

### Or ```` Sealed interface in my case because I don't have common data that I share across...

**Problem:** Or ``` class ConnectionHandler { val socket = Websocket().also { it.connect() } } val handler = remember { ConnectionHandler() } handler.socket... ```

**Solution:** Sealed interface in case because don't have common data that share across states but both are fine

```kotlin
class ConnectionHandler {
  val socket = Websocket().also { it.connect() }
}
...
val handler = remember { ConnectionHandler() }
handler.socket...
```

---

### State machines for managing `remember` variables

**Problem:** Understanding when to use state machines vs. simple state.

**Solution:** State machines are beneficial when many related `remember` variables are needed. Consider using one in `Popup` components.

---

### Do you have a updated list of different projects built using kobweb by different people

**Problem:** Do you have a updated list of different projects built using kobweb by different people?

**Solution:** It's still really early, and think it's OK for people to be nervous about that

---

### What is replacement for LazyLists in kobweb

**Problem:** What is replacement for LazyLists in kobweb?

**Solution:** It's just been a huge pile of higher priorities but 're getting there!

---

### Is there any widget available expanded that will fill available space in a row

**Problem:** Is there any widget available expanded that will fill available space in a row?

**Solution:** If 'd like to sync the project and take a look, might be able to point in the right direction

---

### Can i help in any way? It's exciting that new people keep using Kobweb in new ways

**Problem:** In any way?

**Solution:** Sorry 're working on the bleeding edge...

---

### Where can I find documentation or examples about forms and input data with compose or kobweb

**Problem:** Where can I find documentation or examples about forms and input data with compose or kobweb ?

**Solution:** Basically one form pass text in as an argument, the other the text value comes from the control, think?

---

### Display a svg from a file. For that, I used the object element that allow me to load the...

**Problem:** Display a svg from a file. For that, I used the object element that allow me to load the content of the svg file. But now I also want to acess the svg element and show it on the console for example. I tried this code but the output in the console is "null" or "NodeList[]". Does that...

**Solution:** Also used window.setTimeout (without a time argument) sometimes to postpone using a value for a frame

```kotlin
Object( attrs = {
    ref { objectElement ->

        console.log(objectElement.firstChild)
        console.log(objectElement.firstElementChild)
        console.log(objectElement.childNodes)

        onDispose {  }
    }
    attr("data", "complexShapes.svg")
    attr("type","image/svg+xml")
})
```

---

### How to write code for downloading a file when a button is clicked

**Problem:** How to write code for downloading a file when a button is clicked.

**Solution:** <https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/frontend/compose-html-ext/src/jsMain/kotlin/com/varabyte/kobweb/compose/file/FileUtils.kt#L26>

---

### Using rememberCoroutineScope for file downloads

**Problem:** How to convert a PDF file to a byte array and pass it to `document.saveToDisk()`.

**Solution:** Use `rememberCoroutineScope` to launch a coroutine for the download operation:

```kotlin
val scope = rememberCoroutineScope()

Button(onClick = {
  scope.launch {
    document.saveToDisk(
      "resume.pdf",
      window.http.get("/resume.pdf")
    )
  }
})
```

---

### How to make a request from the browser and avoid invalid origin error

**Problem:** How to make a request from the browser and avoid invalid origin error.

**Solution:** Browser requests are subject to CORS (Cross-Origin Resource Sharing) policies. The server must include appropriate CORS headers (`Access-Control-Allow-Origin`) in its response. For development, configure the server to allow the Kobweb dev server origin. For production, ensure the API server includes the correct CORS headers. Use `window.fetch()` or `window.http.get()` for making requests.

---

### Errors go away when making a slight change, even a comment

**Problem:** Errors resolve when making any change to the source file, even adding a comment.

**Solution:** This is a cache issue caused by Kobweb's aggressive live reload caching. Clear the browser cache or perform a hard refresh. Disable incremental compilation in `build.gradle.kts` if the issue persists. The live reload system caches compiled output aggressively to speed up development, which can sometimes serve stale state.

---

### Exception handling differences between Firefox and Chrome

**Problem:** Exception behavior differs between Firefox and Chrome browsers.

**Solution:** Firefox and Chrome use different rendering engines (Gecko vs Blink) and JavaScript runtimes, which can cause exceptions in one browser but not the other. Test on both browsers during development. When a crash occurs in only one browser, debug in that browser first as it likely reveals the root cause.

---

### Building a productivity tool (Pomodoro) in Kobweb

**Problem:** Building a Pomodoro productivity tool on a website. Browser should notify when timer ends with a sound and PC reminder, then start the break, and restart the Pomodoro when break ends.

**Solution:** Learn how to implement it in JavaScript first, then create a minimal Kobweb project to replicate the functionality.

---

### Is marquee available in Kobweb

**Problem:** Is marquee available in Kobweb.

**Solution:** Kobweb does not include a native marquee composable. Use Compose HTML's `MarqueeContainer` if available, or implement the scrolling effect with CSS animation using the `animation` property in `CssStyle`.

---

### Is the approach in issue #274 possible?

**Problem:** Is the approach described in <https://github.com/varabyte/kobweb/issues/274> feasible?

**Solution:** The approach described in that issue is not currently possible with Kobweb. Reply to the GitHub issue to confirm this limitation.

---

### Where is Kobweb documentation located?

**Problem:** Where is the Kobweb documentation located? Only the GitHub README has been used so far.

**Solution:** Kobweb documentation is hosted at https://kobweb.varabyte.com. Generate sample projects using `kobweb create examples/todo` or `kobweb create app` to see working configurations. Compare these samples with the project setup to identify configuration differences.

---

### What is the Kobweb equivalent of a text input

**Problem:** What is the Kobweb equivalent of a text input in Kobweb.

**Solution:** Use `TextInput` or `Input` composables from the `compose-html-ext` module. These provide text input functionality within Kobweb applications. Refer to the `kobweb create app` sample for usage examples.

---

### How to use externals for JavaScript interop in Kobweb

**Problem:** How to use Kotlin external declarations for JavaScript interop. Example: wrapping the EasyMDE library.

**Solution:** Define external declarations using the `@JsModule` annotation. The JavaScript code:

```
const easyMDE = new EasyMDE({element: document.getElementById('my-text-area')});
```

Translates to the following Kotlin external declaration:

```kotlin
@JsModule("easymde")
@JsNonModule
external fun EasyMDE: dynamic
```

Refer to existing Kotlin/JS interop examples like <https://github.com/bitspittle/firebase-kotlin-bindings> for reference patterns.

---

### How to execute code before a composable launches

**Problem:** How to execute code before a composable launches.

**Solution:** Use `LaunchedEffect` for side effects that run when the composable enters the composition. Use `remember` to initialize state or resources. Use `DisposableEffect` for setup/cleanup lifecycle. Example:

```kotlin
LaunchedEffect(Unit) {
    // runs once when composable enters composition
}
```

---

### How to get the browser language in Kobweb

**Problem:** How to get the browser language in Kobweb.

**Solution:** Access `window.navigator.language` which returns the browser's language setting as a BCP 47 language tag (e.g., "en-US"). In Kobweb, use `window.navigator.language` directly. This is a standard browser API and works the same way as in plain JavaScript.

---

### Creating a custom range input in Kobweb

**Problem:** How to create a custom range input in Kobweb.

**Solution:** Study established UI libraries like Chakra UI's Slider component (<https://chakra-ui.com/docs/components/slider>) for API design patterns. Use the browser element inspector to understand the HTML/CSS structure of range inputs. Implement the custom range input using Kobweb's composable primitives and CSS styling. The `RangeInput` composable provides a starting point that can be customized with CSS.

---

### Understanding browser-based execution in Kobweb

**Problem:** Is the HTTP GET request performed by JavaScript running in the browser?

**Solution:** Kobweb applications compile to JavaScript and execute in the browser. Network requests (including GET) are performed by the browser's JavaScript runtime via APIs like `fetch` or `XMLHttpRequest`. The application code is downloaded when a user visits the site, and all subsequent requests originate from the browser. This is the standard web application model — even though the application source is remote, execution is entirely client-side.

---

### Where to find a list of Kobweb composables

**Problem:** Where to find a list of composables implemented in Kobweb.

**Solution:** Kobweb provides two categories of widgets: general Compose HTML widgets (usable in any Compose HTML project) found in the `compose-html-ext` module, and Kobweb-specific widgets for framework interaction found in the `kobweb-compose` module. Browse the source directories of these modules for the complete list. The general widgets include buttons, inputs, layouts, and typography components.

---

### OnClick doesn't work

**Problem:** OnClick event does not fire.

**Solution:** Review existing code first. Run `kobweb create app` and check the button implementation in the generated project.

---

### Does Kobweb offer a feature to create a home screen?

**Problem:** Does Kobweb offer a feature to create a home screen?

**Solution:** Kobweb does not currently offer a home screen feature. If implemented, it would likely be as an extension in the `kobwebx` namespace rather than in the core library.

---

### Browser devtools for mobile preview in Kobweb

**Problem:** Browser developer tools are available on mobile devices for debugging Kobweb applications.

**Solution:** Mobile browsers (Chrome on Android, Safari on iOS) provide developer tools for debugging. Desktop browsers can also emulate mobile viewports using responsive design mode (Chrome DevTools toggle device toolbar, Firefox Responsive Design Mode). Use these tools to test responsive layouts without a physical device.

---

### Does examples/responsive still exist?

**Problem:** Does examples/responsive still exist?

**Solution:** The `examples/responsive` project was removed after the `app` template was improved to include responsive patterns. Use `kobweb create app` to get the current recommended project structure with responsive layout support.

---

### Showcase channel

**Problem:** Where to find Kobweb showcase projects.

**Solution:** Run `kobweb create` to see the list of available project templates and examples. Browse the Kobweb GitHub repository's examples directory for showcase projects.

---

### Best approach for table layout structure in Kobweb

**Problem:** How to structure a table layout with columns, rows, and boxes in Kobweb.

**Solution:** Use an HTML `<table>` element for tabular data layouts. Kobweb provides `Table`, `Tr`, `Td`, and `Th` composables that map directly to HTML table elements. For complex layouts, consider using CSS Grid or Flexbox via Kobweb's `CssGrid` and `FlexBox` composables.

---

### Add Visa payment to a Kobweb website

**Problem:** How to integrate Visa payment processing into a Kobweb website.

**Solution:** Payment processing requires integration with a payment gateway (e.g., Stripe, Square) that supports Visa. The payment flow involves:
1. A frontend form in Kobweb to collect payment details
2. A backend API to securely process payments using the gateway's SDK
3. Server-side handling of payment confirmation and error responses

Kobweb handles the frontend UI; the payment backend must be implemented separately using the gateway's API.

---

### How to change the website tab title

**Problem:** How to change the website name that appears on the browser tab.

**Solution:** Set the `<title>` tag in `index.html`. To override it dynamically from code:

```kotlin
LaunchedEffect(title) {
    document.title = title
}
```

This updates the browser tab title reactively whenever the `title` variable changes.

---

### Using the Anchor composable for navigation

**Problem:** How to use the Anchor composable for client-side navigation in Kobweb.

**Solution:** The `Anchor` composable provides client-side navigation without full page reloads:

```kotlin
Anchor("/") {
  Text("Home")
}
```

Use `Anchor` with a path string for internal navigation. The composable handles routing within the Kobweb application.

---

### Asciidoctor.js integration with Kobweb throws TypeError

**Problem:** Integrating Asciidoctor.js with Kobweb throws `TypeError: Cannot read properties of undefined (reading 'convert')` with the following declaration:

```kt
external object Asciidoctor {
    fun convert(input: String): String
}
```

Based on the Asciidoctor.js quick tour (<https://docs.asciidoctor.org/asciidoctor.js/latest/setup/quick-tour/>), the JavaScript code is:

```js
import asciidoctor from 'asciidoctor'
const Asciidoctor = asciidoctor()
const html = Asciidoctor.convert(content)
```

**Solution:** `Asciidoctor` in the Kotlin declaration should be an external class that is instantiated, not an external object. The JavaScript code calls `asciidoctor()` as a factory function, returning an instance. Declare it as:

```kt
@JsModule("asciidoctor")
@JsNonModule
external fun asciidoctor(): Asciidoctor

external class Asciidoctor {
    fun convert(input: String): String
}
```

The factory function call `asciidoctor()` returns a new `Asciidoctor` instance, which provides the `convert` method.

---

### Does Kobweb use type-safe styling?

**Problem:** Does Kobweb use type-safe styling?

**Solution:** Kobweb provides type-safe CSS styling through its `CssStyle` API. Styles are defined using Kotlin DSL with compile-time checking of CSS properties and values. This eliminates runtime style bugs and provides IDE autocompletion for CSS properties. Example:

```kotlin
val MyStyle = CssStyle {
    color(Color.red)
    fontSize(20.px)
    margin(0.px, 16.px)
}
```

---

### Adding Kobweb to a KMP project to share business logic code

**Problem:** How to add Kobweb to a Kotlin Multiplatform project to share business logic code.

**Solution:** Refer to https://kobweb.varabyte.com/docs/guides/existing-project for integrating Kobweb into an existing project. The main requirement is creating a `conf.yaml` configuration file, which can be copied from a generated Kobweb project for reference.

---

### State management in Kobweb for form-heavy applications

**Problem:** What is the best approach for managing state in Kobweb, especially in web applications with many forms.

**Solution:** Use Compose's built-in state management primitives: `remember`, `mutableStateOf`, and `rememberCoroutineScope`. For complex forms, organize state into data classes and use `remember` to preserve state across recompositions. Patterns include:
- `mutableStateListOf` for dynamic form lists
- `remember` with derived state for computed values
- Pass state and callbacks as parameters for shared state across components
- `CompositionLocal` for application-wide state

Kobweb does not include a ViewModel equivalent; Compose's state management replaces this pattern.

---

### Programmatically changing tabs in Kobweb

**Problem:** How to programmatically change tabs in Kobweb's Tab component.

**Solution:** Pass a state variable to the `TabsPanel` component as the selected tab index. Update the state variable to change tabs programmatically. If `TabsPanel` does not accept a selected index parameter, manage the tab state externally by tracking the selected tab index and conditionally rendering tab content based on the state value.

---

### Is Kobweb compatible with Kotlin 2.3.0?

**Problem:** Is Kobweb compatible with Kotlin 2.3.0?

**Solution:** Compatibility with Kotlin 2.3.0 is being worked on. A release is expected within a few days.

---
