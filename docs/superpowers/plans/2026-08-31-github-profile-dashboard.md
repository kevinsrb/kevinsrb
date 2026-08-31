# GitHub Profile Dashboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert the `kevinsrb` profile README into a dark graphite and orange hybrid dashboard with a local profile card, a new KR banner, categorized technologies, and three real project cards.

**Architecture:** GitHub-compatible HTML provides a 28/72 desktop layout and keeps important links and text interactive. Two local SVG components provide the visual treatment that GitHub Markdown cannot express, while local avatar and icon assets remove avoidable runtime dependencies. A PowerShell validator checks assets, SVG structure, required content, links, and the two remaining dynamic statistics images.

**Tech Stack:** GitHub Flavored Markdown, GitHub-safe HTML, SVG 1.1, PNG, PowerShell 7, Git, GitHub REST API, Simple Icons and Devicon CDNs as download sources only.

**Spec:** `docs/superpowers/specs/2026-08-31-github-profile-dashboard-design.md`

## Global Constraints

- Preserve `assets/banner-final.svg` and every existing SVG unchanged.
- Preserve commit `0cd0827` as the published rollback point.
- Use `#ff5714` as the primary orange accent over `#071016`, `#0b1117`, and `#111820` graphite backgrounds.
- Display `Kevin Rodriguez`, `kevinsrb`, `San Marcos - Sucre, Colombia`, `kevinsrb.1999@gmail.com`, `linkedin.com/in/kevinsamirrodriguez`, and `Joined Apr 2021` exactly.
- Do not show a static clock, invented repository, or manually maintained follower, star, fork, or contribution count.
- Keep individual GitHub, LinkedIn, email, contact, and repository links outside the SVG files.
- Keep `github-stats-extended.vercel.app` as the only runtime image service required for dynamic data.
- Optimize the 28/72 composition for desktop while keeping all text readable when the right column is approximately 620 px wide.
- Use only public account data and assets that can be committed to this repository.

## File Structure

- Create `scripts/validate-profile.ps1`: repeatable validation for assets, SVGs, README structure, and remote statistics cards.
- Create `assets/avatar-kevin.png`: current public GitHub avatar downloaded from `https://avatars.githubusercontent.com/u/83133879?v=4`.
- Create `assets/icons/*.svg`: local official-color icons used by the banner and technology labels.
- Create `assets/profile-sidebar.svg`: self-contained 320 × 980 profile card with the PNG avatar embedded as a data URI.
- Create `assets/banner-dashboard.svg`: self-contained 1120 × 340 KR hero banner with inline brand vector paths.
- Modify `README.md`: GitHub-safe two-column shell, interactive links, about copy, categorized technology rows, three projects, and dynamic activity cards.
- Keep `assets/banner.svg`, `assets/banner-v2.svg`, `assets/banner-refined.svg`, and `assets/banner-final.svg` untouched.

---

### Task 1: Local asset set and validation harness

**Files:**
- Create: `scripts/validate-profile.ps1`
- Create: `assets/avatar-kevin.png`
- Create: `assets/icons/nodedotjs.svg`
- Create: `assets/icons/typescript.svg`
- Create: `assets/icons/nestjs.svg`
- Create: `assets/icons/express.svg`
- Create: `assets/icons/fastify.svg`
- Create: `assets/icons/dotnet.svg`
- Create: `assets/icons/csharp.svg`
- Create: `assets/icons/angular.svg`
- Create: `assets/icons/react.svg`
- Create: `assets/icons/vuedotjs.svg`
- Create: `assets/icons/postgresql.svg`
- Create: `assets/icons/mysql.svg`
- Create: `assets/icons/mongodb.svg`
- Create: `assets/icons/redis.svg`
- Create: `assets/icons/firebase.svg`
- Create: `assets/icons/amazonwebservices.svg`
- Create: `assets/icons/googlecloud.svg`
- Create: `assets/icons/docker.svg`
- Create: `assets/icons/kubernetes.svg`
- Create: `assets/icons/githubactions.svg`
- Create: `assets/icons/linux.svg`
- Create: `assets/icons/github.svg`
- Create: `assets/icons/linkedin.svg`
- Create: `assets/icons/gmail.svg`

