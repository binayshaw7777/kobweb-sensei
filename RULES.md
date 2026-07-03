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
