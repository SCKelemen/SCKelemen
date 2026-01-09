# Data Visualization System Architecture

A modular ecosystem of Go packages for building terminal and web-based data visualizations and sophisticated TUI applications.

## System Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                      Applications                                 │
├──────────────┬──────────────┬────────────────┬──────────────────┤
│   viz-cli    │   readme     │   tui          │   clix (ext)      │
│   (terminal) │   (web/SVG)  │   (framework)  │   (TUI alt)       │
└──────┬───────┴───────┬──────┴────────┬───────┴──────────────────┘
       │               │               │
       ▼               ▼               ▼
┌──────────────────────────────────────────────────────────────────┐
│                  Visualization & UI Layer                         │
├──────────────────────────────────────────────────────────────────┤
│  dataviz - Dual-mode rendering (SVG + Terminal)                  │
│  • Heatmaps, Line Graphs, Bar Charts, Stat Cards                 │
│  • Supports both web and CLI output                              │
└─────────┬────────────────────────────────────────────────────────┘
          │
          ├──────────┬─────────────┬────────────┐
          ▼          ▼             ▼            ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐
│   svg    │ │   cli    │ │  design  │ │   layout     │
│          │ │          │ │  -system │ │              │
│ SVG gen  │ │ Terminal │ │  Themes  │ │ Flexbox/Grid │
│          │ │ renderer │ │  Tokens  │ │              │
└─────┬────┘ └────┬─────┘ └────┬─────┘ └──────┬───────┘
      │           │            │               │
      └───────────┴────────────┴───────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌──────────┐     ┌──────────┐   ┌──────────┐
│  color   │     │   text   │   │  units   │
│          │     │          │   │          │
│ OKLCH    │     │ Unicode  │   │ CSS-like │
│ Gradients│     │ Width    │   │ px/ch/vw │
└──────────┘     └─────┬────┘   └──────────┘
                       │
                       ▼
                 ┌──────────┐
                 │ unicode  │
                 │          │
                 │ UTR#51   │
                 └──────────┘
```

---

## Repository Details

### Application Layer

#### viz-cli
**Path**: `github.com/SCKelemen/viz-cli`
**Purpose**: Command-line tool for generating data visualizations
**Output Formats**: Terminal (ANSI), SVG
**Dependencies**: dataviz, design-system, cli, text

**Features**:
- Read JSON data from files or stdin
- Generate heatmaps, line graphs, bar charts, stat cards
- Interactive terminal dashboards with Bubbletea
- Theme support (default, midnight, nord, paper, wrapped)

**Examples**:
```bash
viz-cli -type heatmap -format terminal -data contributions.json
viz-cli -type line-graph -format svg > output.svg
```

---

#### readme
**Path**: `github.com/SCKelemen/readme`
**Purpose**: GitHub-specific badge and card generation service
**Output Format**: SVG (via HTTP endpoints)
**Dependencies**: dataviz, design-system, svg, layout, color

**Features**:
- GitHub profile statistics visualization
- Repository contribution heatmaps
- Language usage bar charts
- Embeddable SVG badges for README files
- GitHub Linguist color integration

**Endpoints**:
- `/api/heatmap?username=octocat`
- `/api/line-graph?repo=octocat/hello-world`
- `/api/stat-card?metric=stars&value=1234`

---

#### tui
**Path**: `github.com/SCKelemen/tui`
**Purpose**: Comprehensive TUI framework for building Claude Code-like CLI experiences
**Dependencies**: cli, layout, design-system, text, color
**Status**: Private (Active Development)

**Features**:
- High-level component system (FileExplorer, CommandPalette, StatusBar, Modal, Tabs)
- Focus management with Tab/Shift+Tab navigation
- Keyboard navigation with customizable keymaps
- Mouse support (click, scroll, drag)
- Theme integration via design-system
- Component lifecycle management (Init, Update, View, Focus, Blur)

**Components**:
- **Application**: Main container with component orchestration
- **StatusBar**: Bottom bar with status messages and keybindings
- **FileExplorer**: Tree view with navigation and search (planned)
- **CommandPalette**: Fuzzy-searchable command launcher (planned)
- **Editor**: Text viewing/editing with syntax highlighting (planned)
- **Modal**: Dialog boxes for confirmations and inputs (planned)
- **Tabs**: Multi-view tab management (planned)

**Example**:
```go
app := tui.NewApplication()
statusBar := tui.NewStatusBar()
statusBar.SetMessage("Welcome to TUI")
app.AddComponent(statusBar)