**Interfaces:**
- Consumes: Public GitHub avatar URL plus verified Simple Icons and Devicon CDN SVG responses.
- Produces: `scripts/validate-profile.ps1 -Scope Assets|Sidebar|Banner|Readme|Remote|All`; local files used by Tasks 2–4.

- [ ] **Step 1: Write the failing asset validator**

Create `scripts/validate-profile.ps1` with these exact scopes and checks:

```powershell
param(
  [ValidateSet('Assets', 'Sidebar', 'Banner', 'Readme', 'Remote', 'All')]
  [string]$Scope = 'All'
)

$ErrorActionPreference = 'Stop'
$repoRoot = Split-Path -Parent $PSScriptRoot
$failures = [System.Collections.Generic.List[string]]::new()

function Add-Failure([string]$Message) { $failures.Add($Message) }
function Assert-File([string]$RelativePath) {
  $path = Join-Path $repoRoot $RelativePath
  if (-not (Test-Path -LiteralPath $path -PathType Leaf)) {
    Add-Failure "Missing file: $RelativePath"
    return $null
  }
  return $path
}
function Assert-Svg([string]$RelativePath, [string[]]$RequiredText) {
  $path = Assert-File $RelativePath
  if (-not $path) { return }
  try { [xml]$xml = Get-Content -Raw -LiteralPath $path }
  catch { Add-Failure "Invalid SVG XML: $RelativePath"; return }
  if ($xml.DocumentElement.LocalName -ne 'svg') { Add-Failure "Not an SVG root: $RelativePath" }
  $source = Get-Content -Raw -LiteralPath $path
  foreach ($text in $RequiredText) {
    if (-not $source.Contains($text)) { Add-Failure "Missing '$text' in $RelativePath" }
  }
}

$iconNames = @(
  'nodedotjs','typescript','nestjs','express','fastify','dotnet','csharp',
  'angular','react','vuedotjs','postgresql','mysql','mongodb','redis',
  'firebase','amazonwebservices','googlecloud','docker','kubernetes',
  'githubactions','linux','github','linkedin','gmail'
)

if ($Scope -in @('Assets', 'All')) {
  $avatar = Assert-File 'assets/avatar-kevin.png'
  if ($avatar) {
    $bytes = [IO.File]::ReadAllBytes($avatar)
    if ($bytes.Length -lt 8 -or $bytes[0] -ne 0x89 -or $bytes[1] -ne 0x50) {
      Add-Failure 'assets/avatar-kevin.png is not a valid PNG payload'
    }
  }
  foreach ($name in $iconNames) { Assert-Svg "assets/icons/$name.svg" @('<svg') }
}

if ($Scope -in @('Sidebar', 'All')) {
  Assert-Svg 'assets/profile-sidebar.svg' @(
    'viewBox="0 0 320 980"', 'data:image/png;base64,', 'Kevin Rodriguez',
    'San Marcos - Sucre, Colombia', 'Joined Apr 2021'
  )
}

if ($Scope -in @('Banner', 'All')) {
  Assert-Svg 'assets/banner-dashboard.svg' @(
    'viewBox="0 0 1120 340"', 'KR', 'KEVIN', 'RODRIGUEZ',
    'Backend Focused', 'Clean Architecture', 'Scalable APIs', 'Cloud Native',
    'Node.js', 'TypeScript', 'NestJS', 'AWS', 'GCP', 'Docker', 'Kubernetes'
  )
}

if ($Scope -in @('Readme', 'All')) {
  $readmePath = Assert-File 'README.md'
  if ($readmePath) {
    $readme = Get-Content -Raw -LiteralPath $readmePath
    $required = @(
      './assets/profile-sidebar.svg', './assets/banner-dashboard.svg',
      'https://github.com/kevinsrb',
      'https://www.linkedin.com/in/kevinsamirrodriguez/',
      'mailto:kevinsrb.1999@gmail.com',
      'inventory-management-api', 'create-flex-stack', 'yugioh-app',
      'github-stats-extended.vercel.app/api?username=kevinsrb',
      'github-stats-extended.vercel.app/api/top-langs/?username=kevinsrb'
    )
    foreach ($text in $required) {
      if (-not $readme.Contains($text)) { Add-Failure "README missing: $text" }
    }
    if ($readme -match '[A-Za-z]:\\') { Add-Failure 'README contains a Windows local path' }
    if ($readme.Contains('skillicons.dev')) { Add-Failure 'README still depends on skillicons.dev' }
    if ($readme.Contains('Joined Dec 2019')) { Add-Failure 'README contains the incorrect join date' }
  }
}

if ($Scope -in @('Remote', 'All')) {
  $urls = @(
    'https://github-stats-extended.vercel.app/api?username=kevinsrb&show_icons=true&hide_border=true&bg_color=0b0f12&title_color=ff5714&icon_color=ff5714&text_color=c9d1d9&ring_color=ff5714',
    'https://github-stats-extended.vercel.app/api/top-langs/?username=kevinsrb&layout=compact&hide_border=true&bg_color=0b0f12&title_color=ff5714&text_color=c9d1d9'
  )
  foreach ($url in $urls) {
    try {
      $response = Invoke-WebRequest -Uri $url -UseBasicParsing -TimeoutSec 30
      if ($response.StatusCode -ne 200) { Add-Failure "Remote status $($response.StatusCode): $url" }
      if ($response.Content -notmatch '<svg') { Add-Failure "Remote response is not SVG: $url" }
    }
    catch { Add-Failure "Remote request failed: $url — $($_.Exception.Message)" }
  }
}

if ($failures.Count -gt 0) {
  $failures | ForEach-Object { Write-Error $_ -ErrorAction Continue }
  exit 1
}

Write-Output "Profile validation passed for scope: $Scope"
```

