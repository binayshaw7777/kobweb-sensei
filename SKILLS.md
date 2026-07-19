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
var colorMode by ColorMode.currentState
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

351 problem→solution entries extracted from the `#need_help` Discord channel.

See [FAQ.md](FAQ.md) for the full list.

