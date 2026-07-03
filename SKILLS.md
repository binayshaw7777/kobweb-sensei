# Kobweb Skills Reference

> Kobweb 0.24.0 · Kotlin 2.3.10 · Compose HTML 1.10.2 · Ktor 3.4.0

---

## What Is Kobweb

Opinionated Kotlin framework for websites and web apps. Built on JetBrains Compose HTML. Inspired by Next.js (routing/structure) and Chakra UI (component styling). Uses KSP (Kotlin Symbol Processing) for compile-time code generation + Gradle plugins for automation.

**Solves:** routing complexity, SEO, boilerplate, dark/light mode, CSS organization, project structure.

---

## Installation

```bash
# macOS / Linux
brew install varabyte/tap/kobweb

# Windows
scoop install varabyte/kobweb

# Cross-platform
sdk install kobweb

# Arch Linux
yay -S kobweb
```

Update: `brew upgrade kobweb` / `scoop update kobweb` / `sdk upgrade kobweb`

Check `COMPATIBILITY.md` before upgrading — Kotlin and Compose versions must match.

---

## Quick Start

```bash
kobweb create app        # scaffold new project
cd your-project/site
kobweb run               # dev server at http://localhost:8080
```

Dev server auto-recompiles and live-reloads on file change.

---

## Project Structure

```
your-project/
├── site/                    # Kotlin/JS frontend
│   ├── src/jsMain/
│   │   └── kotlin/
│   │       └── com/you/
│   │           ├── pages/        # @Page routes
│   │           └── components/   # reusable composables
│   ├── build.gradle.kts     # kobweb app plugin
│   └── conf.yaml            # site config
└── server/                  # Fullstack JVM (optional)
    └── src/jvmMain/
        └── kotlin/com/you/api/   # @Api endpoints
```

`site/` is the frontend (Kotlin/JS). `server/` is the JVM backend (optional, fullstack only).

---

## Routing

### Basic Pages

```kotlin
// site/src/jsMain/kotlin/com/you/pages/Index.kt
@Page
@Composable
fun HomePage() {
    Text("Hello, Kobweb!")
}
```

File → route mapping: `pages/Index.kt` → `/`, `pages/About.kt` → `/about`, `pages/BlogPost.kt` → `/blogpost`

### Route Override

```kotlin
@Page(routeOverride = "my-custom-url")
@Composable
fun CustomPage() { }
```

### Package Mapping

Pages under subpackages get nested routes:

```
pages/blog/Overview.kt  →  /blog/overview
```

Override with `routeOverride`.

### Dynamic Routes

```kotlin
// pages/blog/[slug].kt
@Page
@Composable
fun BlogPostPage(ctx: PageContext) {
    val slug = ctx.route.params.getValue("slug")
}
```

#### Page Scope and Active Page Pattern

Kobweb core uses `@Page` annotations. `main.kt` hooks them up with `router.renderActivePage { ... }` which calls the active `@Page` method stored as mutable state.

#### Check if Exporting

```kotlin
if (rememberPageContext().isExporting) return
```

---

## Silk UI System

Silk = component library + styling system built on Compose HTML. Provides Android/Desktop Compose-like APIs.

### Modifier API

```kotlin
Modifier
    .background(Colors.Red)
    .padding(16.px)
    .margin(top = 8.px)
    .fillMaxWidth()
    .fontSize(18.sp)
    .fontFamily("Roboto")
```

### Layout Components

- `Row` — horizontal flex
- `Column` — vertical flex
- `Box` — absolute positioning layer
- `SimpleGrid` — CSS grid wrapper

### ComponentStyle & Variants

```kotlin
// Define a style
val MyButtonStyle = ComponentStyle(prefix = "my-btn") {
    base { Modifier.color(Colors.White).backgroundColor(Colors.Blue).padding(8.px) }
    hover { Modifier.backgroundColor(Colors.DarkBlue) }
}

// Use it
Button("Click", style = MyButtonStyle)
```

### @InitSilk — Theme Customization

```kotlin
@InitSilk
fun initTheme(ctx: SilkContext) {
    ctx.theme.palette.primary = Colors.Blue
    ctx.theme.fonts.default = FontFamily("Roboto", "sans-serif")
}
```

### Color Mode

```kotlin
val colorMode = ColorMode.current
```

`@InitSilk` overrides apply statically. For dynamic per-page overrides, set `colorMode` directly within a composable.

### Responsive Breakpoints

Breakpoints: `Mobile` (0), `Tablet` (768), `Desktop` (1024), `Widescreen` (1280), `Ultrawide` (1920+).

### CSS Animations / Keyframes

```kotlin
val fadeIn = CssKeyframes {
    0% { opacity(0); transform = "translateY(20px)" }
    100% { opacity(1); transform = "translateY(0)" }
}
Div(Modifier.animation(fadeIn, duration = 500.ms, easing = Easing.EaseOut))
```

### Style Variables (CSS Custom Properties)

```kotlin
CssStyle("custom-props") {
    base {
        Modifier
            .setVariable("--my-color", Colors.Red.toString())
            .setVariable("--spacing", "1rem")
    }
}
```

### CSS Layers

```kotlin
@CssLayer("components", after = "base")
```

### DOM Element Access

```kotlin
var myRef by remember { mutableStateOf<Element?>(null) }
Div(Modifier.ref { myRef = it; onDispose { myRef = null } })
```

### Icon Libraries

```kotlin
// Font Awesome
implementation(kobwebx("font-awesome"))
FaCoffee()   // or FaBeer, FaGithub, etc.

// Material Icons
implementation(kobwebx("material-icons"))
MdiHome()
```

### Style Recipes

```kotlin
// Gradient text
val GradientText = CssStyle {
    base {
        Modifier
            .background(linearGradient(180.deg, Colors.Purple, Colors.Pink))
            .backgroundClip(BackgroundClip.Text)
            .property("-webkit-background-clip", "text")
            .property("color", "transparent")
    }
}

// Glassmorphism
val GlassStyle = CssStyle {
    base {
        Modifier
            .background("rgba(255,255,255,0.15)")
            .backdropFilter(BlurFilter(10.px))
            .borderRadius(16.px)
            .border(1.px, LineStyle.Solid, "rgba(255,255,255,0.3)")
            .boxShadow("0 4px 30px rgba(0,0,0,0.1)")
    }
}

// Fullscreen overlay
val OverlayStyle = CssStyle {
    base { Modifier.position(Position.Fixed).fillMaxSize().zIndex(9999) }
}
```

### Color System

#### ColorMode

```kotlin
val colorMode = ColorMode.current

// confusing naming: should be rememberColorModeState
rememberColorMode()
```

#### Theme Palette

```kotlin
@InitSilk
fun initTheme(ctx: SilkContext) {
    ctx.theme.palette.primary = Colors.Blue
    ctx.theme.palette.secondary = Colors.Purple
    ctx.theme.palette.background = Colors.White
    ctx.theme.palette.surface = Colors.Gray
}
```

#### Force ColorMode on Page

```kotlin
var colorMode = ColorMode.current
colorMode = ColorMode.LIGHT
```

---

## Fullstack / Server

### @Api Endpoints

Server methods live in `jvmMain/api/`. Auto-routed by package + filename.

```kotlin
// jvmMain/api/Hello.kt → /api/hello
@Api
suspend fun hello(ctx: ApiContext) {
    ctx.res.setBodyText("Hello, World!")   // auto-sets 200
}
```

Dynamic routes:

```kotlin
// jvmMain/api/user/[id].kt → /api/user/42
@Api
suspend fun getUser(ctx: ApiContext) {
    val id = ctx.route.params["id"] ?: return ctx.res.sendError(400)
    ctx.res.setBodyText("User $id")
}
```

### @ApiInterceptor

```kotlin
@ApiInterceptor(priority = 100)
suspend fun authInterceptor(ctx: ApiInterceptorContext) {
    val token = ctx.req.headers["Authorization"]
    if (token == null) { ctx.res.sendError(401); return }
    ctx.proceed()
}
```

### ApiStream

```kotlin
@Api
suspend fun stream(ctx: ApiContext) {
    ApiStream(ctx.res).use { stream ->
        repeat(10) { stream.write("data: $it\n\n"); delay(1000) }
    }
}
```

### Server Config

```yaml
# conf.yaml
server:
  port: 8080
  host: "0.0.0.0"
```

### API Calling Conventions

```kotlin
// Client calls server API
val response = ctx.fetch("api/hello")
```

---

## Markdown

### File Location

`src/resources/markdown/` → auto-routed same as pages.

### Front Matter

```markdown
---
layout: .components.layouts.BlogLayout
routeOverride: my-custom-url
imports:
  - .components.widgets.*
---

# My Post
```

Special keys: `layout`, `routeOverride`, `imports`.

Access in code:
```kotlin
val meta = rememberPageContext().markdown?.frontMatter
val title = meta?.get("title")?.first()
```

### Embed Composables

```markdown
Block (standalone):
{{{ .components.widgets.VisitorCounter }}}

Inline (mid-sentence):
Press ${.components.widgets.ColorButton} to toggle.
```

No spaces inside `{{{ }}}` or `${ }`.

### Callouts

```markdown
> [!NOTE]
> Standard note

> [!WARNING "Custom Label"]
> Watch out!
```

Types: `NOTE`, `TIP`, `IMPORTANT`, `WARNING`, `CAUTION`, `QUESTION`, `QUOTE`

Silk variants: outlined, filled, left-bordered.

### Global Config (build.gradle.kts)

```kotlin
kobweb {
    app {
        markdown {
            defaultLayout.set(".components.layouts.PageLayout")
            imports.add(".components.widgets.*")
            process.set { entries ->
                entries.forEach { /* transform entries */ }
            }
            addSource("src/resources/extra-markdown", targetPackage = ".extra")
        }
    }
}
```

---

## Web Workers

```kotlin
// Worker definition (shared module)
class PrimeFinder : KobwebWorkerFactory<Int, List<Int>> {
    override fun create(output: OutputDispatcher<List<Int>>) =
        Worker { n -> findPrimes(n) }
}

// Client usage
val worker = rememberWorker { PrimeFinder() }
LaunchedEffect(Unit) {
    worker.postInput(1000)
}
val primes = worker.output.collectAsState(emptyList())
```

---

## Export & Deployment

```bash
# Static layout (SEO-friendly pre-rendered HTML)
kobweb export

# Full-stack (server + static)
kobwebExport -PkobwebLayout=FULLSTACK -PkobwebBuildTarget=RELEASE
```

### GitHub Actions

```yaml
- uses: varabyte/kobweb-actions/export@v1
  with:
    kobweb-layout: static
    fetch-depth: 0
- uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./
    publish_branch: gh-pages
    destination_dir: ./
```

### Deployment Targets

#### Caddy

```caddyfile
yourdomain.com {
    root * /path/to/site
    file_server
    try_files {path} {path}/index.html /index.html
}
```

#### Docker (Fullstack)

```dockerfile
FROM eclipse-temurin:17-jre
COPY server/build/libs/server.jar /app/server.jar
EXPOSE 8080
CMD ["java", "-jar", "/app/server.jar"]
```

```yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - KOBWEB_ENV=production
```

### Export Tips

```bash
# Static layout
kobweb export --layout static

# Run production build locally
kobweb run --env prod --layout static
```

---

## Development Workflow

### IntelliJ

```bash
# Continuous watch mode (Gradle task)
kobwebStart -t

# Stop server
kobwebStop
```

Stack traces in terminal link directly to source lines.

### CLI Commands Summary

| Command | Effect |
|---------|--------|
| `kobweb create app` | scaffold new project |
| `kobweb run` | dev server + live reload |
| `kobweb export` | static site export |
| `kobwebStart -t` | Gradle continuous watch |
| `kobwebStop` | stop server |

### Workflow Recipes

#### Create + Run a Sample

```bash
kobweb create examples/todo
cd todo/site
kobweb run
```

Available: `examples/todo`, `examples/clock`, `examples/chat`, `examples/opengl`, `examples/chatgpt`.

#### Using Gradle Directly

```bash
# From site folder
../gradlew kobwebStart   # includes live reload

# From root
./gradlew :site:kobwebStart   # no live reload
```

#### Move Kobweb to Subfolder

Update `conf.yaml` paths with `..` references. The structure works but paths need adjustment.

#### Check Generated HTML

```bash
# After kobweb run, search build output
find build -name "index.html"
```

#### Live Reload vs Manual Restart

`kobweb run` includes live reload. `kobwebStart` from within site folder also reloads. Direct Gradle task `:site:kobwebStart` from root does not.

### Troubleshooting

#### conf.yaml Empty After Export

Check `conf.yaml` title and paths. Export puts files in `.kobweb/site`.

#### Plugin Not Found

```kotlin
Plugin [id: 'com.varabyte.kobweb.application', version: '...'] was not found
```

Check plugin repository in `settings.gradle.kts`:

```kotlin
pluginManagement {
    repositories {
        mavenCentral()
        maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
        maven("https://us-central1-maven.pkg.dev/varabyte-repos/public")
        gradlePluginPortal()
    }
}
```

#### JVM Target Error

```kotlin
Failed to calculate the value of property 'jvmTarget'.
> Unknown Kotlin JVM target: 20
```

Set explicit JVM target:

```kotlin
jvm {
    compilations.all {
        kotlinOptions {
            jvmTarget = "1.8"
        }
    }
}
```

#### HeadlessException in CI

```bash
kobweb export --layout static --notty  # no TTY mode for headless
```

#### .gitignore for Kobweb

```bash
# Kobweb ignores
.kobweb/*
!.kobweb/conf.yaml
!.kobweb/site
```

### Styling HTML Elements

```kotlin
// Inline style (avoid for perf — not cached)
Div(attrs = {
    style { color(Color("red")) }
}) { }

// CssStyle (preferred — generates stylesheet class)
val MyDivStyle = CssStyle {
    base { Modifier.color(Colors.Red) }
}
Div(MyDivStyle.toAttrs()) { }

// Pseudo-classes in CssStyle
val MyDivStyle = CssStyle {
    base { Modifier.backgroundColor(Colors.Black) }
    hover { Modifier.color(Colors.DarkRed) }
}
Div(MyDivStyle.toAttrs()) { }
```

---

## Utility Modules (Standalone)

Both usable outside Kobweb in any Kotlin/JS project.

### compose-html-ext

- Type-safe CSS wrappers
- Gradient support
- SVG capabilities
- CSS variables
- Transition events
- `calc()` helpers

### browser-ext

- File I/O utilities
- Enhanced `fetch` operations
- Resize / Intersection observers
- DOM traversal helpers

---

## Gradle Artifacts

```kotlin
// build.gradle.kts
kobweb {
    app {
        jsTarget {
            dependencies {
                implementation(kobweb("core"))
                implementation(kobweb("silk"))
                implementation(kobwebx("font-awesome"))
                implementation(kobwebx("material-icons"))
            }
        }
        jvmTarget {
            dependencies {
                implementation(kobweb("server-plugin"))
            }
        }
    }
}
```

---

## When to Use Kobweb

**Good fit:**
- Kotlin-only shops avoiding JS
- Android devs moving to web
- Personal sites, blogs, small apps
- Rapid prototyping

**Not ideal:**
- Large existing projects needing migration
- Enterprise requiring battle-tested stability
- Heavy third-party JS library dependency

**vs Compose Multiplatform:** Kobweb preserves HTML/CSS benefits — SEO, browser devtools, 200–400K bundle vs 2–3MB CMP baseline.

---

## Resources

| Resource | URL |
|----------|-----|
| Docs | https://kobweb.varabyte.com/docs |
| GitHub | https://github.com/varabyte/kobweb |
| Discord | https://discord.gg/5NZ2GKV5Cs |
| Kotlin Slack | `#kobweb` channel |

## Recipes & Fixes

> Problem → solution threads extracted from #need_help Discord.
> Organized by topic. Each entry is a complete problem → solution.

### Styling

### Beginner question: Why does the following code not center horizonal the red column in the middle of the screen