- [ ] **Step 2: Run the asset scope and confirm the red state**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Assets
```

Expected: exit code `1` with `Missing file: assets/avatar-kevin.png` and missing icon messages.

- [ ] **Step 3: Download the exact local assets**

Create `assets/icons/`, download the avatar, and download each official-color SVG. Use this exact mapping:

```powershell
$headers = @{ 'User-Agent' = 'kevinsrb-profile-assets' }
New-Item -ItemType Directory -Path 'assets/icons' -Force | Out-Null
Invoke-WebRequest -Uri 'https://avatars.githubusercontent.com/u/83133879?v=4' -Headers $headers -OutFile 'assets/avatar-kevin.png'

$icons = [ordered]@{
  nodedotjs='https://cdn.simpleicons.org/nodedotjs/5FA04E'
  typescript='https://cdn.simpleicons.org/typescript/3178C6'
  nestjs='https://cdn.simpleicons.org/nestjs/E0234E'
  express='https://cdn.simpleicons.org/express/FFFFFF'
  fastify='https://cdn.simpleicons.org/fastify/FFFFFF'
  dotnet='https://cdn.simpleicons.org/dotnet/512BD4'
  csharp='https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/csharp/csharp-original.svg'
  angular='https://cdn.simpleicons.org/angular/DD0031'
  react='https://cdn.simpleicons.org/react/61DAFB'
  vuedotjs='https://cdn.simpleicons.org/vuedotjs/4FC08D'
  postgresql='https://cdn.simpleicons.org/postgresql/4169E1'
  mysql='https://cdn.simpleicons.org/mysql/4479A1'
  mongodb='https://cdn.simpleicons.org/mongodb/47A248'
  redis='https://cdn.simpleicons.org/redis/FF4438'
  firebase='https://cdn.simpleicons.org/firebase/DD2C00'
  amazonwebservices='https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/amazonwebservices/amazonwebservices-plain-wordmark.svg'
  googlecloud='https://cdn.simpleicons.org/googlecloud/4285F4'
  docker='https://cdn.simpleicons.org/docker/2496ED'
  kubernetes='https://cdn.simpleicons.org/kubernetes/326CE5'
  githubactions='https://cdn.simpleicons.org/githubactions/2088FF'
  linux='https://cdn.simpleicons.org/linux/FCC624'
  github='https://cdn.simpleicons.org/github/FFFFFF'
  linkedin='https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/linkedin/linkedin-original.svg'
  gmail='https://cdn.simpleicons.org/gmail/EA4335'
}
foreach ($entry in $icons.GetEnumerator()) {
  Invoke-WebRequest -Uri $entry.Value -Headers $headers -OutFile "assets/icons/$($entry.Key).svg"
}
```

- [ ] **Step 4: Run the green asset validation**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Assets
```

