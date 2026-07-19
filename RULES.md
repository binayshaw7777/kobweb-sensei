# Kobweb Rules & Gotchas

> Generated from #need_help Discord channel analysis.
> Only concrete, reusable rules extracted.

## Routing

- `Skipped over `@Page fun MyPage`. It is defined under package `othermodule.pages` but must exist under `mainmodule.pages`
- I assume that's what you mean. If so, you need to call `Router#setErrorHandler`, which you should probably do in an `@InitKobweb` method somewhere. Off the top of my head:
- Then, your index page is basically an animated image or CSS animation, which, when it is finished running (if an animation, you can use an animation ending event, or you can use a `window.setTimeout` event in a pinch), use a `PageContext` to navigate to whatever other page you want (`ctx.router.navigateTo("portfolio")`)

## Styling

- Silk also has Modifier.displayIf and displayUntil functions which you can use to completely show / hide things
- Ah actually I was looking into this and I don't think I need it after all. `ComponentStyle` and `ComponentVariant` are not things you instantiate. Rather, they're globals, so they should be the same every time.
- `ComponentStyle` is never passed into methods so I think we're all good there?
- The modifier will drop in v0.11.10 but you can use an attrModifier fallback for now
- My code should easily adapt to that, you need to set `Modifier.fillMaxSize`
- In that case, this is what I'm doing (but I'm not sure yet if it's a recommended pattern): <https://github.com/varabyte/kobweb/blob/5d1259da76d7e589c4781a93da1f61f478fec557/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/theme/colors/ColorMode.kt#L55>
- At the very least, I think the function below it should probably be called `rememberColorModeState()`, that might have been sloppy naming on my part
- Moving forward, before 1.0, I may even remove `colorMode` property from `ComponentStyle`, as 1) doing so would let me clean up so much complex code and 2) if I'm correct, this should result in far fewer recompositions in people's code
- for an arbitrary amount of breakpoints but in general you probably don't need more than the 5 kobweb offers
- For now just use `styleModifier`, as I think you are doing?
- Note that Kobweb Silk color management is totally a separate thing, but if you wanted to set that value anyway, right now you'd have to use `styleModifier` + `property`: <https://github.com/varabyte/kobweb#attrsmodifier-and-stylemodifier>
- Even if my stuff is busted, to repro your screenshot above, remember you can use `styleModifier { property("filter", "....") }`
- Actually, `getColorMode` is fine to use without a remember. Honestly, that value is confusing because I have `rememberColorMode` but that should really be called `rememberColorModeState`
- So are you working yet? I still see an `attrsModifier` in there. YOu should use a `styleModifier` instead if you had to, but there, you shouldn't need to
- `Modifier.width(100.vw).height(100.vh).margin(0.px)` should be perfect
- (Side note, `FirstSectionModifier` should probably be called `FirstSectionStyle` although it is technically fine as is. You can also do `listOf(FirstSectionStyle, FadeContainerStyle).toModifier()` as a shortcut)
- Actually I don't think so. I have a layer in Kobweb called `compose-html-ext` which is actually Kobweb agnostic. That said, you can always call Modifier.toAttrs to adapt.
- If you want more customization, I'd recommend implementing your own grid instead of using `SimpleGrid`.

## Silk

- You shouldn't think about the order of the files mattering, as much as you should assume that things like `@InitSilk` / `@InitKobweb` are not called in any guaranteed order, and you should write them that way. If order is required, you should put all logic in the same `@InitSilk` block.

## Server

- If you aren't using `@Api` then you should probably use the former approach. It's much faster!
- 3. **Output:** Instead of spinning up a server to serve it, Kobweb writes the response body directly to a file in the static output folder.

## Build

- Just found out about this annotation, which should be able to detect the most common case?

## Deployment

- Then, you should do `kobweb export --layout static`
- LornMalvo suggested here <https://discord.com/channels/886036660767305799/886039480316858368/1121442538579046504> adding this bit to `build.gradle.kts` should solve it, but then I get the error:

## General