p := tea.NewProgram(app, tea.WithAltScreen())
p.Run()
```

**Design Philosophy**:
- Built on top of cli renderer for low-level primitives
- Component-based architecture for composability
- Focus on developer experience and ergonomics
- Claude Code-inspired UX patterns

---

#### clix (Related)
**Path**: `github.com/SCKelemen/clix`
**Purpose**: Terminal UI framework
**Status**: External/related project
**Relationship**: Alternative TUI framework approach

---

### Visualization Layer

#### dataviz
**Path**: `github.com/SCKelemen/dataviz`
**Purpose**: Core data visualization library with dual rendering
**Output Formats**: SVG (via svg package), Terminal (via cli package)
**Dependencies**: design-system, svg, cli, color, layout, text

**Components**:
- **Heatmap**: Contribution calendars (linear, weeks layout)
- **Line Graph**: Time series with Braille characters
- **Bar Chart**: Horizontal/vertical bars with labels
- **Stat Card**: Metric cards with trend indicators
- **Scatter Plot**: Point-based visualizations
- **Area Chart**: Filled line graphs

**Dual Rendering**:
```go
// SVG output
renderer := dataviz.NewSVGRenderer()
svg := renderer.RenderHeatmap(data, bounds, config)

// Terminal output
renderer := dataviz.NewTerminalRenderer()
term := renderer.RenderHeatmap(data, bounds, config)
```

**Design Philosophy**:
- Single source of truth for visualization logic
- Renderer-agnostic data structures
- Consistent API across output formats
- Optimized for both web embedding and terminal display

---

### Rendering Layer

#### svg
**Path**: `github.com/SCKelemen/svg`
**Purpose**: Programmatic SVG generation
**Dependencies**: color, units

**Features**:
- Type-safe SVG element creation
- Gradient support (all color spaces: sRGB, OKLCH, LAB, etc.)
- Path building with fluent API
- Transform support (translate, scale, rotate)
- Filter effects support

**Example**:
```go
doc := svg.NewDocument(400, 200)
rect := svg.Rect(10, 10, 380, 180).
    Fill("#2196F3").
    Stroke("#000000", 2)