Expected: `Profile validation passed for scope: Assets` and exit code `0`.

- [ ] **Step 5: Commit the validator and local source assets**

```powershell
git add scripts/validate-profile.ps1 assets/avatar-kevin.png assets/icons
git commit -m "chore: add local profile visual assets"
```

---

### Task 2: Profile sidebar SVG

**Files:**
- Create: `assets/profile-sidebar.svg`
- Test: `scripts/validate-profile.ps1`

**Interfaces:**
- Consumes: `assets/avatar-kevin.png`; exact personal copy in Global Constraints.
- Produces: a self-contained SVG with `viewBox="0 0 320 980"` referenced by `README.md` in Task 4.

- [ ] **Step 1: Confirm the sidebar test is red**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Sidebar
```

Expected: exit code `1` with `Missing file: assets/profile-sidebar.svg`.

- [ ] **Step 2: Confirm the avatar can be converted to an embedded data URI**

Run this read-only conversion and keep the output available for the `href` value written with `apply_patch`:

```powershell
$avatarBytes = [IO.File]::ReadAllBytes((Resolve-Path 'assets/avatar-kevin.png'))
$avatarDataUri = 'data:image/png;base64,' + [Convert]::ToBase64String($avatarBytes)
if ($avatarDataUri.Length -le 1000) { throw 'Avatar data URI is unexpectedly short' }
"Avatar data URI ready: $($avatarDataUri.Length) characters"
```

Expected: `Avatar data URI ready:` followed by a length greater than `1000`.

- [ ] **Step 3: Create the sidebar with fixed visual zones**

Use `apply_patch` to create `assets/profile-sidebar.svg`. Use these dimensions and coordinates:

- Root: `width="320" height="980" viewBox="0 0 320 980"`.
- Background: rounded rectangle at `x=4 y=4 width=312 height=972 rx=18`, fill `#071016`, stroke `#33404a`.
- Avatar: circular clip centered at `(160,126)` with radius `86`; `<image>` at `x=74 y=40 width=172 height=172`, initially using the exact token `__AVATAR_DATA_URI__`; two rings at radii `91` and `96` in `#ff5714` and `#5e2a16`.
- Status: green circle at `(238,194)` radius `12`, with `#081016` outer stroke.
- Identity: `Kevin Rodriguez` at `(28,252)`, `kevinsrb` at `(28,279)`, and `Full Stack Developer · Backend Focused` at `(28,312)`.
- Bio: four lines beginning at `y=356`, each at most 42 characters, describing backend, robust solutions, scalability, and efficiency.
- Divider: from `(28,438)` to `(292,438)`.
- Details: five rows at `y=478, 522, 566, 610, 654` for location, email, LinkedIn, username, and `Joined Apr 2021`. Use inline stroke icons; do not use emoji glyphs.
- Connection panel: rounded rectangle at `x=22 y=710 width=276 height=224 rx=14`, title `Conectemos`, three circular decorative icons for GitHub, LinkedIn, and email, and footer `Backend · Cloud · IA`.
- Typography: `Inter, Segoe UI, Arial, sans-serif`; main text `#f3f4f6`; secondary text `#b6c2cc`; accent `#ff5714`.
- Add `<title>Perfil de Kevin Rodriguez</title>` and `<desc>Desarrollador Full Stack especializado en backend, arquitectura y cloud.</desc>`.

Do not include link elements in the SVG. Task 4 supplies interactive controls outside the image.

Immediately after the `apply_patch`, perform this one mechanical token replacement. The finished SVG must not contain the token:

```powershell
$avatarBytes = [IO.File]::ReadAllBytes((Resolve-Path 'assets/avatar-kevin.png'))
$avatarDataUri = 'data:image/png;base64,' + [Convert]::ToBase64String($avatarBytes)
$sidebarPath = Resolve-Path 'assets/profile-sidebar.svg'
$sidebarSource = [IO.File]::ReadAllText($sidebarPath)
$sidebarSource = $sidebarSource.Replace('__AVATAR_DATA_URI__', $avatarDataUri)
[IO.File]::WriteAllText($sidebarPath, $sidebarSource, [Text.UTF8Encoding]::new($false))
if ([IO.File]::ReadAllText($sidebarPath).Contains('__AVATAR_DATA_URI__')) {
  throw 'Avatar token remains in profile-sidebar.svg'
}
```

- [ ] **Step 4: Verify XML, required copy, and visual rendering**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Sidebar
```

Expected: `Profile validation passed for scope: Sidebar`.

Open `assets/profile-sidebar.svg` at its native size and at 280 px wide. Confirm that the face is centered, the name does not clip, every detail row fits, and the orange ring is continuous.

- [ ] **Step 5: Commit the profile card**

```powershell
git add assets/profile-sidebar.svg
git commit -m "feat: add profile sidebar card"
```

---

### Task 3: KR dashboard banner

**Files:**
- Create: `assets/banner-dashboard.svg`
- Read: `assets/banner-final.svg`
- Read: `assets/icons/nodedotjs.svg`
- Read: `assets/icons/typescript.svg`
- Read: `assets/icons/nestjs.svg`
- Read: `assets/icons/amazonwebservices.svg`
- Read: `assets/icons/googlecloud.svg`
- Read: `assets/icons/docker.svg`
- Read: `assets/icons/kubernetes.svg`
- Test: `scripts/validate-profile.ps1`

**Interfaces:**
- Consumes: the existing KR geometry and verified official icon paths.
- Produces: `assets/banner-dashboard.svg` with `viewBox="0 0 1120 340"` for the right README column.

- [ ] **Step 1: Confirm the banner test is red**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Banner
```

Expected: exit code `1` with `Missing file: assets/banner-dashboard.svg`.

- [ ] **Step 2: Create the banner foundation**

Use `apply_patch` to create the SVG with:

- Root `width="1120" height="340" viewBox="0 0 1120 340"`.
- Rounded background at `x=4 y=4 width=1112 height=332 rx=20`, fill gradient from `#071016` to `#0b1117`, stroke `#33404a`.
- A dotted orange radial pattern in the top-right quadrant at 12 % opacity.
- Circuit lines on the upper-right and lower-right using `#ff5714`, 1.5 px strokes, circular terminals, and no line behind text.
- A 250 px circular technical dial centered at `(168,170)` with three thin rings, radial ticks, and the existing `KR` monogram scaled inside it.
- Text `KEVIN` at `(330,118)` in `#f3f4f6` and `RODRIGUEZ` beginning at `x=555` in `#ff5714`; keep at least 18 px of visible separation.
- Subtitle at `(334,151)`: `FULL STACK DEVELOPER · BACKEND FOCUSED`.

- [ ] **Step 3: Add specialty chips and the technology rail**

Add four chips centered below the title:

- `Backend Focused`, starting at `x=330`.
- `Clean Architecture`, starting at `x=488`.
- `Scalable APIs`, starting at `x=672`.
- `Cloud Native`, starting at `x=820`.

Use `height=34`, `rx=9`, fill `#111820`, stroke `#3a4650`, 13 px text, and orange line icons. Align every icon to the same centerline.

Add a technology rail at `x=318 y=224 width=672 height=82 rx=14`. Inline the exact vector paths read from the seven local icon files; do not reference CDN URLs from the finished SVG. Place 32 px icons and labels at centers `370, 462, 554, 646, 738, 830, 922`, in this order: Node.js, TypeScript, NestJS, AWS, GCP, Docker, Kubernetes.

Add three decorative social circles at `x=1044`, `y=122,170,218` for GitHub, LinkedIn, and a globe; label `@kevinsrb` at `(1002,265)`. Interactive links remain in Task 4.