**Problem:** Beginner question: Why does the following code not center horizonal the red column in the middle of the screen? ``` Row( modifier = Modifier .alignContent(AlignContent.Center) .fillMaxSize()) { Column( horizontalAlignment = Alignment.CenterHorizontally, modifier = Modifier .fillMaxHeight() .width...

**Solution:** **So three things to try** - Use Row's alignHorizontally parameter - Change Row's fillMaxSize to width(800.px) (just temporarily, to debug stuff) - Change Row's background color to, I don't know, blue?

```
Row(    modifier = Modifier        .alignContent(AlignContent.Center)        .fillMaxSize()) {    Column(        horizontalAlignment = Alignment.CenterHorizontally,        modifier = Modifier            .fillMaxHeight()            .width(600.px)            .textAlign(TextAlign.Center)            .backgroundColor(Color.red),    ) {        Header()        content()        Spacer()        Footer()    }}
```

---

### Just another Noob-Compose-Layout-Question. Why does the following Code not center the content in both axis

**Problem:** Just another Noob-Compose-Layout-Question. Why does the following Code not center the content in both axis? ``` Column(Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally) { Row(Modifier.fillMaxHeight(), verticalAlignment = Alignment.CenterVertically) { content() } } ```

**Solution:** Also you probably just want to use a Box in that case

```
Column(Modifier.fillMaxSize(), horizontalAlignment = Alignment.CenterHorizontally) {         
   Row(Modifier.fillMaxHeight(), verticalAlignment = Alignment.CenterVertically) {     
       content()    
   }
}
```

---

### I think my problem is just, that I've no idea how to use html / css stylings *and* Compose

**Problem:** I think my problem is just, that I've no idea how to use html / css stylings *and* Compose. It might be a bad starting point if getting into kobweb or compose for web in general 😄

**Solution:** The way you're doing it is probably the best case possible at the moment though! Learn jetpack Compose and then try to use kobweb. Sadly it's not as smooth migration as I would have hoped back when I started this project, but I think I've done my best based on the different API requirements.

---

### I assumed it's not on my end since I get the page I'm looking for but then kobweb gives me the 404 so idk Just

**Problem:** I assumed it's not on my end since I get the page I'm looking for but then kobweb gives me the 404 so idk

**Solution:** Just to make sure, if you run locally: `kobweb run --env prod --layout static` do you get the behavior you're expecting?

---

### I'm a bit lost. How would you make a horizontally scrollable div

**Problem:** I'm a bit lost. How would you make a horizontally scrollable div?

**Solution:** (Rows and Columns already put you in flex mode so I think you don't need to add the display modifier here)

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

### Does nowrap plus work

**Problem:** Does nowrap plus `justifyContent(JustifyContent.Start)` work?

**Solution:** ``` Row( Modifier .fillMaxWidth() .overflowX(Overflow.Auto) .flexWrap(FlexWrap.Nowrap) ) { for (i in 1..10) { Button(onClick = {}, Modifier.margin(5.px).minWidth(100.px)) { Text("Button $i") } } } ``` Seems to work

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

### touchmove doesn't work for me either Ah gotcha

**Problem:** touchmove doesn't work for me either

**Solution:** Ah gotcha. In that case yeah then this: ``` PageLayout("Welcome to Kobweb!") { val scope = rememberCoroutineScope() Row(Modifier.width(100.px).height(30.px).backgroundColor(Colors.AliceBlue).attrsModifier { addEventListener("click") { scope.launch { delay(800L) println("CLICKED") } } }) { } ```

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

### Where can I create the variant

**Problem:** Where can I create the variant? I tried to do ButtonStyle.addVariant in a top-level property before using it inside Composable, but it tells me `Key ... is missing in the map`

**Solution:** https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-widgets

---

### Probably worth adding to the classes and maybe to

**Problem:** Probably worth adding `hashCode()` to the `ComponentVariant` classes and maybe to `ComponentStyle`?

**Solution:** you or your users probably wouldn't notice in this case, so if not using variants is easier for you it should be fine

---

### So what then with

**Problem:** So what then with `ComponentStyle`?

**Solution:** For exampe something like this: https://github.com/varabyte/kobweb/blob/a46c08e97dd40cb32eebba493f4ecc40b1ac3983/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L62

---

### Is there a way to define something like "every tr element inside table has this modifier" via style components

**Problem:** Is there a way to define something like "every tr element inside table has this modifier" via style components on a table, or should I make a separate component for tr?

**Solution:** Take a look at this sort of syntax: https://github.com/varabyte/kobweb/blob/f82c8fc40b19950fb9679bb8e5575647b3b261c9/frontend/kobweb-compose/src/jsMain/kotlin/com/varabyte/kobweb/compose/style/KobwebComposeStyleSheet.kt#L26 which in this case means "every element that is a direct child of elements that are tagged with the class "kobweb-box". I think what you're looking for is "table > tr" in your case

---

### Hmm, it's for *all* tables. I mean, it fits my purpose, but is there a way to do this just for specific compon

**Problem:** Hmm, it's for *all* tables. I mean, it fits my purpose, but is there a way to do this just for specific component styles? I mean, I guess I can just use string as a class name and do "className > tr" or something 🤔

**Solution:** Not 100% sure, you can play with it, but you might be able to use the `cssRule` method in `ComponentStyle` ``` ComponentStyle("test") { cssRule("> tr") { ... } } ```

---

### Is there a way to create a such that the property isn't set? I want to use which works if I use it as an inlin

**Problem:** Is there a way to create a `Column` such that the `justify-content` property isn't set? I want to use `justify-content: space-evenly` which works if I use it as an inline style. But if I put it in a `ComponentStyle`, then the `kobweb-col...` style overrides it

**Solution:** https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/Arrangement

---

### Using myStyle

**Problem:** Yea I could, but the component style way would be nicer, especially since I want to make other changes based on the breakpoints too

**Solution:** I'll probably add a Flexbox Div right below it as a control.

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

### I think the parameter of doesn't work

**Problem:** I think the `extraModifiers` parameter of `ComponentStyle.addVariant/addVariantBase` doesn't work. It seems to work for creating styles, but not variants

**Solution:** I'll check, I don't think I tested carefully for variants

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

### Is it possible to just have some sort of special alignment with a class that doesn't set the property

**Problem:** Is it possible to just have some sort of special `Unset` alignment with a class that doesn't set the property?

**Solution:** Let me sleep on it but I have this now: ``` val MyStyle = ComponentStyle("my-style") { base { Modifier .width(200.px) .height(200.px) .justifyContent(JustifyContent.SpaceEvenly) } Breakpoint.MD { Modifier.justifyContent(JustifyContent.Start) } } @Page @Composable fun HomePage() { PageLayout("Index") { Column(MyStyle.toModifier().backgroundColor(Colors.LightBlue), horizontalAlignment = Alignment.CenterHorizontally, verticalArrangement = Arrangement.None) { repeat(3) { SpanText("Text") } } } } ```

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

### If I want a variant where no matter the state (hover, focus, ...), a certain colormode aware modifier is appli

**Problem:** If I want a `ButtonStyle` variant where no matter the state (hover, focus, ...), a certain colormode aware modifier is applied, do I have to override all the rules in `ButtonStyle` separately like this? ```kotlin val variant = ButtonStyle.addVariant("variant") { val modifier = Modifier.background...

**Solution:** I think you have to override all of them? Since they'll be layered on top of base

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

### Maybe the base button should have no effects and there can be some sort of which incorporates the palette? I r

**Problem:** Maybe the base button should have no effects and there can be some sort of `ThemedButton` which incorporates the palette?

**Solution:** I really hate how buttons right now have to included "enabled" in each style. I was trying to have "disabled" work but it wasn't. Actually that might be due to something I was experimenting with when things were using a Div under the hood.

---

### For the default , you can just override the (and I guess maybe cursor

**Problem:** For the default `ButtonStyle`, you can just override the `backgroundColor` (and I guess maybe cursor?) and then you don't need `+ enabled` as long as disabled is the last style

**Solution:** I also wished this worked but it doesn't seem to 🙂 ``` val ButtonStableVariant = ButtonStyle.addVariant("stable") { (hover + enabled) { Modifier } (focus + enabled) { Modifier } (active + enabled) { Modifier } (focus + active + enabled) { Modifier } } ```

```kotlin
val ButtonStableVariant = ButtonStyle.addVariant("stable") {
    (hover + enabled) { Modifier }
    (focus + enabled) { Modifier }
    (active + enabled) { Modifier }
    (focus + active + enabled) { Modifier }
}
```

---

### Does border radius 50% not do the trick

**Problem:** Does border radius 50% not do the trick?

**Solution:** If I could create one button style and then just add a single circulebutton variant (or people just add a clip(50%) themselves) and it works across multiple sizes, then less is more for sure

---

### i know that i can use too Interesting, I'll think about that

**Problem:** i know that i can use `modifier = listOf(TextStyle, TextHeaderStyle).toModifier()`too

**Solution:** Interesting, I'll think about that

---

### To make Kobweb even more attractive, we could create a styled UI library using Kobweb. What are your thoughts

**Problem:** To make Kobweb even more attractive, we could create a styled UI library using Kobweb. What are your thoughts?

**Solution:** It's very rudimentary right now but once it gets some love it should explode with a lot of widgets

---

### How do i write this CSS property in Kobweb

**Problem:** How do i write this CSS property in Kobweb? `animation-timing-function: cubic-bezier(0, 1, 1, 0);`

**Solution:** I mean, I don't think there's any point to any animation properties without a name

---

### Alright. Another question. Let's say i've got three states, box moving to three different positions, and a but

**Problem:** Alright. Another question. Let's say i've got three states, box moving to three different positions, and a button toggling those states. Is something like that possible with CSS?

**Solution:** I'm OK leaning on CSS properties for things like animation timings. I'm just thinking I could probably simplify your original CSS using Kotlin

---

### so or something like this Ah yeah it's also what they're using in the stack overflow answer

**Problem:** so `it.clipboardData?.getData("text/plain")` or something like this

**Solution:** Ah yeah it's also what they're using in the stack overflow answer

---

### I mean it's js but ```` Did you use attrModifier { onPaste { evt -> ... } }

**Problem:** I mean it's js but ```js document.querySelector("div").addEventListener("paste", e => { e.preventDefault(); var text = e.clipboardData.getData("text/plain"); document.execCommand("insertHTML", false, text); }); ```

**Solution:** Did you use attrModifier { onPaste { evt -> ... } }?

```js
document.querySelector("div").addEventListener("paste", e => {
  e.preventDefault();
  var text = e.clipboardData.getData("text/plain");
  document.execCommand("insertHTML", false, text);
});
```

```kotlin
onPaste { event ->
    event.preventDefault()
    val text = event.clipboardData?.getData("text/plain")
    document.execCommand("insertHtml", false, text!!)
}
```

---

### Using ServiceCardBackgroundStyle

**Problem:** ```kotlin val ServiceCardBackgroundStyle by ComponentStyle { base { Modifier .border( width = 1.px, style = LineStyle.Solid, color = Color(Theme.LightGray.color) ) .backgroundColor(Colors.White) } hover { Modifier .border( width = 1.px, style = LineStyle.Solid, color = Color(Theme.LightGray.color...

**Solution:** ``` (hover + cssRule(" > *")) { Modifier .border( width = 1.px, style = LineStyle.Solid, color = Color(Theme.Primary.color) ) .backgroundColor(Colors.White) } ```

```kotlin
val ServiceCardBackgroundStyle by ComponentStyle {
    base {
        Modifier
            .border(
                width = 1.px,
                style = LineStyle.Solid,
                color = Color(Theme.LightGray.color)
            )
            .backgroundColor(Colors.White)
    }
    hover {
        Modifier
            .border(
                width = 1.px,
                style = LineStyle.Solid,
                color = Color(Theme.LightGray.color)
            )
            .backgroundColor(Theme.Primary.color)
    }

    cssRule(" > *") {
        Modifier
            .border(
                width = 1.px,
                style = LineStyle.Solid,
                color = Color(Theme.Primary.color)
            )
            .backgroundColor(Colors.Transparent)
    }

    (hover + cssRule(" > *") {
        Modifier
            .border(
                width = 1.px,
                style = LineStyle.Solid,
                color = Color(Theme.Primary.color)
            )
            .backgroundColor(Colors.White)
    })
}
```

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

### Any example of styling the CheckboxInput? Side note since that discussion you linked to

**Problem:** Any example of styling the CheckboxInput?

**Solution:** Side note since that discussion you linked to. You can do this: ``` @InitSilk fun initSilk(ctx: InitSilkContext) { ctx.stylesheet.registerStyle("...") { ... } } ``` instead of creating your own StyleSheet.

```kotlin
@InitSilk
fun initSilk(ctx: InitSilkContext) {
  ctx.stylesheet.registerStyle("...") { ... }
}
```

---

### I was trying to do something crazy, but looks like it's probably impossible

**Problem:** I was trying to do something crazy, but looks like it's probably impossible. Basically, I wanted to use "full" kobweb in an extension. But looks like extensions only run as `.html` files, so I'd need the page to work while keeping the ending. Not sure it would work even then though.

**Solution:** I recall had some initial success using some parts of Kobweb when making an extension (the widgets and the Modifier stuff)

---

### Why don't you try some kind of a hack

**Problem:** Why don't you try some kind of a hack. To actually update the border color of your input field when you click those icons, so that you fake a focus 😁 It's not a good example tho, but still...

**Solution:** https://github.com/varabyte/kobweb/commit/2a97f17da380f5222cec879e21ee1eea5cdd4f8d <- The death of transition property name abuse is coming ☠️

---

### Does this do what you want? ```` FYI, in case it helps you (or anyone else) in the future, you have three opti

**Problem:** Does this do what you want? ``` Surface( modifier = Modifier .align(Alignment.CenterEnd) .margin(right = 12.px) .onMouseDown { it.preventDefault() } .onClick { value = "" } ) { ```

**Solution:** FYI, in case it helps you (or anyone else) in the future, you have three options with events 1) preventDefault - tells the underlying element type (in this case, a div?) not to handle the event (here, it was probably taking focus or something?) 2) stopPropogation - stops the event from being handled by *other* elements (e.g. elements underneath the current one) 3) stopPropogationImmediately - stops the event from being handled by *other* elements *and* any other event handlers registered with the current element

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

### i've got a side bar i can open/close. any trick for preserving open/close state when changing page

**Problem:** i've got a side bar i can open/close. any trick for preserving open/close state when changing page?

**Solution:** You wouldn't even need to use the `lazy` trick. I only do that because `colorModeState` depends on configuration that the user might change on startup. You would just have a mutableState value somewhere that you could set from anywhere.

---

### then i can give you a demo 🙂 Like if you have two composables that both call first and calls , then despite yo

**Problem:** then i can give you a demo 🙂

**Solution:** Like if you have two composables that both call `PageLayout` first and `PageLayout` calls `SideNavBar`, then despite you moving to a new page, the old side nav bar shouldn't get recomposed? But if elements change in a way that shift the DOM, then I believe all bets are off.

---

### does this concept exist in css

**Problem:** does this concept exist in css?

**Solution:** I think a year ago I saw on the Slack someone implemented LazyColumn for web but I haven't yet looked into it

---

### how do i hide the scrollbar on my column with overflow

**Problem:** how do i hide the scrollbar on my column with overflow?

**Solution:** https://www.w3schools.com/css/tryit.asp?filename=trycss_overflow_hidden

---

### I have a side menu on the left and a column on the right with different sections. I want the column to smoothl

**Problem:** I have a side menu on the left and a column on the right with different sections. I want the column to smoothly scroll to the section top when a menu item is selected. How to?

**Solution:** https://www.w3schools.com/howto/howto_css_smooth_scroll.asp this also uses the fragment trick

```kotlin
var scrollableColumn: Element? by remember { mutableStateOf(null) }

    LaunchedEffect(state.selectedSection) {
        scrollableColumn?.let { scrollableColumn ->
            val section = scrollableColumn.querySelector(state.selectedSection.title)
            section?.scrollIntoView("smooth")
        }
    }
```

```kotlin
.attrsModifier {
                        ref {
                            scrollableColumn = it
                            onDispose { scrollableColumn = null }
                        }
                    }
```

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

### Chat GPT says 😄 CSS itself does not provide a way to create smooth scrolling effects

**Problem:** Chat GPT says 😄 CSS itself does not provide a way to create smooth scrolling effects. However, you can achieve smooth scrolling effects using JavaScript. Here's an example of how you can smooth scroll to a position using JavaScript: HTML: ```html <a href="#" class="smooth-scroll" data-target="#se...

**Solution:** https://codepen.io/jasongart/pen/qgmGxB Not sure yet but seems like this is doing something without changing the href address

---

### Hey everyone, trying to add a styleModifier, but the property doesn't show up on the element. Here's the code 

**Problem:** Hey everyone, trying to add a styleModifier, but the property doesn't show up on the element. Here's the code ``` Row(Modifier .fontFamily("Koulen") .fontStyle(FontStyle.Normal) .fontWeight(400) .fontSize(64.px) .color(rgb(233, 0, 0)) .styleModifier { property("text-shadow", "3px, 4px, 7px, 0px")...

**Solution:** It's syntax is pretty simple, unlike some of the modifiers I had to add recently. I should bump up textshadow's priority, probably add it for the next release.

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

### Any support for multiple text shadows? Ah I see it now

**Problem:** Any support for multiple text shadows?

**Solution:** Ah I see it now. I'll add support for that.

---

### Just curious, why not ```` which I actually kind of prefer

**Problem:** Just curious, why not `TextShadow`

**Solution:** ``` sealed class TextShadow { class Keyword : TextShadow class Params : TextShadow } fun textShadow(vararg params: TextShadow.Params) fun textShadow(keyword: TextShadow.Keyword) ``` which I actually kind of prefer? But it feels like that might be inconsistent with the rest of the framework

```
sealed class TextShadow {
   class Keyword : TextShadow
   class Params : TextShadow
}

fun textShadow(vararg params: TextShadow.Params)
fun textShadow(keyword: TextShadow.Keyword)
```

---

### Also will there be somthing like in future to manage styles ```` I'll file a bug for that

**Problem:** Also will there be somthing like in future to manage styles ```kotlin object ProjectStyles { const val container by ComponentStyle { ... } const val items by ComponentStyle { ... } } @Composable fun Project(){ Box(ProjectStyles.container.toModifier()) { ... Box(ProjectStyles.items.toModifier()){}...

**Solution:** I'll file a bug for that. It should be possible. I have to be really careful with that feature because my current parsing code can get complex fast. Note that the name for those styles would be "container" and "items", which it will be easy to copy/paste that code somewhere else, change the parent object, and get a frustrating error at runtime.

```kotlin
object ProjectStyles {
const val container by ComponentStyle {
...
}
const val items by ComponentStyle {
...
}
}

@Composable
fun Project(){
Box(ProjectStyles.container.toModifier()) {
     ... 
    Box(ProjectStyles.items.toModifier()){}
   }
}
```

**Note:** Note that the name for those styles would be "container" and "items", which it will be easy to copy/paste that code somewhere else, change the parent object, and get a frustrating error at runtime.

---

### can't you have all functions return the

**Problem:** can't you have all functions return the `StyleResult`?

**Solution:** Here's what I declared once in my project: ``` ctx.stylesheet.apply { registerBaseStyle("@font-face") { Modifier .fontFamily("CardTitle") .styleModifier { property("src", "url(fonts/CardTitle.ttf") } } ```

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

### I think you'd need here at least with the current implementation, I think the helper method would be a good ex

**Problem:** I think you'd need `base { Modifier } ` here at least with the current implementation, I think the helper method would be a good explicit approach to empty styles

**Solution:** And your font is correctly named and in `resources/public/font/CentraNo2-Book.ttf` ?

---

### ```` I think it could be due to registering multiple styles

**Problem:** ```kotlin @InitSilk fun updateTheme(ctx: InitSilkContext) { ctx.stylesheet.registerStyle("@font-face") { base { Modifier .fontFamily("Centra") .fontWeight(700) .styleModifier { property("src", "url(font/CentraNo2-Bold.ttf)") } } } ctx.stylesheet.registerStyle("@font-face") { base { Modifier .font...

**Solution:** I think you can actually copy the HTML and paste it into a separate text editor

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

### Hello All, I am trying to learn how to integrate existing javascript libraries with KobWeb

**Problem:** Hello All, I am trying to learn how to integrate existing javascript libraries with KobWeb. To do this, I am using https://sweetalert2.github.io/#usage I got success showing alert with `js` function. But, now I want to know how to handle callback in `js`? Here is the sample code of my Composable ...

**Solution:** You should probably try to get out of JS as fast as possible. Kotlin/JS interactions are notoriously tricky. Probably you want to write some JS code, cast its result to a `Promise`, and then handle the promise from Kotlin.

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

### How to add conditional Style? I am creating a Header component which has a and multiple

**Problem:** How to add conditional Style? I am creating a Header component which has a `Row` and multiple `Links`. I have added style so that when I hover over it, some style is applied. Now, when I click on any link, I want it to be selected and have selected style applied on it. So how can we do that here ...

**Solution:** Something like this is possible: <https://github.com/varabyte/kobweb/blob/369b94fe7349d22dc95a28004747353244641c24/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L51>

---

### Actually I have already quit all java process manually because of heating issue

**Problem:** Actually I have already quit all java process manually because of heating issue. Right now there are only 4 processes running.

**Solution:** I have a sticky header in my blog site but I actually need to revisit it because it's using zindex. I'm going to try and fix it later using `deferRender` but I haven't had a chance to yet. Still, you can take a look at what I'm doing: <https://github.com/bitspittle/bitspittle.dev/blob/222310af50d8f17973d733ee9dfee0d49aceb421/site/src/jsMain/kotlin/dev/bitspittle/site/components/sections/NavHeader.kt#L38> (and read up on the "sticky" CSS position)

---

### The header shouldn't be in the box

**Problem:** The header shouldn't be in the box. ```kotlin Column { Header() ScrollableColumn { Content() } } ``` (Sorry for the syntax, I have no idea how you write that with Kobweb)

**Solution:** ScrollableColumn would be a `Column` with the right modifier

```kotlin
Column {
    Header()

    ScrollableColumn {
        Content()
    }
}
```

---

### scrollablecolumn isn't a thing is it

**Problem:** scrollablecolumn isn't a thing is it?

**Solution:** There's a mutex concept in Web?

---

### But does that means I can't put the attributes in a variable and uses them like that : ```Modifier`

**Problem:** Thank you ! But does that means I can't put the attributes in a variable and uses them like that : ```Kotlin // I know it's not the right syntax val attrs = AttrBuilderContext<HTMLVideoElement> { attr("width", "100%") onMouseEnter { (it.target as HTMLVideoElement).setAttribute("controls", "") } o...

**Solution:** Attributes are a weak surface area in Kobweb right now because it's not easy to expose things like that in `Modifier`

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

### attributes like can't be manipulated by css though ````

**Problem:** attributes like `controls` can't be manipulated by css though

**Solution:** ``` Video(attrs = Modifier.toAttrs { /* .. video attributes go here .. */ } ```

```
Video(attrs = Modifier.toAttrs {
   /* .. video attributes go here .. */
}
```

---

### this version is working correctly : `````toAttrs` (but not 100% sure, just guessing)

**Problem:** this version is working correctly : ```Kotlin val attrs = Modifier.toAttrs<AttrsScope<HTMLVideoElement>> { attr("width", "100%") onMouseEnter { (it.target as HTMLVideoElement).setAttribute("controls", "") } onMouseLeave { (it.target as HTMLVideoElement).removeAttribute("controls") } } Video(attrs...

**Solution:** You might be able to also do: ``` Modifier .onMouseEnter { it.setAttribute("controls", "") } .onMouseLeave { it.removeAttribute("controls") } .toAttrs { attr("width", "100%") } ``` if you wanted to minimize the part inside the `toAttrs` (but not 100% sure, just guessing)

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

### Side note: is there a reason the generic type of requires you to specify the as opposed to just letting you do

**Problem:** Side note: is there a reason the generic type of `Modifier.toAttrs` requires you to specify the `.toAttrs<AttrScope<*>>` as opposed to just letting you do `.toAttrs<*>` and the `AttrScope` handled internally?

**Solution:** Yeah looks like a smooth transition. Possibly not backwards compatible, but better to rip the bandaid off sooner than later

---

### Ah, in that case maybe you can just have both

**Problem:** Ah, in that case maybe you can just have both?

**Solution:** ```kotlin fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null): AttrsScope<E>.() -> Unit = this.toAttrs<AttrsScope<E>>(finalHandler) ``` seems to be choosing the right one.

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null): AttrsScope<E>.() -> Unit =
    this.toAttrs(finalHandler)
```

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null): AttrsScope<E>.() -> Unit =
    this.toAttrs<AttrsScope<E>>(finalHandler)
```

---

### I took it from ```` is compiling yeah but have to test it

**Problem:** I took it from `Div(attrs: AttrBuilderContext<HTMLDivElement>? = null)`

**Solution:** ``` fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null) = this.toAttrs<AttrsScope<E>>(finalHandler) ``` is compiling yeah but have to test it

```kotlin
fun <E : Element> Modifier.toAttrs(finalHandler: (AttrsScope<E>.() -> Unit)? = null) =
    this.toAttrs<AttrsScope<E>>(finalHandler)
```

---

### Check out this article <https://css-tricks.com/styling-cross-browser-compatible-range-inputs-css/> and this to

**Problem:** Check out this article <https://css-tricks.com/styling-cross-browser-compatible-range-inputs-css/> and this tool <http://danielstern.ca/range.css/?ref=css-tricks#/>

**Solution:** 🤩 for all the switch talk. I need to add a widget to Silk, this discussion looks really helpful. I like that you both explored transition and animation approaches.

---

### this is the error. i can't send big message in discord it automatically converted to text file Look at the par

**Problem:** this is the error. i can't send big message in discord it automatically converted to text file

**Solution:** Look at the part where it's a 3 by ??? grid of features

---

### this version of Circle is not a shape from kobweb, it's a Circle from svg library

**Problem:** this version of Circle is not a shape from kobweb, it's a Circle from svg library. I need a shape from this library to be able to apply the special svg style property like "stroke-dasharray"

**Solution:** Launched effect is a good way to get a coroutine yeah

---

### Yeah, I understood that, I just want a canvas that resize itself with the browser window. Do you know a better

**Problem:** Yeah, I understood that, I just want a canvas that resize itself with the browser window. Do you know a better method maybe?

**Solution:** I'd look into setting a max size on the buffer size so you don't blow away RAM on people with ultra-wide 4K monitors, but as for the style, resizing often should be fine

---

### I should be able to get the lang variable from any page of MyApp with a compositionLocalProvider but I don't k

**Problem:** I should be able to get the lang variable from any page of MyApp with a compositionLocalProvider but I don't know how to access it in my pages. ```Kotlin @App @Composable fun MyApp(content: @Composable () -> Unit) { var lang by remember { mutableStateOf("french") } val LocalLanguage = composition...

**Solution:** Here's how `AppGlobals` works: <https://github.com/varabyte/kobweb/blob/5d1259da76d7e589c4781a93da1f61f478fec557/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/core/AppGlobals.kt#L26>

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

---

### how do you set colors based on the colormode then

**Problem:** how do you set colors based on the colormode then?

**Solution:** Imagine something like this: ```kotlin // Option #1 - Classic val MyWidgetStyle by ComponentStyle { base { Modifier.color(colorMode -> ...) } } // Option #2 - Kobweb support // No new class for variables, Kobweb plugin finds all // ColorComputers and registers them for you val MyWidgetColorVar by StyleVariable<CSSColorValue>() val MyWidgetColorComputer = ColorComputer(MyWidgetColorVar) { colorMode -> ... } val MyWidgetStyle by ComponentStyle { base { Modifier.color(MyWidgetColorVar.value()) } } // Option #3 - Hybrid // New class for color variables. Convenient, but maybe // there are lots of other cases I'm missing (like variables // based on other state changes) val MyWidgetColorVar by SilkColorVariable { colorMode -> ... } val MyWidgetStyle by ComponentStyle { base { Modifier.color(MyWidgetColorVar.value()) } } // Option #4 - Explicit // What you'd have to do if you wanted to handle it yourself val MyWidgetColorVar by StyleVariable<CSSColorValue>() val MyWidgetStyle by ComponentStyle { base { Modifier.color(MyWidgetColorVar.value()) } } @Composable fun MyApp { val root = remember { document.getElementById("root") as HTMLElement } val colorMode by rememberColorMode() root.setVariable(MyWidgetColorVar, colorMode -> ...) } ```

```kotlin
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
// New class for color variables. Convenient, but maybe
// there are lots of other cases I'm missing (like variables
// based on other state changes)
val MyWidgetColorVar by SilkColorVariable { colorMode -> ... }

val MyWidgetStyle by ComponentStyle {
  base { Modifier.color(MyWidgetColorVar.value()) }
}

// Option #4 - Explicit
// What you'd have to do if you wanted to handle it yourself
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

### Hi, People. I'm looking for some help to create a scrollable function to scroll some images in a Column. The C

**Problem:** Hi, People. I'm looking for some help to create a scrollable function to scroll some images in a Column. The Column from the varybite repositories do not have this property using modifier. Any idea? Thnaks in advance!!

**Solution:** I recommend getting used to searching "css $propety" and hit the Mozilla url that shows up, here is: https://developer.mozilla.org/en-US/docs/Web/CSS/overflow

---

### also I'm wondering If can make a composable aware of the screen size

**Problem:** also I'm wondering If can make a composable aware of the screen size? maybe with `produceState(initialValue = window.innerWidth) { value = window.innerWidth }` ?

**Solution:** Any reason you're doing this on your own instead of configuring the breakpoints Kobweb provides for you? (Apologies if you said it earlier)

```kotlin
val SideBarColumnStyle by ComponentStyle {
    base { ... }
    Breakpoint.MD{
        Modifier.left((-100).px)
    }
}
```

---

### can I use multiple box shadow with

**Problem:** can I use multiple box shadow with `Modifier.boxShadow()` ?

**Solution:** Ah sorry doesn't look like it

---

### how to implement dotted lines like this

**Problem:** how to implement dotted lines like this. i was going to add icon but then i saw listStyleType in modifier but i dont know how to use it

**Solution:** Here's what it looks like using it in Compose HTML: <https://github.com/varabyte/kobweb/blob/47c4e10739bd8983fbf57840b6d295a75794cdfa/frontend/kobweb-silk/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/document/Toc.kt#L69>

---

### does anyone know that how to hide scrollbar using modifier

**Problem:** does anyone know that how to hide scrollbar using modifier?

**Solution:** You can change the overflow using a breakpoint block in a style component, would that help?

---

### Ahh. how to use the above css example in the modifier

**Problem:** Ahh. how to use the above css example in the modifier?

**Solution:** I'm guessing you're already looking at <https://www.w3schools.com/howto/howto_css_hide_scrollbars.asp>

---

### this works ```` Sorry that makes sense

**Problem:** this works ``` val SectionStyle by ComponentStyle { cssRule("::-webkit-scrollbar") { Modifier.display(DisplayStyle.None) } } ```

**Solution:** Sorry that makes sense. Yeah Component Style is probably the way here, nice catch.

```kotlin
val SectionStyle by ComponentStyle {
    cssRule("::-webkit-scrollbar") {
        Modifier.display(DisplayStyle.None)
    }
}
```

---

### how can I animate content visibility

**Problem:** how can I animate content visibility? like we do on AnimatedVisiblity on compose

**Solution:** You can also declare an animation, something like ``` val FadeInKeyframes by Keyframes { 0.percent { Modifier.opacity(0) } 100.percent { Modifier.opacity(1) } } ``` and then later ``` Modifier.animation(FadeInKeyframes.toAnimation()) ```

```kotlin
val FadeInKeyframes by Keyframes {
  0.percent { Modifier.opacity(0) }
  100.percent { Modifier.opacity(1) }
}
```

```
Modifier.animation(FadeInKeyframes.toAnimation())
```

---

### what i want is something like animateContentSize() Ah not sure, what happens if you set a transition for heigh

**Problem:** what i want is something like animateContentSize()

**Solution:** Ah not sure, what happens if you set a transition for height? ``` Modifier .transition(CSSTransition("height", 1.s)) ```

```
Modifier
  .transition(CSSTransition("height", 1.s))
```

---

### doesn't the padding stay the same

**Problem:** doesn't the padding stay the same?

**Solution:** This would be so easy if browsers allowed you to animate auto heights, but I guess that's technically complex

---

### Depends how many you have, you can divide it based on the number of columns/rows

**Problem:** Depends how many you have, you can divide it based on the number of columns/rows.

**Solution:** I should probably add ColumnScope / RowScope weight methods (which just call flexGrow). I'll need to check weight to make sure it isn't crazy complicated

---

### What I've settled into is using for general sizing, within widgets when I can, and for small things like borde

**Problem:** What I've settled into is using `cssRem` for general sizing, `percent` within widgets when I can, and `px` for small things like border radius. Not sure if it's the best but it's been working fine for me.

**Solution:** I might have remembered it wrong

---

### ```` I have this page Sure it will be included on other pages where you don't need it but maybe that doesn't m

**Problem:** ```kotlin @Page @OptIn(ExperimentalComposeWebApi::class) @Composable fun LoginPage() { var email by remember { mutableStateOf("") } var password by remember { mutableStateOf("") } Form(attrs = Modifier .fillMaxWidth(80.percent) .position(Position.Absolute) .top(50.percent) .left(50.percent) .tran...

**Solution:** Sure it will be included on other pages where you don't need it but maybe that doesn't matter? And it could check if the issue is because you're trying to dynamically add the script later

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

### might need help with interactive buttons Yes, exactly

**Problem:** might need help with interactive buttons

**Solution:** Yes, exactly. Just note that "light only" is not well supported across browsers at the moment: <https://developer.mozilla.org/en-US/docs/Web/CSS/color-scheme#browser_compatibility>

**Note:** note that "light only" is not well supported across browsers at the moment: <https://developer.

---

### that's weird, why light only is not well supported and dark only is supported everywhere Silk has its own conc

**Problem:** that's weird, why light only is not well supported and dark only is supported everywhere

**Solution:** Silk has its own concept of dark mode. You can configure the initial setup like this: <https://github.com/bitspittle/bitspittle.dev/blob/8637f2af8787aa96ee7f471ad6dd66f79aa68d82/site/src/jsMain/kotlin/dev/bitspittle/site/AppStyles.kt#L36>

---

### that's what I'm doing right now ````

**Problem:** that's what I'm doing right now

**Solution:** ```kotlin Column(Modifier.gap(1.cssRem)) { var showSecondBox by remember { mutableStateOf(false) } Box(Modifier.size(200.px).backgroundColor(Colors.Red) .onMouseOver { showSecondBox = true }.onMouseOut { showSecondBox = false }, contentAlignment = Alignment.Center) { Text("Hover me!") } Box( Modifier.size(200.px).backgroundColor(Colors.Blue).transition(CSSTransition("scale", 1.s)) .scale(0).thenIf(showSecondBox, Modifier.scale(1)) ) } ```

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

### how to add a hover style to a specific element inside a nested view To bring the comments from and together, y

**Problem:** how to add a hover style to a specific element inside a nested view

**Solution:** To bring the comments from and together, you should look at `window.fetch` (provided in the stdlib but Kobweb provides a few extensions). But you should also check out `window.http` which I added which layers on top of `window.fetch` (e.g. `window.http.get("https://www.google.com/search?q=varabyte+kobweb")`

---

### I've tried lucide l-react icons, i think we should also have those in kobweb how would I go about adding it 🤔 

**Problem:** I've tried lucide l-react icons, i think we should also have those in kobweb how would I go about adding it 🤔

**Solution:** That would be great if you wanted to give it a try. Check out <https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-icons-fa> and <https://github.com/varabyte/kobweb/tree/main/frontend/kobweb-silk-icons-mdi> to see how it's done for font awesome and Google material design icons (the latter submitted by )

---

### ~~What if I remove kobweb

**Problem:** ~~What if I remove kobweb?~~

**Solution:** Honestly I think css got overly complicated because they tried to make these catch all container types that do everything

---

### But what if you have a component style which includes breakpoints, hover,state,inactive..etc effects

**Problem:** But what if you have a component style which includes breakpoints, hover,state,inactive..etc effects.

**Solution:** I can appreciate some users in the future might use TW and exclude Silk. And also a bunch of existing component libraries in the wild users can leverage, like yzziK's impressive controls demo.

---

### I really don't see how tw classes are better in this case

**Problem:** I really don't see how tw classes are better in this case. opposed to readable and reusable component styles <a:shruganimated:421995622346784768>

**Solution:** Well Kobweb can be minimally good for routing and re-using multiplatform Kotlin code

---

### Classes isn't a styleModifier though, right

**Problem:** Classes isn't a styleModifier though, right?

**Solution:** I think font size is the way to go, or use a `rememberBreakpoint` and pass in a different size into your fa icon calls

---

### how do you get console on android browser

**Problem:** how do you get console on android browser ?

**Solution:** https://stackoverflow.com/questions/37256331/is-it-possible-to-open-developer-tools-console-in-chrome-on-android-phone ?

---

### for backgroundImage , url of image should provide from where i mean from site/.../resource/public/image.png or

**Problem:** for backgroundImage , url of image should provide from where i mean from site/.../resource/public/image.png or anything else?

**Solution:** I believe if your image is under "jsMain/resources/public" then it should be `url("/image.png")`

---

### i can make any component focusable/non in compose I think tabindex is how you do it in css

**Problem:** i can make any component focusable/non in compose

**Solution:** I think tabindex is how you do it in css? <https://stackoverflow.com/questions/716235/how-to-make-a-div-unfocusable>

---

### how do i prevent content going outside of a container Check out <https://developer.mozilla.org/en-US/docs/Web/

**Problem:** how do i prevent content going outside of a container

**Solution:** Check out <https://developer.mozilla.org/en-US/docs/Web/CSS/overflow> ?

---

### can you share how you are applying the animation in a Modifier

**Problem:** can you share how you are applying the animation in a Modifier?

**Solution:** What happens if you set the duration to 10 seconds or something ridiculous?

---

### Adding a background color to the style gives me this, am i helping in debugging

**Problem:** Adding a background color to the style gives me this, am i helping in debugging ?

**Solution:** 1) you can query `colorMode` inside of it as a property if you want to have that effect of a different filter (although don't add it yet until you're done testing why it's not working) 2) you don't need a prefix if the style is in your code, that's just meant for libraries

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

### Honestly, that value is confusing because I have but that should really be called

**Problem:** Well thanks everyone for help in animations yesterday and filter today i've added a navbar and social media links buttons Tbh the coolest thing is the overlapping feature when nav bar overlaps with a section is changes the title of the nav bar https://elkhoudiry.github.io/

**Solution:** Actually, `getColorMode` is fine to use without a remember. Honestly, that value is confusing because I have `rememberColorMode` but that should really be called `rememberColorModeState`

---

### when adding filter to an image, why its moving to the top layer

**Problem:** when adding filter to an image, why its moving to the top layer ?? in a box of overlay bg ,content ,something else, filter applied bg comes to the front 🤔

**Solution:** "Congratulations" I think you get to learn about CSS stacking contexts now: <https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_positioned_layout/Understanding_z-index/Stacking_context>

---

### ```` the bg just go black here opacity nothing wont be applied to it\ And you want it to also serve your Kobwe

**Problem:** ```kt Box( modifier = Modifier .width(100.vw).height(100.vh), contentAlignment = Alignment.Center ) { PatternBackground ( Modifier.styleModifier { filter { grayscale(50.percent) } } .fillMaxSize() .objectFit(ObjectFit.Cover) ) Column( modifier = Modifier.isolation(Isolation.Isolate) ) { DesktopCo...

**Solution:** And you want it to also serve your Kobweb files?

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

### And I hit it on moible just because of the reduced dimensions

**Problem:** Hello! I have a trouble: on phone my site has strange white background, there is code: ```kt @Composable fun markdownLayout(content: @Composable () -> Unit) { Box( Modifier .fillMaxSize() .minHeight(100.percent) .gridTemplateRows { size(1.fr); size(minContent) } ) { Column(Modifier.fillMaxSize())...

**Solution:** so the web things the page is wider than 100% because of it. And I hit it on moible just because of the reduced dimensions.

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

### how do I controll the size of a text area

**Problem:** how do I controll the size of a text area. Setting this doesnt seem to work ``` TextArea(attrs = Modifier.attrsModifier { style { this@attrsModifier.width(1000.px) } }.toAttrs(), value = postBody.value) ```

**Solution:** ``` attrs = { width(...) } ```

```
TextArea(attrs = Modifier.attrsModifier {
                style {
                    this@attrsModifier.width(1000.px)
                }
            }.toAttrs(), value = postBody.value)
```

---

### Can anyone tell me how to give a gradient as a background color please

**Problem:** Can anyone tell me how to give a gradient as a background color please. I tried this, but it doesn't seem to be working

**Solution:** Yeah, sorry, Kobweb can only protect users so much from CSS. Fun fact, the "background" vs "background color" difference between Android Jetpack Compose and CSS was the main thing that made me give up fighting CSS and deciding to let CSS win. (At first I wanted Kobweb to provide a Jetpack Compose API that happened to delegate to HTML / CSS)

---

### Also text-transform: uppercase converts to .textTransform(TextTransform.Uppercase) which is not even available

**Problem:** Also text-transform: uppercase converts to .textTransform(TextTransform.Uppercase) which is not even available. Any suggestions on this one?

**Solution:** If things are missing in Kobweb that's my bug. I'll check Text transform shortly. LoP can look at the other ones and let me know if Kobweb is doing it wrong

---

### Another question, right now I'm padding single column divs to give space between them. Is there a better way t

**Problem:** Another question, right now I'm padding single column divs to give space between them. Is there a better way to handle adding space between columns in the SimpleGrid?

**Solution:** `Modifier.gap` is amazing, I use it frequently

---

### What's in the divs is not relevant But what you're doing is creating a 3 column div inside a simple grid right

**Problem:** What's in the divs is not relevant

**Solution:** But what you're doing is creating a 3 column div inside a simple grid right?

---

### I'm guessing that what's happening now is that the Link's style is taking precedence over your ComponentStyle,

**Problem:** I'm guessing that what's happening now is that the Link's style is taking precedence over your ComponentStyle, you should be able to check whether that's the case in dev tools

**Solution:** `(Breakpoint.MD + anyLink) { ... }` try that?

---

### Looks like you want fillmaxwidth

**Problem:** Looks like you want fillmaxwidth?

**Solution:** It's still not clear to me what you're trying to do.

---

### I want to center the green container as you see in the image 😄 ````

**Problem:** I want to center the green container as you see in the image 😄

**Solution:** ```kotlin Column(Modifier.fillMaxWidth(), horizontalAlignment = CenterHorizontally) { Column(...) { } } ```

```kotlin
Column(Modifier.fillMaxWidth(), horizontalAlignment = CenterHorizontally) {
   Column(...) {
   }
}
```

---

### Using silk-specific 'toModifier', 'SpanText' causes an error. Is there a workaround

**Problem:** Using silk-specific 'toModifier', 'SpanText' causes an error. Is there a workaround?

**Solution:** It looks like you're either not applying the Kobweb plugin/ you're importing silk foundation only

---

### My usecase requires rendering Markdown text, which I achieve using md-block library, which basically creates a

**Problem:** My usecase requires rendering Markdown text, which I achieve using md-block library, which basically creates a custom html tag that does it under the hood. In Compose HTML I implement that using TagElement composable. To include that in a Kobweb composable do I need to do anything special, or doe...

**Solution:** Actually I don't think so. I have a layer in Kobweb called `compose-html-ext` which is actually Kobweb agnostic. That said, you can always call Modifier.toAttrs to adapt.

---

### Hello Everybody, anybody know tell me how manipulate cssRule for modify only a Box into a Column

**Problem:** Hello Everybody, anybody know tell me how manipulate cssRule for modify only a Box into a Column? Like:

**Solution:** What if you tag the horizonal line with an ID or class name and then use the selector for that? Also, I believe the horizontal rule has to be a child?

---

### Is there a "visibility" property in CSS

**Problem:** Is there a "visibility" property in CSS? Also, where can I see all existing CSS properties?

**Solution:** Please read https://github.com/varabyte/kobweb#learning-css-through-kobweb it should help a lot!

---

### Is there easy way of converting CSSColorValue to Kobweb Color

**Problem:** Is there easy way of converting CSSColorValue to Kobweb Color?

**Solution:** Why do you need a Kobweb color? The CSSColorValue class is unfortunately totally opaque so I don't think there's a way to pull the r, g, b values out of it.

---

### Hi guys, I'm trying to apply a box-shadow to my header, but it is not applied, by inspecting the page I can se

**Problem:** Hi guys, I'm trying to apply a box-shadow to my header, but it is not applied, by inspecting the page I can see that the css property has been applied to the header, would anyone have a suggestion as to what could be impacting it? I am applying it as follows: ``` .boxShadow(offsetX = 5.px, offset...

**Solution:** When fighting CSS issues my recommendation is to use browser dev tools to discover values that work there first

```
.boxShadow(offsetX = 5.px, offsetY = 1.px, blurRadius = 4.px, spreadRadius = 0.px, color = Color.rgba(46, 46, 70, 1f))
.styleModifier {
       property("-webkit-box-shadow", "5px 1px 4px 0px rgba(46,46,70,1)")
       property("-moz-box-shadow", "5px 1px 4px 0px rgba(46,46,70,1)")
}
```

---

### Hello, how to implement adaptive layout on kobweb By adaptive, you mean mobile vs desktop

**Problem:** Hello, how to implement adaptive layout on kobweb

**Solution:** By adaptive, you mean mobile vs desktop?

---

### You can define your own s (https://github.com/varabyte/kobweb?tab=readme-ov-file#componentstyle): ```` Wow tha

**Problem:** You can define your own `ComponentStyle`s (https://github.com/varabyte/kobweb?tab=readme-ov-file#componentstyle): ```kt val MyDropDownStyle by ComponenetStyle { //... } BSDropdown(modifier = MyDropDownStyle.toModifier() ```

**Solution:** Wow that's been in there for a while. Fixing!

```kt
val MyDropDownStyle by ComponenetStyle {
//...
}
BSDropdown(modifier = MyDropDownStyle.toModifier()
```

---

### I have added Tailwind CSS to Kobweb project. When I add any Tailwind class using Modifier, it works. The expec

**Problem:** I have added Tailwind CSS to Kobweb project. When I add any Tailwind class using Modifier, it works. The expected change gets reflected on the website. **My question is**: Can we make our IntellijIdea provide suggestions when we write Tailwind class? Generally I noticed that when I use HTML or CS...

**Solution:** There's tailwind support in ultimate (<https://www.jetbrains.com/help/idea/tailwind-css.html#ws_css_tailwind_configuration>) but I'm pretty sure not kt files but I could be wrong

---

### One thing you can try is just taking a list of tailwind classes and generating a simple kotlin file like ```` 

**Problem:** One thing you can try is just taking a list of tailwind classes and generating a simple kotlin file like ```kt fun Modifier.w24() = this.classes("w-24") fun Modifier.textLg() = this.classes("text-lg") ... ```

**Solution:** Maybe a future Kobweb-aware library? 😛

```kt
fun Modifier.w24() = this.classes("w-24")
fun Modifier.textLg() = this.classes("text-lg")
...
```

```kt
sealed interface TailwindClass

fun Modifier.classNames(vararg classes: TailwindClass) = this.classNames(*classes.unsafeCast<Array<String>>()) // or Modifier.tw(...) if you like brevity

inline val w24 get() = "w-24".unsafeCast<TailwindClass>()
inline val textLg get() = "text-lg".unsafeCast<TailwindClass>()
```

---

### doesn't work for me unless I directly add to my page `````` after setting the initial color mode

**Problem:** doesn't work for me unless I directly add to my page ```kt var colorMode = ColorMode.current colorMode = ColorMode.LIGHT```

**Solution:** What happens if you do ``` println(ColorMode.current.name) ``` after setting the initial color mode?

```kt
var colorMode = ColorMode.current
    colorMode = ColorMode.LIGHT
```

---

### Is there a way to use Bootstrap but not have it be so aggressive with its css layers? Its prioritized over my 

**Problem:** Is there a way to use Bootstrap but not have it be so aggressive with its css layers? Its prioritized over my css styles. Saw some older discussion through searching here, but wasn't able to find any solution. Anyone have recommendations?

**Solution:** <https://github.com/varabyte/kobweb?tab=readme-ov-file#importing-third-party-styles-into-layers>

---

### Was ComponentStyle removed? I feel so out of touch

**Problem:** Was ComponentStyle removed? I feel so out of touch...

**Solution:** So just wanted to let you know that was the biggest change by far. We don't have any more plans that significant from now until 1.0

---

### with the new layout stuff, how do I pass data from markdown files to layouts

**Problem:** with the new layout stuff, how do I pass data from markdown files to layouts?

**Solution:** Markdown values + @Layout

---

### isn't that how it's supposed to work

**Problem:** isn't that how it's supposed to work?

**Solution:** Correct, for `attrsModifier` you'd need to manually cast the `this` scope (maybe? Now that I've typed that I'm not sure if it will work)

---

### I have a simple question: With the current state of Silk, is it possible to create an entire responsive websit

**Problem:** I have a simple question: With the current state of Silk, is it possible to create an entire responsive website without divs (only using row, column and box) and only rely on low level Compose HTML compostables for specific styling and functionality (like input, span, etc)?

**Solution:** It's hard to say something definitive because every now and then you run across a case where CSS wins and at that point it can be better to use a raw html element. But generally I sculpt a site with high level Compose concepts yes

---

### Routing

### so this is for seo for crawlers that don't exercise the JS? I don't understand/remember how dynamic pages prov

**Problem:** so this is for seo for crawlers that don't exercise the JS? I don't understand/remember how dynamic pages provide content for crawlers so they can be indexed.

**Solution:** Also, I figure maybe later I can look into hydration more -- I'm not sure it's currently possible with the way compose works, but that might change

---

### '''2022-02-07T15:56:17.135717+00:00 app[web.1]: Unable to list file systems to check whether they can be watched

**Problem:** '''2022-02-07T15:56:17.135717+00:00 app[web.1]: Unable to list file systems to check whether they can be watched. Assuming all file systems can be watched. Reason: Could not query file systems: could not open mount file (errno 2: No such file or directory) 2022-02-07T15:56:26.637035+00:00 heroku[...

**Solution:** So what's recommended is running these commands, you can do this locally to test for now ``` > kobweb export --mode dumb > kobweb run --env prod --mode dumb ```

```
> kobweb export --mode dumb
> kobweb run --env prod --mode dumb
```

---

### do you need this verticalAlignment there? Row want be any bigger than your image anyway It's tricky

**Problem:** do you need this verticalAlignment there? Row want be any bigger than your image anyway

**Solution:** It's tricky. I mean, I want it to work the same as much as possible. I'm on mobile now but will look more at your question and link to the official Compose docs later.

---

### I take it you meant this one? https://github.com/varabyte/kobweb-site/blob/main/Dockerfile Like, I started som

**Problem:** I take it you meant this one? https://github.com/varabyte/kobweb-site/blob/main/Dockerfile

**Solution:** Like, I started something with feature A which, behind my back, registered feature B for me. And then when I stopped feature A, feature B was still sitting there demanding payment

---

### BTW, this is the code I have to make step 2 in the guide automatically apply to my ```` That's amazing

**Problem:** BTW, this is the code I have to make step 2 in the guide automatically apply to my `index.html` ``` kobweb { index { description.set("Powered by Kobweb") head.add { script(type="text/javascript"){ consumer.onTagContent( """ | | (function(l) { | if (l.search[1] === '/' ) { | var decoded = l.search...

**Solution:** That's amazing. I may create a blog post about it at some point if that's alright with you.

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

### I'll try it and see what happens

**Problem:** I'll try it and see what happens. More likely, I'll try it then wrap it up for tonight and try tomorrow 🙂

**Solution:** (What I use chrome for is loading the html + js generated by Kotlin, which at that point is one giant single page app, visit each route, let the js engine process the page, and then save the resulting dom out into an html snapshot)

---

### I was reading readme last night and in the section Markdown you can explain how to create links between markdo

**Problem:** I was reading readme last night and in the section Markdown you can explain how to create links between markdowns. I worked it myself. Easy anyway, but may be helpful for new users.

**Solution:** Maybe I should allow overriding that in the YAML too

---

### What's the point of

**Problem:** What's the point of `focus + active` ? Shouldn't the `active` block override `focus` anyway?

**Solution:** Before the widget was backed by a div, and now it's backed by an actual button

---

### hi all! i'm playing again with kobweb and im running into an issue: if i try to run from a submodule (so non-t

**Problem:** hi all! i'm playing again with kobweb and im running into an issue: if i try to run `gradlew kobwebStart` from a submodule (so non-top level project), kobweb server won't start reproducer: https://github.com/DVDAndroid/kobweb-issue1 i've tried to fix it by changing the command working directory b...

**Solution:** The error message means it can't find a .kobweb folder. But you have one?

---

### So, I started attempting to convert my project to a multimodule format. Mostly good so far, but I've hit one i

**Problem:** So, I started attempting to convert my project to a multimodule format. Mostly good so far, but I've hit one issue. Pages in my non-main module (with no .kobweb) seem to not work. I get a 404 error when I try to navigate to one, and I'd assume that's cuz `build\processedResources\js\main\META-INF...

**Solution:** I tested it on windows at one point but maybe it broke again

---

### chat example does work Yeah I just looked up the output for , looks like:

**Problem:** chat example does work

**Solution:** Yeah I just looked up the `META-INF` output for `chat/auth`, looks like: `{"pages":[{"fqn":"chat.auth.pages.account.CreateAccountPage","route":"/account/create"},{"fqn":"chat.auth.pages.account.LoginPage","route":"/account/login"}],"kobwebInits":[],"silkInits":[],"silkStyles":[],"silkVariants":[]}`

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

### Don't we have for text in Kobweb yet

**Problem:** Don't we have `max-lines: [none | <integer>]` for text in Kobweb yet?

**Solution:** Got a link to the API? I did a quick search but it seems like max-lines is experimental?

---

### Saw this bug about allowing paths ending in , is it by any chance possible to do some hack that would allow th

**Problem:** Saw this bug about allowing paths ending in `.html`, is it by any chance possible to do some hack that would allow this? https://github.com/varabyte/kobweb/issues/153

**Solution:** Somewhere in your code, add ``` @Composable private fun CustomErrorPage(errorCode: Int) { println(window.location.href) Div { Text("Error code: $errorCode") } } @InitKobweb fun initKobweb(ctx: InitKobwebContext) { ctx.router.errorPage = CustomErrorPage } ```

```kotlin
@Composable
private fun CustomErrorPage(errorCode: Int) {
   println(window.location.href)
   Div {
        Text("Error code: $errorCode")
   }
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

### What are your thoughts on having it work like the new SilkPalette? ```` seems nice to me and gets rid of the r

**Problem:** What are your thoughts on having it work like the new SilkPalette? ``` addRouteInterceptor { if (path == "entry.html") path = "entry" } ``` seems nice to me and gets rid of the returning null for no changes which didn't seem ideal

**Solution:** I was worried someone might intercept a URL, return a slightly modified path, and suddenly the query params and fragment order would flip around. But it shouldn't happen

---

### Just a note that it's possible already to make links ending in html work using a route interceptor, but the op

**Problem:** Just a note that it's possible already to make links ending in html work using a route interceptor, but the option needs to specifically keep the html suffix in the url, as otherwise reloading can send you to an invalid path

**Solution:** Like, if you want ".html" endings to be supported, should Kobweb always append them? Even if you visit a link without putting it there explicitly?

---

### Make a kobweb template for a data deletion site and profit? Technically it should be fairly trivial

**Problem:** Make a kobweb template for a data deletion site and profit?

**Solution:** Technically it should be fairly trivial. You basically need one page (in jsMain) and one API route (in jvmMain). The jsMain page has a message, an input field for the account, and a button to submit the delete request. The API request handles the request, sends an API request to some other server to handle the deletion, and then responds back with the result.

---

### So i can just use val socket = remember { Websocket() }

**Problem:** So i can just use val socket = remember { Websocket() } ... socket.connect()

**Solution:** ```kotlin sealed interface PageState { object Connecting : PageState, class Connected(websocket: Websocket) : PageState, ... } @Page @Composable fun somePage() { var state by remember { mutableStateOf(PageState.Connecting) } when (state) { is PageState.Connecting { LaunchedEffect(Unit) { val websocket = Websocket() websocket.connect() state = PageState.Connected(websocket) } } is PageState.Connected { ... } } } ```

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

### I have a file.mp4 that I would like to display on a webpage. How can I do that with kobweb

**Problem:** I have a file.mp4 that I would like to display on a webpage. How can I do that with kobweb ?

**Solution:** Actually the mp4 should probably be in a different folder called `videos`, I'll fix that shortly, but you can see I put it inside the `resources/public` folder: <https://github.com/varabyte/kobweb-site/blob/f2f832734838241a3862eb123486dd222e519a12/src/jsMain/resources/public/images/kobweb-cli.mp4>

---

### can i override the server status pages? I assume that's what you mean

**Problem:** can i override the server status pages?

**Solution:** I assume that's what you mean. If so, you need to call `Router#setErrorHandler`, which you should probably do in an `@InitKobweb` method somewhere. Off the top of my head: ``` @InitKobweb fun initKobweb(ctx: InitKobwebContext) { ctx.router.setErrorHandler { errorCode -> ... } ```

```kotlin
@InitKobweb
fun initKobweb(ctx: InitKobwebContext) {
  ctx.router.setErrorHandler { errorCode ->
   ...
}
```

---

### how do we implement drop down with kobweb? Note that a pulldown element is planned to be added to silk in the 

**Problem:** how do we implement drop down with kobweb?

**Solution:** Note that a pulldown element is planned to be added to silk in the next few versions. But using the link shared above you can definitely implement it yourself.

**Note:** Note that a pulldown element is planned to be added to silk in the next few versions.

---

### the default page of kobweb is always Index.kt right. just curious how to change the default page to something 

**Problem:** the default page of kobweb is always Index.kt right. just curious how to change the default page to something else?

**Solution:** Other ways you can do this are... 1) have your index page simply delegate to another page by calling it directly 2) Use a route interceptor for redirects: <https://github.com/varabyte/kobweb/blob/fcda655ef9ba10512377c925ccde758517bf12ea/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/Router.kt#L204>, catch path == "" and change it to where you want it to go

---

### i can just use a call back instead Something like: ````

**Problem:** i can just use a call back instead

**Solution:** Something like: ``` private inline fun <reified T> HTMLElement.nextSiblingOf(): T? { var next = nextSibling while (next != null) { if (next is T) { return next } next = next.nextSibling } return null } Input( InputType.Text, attrs = { lateinit var element: HTMLElement ref { element = it; onDispose { } } placeholder("Input #1") onKeyDown { event -> if (event.key == "Enter") { element.nextSiblingOf<HTMLInputElement>()?.focus() } } } ) ```

```
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

### I think that might happen if an exception occurs? Definitely check the console if that ever happens

**Problem:** I think that might happen if an exception occurs? Definitely check the console if that ever happens. If not though sounds like a bug

**Solution:** and if an exception is thrown, route to another page

---

### ~~You can set the cursor properties with classes~~ In addition to what LoP said (which is spot on), you can pr

**Problem:** ~~You can set the cursor properties with classes~~

**Solution:** In addition to what LoP said (which is spot on), you can probably find a lot of articles like <https://karlgroves.com/links-are-not-buttons-neither-are-divs-and-spans/> and <https://javascript.plainenglish.io/stop-using-divs-for-buttons-87a0b3d7945e>. Basically, you have three options you can use to create "buttons" -- <button>, <a>, and <span>. There are cases I think to use <a> sometimes, when your button is really just a glorified link, but.... in general, if you think of something conceptually as a button, you should probably use a button.

---

### Wait so static sites can't have markdown pages

**Problem:** Wait so static sites can't have markdown pages ?

**Solution:** I would need to extract Kobweb server logic out into a library, which is trickier than it sounds because I'm doing a bunch of sophisticated stuff for live reloading

---

### Using ComponentPreview

**Problem:** oh i managed to make my react components work without using any wrapper for each component i think i dont need import extension property anymore now im just calling it like ```kotlin {{{ .components.CPreview { example.shadcn_kotlin.ui.pages.components.demo.AccordionDemo {} } }}} ``` and this is m...

**Solution:** Hey just got back from a morning out and the more I think about your import request the more I love it

```kotlin
{{{ .components.CPreview { example.shadcn_kotlin.ui.pages.components.demo.AccordionDemo {} } }}}
```

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

```kotlin
external interface ComponentPreviewProps: Props {
    var code: String
    var component: ReactNode
    // Syntax Highlighting doesn't work in Components Preview as the code tab is not visible
    // when `Prism.highlightAll()` is called
    var lang: String
    var previewChild: (ChildrenBuilder) -> Unit
}
```

---

### Did you setup a "redirect" or a "rewrite" to index.html

**Problem:** Did you setup a "redirect" or a "rewrite" to index.html. From what I understand a rewrite doesn't change the client side url, which would then make it possible for kobweb to serve the dynamic route page even from index.html

**Solution:** According to your error message you also have Java 20 installed somewhere on your machine and gradle seems to be finding that one

---

### how does one register an event listener and remove it

**Problem:** how does one register an event listener and remove it ? in react they just put the remove call in return callback of useEffect

**Solution:** by the way latest Kobweb has a helper EventListenerManager class which makes disposal a bit easier if you have multiple listeners

---

### should'nt this in frontmatter ```````` maybe im writing yaml incorrectly

**Problem:** should'nt this in frontmatter ```yaml radix: - link: https://www.radix-ui.com/docs/primitives/components/toast - api: https://www.radix-ui.com/docs/primitives/components/toast#api-reference ``` get converted into ```kotlin "radix" to listOf("link" to "https://www.radix-ui.com/docs/primitives/comp...

**Solution:** You can probably import a yaml parser (I use kaml) to parse the children further

```kotlin
"radix" to listOf("link" to "https://www.radix-ui.com/docs/primitives/components/toast", "api" to "https://www.radix-ui.com/docs/primitives/components/toast#api-reference")
```

```kotlin
"radix" to listOf("link: https://www.radix-ui.com/docs/primitives/components/toast", "api: https://www.radix-ui.com/docs/primitives/components/toast#api-reference")
```

```kotlin
"radix" to listOf(), "link" to listOf("https://www.radix-ui.com/docs/primitives/components/toast"), "api" to listOf("https://www.radix-ui.com/docs/primitives/components/toast#api-reference"))
```

---

### actually, does the ktor auth plugin work for the kobweb backend

**Problem:** actually, does the ktor auth plugin work for the kobweb backend?

**Solution:** However, if you just want the sort of auth you get when you visit a page, like "Please enter your email and password to continue", that should be possible entirely on the frontend

---

### So if we add pages that are not in the initial directory and provide a route, ie web dev service pages go to r

**Problem:** So if we add pages that are not in the initial directory and provide a route, ie web dev service pages go to route services/web, how to get the images to display?

**Solution:** Images should go into your resources/public directory

---

### Sorry to keep bothering you but how about this google analytics script that needs to be added right after the 

**Problem:** Sorry to keep bothering you but how about this google analytics script that needs to be added right after the <head> element: ```<!-- Google tag (gtag.js) --> <script async src="https://www.googletagmanager.com/gtag/js?id=G-K9W7KBLCQ4"></script> <script> window.dataLayer = window.dataLayer || [];...

**Solution:** Ah sorry, at some point I want to support body tags in the kobweb block as well

---

### Can you post the code

**Problem:** Can you post the code?

**Solution:** The problem is, the way Kobweb works is it renders your page first with a snapshot, then reruns javascript and runs it again

---

### Can you fill an issue ? My knowledge about ksp is

**Problem:** Can you fill an issue ? My knowledge about ksp is... zero 😅

**Solution:** In general, even if your knowledge about something is zero, a useful way to write a bug is: ``` I'm seeing this error: ... Here a link to my project: ... To repro, do this: ... ```

---

### I have this snippet here and no recomposition takes places when the fields on change ```Textsalso```

**Problem:** Hey all. I have this snippet here and no recomposition takes places when the fields on `patient` change ```kotlin var patient: Patient by remember { mutableStateOf(Patient()) } //Default constructor just inits everything with "N/A" LaunchedEffect(patientId) { patient = getPatientData(patientId) /...

**Solution:** And for sure it's actually returning successfully? A neat trick is to use `also` like this (one of its few use cases IMO): ```kotlin suspend fun getPatientData() { ... a bunch of stuff ... return patientData .also { println("Returning patient data: ${it.name}") } } ```

```kotlin
var patient: Patient by remember { mutableStateOf(Patient()) }  //Default constructor just inits everything with "N/A"
LaunchedEffect(patientId) {
    patient = getPatientData(patientId) //API call where the actual values are downloaded
}
Column {
    Row {Text("Name: ${patient.name}")}
    Row {Text("Surname: ${patient.surname}")}
    Row {Text("Disease: ${patient.disease}")}
}
```

```kotlin
suspend fun getPatientData() {
   ... a bunch of stuff ...
   return patientData
     .also { println("Returning patient data: ${it.name}") }
}
```

```kotlin
private suspend fun getPatientData(patientUuid: String): Patient {
    println("getPatientData called")
    var result = Patient()
    runCatching {
        result = SupabaseModule.getPatient(patientUuid)
    }
        .onSuccess {
            println("getPatientData result: $result")
            return result
        }
        .onFailure {
            if (it is NoSuchElementException)
                getPatientData(patientUuid)
        }
    return result
}
```

---

### Using patient

**Problem:** It certainly seems like the values should get updated, not sure why they wouldn't be

**Solution:** but I've done stuff like this before: ```kotlin val patient = CompletableDeferred<Patient>() somethingWithCallbacks .onSuccess { patient.complete(result) } .onFailure { patient.complete(Patient.from(patientUuid)) } return patient.await() ```

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

### Hello all, How you guys support navigation in Kobweb with managing state? Suppose I have 2 pages: Home and Pro

**Problem:** Hello all, How you guys support navigation in Kobweb with managing state? Suppose I have 2 pages: Home and Projects. In the home screen, I have dropdown which I have clicked and opened. Now, how do I navigate to project page so that when I came back to Home, that drop-down will still be opened.

**Solution:** Search this server history for "LocalStorage" and "SessionStorage", pretty sure you want one of those two (depending if you want the pulldown value to survive for a long time or just until you close the tab)

---

### This is Kobweb's poweeeeeeeeeer

**Problem:** Good evening everyone, I finished my page and I'm here to share it in the group, thank you to everyone who helped me answer my questions in this process. This is Kobweb's poweeeeeeeeeer! https://fmoreiradeveloper.com/

**Solution:** Congratulations!! Yeah I'll pin it in showcase

---

### Hello all, With Kobweb backend, when i use , it gets me all query parameters. Do we have support for path para

**Problem:** Hello all, With Kobweb backend, when i use `ctx.req.params`, it gets me all query parameters. Do we have support for path parameters?

**Solution:** The reason they're supported on the frontend is because it leads to really nice, clean urls that users can see and share. But the backend is triggered by code only, so we decided it wasn't necessary there.

---

### Just to clarify, when exactly does this happen? When first starting the server, when rebuilding after code cha

**Problem:** Just to clarify, when exactly does this happen? When first starting the server, when rebuilding after code changes, and/or whenever you reload the page?

**Solution:** I haven't particularly noticed myself, but my guess is this is just the result of being a dev build. Try running `kobweb export` followed by `kobweb run --env prod` to see if it's much faster? I could also double check that I'm not doing anything accidentally anywhere in Kobweb.

---

### **navigateTo and States** Hello everyone, I would like some help related to navigateTo and state management, h

**Problem:** **navigateTo and States** Hello everyone, I would like some help related to navigateTo and state management, have a SideBar that receives an inner side bar specific to each page, inside the SideBar I have the following code ```Kotlin @Composable fun SideBar(innerSideBar: @Composable (() -> Unit)?...

**Solution:** <https://github.com/bitspittle/bitspittle.dev/blob/f4e419bf1c017551cc6c1ea7d95b9a1d157c157f/site/src/jsMain/kotlin/dev/bitspittle/site/components/layouts/PageLayout.kt#L56> sends an analytics ping every time a new page is visited. This code is a bit old and now I think I'd be using the page context instead but the idea is the same

```kotlin
@Composable
fun SideBar(innerSideBar: @Composable (() -> Unit)? = null) {
    val innerBarIsPresent = innerSideBar == null

    var sideBarIsClosed by remember { mutableStateOf(innerBarIsPresent) }

    fun setSideBarIsClosed() {
        sideBarIsClosed = !sideBarIsClosed
    }
    
    // more code here...
}
```

---

### Hey yall, I have been using the routePrefix settings: https://github.com/varabyte/kobweb?tab=readme-ov-file#se

**Problem:** Hey yall, I have been using the routePrefix settings: https://github.com/varabyte/kobweb?tab=readme-ov-file#setting-your-sites-route-prefix I noticed that they are case sensitive. myexample.com/myroute is not accessable with myexample.com/MYROUTE. Anyone have tips to get passed this? Thank you!

**Solution:** <https://github.com/varabyte/kobweb/blob/38c22650e36eedf103f09dfe7e43739f62a52c01/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/Router.kt#L288>

---

### Hey! Is there an easy way to use a ts npm library in kobweb? I saw that with Kotlin/JS you have to write the t

**Problem:** Hey! Is there an easy way to use a ts npm library in kobweb? I saw that with Kotlin/JS you have to write the type definitions by hand, but the lib is in TS so it already contains all the types. Any advice?

**Solution:** I think there was at one point something experimental that JB provided but they pulled back on. Not 100% sure. Kotlin != JS/TS in some fundamental ways so it's probably not trivial.

---

### Heyo, ran into a weird error. My site was working fine running on localhost. Inside my project I created a fil

**Problem:** Heyo, ran into a weird error. My site was working fine running on localhost. Inside my project I created a file, then realized I didn't need it and also deleted another one I saw I didn't need, just cleaning up. Both Kotlin files had nothing in them. After deleting, my index page says 'Error code...

**Solution:** I'm sure it's fine. Maybe our KSP logic got confused. Maybe quit the running server, run `./gradlew clean`, and try starting it again

---

### And how to navigate to the defined root

**Problem:** And how to navigate to the defined root ? ctx.router.navigateTo("/") sends me to https://xibalbam.github.io/ instead of https://xibalbam.github.io/AJTextGameEngine/

**Solution:** Use [`RoutePrefix.prepend("/")`](<https://github.com/varabyte/kobweb/blob/da9ea932d01d770cdd6cf2afce50edc9b615fde2/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/navigation/RoutePrefix.kt#L25>)

---

### I did, I had to refresh the page manually each time

**Problem:** I did, I had to refresh the page manually each time... In the end, I added a `<meta>` to auto-refresh, since you showed me how

**Solution:** `./gradlew :site:kobwebStart ...` does not live reload `../gradlew kobwebStart ...` from within the site folder *does* reload.

---

### Is there a page that shows/demos all the available Silk widgets? If you want to see something a bit more raw i

**Problem:** Is there a page that shows/demos all the available Silk widgets?

**Solution:** If you want to see something a bit more raw in the meanwhile, clone the Kobweb project. ```bash $ git clone https://github.com/varabyte/kobweb $ cd kobweb/playground/site $ kobweb run ```

```
$ git clone https://github.com/varabyte/kobweb
$ cd kobweb/playground/site
$ kobweb run
```

---

### Hi, Is there a recommended way to read a properties file in a Kobweb static site? I’m considering placing a co

**Problem:** Hi, Is there a recommended way to read a properties file in a Kobweb static site? I’m considering placing a config.properties file in the resources directory, then using window.fetch("<path>/config.properties").await() to retrieve it and parse it. Would this be the correct approach?

**Solution:** Do you need the information to be dynamic by the way, or edited by someone that's not you? You can also use app globals if that's easier

---

### Is there a way to display an image thumbnail, dynamically in the open graph meta tags, when pasting the link o

**Problem:** Is there a way to display an image thumbnail, dynamically in the open graph meta tags, when pasting the link on social media?

**Solution:** I don't think Open Graph meta tags are meant to be personalized like that? Maybe I'm wrong. I think they're supposed to show up for when, say, I share a link to a site in Discord, and Discord wants to show a preview for everyone?

---

### that preview when sharing a link, currently it's taking the first image from the page, but I need to customize

**Problem:** that preview when sharing a link, currently it's taking the first image from the page, but I need to customize it based on the thumbnail data which is fetched from the database

**Solution:** Are you already doing something like this? <https://kobweb.varabyte.com/docs/concepts/foundation/page-metadata#page-specific-metadata>

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

### Hi all, I want to implement a 'locking a resource' mechanism where i would use the API Streams to broadcast th

**Problem:** Hi all, I want to implement a 'locking a resource' mechanism where i would use the API Streams to broadcast that event for all clients observing that resource. However, i find it difficult to do that. The broadcast method, even sendTo are only part of the context (LimitedStream) I get from the ca...

**Solution:** You have access to the `data` user map so you can create some managing object and store it in that. The streams themselves are indeed short lived but you can save ids around as those are stable.

---

### Is there a way to remove the generated markdown About.kt so I can use that route

**Problem:** Is there a way to remove the generated markdown About.kt so I can use that route?

**Solution:** Just delete the about.md file that came with the template

---

### Maybe you guys can help figure this problem out. I'm trying to add dynamic routing to sorenkai.com writings pa

**Problem:** Maybe you guys can help figure this problem out. I'm trying to add dynamic routing to sorenkai.com writings page so that writings, which currently are stored in MongoDB and loaded through a backend server, can have canonical links, such as https ://www.sorenkai.com/en/writings/ {id} The site is c...

**Solution:** (Can you put the code into a thread?)

---

### ```` am I doing anything wrong here

**Problem:** `TypeError: Failed to execute 'fetch' on 'Window': Failed to read the 'cache' property from 'RequestInit': The provided value 'null' is not a valid enum value of type RequestCache.` ```kt window.fetchBytes( HttpMethod.POST, resource = "https://bla", headers = mapOf("Content-Type" to "application/...

**Solution:** Let me look at the Kobweb code one sec

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

---

### seems like this is related to kobweb's code

**Problem:** seems like this is related to kobweb's code. when I use window.fetch directly and fill in some `undefined` params it's working ```kt window.fetch( input = "https://", init = RequestInit( method = "POST", headers = Headers().apply { append("Content-Type", "application/json") }, body = """ { "form"...

**Solution:** <https://github.com/varabyte/kobweb/blob/174e2597e6c6f12ffa8c3fc44cf5f2b8d42c0d96/frontend/browser-ext/src/jsMain/kotlin/com/varabyte/kobweb/browser/http/Fetch.kt#L115> is Kobweb code

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

### all right i got kobweb running 🙂 will see if i can contribute in any way later

**Problem:** all right i got kobweb running 🙂 will see if i can contribute in any way later. Need to finish what i started earlier first. 🙂 playing with Ktor again, really like it 🙂

**Solution:** But it's not terrible, exactly, but it's the heartbeat of the server, and some of the refactoring I did in there may be tricky to follow because I tried to share code that could handle running a dev server, a production server, and a static server

---

### well what do you think about moving Kobweb's config into build.gradle.kts

**Problem:** well what do you think about moving Kobweb's config into build.gradle.kts. So it looks something like that: ```kobweb { site { title = "Buddy" } server { port = 8080 dev { contentRoot = "build/processedResources/js/main/public" script = "../build/js/packages/cricket-kmm/kotlin/cricket-kmm.js" api...

**Solution:** I ended up using gradle because that was the best way to read through your code and understand your project, but I like the idea that you could use anything to build your code and Kobweb really shouldn't care

---

### I don't remember doing anything special. exported the whole thing, got .html and the js files and put them on 

**Problem:** I don't remember doing anything special. exported the whole thing, got .html and the js files and put them on my server. I can invite you to the repo though if that helps

**Solution:** Could you share your repo with me and tell me what steps you did?

---

### ah yeah I probably should learn how to use docker properly

**Problem:** ah yeah I probably should learn how to use docker properly. I was already fighting with deploying my ktor api

**Solution:** https://docs.plesk.com/en-US/obsidian/administrator-guide/web-servers/apache-and-nginx-web-servers-linux/apache-and-nginx-configuration-files.68678/ I have no idea is this is useful for you 🙂

---

### yes but either way your reference to it can't be cached ````

**Problem:** yes but either way your reference to it can't be cached

**Solution:** ``` First time: browser requests kobweb.js Server replies: Here you go, ETag: 1234567 Second time: browser requests Kobweb.js, saying it already has copy 1234567 Server replies: you already got it yo Third time: browser requests Kobweb.js, saying it already has copy 1234567 Server replies: here's a new copy 9876543 ```

```
First time: browser requests kobweb.js

Server replies: Here you go, ETag: 1234567

Second time: browser requests Kobweb.js, saying it already has copy 1234567

Server replies: you already got it yo

Third time: browser requests Kobweb.js, saying it already has copy 1234567

Server replies: here's a new copy 9876543
```

---

### Particularly, UI stuff

**Problem:** Hi! So, I found kobweb after about two weeks of developing my project on my own, and I'm interested in *some* of the features, but not the rest of them. Particularly, UI stuff. Writing HTML tags in pure Compose Web gets pretty annoying really quick. So I'm interested, is it possible to use *only*...

**Solution:** I threw together a rough README there but let me know if any details are missing.

---

### Is there a way to run the code in the jvmMain module without the task running

**Problem:** Is there a way to run the code in the jvmMain module without the `kobwebGenApi` task running?

**Solution:** Oh you want to put code in the JVM folder that's not consumed by the server

---

### Currently I 've added ```ApisFactoryImpl`

**Problem:** Currently I 've added ``` fun main() { println("Hello, world!") } ``` in the jvmMain directory. When I run it, it works, but also generates `ApisFactoryImpl` . I'm just trying to not have that generation happen.

**Solution:** I might need to add a setting to disable the kobweb plugin from doing stuff

```kotlin
fun main() {
    println("Hello, world!")
}
```

---

### is there built in intersectionObserver

**Problem:** is there built in intersectionObserver?

**Solution:** Filed https://github.com/varabyte/kobweb/issues/223

---

### Is it possible to add support for canvas https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D

**Problem:** Is it possible to add support for canvas

**Solution:** https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D

---

### so what's the exactly do

**Problem:** so what's the `includeServer` exactly do? I assume I set this to false if I don't need the jvm part?

**Solution:** (although if you're not using my kobweb server stuff, just set it to false)

---

### This only happens when gradle jvm is set to java 17 or higher ,similar thing happened for my android project r

**Problem:** This only happens when gradle jvm is set to java 17 or higher ,similar thing happened for my android project remember ? The newer AS comes with jdk17

**Solution:** I'll add a note to take a look

---

### How would one go on to create a multimodule kobweb project I'm thinking of putting the websocket files under a

**Problem:** How would one go on to create a multimodule kobweb project I'm thinking of putting the websocket files under a different module

**Solution:** Oh that's not good. Serialization is tough because I think you have to be consistent with your versions across modules?

---

### It can be anything ```` maybe

**Problem:** It can be anything

**Solution:** ``` data class Payload( @SerializedName("t") val t: String? = null, @SerializedName("s") val s: Int? = null, @SerializedName("op") val op: OpCodes? = null, @SerializedName("d") val d: JsonObject? = null ) ``` maybe?

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

### Thats interesting. What else would the plugin do

**Problem:** Thats interesting. What else would the plugin do?

**Solution:** <https://github.com/varabyte/kobweb/issues/252>

---

### btw how do we use headers on our call

**Problem:** btw how do we use headers on our call ?

**Solution:** <https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/frontend/kobweb-core/src/jsMain/kotlin/com/varabyte/kobweb/browser/ApiFetcher.kt#L64>

---

### do you think i should @initApi to initialize mongodb

**Problem:** do you think i should @initApi to initialize mongodb? currently doing it via koin and injecting UseCase direct into @Api fun argument!

**Solution:** The thing is I'm issuing commands telling the terminal to clear and move up 'n' times but when it hits the top it just ignores my command (if I recall correctly)

---

### Does your function meet these requirements

**Problem:** Does your function meet these requirements? https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/-js-export/

**Solution:** <https://github.com/bitspittle/bitspittle.dev/blob/8637f2af8787aa96ee7f471ad6dd66f79aa68d82/site/src/jsMain/kotlin/dev/bitspittle/site/components/layouts/BlogLayout.kt#L41> for example

---

### ```` I have to run for a while

**Problem:** ``` val jvmMain by getting { dependencies { implementation(libs.kobweb.api) val ktorVersion = "2.2.2" implementation("io.ktor:ktor-client-core:$ktorVersion") implementation("io.ktor:ktor-client-json:$ktorVersion") implementation("io.ktor:ktor-client-serialization:$ktorVersion") implementation("io...

**Solution:** I have to run for a while. My regulars here know I'm actually out for the week on a trip, but we got caught inside by some unexpected rain. However, it just cleared up! Wife and I are heading out

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

### can he make direct api calls from js

**Problem:** can he make direct api calls from js?

**Solution:** Even if you just have a form on your site that sends emails and it technically works, I feel like a bad actor can spam click that button on your site and cause trouble

---

### As said in the video, the kobweb directory was created manually

**Problem:** Please don't spam. As said in the video, the kobweb directory was created manually. You can choose to put your project in whatever directory you want, which you can create in windows explorer (or any other method you prefer).

**Solution:** Thank you very much for posting this. This goes for everyone! As our server grows, we need to grow mindful of spamming, even if intentions are good. I'll post some official server rules soon, but please treat this server as a precious resource as people here are volunteering and have their own lives and jobs. Repeatedly abusing the server can lead to a ban, thanks all for understanding!

---

### Does it always generate jvm files even though i'm not using server

**Problem:** Does it always generate jvm files even though i'm not using server? I have another jvm target but it's called vertxMain

**Solution:** I thought it should only do something for a jvmMain target inside a kobweb application module with includeServer = true

---

### what exactly doesn't work? Hmmm, I'm actually starting to think the rule may be to treat the jvmMain module in

**Problem:** what exactly doesn't work?

**Solution:** Hmmm, I'm actually starting to think the rule may be to treat the jvmMain module inside a Kobweb application as the server only. If you want to create a non-server jvm module (like a standalone app or a library), it should probably be a totally separate module.

---

### just use jvm main and remove kobweb api

**Problem:** just use jvm main and remove kobweb api?

**Solution:** Basically the code treats a module tagged with the Kobweb Application Plugin as a full stack unit

---

### I have this function located at jvmMain/kotlin/my/project/name/api/Echo.kt if I go to localhost:8080/api/echo?

**Problem:** I have this function located at jvmMain/kotlin/my/project/name/api/Echo.kt if I go to localhost:8080/api/echo?message=hello, I get an error 404 page If I click this button: Then the output is the HTML Source code of the non-existing 404 page. What am I doing wrong here?

**Solution:** First pass: Does your build script have `configAsKobwebApplication(includeServer = true)` in it?

---

### Let me take a moment and ask a basic question which is bugging me for many months now. We have 2 terms. Static

**Problem:** Let me take a moment and ask a basic question which is bugging me for many months now. We have 2 terms. Static site and Dynamic Site. Suppose I have a portfolio site with Kobweb. In the Home composable, I am calling weather API using Ktor and showing data on my site. This is done using LaunchedEf...

**Solution:** I don't use the term "dynamic" for the oppoiste case, but rather "full stack", which implies you are writing the logic for both the frontend and the backend, and pushing them both up together.

---

### Hey, is kotlinx.serialization compatible with Kobweb

**Problem:** Hey, is kotlinx.serialization compatible with Kobweb?

**Solution:** Yeah try creating the Todo example and check the build script in that one

---

### Someone already used https://github.com/Foso/Ktorfit ? I have some problems to addind it to my project

**Problem:** Someone already used https://github.com/Foso/Ktorfit ? I have some problems to addind it to my project... ``` A problem occurred configuring project ':site'. > Could not create task ':site:kobwebExport'. > Task with name 'kspKotlinJs' not found in project ':site'. ``` 5I tried to follow this guid...

**Solution:** I'm sure I tried an export before but trying again now to make sure

```
A problem occurred configuring project ':site'.
> Could not create task ':site:kobwebExport'.
> Task with name 'kspKotlinJs' not found in project ':site'.
```

---

### Does anyone have experience with the ktor js client

**Problem:** Does anyone have experience with the ktor js client? I'm trying to capture a custom header in a response with ktor but without success, but in postman the header appears normally

**Solution:** Do you own the server or are you using Kobweb for that?

---

### By get request, you mean a get request to an API in the JVM module, right? JS doesn't have bundled resources

**Problem:** By get request, you mean a get request to an API in the JVM module, right?

**Solution:** JS doesn't have bundled resources. Instead, you host the resource on a server somewhere and fetch it

---

### Hello, I'm trying to inspect variables for debugging using println. For the jsMain module, I know I can look a

**Problem:** Hello, I'm trying to inspect variables for debugging using println. For the jsMain module, I know I can look at stuff in the "Inspect -> Console" of my browser, but I how can I check variables from jvmMain? (using IntelliJ IDEA btw).

**Solution:** The `ctx` object in API routes also has a logger, so you can log values and then check `.kobweb/server/logs/kobweb-server.log`

---

### Folks, can someone give me a little help? I'm trying to create a button that calls an endpoint in my front end

**Problem:** Folks, can someone give me a little help? I'm trying to create a button that calls an endpoint in my front end. Like this: ``` Button(onClick = { coroutineScope.launch { playing = true window.http.post( "http://localhost:8080/api/myGame/createGame", body = JsonSerializer .encodeToString(GameStart...

**Solution:** To log something, use the `ctx.logger` property passed into the API route

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

### Hello, Im trying to retrieve a Double from a MongoDB using window.api suspend function. I pass in parameter id

**Problem:** Hello, Im trying to retrieve a Double from a MongoDB using window.api suspend function. I pass in parameter id : String and I can return Strings in other api calls all day long but can't return other Primitives. I don't know how to append the right calls to get the Double. Can anyone help? ``` su...

**Solution:** How are you serializing the data on the server side?

---

### It's an odd exception that I don't know why I'm getting

**Problem:** It's an odd exception that I don't know why I'm getting. Figured that repeating the call usually brings me the correct result. So in theory it could run forever but in practice it does not

**Solution:** Might even structure the code without recursion, something like... ```kotlin var retryCount = 3 val patientDeferred = CompletableDeferred<Patient>() while (retryCount > 0 && !patient.isCompleted) { try { patientDeferred.complete(fetchPatientFromServer()) } catch (ex: WeirdException) { --retryCount if (retryCount == 0) { // handle failure case here patientDeferred.completeExceptionally(ex) } } } return patientDeferred.await() ``` (Feel free not to use this, just pasting in case it's generally useful to anyone here who is observing)

```kotlin
var retryCount = 3
val patientDeferred = CompletableDeferred<Patient>()
while (retryCount > 0 && !patient.isCompleted) {
   try {
       patientDeferred.complete(fetchPatientFromServer())
   } catch (ex: WeirdException) {
       --retryCount
       if (retryCount == 0) {
           // handle failure case here
           patientDeferred.completeExceptionally(ex)
       }
   }
}

return patientDeferred.await()
```

---

### How do provide Server Side Rendering in Kobweb? Kobweb / Compose HTML does not do SSR at the moment

**Problem:** How do provide Server Side Rendering in Kobweb?

**Solution:** Kobweb / Compose HTML does not do SSR at the moment. You may want to check out <https://github.com/kwebio/kweb-core>, is that the sort of solution you're looking for?

---

### Hi Guys I want a dropdown like this Can anyone help

**Problem:** Hi Guys I want a dropdown like this Can anyone help?

**Solution:** <https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault>

---

### Hey I have a quick question, I'm planning to build a mono-repo that includes CMP + KTOR Server side code as we

**Problem:** Hey I have a quick question, I'm planning to build a mono-repo that includes CMP + KTOR Server side code as well as Kobweb in a single repo. How can I achieve this, wouldn't there be any gradle conflict? Do I need to make a sperate module and then cut-paste the kobweb code in that project?

**Solution:** <https://github.com/varabyte/kobweb#adding-kobweb-to-an-existing-project> advice probably applies here too actually

---

### And see how it's done there

**Problem:** And see how it's done there. Don't think we already have an example

**Solution:** but I am open to supporting changing the API concept to be per method instead of per file

---

### Hey ! Is there a good api to draw abitrary lines using code in kobweb

**Problem:** Hey ! Is there a good api to draw abitrary lines using code in kobweb ? like canvas in vanilla js

**Solution:** Check out Canvas2d, or `kobweb create examples/clock`

---

### Hey everyone! How does one do call logging and print statements within the api portion of kobweb

**Problem:** Hey everyone! How does one do call logging and print statements within the api portion of kobweb. Got an app set up but it's failing at a spot and I can't see the data that's casuing it.

**Solution:** You can read more here: <https://github.com/varabyte/kobweb?tab=readme-ov-file#kobweb-server-logs>

---

### I was looking for what ApiStream does <https://kobweb.varabyte.com/docs/concepts/server/fullstack#define-api-streams>

**Problem:** I was looking for what ApiStream does

**Solution:** <https://kobweb.varabyte.com/docs/concepts/server/fullstack#define-api-streams>

---

### Is there no dynamic loading of markdown files

**Problem:** Is there no dynamic loading of markdown files?

**Solution:** Getting js libraries bound into Kotlin is cool but it can be tricky if it's your first time (and also depends on how extensive the library API is), so give it a shot and ask questions if you got em

---

### Hello. Does Kobweb have a concept like Next.js's server component as opposed to client component

**Problem:** Hello. Does Kobweb have a concept like Next.js's server component as opposed to client component? Specifically so that secrets can be used in the kotlinjs without worry about them being publicaly leaked?

**Solution:** Yeah, my understanding was in a Kotlin project structure, it was better to embrace its more strict nature instead of fighting it. I feel like when you navigate a NextJS codebase you have to build a mental model of methods that will be picked up and only used on the backend and only used on the frontend, or some methods that magically are only called at compile time and not runtime. In Kobweb, you have a frontend bundle of code (jsMain), backend (jvmMain), and compile time code (build script)

---

### Deployment

### Yeah, I'm bouncing between the kobweb for developing the site and the runtime which is web-compose + silk +

**Problem:** Yeah, I'm bouncing between the kobweb for developing the site and the runtime which is web-compose + silk + ??. BTW, I noticed that the exports for todo seem to not generate the UI. The other examples appear fine. Cool concept.

**Solution:** But in general for our site we have a docker file which downloads kobweb, clones a repo, and exports stuff

---

### What's your plans for Kobweb

**Problem:** What's your plans for Kobweb? Do you want it to more like Jetpack Compose or like different flavour of Compose for Web?

**Solution:** Kobweb may still be useful for creating and running projects that use their canvas rendering stuff, but I think that approach won't work for all projects, so I plan to build Kobweb for those cases

---

### whats about deploying Kobweb app to for example Heroku? Please let me know how it goes, either if you hit wall

**Problem:** whats about deploying Kobweb app to for example Heroku?

**Solution:** Please let me know how it goes, either if you hit walls or if it works. I plan to document ways to deploy your Kobweb app at some point and it would be nice to talk about more than just Google's approach

---

### does dynamic routing work for static sites

**Problem:** does dynamic routing work for static sites?

**Solution:** That said I do need to document it these thoughts down (but at least there's a bug for that https://github.com/varabyte/kobweb/issues/134)

---

### Trying to run cli on github actions, but regardless of mac/linux, it seems "stuck". Any thoughts

**Problem:** Trying to run `kobweb` cli on github actions, but regardless of mac/linux, it seems "stuck". Any thoughts? I am running it to build a static site. For now I think I'll build locally and change `.gitignore` following https://bitspittle.dev/blog/2022/staticdeploy, but curious if there is better gui...

**Solution:** If you're not looking at the output anyway it's probably fine either way but you can try!

---

### Yea it is. Don't know if you know, but you can use a github actions workflow like this to automatically genera

**Problem:** Yea it is. Don't know if you know, but you can use a github actions workflow like this to automatically generate the static files on every commit

**Solution:** Does Kobweb export work with GitHub actions??

---

### In non static everything work excellent, I don’t understand what is a problem That's very strange

**Problem:** In non static everything work excellent, I don’t understand what is a problem

**Solution:** That's very strange. This has nothing to do with Kobweb, it should all be JS

---

### Maybe I think, but you can add in instruction of static build, that same name is very important Good luck

**Problem:** Maybe I think, but you can add in instruction of static build, that same name is very important

**Solution:** Good luck!! I'm my head, a finished Kobweb would be even better, with a huge widget set you could choose from

---

### can the kobweb generated js file be used with the async attribute

**Problem:** can the kobweb generated js file be used with the async attribute?

**Solution:** Also it's a static file so you'd have to manually update it each time

---

### hey , to export the website as static we use 'kobweb export --layout static', what is gradle task equivalent t

**Problem:** hey , to export the website as static we use 'kobweb export --layout static', what is gradle task equivalent to this?

**Solution:** Export is the worst one. One sec as LoP has a workflow that uses it

---

### hey again, can anyone spot out anything wrong with this

**Problem:** hey again, can anyone spot out anything wrong with this? i'm trying to add these variables to the task, but it doesn't seem to be adding them ```kt tasks.register("run-all") { group = "application" val site = project(":site") val export = site.tasks.findByName("kobwebExport")!! export.inputs.prop...

**Solution:** Here's how I read them on my end: ```kotlin val env = project.findProperty("kobwebEnv") val runLayout = project.findProperty("kobwebRunLayout") ... etc ... ```

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

### gpt told me this and I tried it and it seems to work

**Problem:** gpt told me this and I tried it and it seems to work? ``` export.extra["kobwebReuseServer"] = false export.extra["kobwebEnv"] = "DEV" export.extra["kobwebRunLayout"] = "KOBWEB" export.extra["kobwebBuildTarget"] = "RELEASE" export.extra["kobwebExportLayout"] = "STATIC" ```

**Solution:** Probably had an existential crisis: "I am a master of vast amounts of knowledge, enough to know I do *not* want to be answering Gradle questions"

```
export.extra["kobwebReuseServer"] = false
    export.extra["kobwebEnv"] = "DEV"
    export.extra["kobwebRunLayout"] = "KOBWEB"
    export.extra["kobwebBuildTarget"] = "RELEASE"
    export.extra["kobwebExportLayout"] = "STATIC"
```

---

### You were saying in i think that kobweb export --layout static is worst , why ? How should we export the site t

**Problem:** You were saying in i think that kobweb export --layout static is worst , why ? How should we export the site then🤔

**Solution:** That's a separate discussion about writing kobweb templates. You almost certainly can ignore it 🙂

---

### When running , what gradle command(s) produce the prod script that's used

**Problem:** When running `kobweb export --layout static`, what gradle command(s) produce the prod script that's used? I thought it'd be `jsBrowserProductionWebpack` but that gets me a slightly different script

**Solution:** https://github.com/varabyte/kobweb-cli/blob/897edd9ec5d688089e61e49171090fafc3bdd681/kobweb/src/main/kotlin/com/varabyte/kobweb/cli/common/GradleUtils.kt#L126

---

### This has lots of potential

**Problem:** Yes. This has lots of potential. I can't express how easy it feels to create good looking static websites with Composables using KobWeb. As I use Jetpack Compose daily, it feels home to this. But, when it comes to using JS libs, everything becomes tricky.

**Solution:** ``` js(""" Swal.fire({ ... }) """.trimIndent()) ``` should be a lot cleaner

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

### This site can’t be reached localhost refused to connect

**Problem:** This site can’t be reached localhost refused to connect.

**Solution:** If you're working then I guess it doesn't matter -- I was just worried maybe a new Chrome drop landed that destroyed Kobweb 🙂 Glad that's not the case

---

### has been running for about 15 minutes now. Is that normal

**Problem:** `kobwebExport` has been running for about 15 minutes now. Is that normal?

**Solution:** Yeah it's downloading and caching a browser.. Depending on people's locations it might get stuck

---

### is there any specific requirements to host kobweb website or is it possible to host in any platform

**Problem:** is there any specific requirements to host kobweb website or is it possible to host in any platform?

**Solution:** BTW did you happen to see my blog posts? <https://bitspittle.dev/blog/2022/staticdeploy> and <https://bitspittle.dev/blog/2023/clouddeploy>

---

### In nextjs i added an api route then call api from client side and then call discord api from the api server

**Problem:** In nextjs i added an api route then call api from client side and then call discord api from the api server. So now the request remain of same origin. I could do that kobweb and it will work but then i can't deploy it to vercel or any static site hosting platform

**Solution:** If you don't set up cors for your render site correctly, it will reject your get / post requests

---

### can i do this if site is hosted in a subdomain? sounds like you're still just beginning

**Problem:** can i do this if site is hosted in a subdomain?

**Solution:** sounds like you're still just beginning. I assume you're worried about getting yourself stuck in the future, and I get it. However, my recommendation for now would be to create a really small project just to get the feel of Kobweb.

**Note:** However, my recommendation for now would be to create a really small project just to get the feel of Kobweb.

---

### Jul 16 09:20:23 PM Dockerfile:46 Jul 16 09:20:23 PM -------------------- Jul 16 09:20:23 PM 44 | echo "org.gra

**Problem:** Jul 16 09:20:23 PM Dockerfile:46 Jul 16 09:20:23 PM -------------------- Jul 16 09:20:23 PM 44 | echo "org.gradle.jvmargs=-Xmx256m" >> ~/.gradle/gradle.properties Jul 16 09:20:23 PM 45 | Jul 16 09:20:23 PM 46 | >>> RUN kobweb export --notty Jul 16 09:20:23 PM 47 | Jul 16 09:20:23 PM 48 | #-------...

**Solution:** <https://github.com/bitspittle/kobweb-todo-on-render/blob/a5244fa568b0f358fb670e996235441e77b54ffc/Dockerfile#L5> should probably be set to "site" for most project

---

### can i just compile the kobweb markdown plugin without having to clone and compile the whole kobweb repo ? i ne

**Problem:** can i just compile the kobweb markdown plugin without having to clone and compile the whole kobweb repo ? i need a certain functionality to be present in markdown plugin. ive also opened an issue about that maybe he forgot about it🤔

**Solution:** (Doing a static export and writing your own server to serve them is totally fine btw, just want to make sure you need it)

---

### I just executed command. I want to host my portfolio site on Linode. Should I copy paste my whole project on L

**Problem:** I just executed `kobweb export` command. I want to host my portfolio site on Linode. Should I copy paste my whole project on Linode and then use `kobweb run --env prod`? Or is there any specific folder which is generated which I can upload on Linode and run with prod environment?

**Solution:** The way I do it for my own site is I do an export locally and then run `firebase deploy` which finds the Kobweb directory and uploads it.

---

### I just executed command. After that I explored directory inside folder. There is one directory called which ha

**Problem:** I just executed `kobweb export --layout static` command. After that I explored `build` directory inside `site` folder. There is one directory called `distribution` which has one folder `public` and one file `portfolio.js` Should I upload these on the server if I want to host my site? If yes, when...

**Solution:** You can change the folder to a different location if you need to, but if all you needed was the files, sounds like you're good to go

---

### > .kobweb/conf.yaml? I was shocked I didn't mention anywhere in there before now

**Problem:** > `kobweb.conf` .kobweb/conf.yaml?

**Solution:** I was shocked I didn't mention `kobweb export` anywhere in there before now. Long overdue, even if this first pass might be a bit shoe-horned in.

---

### What is the purpose of this task exactly

**Problem:** What is the purpose of this task exactly ? 🤔

**Solution:** In a pinch you can just run `kobweb export --layout static` yourself, files will be in .kobweb/site when you're done

---

### Hey ! How can I access Kobweb from my phone ? My PC is connected with Wifi to my box, and my phone also, but w

**Problem:** Hey ! How can I access Kobweb from my phone ? My PC is connected with Wifi to my box, and my phone also, but when I go to `localhost:8080` I get an `ERR_CONNECTION_REFUSED` error

**Solution:** I think you have to set up a proxy. I don't think you can access localhost directly from another machine (but don't quote me on that)

---

### I mean I distribute my website on Cloudflare, and iirc Kobweb doesn't generates edge functions like JS framewo

**Problem:** I mean I distribute my website on Cloudflare, and iirc Kobweb doesn't generates edge functions like JS frameworks ?

**Solution:** Meanwhile my first thought is create convention plugins and off-load as much logic as you can to them

---

### Build

### Apologies for the sharp corners in advance

**Problem:** One is general modularity, and in my case you can't use the compose and kotlinx serialization gradle plugins in the same module

**Solution:** Anyway thanks for tinkering and for this initial feedback. Apologies for the sharp corners in advance!

---

### so if i can run backend how do i check if it works

**Problem:** so if i can run backend how do i check if it works?

**Solution:** They use a gradle feature where they import another project from disk as if it were a dependency

---

### how would I version the kobweb generated js file

**Problem:** how would I version the kobweb generated js file?

**Solution:** Once you've done that, read it like so: https://github.com/bitspittle/morple/blob/40d9484c07178a235dd4d64db0675403fd4b02e8/site/src/jsMain/kotlin/dev/bitspittle/morple/components/sections/Header.kt#L54

```kotlin
link(
                rel = "stylesheet",
                href = "/checkboxes.css?v=${Random.nextInt()}"
            )
```

---

### well the only difference is that I make it a job I can cancel and I use attrsModifier instead of asAttributesBuilder e.g

**Problem:** well the only difference is that I make it a job I can cancel and I use attrsModifier instead of asAttributesBuilder

**Solution:** e.g. if you do something like ``` val scope = ... blah { scope.launch { ... } } ``` and that works, but later add: ``` val scope = ... blah { anotherblah { scope.launch { ... } } } ``` and `anotherblah` itself has its own "scope" field, you will still compile but behavior might change unexpectedly

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

### the code is just simple fetching ``````

**Problem:** the code is just simple fetching ```kt @Serializable data class Product( val id: Int, val title: String, val price: Double, val description: String, val category: String, val image: String, val rating: Rating, ) @Serializable data class Rating( val rate: Double, val count: Int, ) suspend fun fetc...

**Solution:** You have to put this in your build script somewhere: ```kotlin import org.jetbrains.kotlin.gradle.dsl.KotlinCompile .... compose { kotlinCompilerPlugin.set("1.4.0") } project.tasks.withType<KotlinCompile<*>>().configureEach { kotlinOptions.apply { freeCompilerArgs = freeCompilerArgs + listOf("-P", "plugin:androidx.compose.compiler.plugins.kotlin:suppressKotlinVersionCompatibilityCheck=1.8.10") } } ```

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

### ```` I think Kobweb stresses Gradle out (or here, maybe the Kotlin compiler daemon

**Problem:** ``` ./gradlew clean && ./gradlew --stop ```

**Solution:** I think Kobweb stresses Gradle out (or here, maybe the Kotlin compiler daemon?) by pushing people into a live reloading experience

```
./gradlew clean && ./gradlew --stop
```

---

### Can you give it a try

**Problem:** Can you give it a try? Here is the simplified version:

**Solution:** ``` .toAttrs { onInput { value = it.value } ``` onInput is not resolving for me for some reason

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

### Hello! I'm trying to import koin into my project, but is failing and showing no error message... what am I doi

**Problem:** Hello! I'm trying to import koin into my project, but is failing and showing no error message... what am I doing wrong?

**Solution:** And yeah, be sure you did a Gradle sync

---

### Yes, but in my case it really does nothing after loading the project and indexing stuff. As you can see in my 

**Problem:** Yes, but in my case it really does nothing after loading the project and indexing stuff. As you can see in my gradle, it says "Nothing to show". Kinda weird right?. For a newbie like me I felt stuck for a moment haha maybe you can add this in ReadMe. If intelliJ doesn't load your project you can ...

**Solution:** It's a hard balance. Put everything into the README and no one can find anything 🙂 I'm not sure I can answer general IntelliJ questions in there, that will be endless... 🙂

---

### It depends how you open the project

**Problem:** It depends how you open the project. If you open directly the settings.gradle.kts file, it figures out everything automatically. If you open via VCS, it asks you if you want to load it. If you open the folder, sometimes it finds it, sometimes not...

**Solution:** These days pretty much everything is an indeterminate spinner

---

### Or maybe there's a Gradle task for it

**Problem:** Or maybe there's a Gradle task for it? I don't remember

**Solution:** https://kotlinlang.org/docs/using-packages-from-npm.html seems to imply manual work

---

### So i can generate material3 for kobweb

**Problem:** So i can generate material3 for kobweb ?

**Solution:** In a magic world with fairies and dragons you would just add an `npm` dependency and then just start calling into it with Kotlin

---

### If it consistently gets bad, as a temporary fix you can use gradlew kobwebStart -t and get mostly the same behavior

**Problem:** If it consistently gets bad, as a temporary fix you can use gradlew kobwebStart -t and get mostly the same behavior. That fixed things for me when I was having similar issues on windows. Just make sure to run kobwebStop when you're done.

**Solution:** That's not good, I'll look into it later this week

---

### where i can find guide for kobweb? We are still early days of Kobweb so guides are sparse

**Problem:** where i can find guide for kobweb?

**Solution:** We are still early days of Kobweb so guides are sparse. However, fairly quickly you'll see it's a Kotlin version of JavaScript / CSS, so usually if you start searching how to do things using CSS, figuring out what to do in Kobweb is usually not too hard

---

### does kobweb not support adding favicon other than the default one

**Problem:** does kobweb not support adding favicon other than the default one ?

**Solution:** Icon set here: <https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/gradle-plugins/application/src/main/kotlin/com/varabyte/kobweb/gradle/application/extensions/AppBlock.kt#L42>

---

### if you need my gradle task. i can share with you I mean, what needs to be shared between your ktor module and 

**Problem:** if you need my gradle task. i can share with you

**Solution:** I mean, what needs to be shared between your ktor module and Kobweb?

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

### Hello guys! i have a problem with bulding project What i need to do either? Gradle has its own settings

**Problem:** Hello guys! i have a problem with bulding project What i need to do either?

**Solution:** Gradle has its own settings. It should be set to project settings but yeah double check is good

---

### Maybe worth setting explicitly in

**Problem:** Maybe worth setting explicitly in `build.gradle.kts`? ``` tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> { compilerOptions.jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_11) } ```

**Solution:** Play around with Kobweb, see what you think! Hope you enjoy working with it 👍

```
tasks.withType<org.jetbrains.kotlin.gradle.tasks.KotlinCompile> {
    compilerOptions.jvmTarget.set(org.jetbrains.kotlin.gradle.dsl.JvmTarget.JVM_11)
}
```

---

### ah yeah if you inspect the site in your browser those tags will be empty

**Problem:** ah yeah if you inspect the site in your browser those tags will be empty. not sure why

**Solution:** At export time, I actually turn those invisible style tags visible with this code: <https://github.com/varabyte/kobweb/blob/fcda655ef9ba10512377c925ccde758517bf12ea/gradle-plugins/application/src/main/kotlin/com/varabyte/kobweb/gradle/application/tasks/KobwebExportTask.kt#L76>

---

### What version of kobweb are you using

**Problem:** What version of kobweb are you using?

**Solution:** If you can't upgrade, you can use the stdlib window.fetch if you want but it kind of sucks.

---

### wait, how are you able to generate toc from headings

**Problem:** wait, how are you able to generate toc from headings ? you dont have them in markdown page in yaml section, you don't have set any gradle configuration for that im talking about this line <https://github.com/bitspittle/bitspittle.dev/blob/2f40c1f931e31e0f687547cb7bebdf2cb61eedc2/site/src/jsMain/k...

**Solution:** https://raw.githubusercontent.com/bitspittle/bitspittle.dev/2f40c1f931e31e0f687547cb7bebdf2cb61eedc2/site/src/jsMain/resources/markdown/blog/2022/KoverBadge.md

```kotlin
mdCtx.frontMatter["toc-min"]?.singleOrNull()?.toIntOrNull() ?: 2,
mdCtx.frontMatter["toc-max"]?.singleOrNull()?.toIntOrNull() ?: 3,
```

---

### You can try manually providing that compositionlocal, though I assume you'll run into new issues anyway What v

**Problem:** You can try manually providing that compositionlocal, though I assume you'll run into new issues anyway

**Solution:** What version of Kobweb is your project using?

---

### ``tf`` Can we do this too in kobweb

**Problem:** ```<!DOCTYPE html> <html lang="en-US"> <head> <meta charset="utf-8" /> <title>TensorFlow.js browser example</title> <!-- Load TensorFlow.js from a script tag --> <script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@latest/dist/tf.min.js"></script> </head> <body> <h1>TensorFlow.js example</h...

**Solution:** spend some time trying to fix it

---

### yes you can add the script in head.add, and then call in your page Best thing for now is to add just the scrip

**Problem:** yes you can add the script in head.add, and then call `js("const model...")` in your page

**Solution:** Best thing for now is to add just the script inside your build.gradle script and then run the js logic inside your code somewhere (maybe inside `MyApp`?)

---

### I think you can't use **npm** dependency by calling them inside js(). I think it's only for packages available

**Problem:** I think you can't use **npm** dependency by calling them inside js(). I think it's only for packages available through cdn ?. For npm libs you need to create external declarations to use them if I'm not wrong

**Solution:** Open up a browser and go to its console in its dev tools. You can experiment with typing js in there

---

### Not registering keyframes definition , as only top-level definitions are supported at this time

**Problem:** Not registering keyframes definition `val ShiftRight`, as only top-level definitions are supported at this time. Although fixing this is recommended, you can manually register your keyframes inside an @InitSilk block instead (`ctx.stylesheet.registerKeyframes(ShiftRight)`). Suppress this message ...

**Solution:** It should be a top level global public var so the Kobweb gradle plugin can find it and register it

---

### What properties are available to configure for the meta tag in the build gradle

**Problem:** What properties are available to configure for the meta tag in the build gradle?

**Solution:** Just in case, would be in code like this (but I'm not using meta): <https://github.com/bitspittle/bitspittle.dev/blob/8037bc0ad7ab182caf4b7d50d26e2b2d50685e43/site/build.gradle.kts#L36>

---

### 11 can have issues sometimes

**Problem:** 11 can have issues sometimes.

**Solution:** So yeah try removing this block and try again: <https://github.com/varabyte/kobweb/blob/db534eff808f585542cae80b975dd21d4efedb07/buildSrc/build.gradle.kts#L32C1-L32C1>

---

### Is the issue when you run kobwebExport or when you build the project

**Problem:** Is the issue when you run kobwebExport or when you build the project?

**Solution:** My guess is Kobweb and <https://plugins.gradle.org/plugin/de.comahe.i18n4k> are interacting in a bad way

---

### I'm getting this error, need help

**Problem:** I saw https://github.com/varabyte/kobweb/#adding-kobweb-to-an-existing-project and added a site from kobweb to my KotlinMultiPlatform project. I'm getting this error, need help! ----- TestKmm3/build.gradle.kts' at line: 1 Plugin [id: 'com.android.application', version: '8.1.0', apply: false] not ...

**Solution:** > `org.gradle.api.plugins.UnknownPluginException: Plugin [id: 'com.android.application', version: '8.1.0', apply: false] not found in `one of the following sources:

---

### Did you modify the root gradle or settings file when adding the site module? It's possible that by becoming mu

**Problem:** Did you modify the root gradle or settings file when adding the site module?

**Solution:** It's possible that by becoming multi module you are making gradle resolve a plugin in a place you're not expecting (e.g. the site module probably)

---

### So, my problem when i create kobweb project is on Node? You can try running instead of

**Problem:** So, my problem when i create kobweb project is on Node?

**Solution:** You can try running `./gradlew kobwebStart -t` instead of `kobweb run`. It's basically the same thing. Maybe you'll get a clearer error message that way?

---

### I'm curious like how can I achieve this the same using .js file? Let's say I created a .js file and pasted the

**Problem:** I'm curious like how can I achieve this the same using .js file? Let's say I created a .js file and pasted the same script. How can I call that function without making it .kt file? Ahh, this is so confusing sometimes..

**Solution:** You put it in your public directory, include it as a <script> in your build gradle kts file, and then if necessary create external bindings to it

---

### playground uses symlinks for gradle wrapper which makes it impossible to run on windows or maybe i need a plug

**Problem:** playground uses symlinks for gradle wrapper which makes it impossible to run on windows or maybe i need a plugin for that ?

**Solution:** Ah sure, although symlinks are amazing and it's a tragedy they weren't adopted on Windows earlier! I would highly recommend setting it up at some point, it's not too hard.

---

### Hi guys, can anyone help me with this

**Problem:** Hi guys, can anyone help me with this. My website in mobile is loading like the desktop mode first and 1s later move all to the mobile version https://www.carlosgub.dev , I want to fix this 😄

**Solution:** Sadly, yeah that has a stutter in it. It wasn't something I realized when I first started working on Kobweb. It can still be useful but I should probably document the method with what I learned

---

### Well, just updated kobweb to 0.20.0 and this happened... The build just stuck. kobweb run/export also are stuc

**Problem:** Well, just updated kobweb to 0.20.0 and this happened... The build just stuck. kobweb run/export also are stuck Any ideas why this happening? Kotlin, compose compiler versions etc just like in the latest sample for 0.20.0 May be this happening because of kobweb-lib. Will try now to research and m...

**Solution:** Amazing thanks. If you can find repro steps I'll investigate this as a high priority.

---

### idk, I'm running , and it does recompile, but the browser doesn't reload Yes that should support live reloading

**Problem:** idk, I'm running `./gradlew :site:kobwebStart -PkobwebEnv=DEV --continuous`, and it does recompile, but the browser doesn't reload

**Solution:** Yes that should support live reloading.

---

### I can't try it out today, I'll update you in a few days Ah it's all good

**Problem:** I can't try it out today, I'll update you in a few days

**Solution:** Ah it's all good. If we can repro it here, that should be enough to dig into it. I assume it's a me issue and not a Gradle issue.

---

### Hey, I'm running into a weird problem I've not run into before

**Problem:** Hey, I'm running into a weird problem I've not run into before. I'm adding Kobweb to the webApp module of the KindredCircl KMP and it's now looking for commonMain, commonTest, jsMain, and jsTest in the :site directory jsMain is obviously declared, and I can add jsTest, but the others aren't neces...

**Solution:** New Gradle sourceset issue

---

### If the gradle command works I'd expect the cli to work too. Do other kobweb commands work

**Problem:** If the gradle command works I'd expect the cli to work too. Do other kobweb commands work?

**Solution:** Maybe it's how my CLI chooses gradle to run?

---

### Silk

### I've got TextInput and icons on top. When I click the clear button, i lose focus. how do i fix this

**Problem:** I've got TextInput and icons on top. When I click the clear button, i lose focus. how do i fix this? can i override and set focus myself state?

**Solution:** I do something like that in button; https://github.com/varabyte/kobweb/blob/a8910bf5168a3e27be88ab49fce8b0a86322caac/frontend/kobweb-silk-widgets/src/jsMain/kotlin/com/varabyte/kobweb/silk/components/forms/Button.kt#L72

---

### Does removing FaIcons, silk break things ? If I'm not using it

**Problem:** Does removing FaIcons, silk break things ? If I'm not using it.

**Solution:** Removing icons is easy, Silk may be harder as it contains a bunch of useful things

---

### Has anyone created an OTP field with silk or compose

**Problem:** Has anyone created an OTP field with silk or compose?

**Solution:** Is that 6 text fields in a row?

---

### is there one for buttons as well? That said, I wonder if we want some way to tell Silk widgets to default to u

**Problem:** is there one for buttons as well?

**Solution:** That said, I wonder if we want some way to tell Silk widgets to default to using a variant everywhere. Probably not though -- I think the Compose way is to let codebases create their own wrapping composables? Not sure

---

### Markdown

### I'm trying to have a markdown editor in my website, anyone ever implement something like that

**Problem:** I'm trying to have a markdown editor in my website, anyone ever implement something like that?

**Solution:** I looked at it and gave up on it because it was not well documented, and I think it was missing yaml extensions, but you might want to look at it

---

### I am setting up highliter js for markdown code block

**Problem:** I am setting up highliter js for markdown code block. but getting hljs is not defined error. please let me know what i am mission.

**Solution:** I can appreciate they're trying to learn from my blog site. FWIW I do plan to move away from highlight.js at some point (I want an engine that allows me to highlight individual lines), but highlight.js is definitely not a bad solution.

---

### General

### Hi. how do i get png showing

**Problem:** Hi. how do i get png showing? Img( "/logo.png" ) png in resources and.. nothing

**Solution:** (The reason for the "/public" prefix is so you can put other things in the resource file that don't automatically get exposed to users, but when kobweb runs, it strips it)

---

### my cards are in svg format, can the canvas display svgs though

**Problem:** my cards are in svg format, can the canvas display svgs though?

**Solution:** I'm trying to remember - I played around with svgs like 6 months ago and there were issues creating them but I think that's since been fixed

---

### Hi team, I've the problem, that if I run no wizard occur but also no error message why it does not work is dis

**Problem:** Hi team, I've the problem, that if I run `kobweb create site` no wizard occur but also no error message why it does not work is displayed. Has someone a tip how to fix it? 🙂

**Solution:** Let me give you the command so you can try it without kobweb, see what happens

---

### yes. But Tobo asked for centering Row, did he

**Problem:** yes. But Tobo asked for centering Row, did he? what in compose is Row( horizontalArrangement = Arrangement.Center )

**Solution:** You can use that as well, that's true

---

### thanks for the beer, but whaaaaat does it mean

**Problem:** thanks for the beer, but whaaaaat does it mean? 😄

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

### Have you tried putting the image in the pre-existing resources/public folder

**Problem:** Have you tried putting the image in the pre-existing resources/public folder?

**Solution:** Cabs not sure if you saw but I'm mid move, we're just unloading now

---

### but how's that relevant I found this

**Problem:** but how's that relevant

**Solution:** I found this. Not sure it's exactly right but it talks about the mechanism for how the browser knows if it should get a new copy of the file or not

```kotlin
script {
  src = "/fluense_web.js?v=${Random.nextInt()}"
  async = true
}
```

---

### I didn't mean to start a riot

**Problem:** I didn't mean to start a riot. I can always just have something run over the generated html files to do that myself. loading the script in header vs body is what I'm invested in though

**Solution:** You can find the js file inside your .kobweb/site folder and manually edit it before uploading to your vps

---

### or what do you mean Like, the minimum we want to test is ````

**Problem:** or what do you mean

**Solution:** Like, the minimum we want to test is ``` addEventListener("touchstart") { scope.launch { delay(800L) }} ```

---

### The other thing I was testing with was having ```` which causes recomposition without the box I have no idea w

**Problem:** The other thing I was testing with was having ``` SideEffect { console.log("Variant recomposed ${componentVariant.hashCode()}") } ``` which causes recomposition without the box

**Solution:** I have no idea what's going on. My equals and hashcode methods aren't even being called.

---

### Is the class immutable

**Problem:** Is the `ComponentVariant` class immutable?

**Solution:** ``` Stable -- Indicates a type that is mutable, but the Compose runtime will be notified if and when any public properties or method behavior would yield different results from a previous invocation. ``` Yeah I definitely don't intend to do that 🙂

---

### Unsure why, but kobweb actually works in the subfolder with no changes If it works for you though that's great

**Problem:** Unsure why, but kobweb actually works in the subfolder with no changes

**Solution:** If it works for you though that's great!

---

### is the overlaying text more content of the file or something else

**Problem:** is the overlaying text more content of the file or something else?

**Solution:** Thank you, I'll take a look first thing tomorrow

---

### Sorry to keep bothering you, but are image resources supposed to work from the non main module? Resources are 

**Problem:** Sorry to keep bothering you, but are image resources supposed to work from the non main module?

**Solution:** Resources are supposed to work, yes. They should be in the public folder.

---

### I have a question. This white area is a Div element that has 'contentEditable' enabled. Which means I can writ

**Problem:** I have a question. This white area is a Div element that has 'contentEditable' enabled. Which means I can write text inside it. When I copy something from my IDE and paste it there, I get the whole formatting, instead of just the text. Is there a way to fix this, instead of using textarea component?

**Solution:** You don't want the formatting at all? Or you want the formatting to extend to the whole area?

```kotlin
onPaste { it.clipboardData?.clearData() }
```

---

### Is it possible to share components between two kobweb application modules directly, not using a library module

**Problem:** Is it possible to share components between two kobweb application modules directly, not using a library module?

**Solution:** Kobweb libraries put metadata in jars read by Kobweb applications

---

### Hi! can anyone tell me why my column items are vertically spaced

**Problem:** Hi! can anyone tell me why my column items are vertically spaced?

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

**Solution:** Like, I just want to use ``` Column { Button Text TextInput Whatever } ```

```
Column {
   Button
   Text
   TextInput
   Whatever
}
```

---

### I'm sadly not an expert here at all, just wanted to throw accessibility into the why a and not b discussion I 

**Problem:** I'm sadly not an expert here at all, just wanted to throw accessibility into the why a and not b discussion

**Solution:** I appreciate it. To be honest I hadn't considered it.

---

### Can we do firebase database integration

**Problem:** Can we do firebase database integration?

**Solution:** But yeah I have a WIP web card game (on hold now since Kobweb seems to need more attention at the moment) that is successfully using firebase for auth, database, and very basic analytics

---

### How are you removing embeds

**Problem:** How are you removing embeds!! 🤔

**Solution:** See also: https://github.com/varabyte/kobweb/blob/main/COMPATIBILITY.md

---

### How can I "reload" my kobweb site when I update the code

**Problem:** How can I "reload" my kobweb site when I update the code? I think it always did it automatically...? 😅

**Solution:** I also set my IntelliJ to autosave on lose focus

---

### Is there anything similar to runBlocking in multi-platform You can use or to get into suspend fun code on JS

**Problem:** Is there anything similar to runBlocking in multi-platform

**Solution:** You can use `LaunchedEffect` or `rememberCoroutineScope` to get into suspend fun code on JS

---

### or ```` Sealed interface in my case because I don't have common data that I share across states but both are fine

**Problem:** or ``` class ConnectionHandler { val socket = Websocket().also { it.connect() } } ... val handler = remember { ConnectionHandler() } handler.socket... ```

**Solution:** Sealed interface in my case because I don't have common data that I share across states but both are fine

```kotlin
class ConnectionHandler {
  val socket = Websocket().also { it.connect() }
}
...
val handler = remember { ConnectionHandler() }
handler.socket...
```

---

### Both looks same to me nowadays idk why State machines are good if you start getting tons of remember variables

**Problem:** Both looks same to me nowadays idk why

**Solution:** State machines are good if you start getting tons of remember variables that all related to each other. (Maybe I should consider using one in Popup...)

---

### do you have a updated list of different projects built using kobweb by different people

**Problem:** do you have a updated list of different projects built using kobweb by different people?

**Solution:** There's been some amazing stuff done with Kobweb already but it's still really early, and I think it's OK for people to be nervous about that

---

### .Hello! What is replacement for LazyLists in kobweb

**Problem:** .Hello! What is replacement for LazyLists in kobweb?

**Solution:** I do want them to be part of Kobweb someday, it's just been a huge pile of higher priorities but we're getting there!

---

### is there any widget available expanded that will fill available space in a row

**Problem:** is there any widget available expanded that will fill available space in a row?

**Solution:** If you'd like to sync the project and take a look, I might be able to point you in the right direction

---

### .can i help in any way? It's exciting that new people keep using Kobweb in new ways

**Problem:** .can i help in any way?

**Solution:** It's exciting that new people keep using Kobweb in new ways. Sorry you're working on the bleeding edge...

---

### where can I find documentation or examples about forms and input data with compose or kobweb

**Problem:** where can I find documentation or examples about forms and input data with compose or kobweb ?

**Solution:** Basically one form you pass text in as an argument, the other the text value comes from the control, I think?

---

### I'm trying to display a svg from a file. For that, I used the object element that allow me to load the content

**Problem:** I'm trying to display a svg from a file. For that, I used the object element that allow me to load the content of the svg file. But now I also want to acess the svg element and show it on the console for example. I tried this code but the output in the console is "null" or "NodeList[]". Does that...

**Solution:** I've also used window.setTimeout (without a time argument) sometimes to postpone using a value for a frame

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

**Problem:** How to write code for downloading a file when a button is clicked ?

**Solution:** <https://github.com/varabyte/kobweb/blob/4284e7e50511a8a2611b52943da5fff58e5bb940/frontend/compose-html-ext/src/jsMain/kotlin/com/varabyte/kobweb/compose/file/FileUtils.kt#L26>

---

### Using scope

**Problem:** How to convert pdf file to byte Array type for passing it to document.savetodisk(content)

**Solution:** All together, it should look something like: ```kotlin val scope = rememberCoroutineScope() Button(onClick = { scope.launch { document.saveToDisk( "resume.pdf", window.http.get("/resume.pdf") ) } }) ```

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

### Do you know how I can make a request from the browser and avoid invalid origin error ? Sorry that at this poin

**Problem:** Do you know how I can make a request from the browser and avoid invalid origin error ?

**Solution:** Sorry that at this point we're still just guessing. Disabling incremental compilation may make things a bit slower, let me know if you see that

---

### the thing is that these error go away when i make a slight change even a comment does it Yeah it's a cache problem

**Problem:** the thing is that these error go away when i make a slight change even a comment does it

**Solution:** Yeah it's a cache problem. Not kobweb's fault but I think because Kobweb is aggressively promoting live reload you're hitting it

---

### don't remember if checked for exception or not. shouldn't it be doing the same for chrome then

**Problem:** don't remember if checked for exception or not. shouldn't it be doing the same for chrome then?

**Solution:** Firefox and Chrome differ enough that sometimes you get a crash in one and not the other, but not likely. I was just saying check that one since it was the one that didn't work

---

### I want to do to productivity tool, and I need put these tools on a website. I will start with a Pomodoro. I ne

**Problem:** I want to do to productivity tool, and I need put these tools on a website. I will start with a Pomodoro. I need that the web browser reminds me when the time ends with a sound and a reminder on my pc, then it will start the break, when the break finishes this starts the Pomodoro. You could guide...

**Solution:** Find out how to do it in JS and then try to create a very very very simple project to do it in Kobweb

---

### Is marquee available in kobweb

**Problem:** Is marquee available in kobweb ?

**Solution:** Did you try adding setTimeout yet?

---

### <https://github.com/varabyte/kobweb/issues/274> wait is this really possible? (Pretty sure) not possible, I ne

**Problem:** <https://github.com/varabyte/kobweb/issues/274> wait is this really possible?

**Solution:** (Pretty sure) not possible, I need to reply to that bug. Will do it now!

---

### It did. If that's documented somewhere, do you know where

**Problem:** It did. If that's documented somewhere, do you know where? I've only been using the github readme for Kobweb.

**Solution:** Until then, asking here is great. And something else you can do in the future is try creating some of the samples (e.g. `kobweb create examples/todo` or even just `kobweb create app`), see if it's working, and then compare that with your own project

---

### Hallo. I'm having fun with Kobweb, but I'm not sure what the Kobweb equivalent of a text input is

**Problem:** Hallo. I'm having fun with Kobweb, but I'm not sure what the Kobweb equivalent of a text input is?

**Solution:** You're welcome to take a look at those and use those if they help. I just need to take more time thinking about all the different use cases before I stamp something as official.

---

### how do i do something like this with externals ```` Best to see what other people do

**Problem:** how do i do something like this with externals ``` const easyMDE = new EasyMDE({element: document.getElementById('my-text-area')}); ```

**Solution:** Best to see what other people do. You can take a look at <https://github.com/bitspittle/firebase-kotlin-bindings> to see what I do

```
const easyMDE = new EasyMDE({element: document.getElementById('my-text-area')});
```

```kotlin
@JsModule("easymde")
@JsNonModule
external fun EasyMDE: dynamic
```

---

### how can i do something before a composable launches

**Problem:** how can i do something before a composable launches?

**Solution:** You can use key or remember

---

### Hey there, does anyone know how get browser language in Kobweb? I tried to search for it but i couldnt find th

**Problem:** Hey there, does anyone know how get browser language in Kobweb? I tried to search for it but i couldnt find the answer. Im new to web development.

**Solution:** The question you'd want to ask here is how to get the browser language in JavaScript. Do you know? I can look it up, I don't know at the moment.

---

### Sorry for interupting the ongoing conversation

**Problem:** Sorry for interupting the ongoing conversation. I need some help regarding RangeInput. I want to create a custom range input like the first image or equivalent. But the second image is what I achieved till now.

**Solution:** I tend to look at chakra UI and see what they're doing: <https://chakra-ui.com/docs/components/slider>, play around a bit with the element inspector

---

### And that "get" is performed via JS code operated by the browser? One thing that's weird to get used to is even

**Problem:** And that "get" is performed via JS code operated by the browser?

**Solution:** One thing that's weird to get used to is even your application is external, at first. People have to visit a website to download and run it. The only thing local at first is just your browser!

---

### Uhm I think I have an even noobier question haha. So, is there a list somewhere where I can find the composabl

**Problem:** Uhm I think I have an even noobier question haha. So, is there a list somewhere where I can find the composables implemented so far in kobweb?

**Solution:** The first set are general widgets, mostly what you want is there. The second set are widgets that are built specifically for interacting with Kobweb. (You can use the first set of widgets even in a non-Kobweb Compose HTML project, in other words)

---

### onClick doesn't work I think you're asking questions a little bit aggressively here

**Problem:** onClick doesn't work

**Solution:** I think you're asking questions a little bit aggressively here. Please look at existing code a bit first before immediately giving up and pinging here. `kobweb create app` has a button in it you can check.

---

### Out of curiosity, does Kobweb offer a feature to create a home screen? Nope nothing at the moment

**Problem:** Out of curiosity, does Kobweb offer a feature to create a home screen?

**Solution:** Nope nothing at the moment. If it was ever added, it would be as a non-core library, probably something in the kobwebx namespace

---

### yep, I have a browser with devtools available on my mobile I'm just pointing out you don't need to have a phon

**Problem:** yep, I have a browser with devtools available on my mobile

**Solution:** I'm just pointing out you don't need to have a phone to preview the mobile experience, in case that's what you were doing. I don't think so in your case, but just mentioning it especially if other people are lurking and didn't know

---

### Does examples/responsive still exist? I was just remembering that

**Problem:** Does examples/responsive still exist?

**Solution:** I was just remembering that. I deleted it after we made `app` better

---

### Showcase channel

**Problem:** Showcase channel?

**Solution:** `kobweb create` and then look at the list of choices

---

### Oh nice But not looking for that, more like structure advice My current table is columns and rows (and boxes),

**Problem:** Oh nice But not looking for that, more like structure advice My current table is columns and rows (and boxes), is there a better method ?

**Solution:** I would use an html table element to be honest

---

### I want to add Visa payment to my website What can I use

**Problem:** I want to add Visa payment to my website What can I use?

**Solution:** ``` product/ Create.kt Delete.kt Update.kt ```

---

### how do I change the name of the website that appears on the tab

**Problem:** how do I change the name of the website that appears on the tab?

**Solution:** You can also override it from code doing something like: ``` LaunchedEffect(title) { document.title = title } ```

---

### No problem, my apps aren't used by anyone for the moment and I can always wait a bit before updating ````

**Problem:** No problem, my apps aren't used by anyone for the moment and I can always wait a bit before updating

**Solution:** ```kotlin Anchor("/") { Text("Home") } ```

```kotlin
Anchor("/") {
  Text("Home")
}
```

---

### given the first two code snippets <https://docs.asciidoctor.org/asciidoctor.js/latest/setup/quick-tour/> is th

**Problem:** given the first two code snippets <https://docs.asciidoctor.org/asciidoctor.js/latest/setup/quick-tour/> is this not supposed to work? I get `TypeError: Cannot read properties of undefined (reading 'convert')` ```kt external object Asciidoctor { fun convert(input: String): String } ```

**Solution:** Asciidoctor should be a class that you instantiate actually

```kt
external object Asciidoctor {
    fun convert(input: String): String
}
```

```js
import asciidoctor from 'asciidoctor' // (1)

const Asciidoctor = asciidoctor()
const content = 'http://asciidoctor.org[*Asciidoctor*] ' +
  'running on https://opalrb.com[_Opal_] ' +
  'brings AsciiDoc to Node.js!'
const html = Asciidoctor.convert(content) // (2)
console.log(html) // (3)
```

---

### Can I say that Kobweb use type-safe styling

**Problem:** Can I say that Kobweb use type-safe styling?

**Solution:** Ooof flutter still not being ready for production -- that's rough to hear! But yeah having experience with Flutter means you know enough to make an informed decision between C4W and Kobweb

---

### Hi. I'm thinking about adding Kobweb to a KMP project to share business logic code. Is this possible

**Problem:** Hi. I'm thinking about adding Kobweb to a KMP project to share business logic code. Is this possible?

**Solution:** BTW you may also want to check out https://kobweb.varabyte.com/docs/guides/existing-project The hardest part of adding Kobweb to an existing project at the moment is creating a conf.yaml file but it's easy to borrow it from another project

---

### In android we use viewModel for managing the state of our app

**Problem:** In android we use viewModel for managing the state of our app. I searched for the view models in kobweb on Google but didn't find anything.. Can anyone tell me what's the best method of managing the state in KobWeb especially when ur web app has a lot of forms

**Solution:** This question is coming up a lot recently. I should probably write a docs article about it...

---

### for Tabs there's no way to programmatically change tabs, is there

**Problem:** for Tabs there's no way to programmatically change tabs, is there?

**Solution:** Basically you would pass something into TabsPanel I think? But it's possible there isn't a way

---

### Is Kobweb compatible with Kotlin 2.3.0 ? I'm recovering from jet lag and it's taking way longer than I expecte

**Problem:** Is Kobweb compatible with Kotlin 2.3.0 ?

**Solution:** I'm recovering from jet lag and it's taking way longer than I expected. Aiming for a new release within a few days.

---