doc.Append(rect)
```

---

#### cli
**Path**: `github.com/SCKelemen/cli`
**Purpose**: Terminal rendering engine
**Dependencies**: layout, color, text, unicode

**Modules**:
- **renderer**: Screen buffers, ANSI output, styled nodes
- **components**: Pre-built TUI components (cards, charts, messages)

**Features**:
- Double-buffered screen rendering
- ANSI TrueColor support with fallbacks (256, 16 color)
- Border styles (rounded, double, thick, normal)
- Layout integration (flexbox, grid)
- Unicode-aware text handling

**Border Styles**:
```go
style := &renderer.Style{
    BorderColor: &accentColor,
}
style.WithBorder(renderer.RoundedBorder)
```

---

### Design System Layer

#### design-system
**Path**: `github.com/SCKelemen/design-system`
**Purpose**: Design tokens, themes, and styling configuration
**Dependencies**: color, units

**Token Types**:
- **DesignTokens**: Colors, typography, spacing, radii
- **LayoutTokens**: Spacing scales, component dimensions, grid defaults
- **MotionTokens**: Animation durations and amplitudes

**Themes**:
```go
design.DefaultTheme()    // Blue accent, light background
design.MidnightTheme()   // Purple accent, dark background
design.NordTheme()       // Arctic color palette
design.PaperTheme()      // Warm, paper-like aesthetic
design.WrappedTheme()    // Spotify-wrapped inspired
```

**Philosophy**:
- Single source of truth for visual consistency
- Centralized theme management
- Reusable across all visualization contexts

---

#### layout
**Path**: `github.com/SCKelemen/layout`
**Purpose**: CSS-like layout engine for terminal and web
**Dependencies**: units

**Layout Modes**:
- **Flexbox**: `display: flex` with flex-direction, justify-content, align-items
- **Grid**: `display: grid` with template rows/columns, gap
- **Block**: Traditional block layout

**Features**:
- Constraint-based sizing
- Viewport-relative units (vw, vh)
- Character-relative units (ch)
- Padding, margin, gap support
- Nested layout trees

**Example**:
```go
node := &layout.Node{
    Style: layout.Style{
        Display:       layout.DisplayFlex,
        FlexDirection: layout.FlexDirectionRow,
        Width:         layout.Vw(100),
        Height:        layout.Ch(20),
        Padding:       layout.Uniform(layout.Ch(1)),
    },
}
```

---

### Foundation Layer

#### color
**Path**: `github.com/SCKelemen/color`
**Purpose**: Color manipulation and conversion
**Dependencies**: None

**Color Spaces**:
- sRGB (standard web)
- OKLCH (perceptually uniform)
- LAB/OKLAB
- HSL/HSV
- Linear RGB

**Features**:
- Gradient generation with easing functions
- Color space conversions
- ANSI color approximation (for terminal fallbacks)
- Accessibility utilities (contrast ratios)

**Gradients**:
```go
gradient := color.Gradient(start, end, steps, color.GradientOKLCH)
```

---

#### text
**Path**: `github.com/SCKelemen/text`
**Purpose**: Unicode-aware text width measurement
**Dependencies**: unicode

**Features**:
- Correct visual width calculation (East Asian width, emoji)
- ANSI escape code stripping
- Terminal-specific text handling
- Multi-codepoint emoji support

**Why Needed**:
```go
// Naive len() gives 4, actual display width is 2
str := "你好"
width := text.NewTerminal().Width(str) // Returns 4 (2 chars × 2 width)
```

---

#### unicode
**Path**: `github.com/SCKelemen/unicode`
**Purpose**: Unicode property databases
**Standard**: UTR#51 (Unicode Emoji)

**Features**:
- East Asian Width property lookup
- Emoji sequence detection
- Grapheme cluster handling
- Unicode version tracking

---

#### units
**Path**: `github.com/SCKelemen/units`
**Purpose**: CSS-like unit system
**Dependencies**: None

**Unit Types**:
- **Absolute**: px (pixels), ch (character width)
- **Relative**: vw (viewport width), vh (viewport height)
- **Fraction**: fr (fraction of available space)

**Example**:
```go
width := layout.Vw(50)  // 50% of viewport width
padding := layout.Ch(2) // 2 character widths
```

---

### Testing Layer

#### wpt-test-gen
**Path**: `github.com/SCKelemen/wpt-test-gen`
**Purpose**: Web Platform Tests generator
**Use Case**: Generate conformance tests from W3C specifications

**Related To**: Ensures svg package conforms to SVG 2 spec

---

## Dependency Graph

```
Applications
    viz-cli    → dataviz, design-system, cli, text
    readme     → dataviz, design-system, svg, layout, color
    tui        → cli, layout, design-system, text, color

Core Libraries
    dataviz    → design-system, svg, cli, color, layout, text
    svg        → color, units
    cli        → layout, color, text, unicode

Design System
    design-system → color, units
    layout        → units

Foundation
    text       → unicode
    color      → (no deps)
    unicode    → (no deps)
    units      → (no deps)
```

---

## Data Flow

### Terminal Visualization Pipeline

```
User Data (JSON)
    ↓
dataviz.HeatmapData / LineGraphData / etc.
    ↓
dataviz.TerminalRenderer
    ↓
cli.StyledNode (with borders, colors)
    ↓
layout.Node (flexbox/grid positioning)
    ↓
cli.Screen (ANSI buffer)
    ↓
Terminal Output (with colors, borders, Unicode)
```

### SVG Visualization Pipeline

```
User Data (JSON)
    ↓
dataviz.HeatmapData / LineGraphData / etc.
    ↓
dataviz.SVGRenderer
    ↓
svg.Document (SVG elements)
    ↓
SVG String Output
    ↓