- [ ] **Step 4: Verify XML, alignment, and icon identity**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Banner
```

Expected: `Profile validation passed for scope: Banner`.

Open `assets/banner-dashboard.svg` at 1120 px and 620 px wide. Confirm that `KEVIN RODRIGUEZ` is separated, KR reads as two letters, the four chips share one baseline, and the AWS, Docker, and Kubernetes marks match their local source icons.

- [ ] **Step 5: Commit the hero banner**

```powershell
git add assets/banner-dashboard.svg
git commit -m "feat: add dashboard hero banner"
```

---

### Task 4: Interactive README dashboard

**Files:**
- Modify: `README.md`
- Read: `assets/profile-sidebar.svg`
- Read: `assets/banner-dashboard.svg`
- Read: `assets/icons/*.svg`
- Test: `scripts/validate-profile.ps1`

**Interfaces:**
- Consumes: both local SVG components, the local icon set, verified personal copy, and three public repository URLs.
- Produces: the complete GitHub profile README with native links and dynamic activity cards.

- [ ] **Step 1: Confirm the README test is red**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Readme
```

Expected: exit code `1` reporting the missing sidebar, dashboard banner, and `yugioh-app` references.

- [ ] **Step 2: Replace the page shell with the 28/72 layout**

Use `apply_patch` to replace `README.md` with one outer presentation table:

```html
<table role="presentation">
  <tr>
    <td width="28%" valign="top">
      <a href="https://github.com/kevinsrb"><img src="./assets/profile-sidebar.svg" width="100%" alt="Perfil de Kevin Rodriguez, desarrollador Full Stack especializado en backend"></a>
    </td>
    <td width="72%" valign="top">
      <img src="./assets/banner-dashboard.svg" width="100%" alt="Kevin Rodriguez — Full Stack Developer, Backend Focused">
    </td>
  </tr>
</table>
```

- [ ] **Step 3: Add native action links and the two-column about section**

Under the sidebar SVG, add centered GitHub, LinkedIn, and email links using local 18 px icons and visible text. Under the banner, add four links in this exact order:

1. `GitHub` → `https://github.com/kevinsrb`.
2. `LinkedIn` → `https://www.linkedin.com/in/kevinsamirrodriguez/`.
3. `Email` → `mailto:kevinsrb.1999@gmail.com`.
4. `Contacto` → `mailto:kevinsrb.1999@gmail.com?subject=Contacto%20profesional%20desde%20GitHub`.

Use `<kbd>` inside each anchor to create a GitHub-native button surface without CSS or a badge service.

Add `Sobre mí` with one introductory paragraph and a nested two-column table. The left side covers robust APIs, Node.js/TypeScript/NestJS/.NET, and AWS/GCP/Docker/Kubernetes. The right side covers relational and NoSQL databases, IA/automation/DevOps, and the objective of building maintainable products. Keep the existing claim `+6 años de experiencia`.

- [ ] **Step 4: Add categorized local technology rows**

Add four labeled rows and use local 18 px `<img>` icons followed by visible `<code>` names:

- Backend: Node.js, TypeScript, NestJS, Express, Fastify, .NET, C#.
- Frontend: Angular, React, Vue.js.
- Bases de datos: PostgreSQL, MySQL, MongoDB, Redis, Firestore; use `firebase.svg` for Firestore.
- DevOps & Cloud: AWS, GCP, Docker, Kubernetes, GitHub Actions, Linux.

Every icon must include an `alt` attribute equal to its visible technology name. Do not include `skillicons.dev` or `img.shields.io` technology images.

- [ ] **Step 5: Add three real project cards**

Create a three-cell table and use this verified content:

- `inventory-management-api`: “API REST de inventario con NestJS y TypeScript, diseñada con arquitectura limpia, reglas de dominio explícitas y consistencia de datos.” Stack: NestJS, TypeScript, PostgreSQL, Prisma, Jest, Docker.
- `create-flex-stack`: “Generador TypeScript para crear APIs Express y frontends React + Vite con arquitectura, persistencia y herramientas opcionales.” Stack: TypeScript, Express, React, Prisma, Docker.
- `yugioh-app`: “Explorador frontend de cartas de Yu-Gi-Oh! con búsqueda, filtros, favoritos persistentes, caché de peticiones y modo oscuro.” Stack: React, Vite, TypeScript, TanStack Query, Tailwind CSS, Zustand.

Link the titles to `https://github.com/kevinsrb/inventory-management-api`, `https://github.com/kevinsrb/create-flex-stack`, and `https://github.com/kevinsrb/yugioh-app`, respectively. Add one `Ver todos los repositorios →` link to `https://github.com/kevinsrb?tab=repositories`. Do not add star or fork counts.

- [ ] **Step 6: Add dynamic activity with graceful fallback copy**

Reuse the two existing `github-stats-extended.vercel.app` URLs without changing their query parameters. Keep the existing Spanish `alt` text and wrap the section in a link to `https://github.com/kevinsrb` so the account remains reachable if an image fails.

Add the visible fallback sentence: `Consulta mi actividad y repositorios directamente en GitHub.` with the same profile link.

- [ ] **Step 7: Run the README validator and inspect the diff**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope Readme
git diff --check
git diff -- README.md
```

Expected: README scope passes, `git diff --check` prints nothing, only the approved content appears in the README diff, and all four action URLs are visible.

- [ ] **Step 8: Commit the README composition**

```powershell
git add README.md
git commit -m "feat: redesign GitHub profile dashboard"
```

---

### Task 5: Full verification, GitHub rendering, and publication

**Files:**
- Verify: `README.md`
- Verify: `assets/profile-sidebar.svg`
- Verify: `assets/banner-dashboard.svg`
- Verify: `assets/avatar-kevin.png`
- Verify: `assets/icons/*.svg`
- Verify: `scripts/validate-profile.ps1`

**Interfaces:**
- Consumes: the complete output of Tasks 1–4.
- Produces: a verified commit sequence on `main`, pushed to `origin/main`, with `0cd0827` still available for rollback.

- [ ] **Step 1: Run complete local and remote validation**

Run:

```powershell
pwsh -NoProfile -File scripts/validate-profile.ps1 -Scope All
git diff --check
git status --short --branch
```

Expected: validator exit code `0`, no whitespace errors, no uncommitted implementation files, and `main` ahead of `origin/main` by the new commits.

- [ ] **Step 2: Verify GitHub's Markdown conversion**

Run:

```powershell
$payload = @{
  text = Get-Content -Raw 'README.md'
  mode = 'gfm'
  context = 'kevinsrb/kevinsrb'
} | ConvertTo-Json
$rendered = Invoke-RestMethod -Uri 'https://api.github.com/markdown' -Method Post -Headers @{ 'User-Agent' = 'kevinsrb-profile-validation' } -ContentType 'application/json' -Body $payload
$requiredHtml = @('profile-sidebar.svg', 'banner-dashboard.svg', 'inventory-management-api', 'create-flex-stack', 'yugioh-app')
foreach ($item in $requiredHtml) {
  if (-not $rendered.Contains($item)) { throw "GitHub-rendered HTML missing: $item" }
}
'GitHub Markdown conversion contains every required component.'
```

Expected: the final success line and no exception.

- [ ] **Step 3: Perform visual QA at both target widths**

Open `assets/profile-sidebar.svg` at 320 px and 280 px. Open `assets/banner-dashboard.svg` at 1120 px and 620 px. Then preview the complete rendered README at approximately 980 px and 760 px.

Reject the visual if any of these conditions occur: clipped name, unreadable detail text, distorted avatar, joined words in `KEVIN RODRIGUEZ`, incorrect AWS/Docker/Kubernetes mark, specialty chip off its baseline, or project cards with unequal top alignment. Fix the relevant SVG or README with `apply_patch`, rerun the affected validator scope, then rerun `-Scope All`.

- [ ] **Step 4: Verify the final repository state and push**

Run:

```powershell
git log --oneline --decorate -8
git status --short --branch
git push origin main
git status --short --branch
```

Expected: push succeeds, the final status is `## main...origin/main`, and commit `0cd0827` remains visible in the log.

- [ ] **Step 5: Verify the published profile**

Open `https://github.com/kevinsrb` after the push. Confirm that both columns, local images, native links, technology labels, three project cards, and two dynamic statistics images render. If GitHub's sanitizer removes an element, replace only that unsupported element with a simpler table, anchor, image, `strong`, or `code` element and repeat Steps 1–5.