- <@!304331536881156099> OK, apologies but you'll have to delete your existing project (it's busted anyway). if you redownload the 0.6.3 kobweb binary and re-run `kobweb create site` it should work.
- (The reason for the "/public" prefix is so you can put other things in the resource file that don't automatically get exposed to users, but when kobweb runs, it strips it)
- If you're in a non-Kobweb folder and run ":site:kobwebStart" I guess that should work
- var backingElement: HTMLInputElement? by remember { mutableStateOf(null) }
- var htmlInputElement: HTMLInputElement? by remember { mutableStateOf(null) }
- It should be something like `.kobweb/site` I believe by default
- In general, you should just be able to run `kobweb create app` and then `cd app/site; kobweb run`
- Note that you can probably use `deferRender` instead of z-index. It's usually ok if you don't overuse it, but if you start using z-index in multiple places, it becomes complicated to keep track
- If you unzip the file I shared, then `cd cropperdemo/site; kobweb run`, you should see cropper.js in action

---

## From #discuss

> Design rationale, workarounds, and Compose-different patterns extracted from 12,172 #discuss messages (2021-11-08 to 2026-06-28).

### Styling

- Unlike Android Compose (where multiple padding() modifiers simulate margin), Kobweb uses distinct Modifier.padding() (for CSS padding) and Modifier.margin() (for CSS margin) which map directly to their CSS counterparts.
- Modifier chaining on Web is flat CSS (last-wins), not nested like Android Compose. Use nested containers for stacked spacing.
- Using Kotlin stdlib's `.apply` block on a Modifier silently discards chained modifiers (since it returns the original receiver). Use `thenIf` or standard chaining instead.
- Wrap Modifier instances in `remember { }` to avoid unnecessary child recomposition on Web Compose.
- styleModifier { property(...) } is the escape hatch for any CSS property Kobweb doesn't wrap.
- ComponentStyles must be top-level and public (or registered manually via @InitSilk + ctx.theme.registerComponentStyle()).
- Use ComponentStyles for reusable widgets; use inline Modifier for one-off layouts. ComponentStyles are overkill for single-use.
- Keyframes must be top-level file properties, not defined inside composable functions — they're statically registered.
- CSSPosition uses typed Edge class — CenterX/CenterY cannot be combined with offset values (compile error, not runtime).
- To get CSS 'auto' where only CSSNumeric is accepted: create anonymous CSSNumericValue that toStrings to "auto".

### Silk

- Surface color transitions are opt-in since 0.11.0: pass variant = AnimatedColorSurfaceVariant.
- SilkPalette is for Silk built-in widgets only — create your own palette for app-specific colors.
- Every Silk widget follows this contract: ComponentStyle + Modifier + variant + ref.
- CSS :focus-visible is preferred over :focus for button/keyboard focus rings (prevents mouse-click focus rings).
- CSS outline does NOT follow clip paths — use borderRadius(50.percent) for rounded focus indicators, not clip.
- Ref API has 3 flavors: ref(keys) for side effects, disposableRef(keys) { onDispose } for cleanup, refScope { ... } for multiple refs.
- Lambda-in-lambda wrappers create new instances each recompose — use remember { } for stable references in callbacks.
- Surface transitions used * selector originally — that wipes child element transitions. Fixed by targeting ' div' only.
- StyleVariable is Kobweb's own CSS variable class (JB's Compose HTML impl is broken). Cannot use CSS vars in inline styles.
- Palette colors = public widget contract. CSS variables = internal implementation. Use modifyComponentStyleBase + setVariable to customize.
- ComponentStyle cannot use attrsModifier inside — use extraModifiers constructor parameter instead.

### Routing

- ctx.route.params["id"] returns empty string "" for missing params, not null — check both null and empty.
- Route params {id} and {id?} cannot coexist (compiles to same pattern like String vs String?). Use separate /realm/ page.
- rememberPageContext() leaks into deferRender/tooltips in versions before 0.13.x — upgrade or avoid page context in deferred blocks.
- For page transitions: use Modifier.transition() + thenIf(routing, opacity(0)) + onTransitionEnd { router.tryRoutingTo() }.
- Link component renders <a> tag (right-click open in new tab works). Button onClick does NOT support right-click.
- movableContentOf + CompositionLocal preserves widget state across @Page navigation.

### Server

- Ktor coroutines swallow exceptions silently — use @InitApi History class to log errors, expose via debug endpoint.
- Kotlin reflection (kotlin-reflect) and Java ServiceLoader may not work on Kobweb Ktor server — classloader doesn't include user code.
- Full-stack server features get less dev attention since most users prefer static export. Expect less polish.
- Backend rate limiting with Token Bucket and Fixed Window algorithms available via community PR.

### Build

- Kobweb = Compose for Web + Gradle plugin (code gen + dev server) + optional Silk. 90%+ of code is plugin-independent.
- Multimodule (since 0.11.0): kobwebx merged into kobweb block, appGlobals → app.globals. Library modules use com.varabyte.kobweb.library plugin.
- Kotlin 1.9.0 + Compose 1.4.3 has JS IR compiler bug (KT-60852) — hold off until 1.9.10+.
- conf.yaml predates Gradle plugin — server config (ports, scripts) stays in YAML; build concerns in Gradle.
- Kobweb disables Compose TRACE code (Android-only) to reduce production bundle size.
- Kotlin/JS Gradle plugin ignores Node.js download=false config — still tries to download Node regardless of setting.

### Deployment

- --layout static is recommended over Kobweb server for most deployments (GitHub Pages, Netlify, Vercel, Firebase).
- Kobweb inherits single-JS-file output from Compose for Web — no code splitting. Mitigate with dynamic routes or separate subdomain projects.
- Static export snapshots only capture currently visible elements. Content behind if (state) won't appear in exported HTML for crawlers. Use ctx.isExporting check.
- Split landing page and webapp into separate Kobweb subdomain projects (domain.com vs app.domain.com) for smaller bundles and better SEO.
- Markdown-heavy builds bottleneck on Terser minification (~10 min) and unparallelized snapshot generation (~7 min).
- CSS layers broke old iPhone users (older Safari) — check browser baseline before using new CSS features.
- During kobweb export, side effects like window.location.replace() run server-side and can break. Check ctx.params for 'kobweb-export' key.
- Per-page SEO meta tags: use document.createElement('meta') + document.head.appendChild() — NOT document.head.append(string) which escapes HTML.

### General

- Kobweb is client-side only — no SSR. Fetch raw data from server, build UI on client. Consider Kilua if SSR needed.
- DI frameworks (Hilt, Koin) are unnecessary in Kobweb/Compose — use `val x = remember { UseCase() }` directly.
- LaunchedEffect(key) + delay() provides native debouncing — coroutine cancels and restarts on key change.
- deferRender creates a separate rendering layer — elements always appear on top, no z-index needed.
- Overlay transitions need batched delay — browsers process DOM additions before removals regardless of call order.
- Disabled HTML elements (Button, Input) stop mouse events entirely — tooltips need wrapper element.
- CSS opacity(0) creates a stacking context — fix overlapping content with styleModifier { property("isolation", "isolate") }.
- GitHub Gist embedding needs postscribe library workaround (document.write doesn't work after page load).
- Kobweb/Compose HTML cannot share UI code with Android/iOS/Desktop (unlike Compose Multiplatform which is Canvas-based).
- Compose Multiplatform (canvas) can't replace DOM-based UI for SEO, a11y, devtools, print stylesheets.