Web Browser / README.md
```

---

## Design Principles

### 1. Modularity
Each package has a single, well-defined responsibility. Packages can be used independently or composed together.

### 2. Dual Output
Visualizations work in both terminal and web contexts without code duplication.

### 3. Unicode Correctness
All text handling respects Unicode properties (width, combining characters, emoji).

### 4. Color Science
Gradients use perceptually uniform color spaces (OKLCH) for smooth transitions.

### 5. Layout Parity
Terminal layouts mirror web layout concepts (flexbox, grid, viewport units).

### 6. Design Tokens
Visual consistency through centralized theme management.

---

## Use Cases

### 1. Terminal Data Dashboards
**Stack**: viz-cli → dataviz → cli → layout + text
**Output**: Interactive TUI with real-time updates

### 2. GitHub Profile Badges
**Stack**: readme → dataviz → svg → color
**Output**: Embeddable SVG images

### 3. CLI Data Exploration
**Stack**: viz-cli → dataviz (terminal mode)
**Output**: Quick data visualization from command line

### 4. Custom Web Visualizations
**Stack**: dataviz → svg → color
**Output**: Programmatic SVG generation

### 5. Claude Code-like TUI Applications
**Stack**: tui → cli → layout + design-system + text
**Output**: Sophisticated terminal applications with modern UX patterns

---

## Development Status

| Package | Status | Version |
|---------|--------|---------|
| color | ✅ Stable | v1.x |
| unicode | ✅ Stable | v1.x |
| units | ✅ Stable | v1.x |
| text | ✅ Stable | v1.x |
| layout | 🚧 Active Development | v0.x |
| svg | 🚧 Active Development | v0.x |
| cli | 🚧 Active Development | v0.x |
| design-system | 🚧 Active Development | v0.x |
| dataviz | 🚧 Active Development | v0.x |
| tui | 🚧 Active Development (Private) | v0.x |
| viz-cli | 🚧 Active Development | v0.x |
| readme | 🚧 Active Development | v0.x |

---

## Getting Started

### Install viz-cli
```bash
go install github.com/SCKelemen/viz-cli@latest
```

### Use dataviz in Your Project
```bash
go get github.com/SCKelemen/dataviz@latest
go get github.com/SCKelemen/design-system@latest
```

### Example: Terminal Heatmap
```go
package main

import (
    "github.com/SCKelemen/dataviz"
    design "github.com/SCKelemen/design-system"
)

func main() {
    data := dataviz.HeatmapData{/* ... */}

    renderer := dataviz.NewTerminalRenderer()
    bounds := dataviz.Bounds{Width: 70, Height: 10}
    config := dataviz.RenderConfig{
        DesignTokens: design.DefaultTheme(),
        Color:        "#2196F3",
    }

    output := renderer.RenderHeatmap(data, bounds, config)
    fmt.Print(output.String())
}
```

---

## Future Directions

### Planned Features
- [ ] WebAssembly support for browser-based rendering
- [ ] More chart types (pie, radar, treemap)
- [ ] Animation support in terminal mode
- [ ] Accessibility improvements (screen reader support)
- [ ] Performance optimizations for large datasets

### Potential Integrations
- [ ] Integration with popular CLI frameworks (Cobra, urfave/cli)
- [ ] Prometheus metrics visualization
- [ ] Database query result visualization
- [ ] Log file analysis and visualization

---

## Contributing

Each repository has its own contribution guidelines. Generally:

1. Foundation packages (color, unicode, units, text) prioritize correctness and standards compliance
2. Layout and rendering packages (layout, svg, cli) focus on performance and correctness
3. Application packages (dataviz, viz-cli, readme) balance features with maintainability

---

## License

All packages are open source. Check individual repositories for specific license information.

---

## Links

- **Main Organization**: https://github.com/SCKelemen
- **viz-cli**: https://github.com/SCKelemen/viz-cli
- **dataviz**: https://github.com/SCKelemen/dataviz
- **cli**: https://github.com/SCKelemen/cli
- **tui**: https://github.com/SCKelemen/tui (private)
- **design-system**: https://github.com/SCKelemen/design-system

---

*Last Updated: 2026-01-09*
