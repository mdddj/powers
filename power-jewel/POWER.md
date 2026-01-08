---
name: "jewel"
displayName: "Jewel - Compose for Desktop Theme"
description: "Build IntelliJ-style desktop applications using Jewel, a Compose for Desktop theme that provides IntelliJ Platform look and feel components."
keywords: ["jewel", "compose", "desktop", "kotlin", "intellij", "theme", "ui", "components", "jetbrains", "swing", "multiplatform"]
---

# Jewel - Compose for Desktop Theme

Jewel is a Compose for Desktop theme that provides IntelliJ Platform look and feel. It allows you to build desktop applications with the same visual style as JetBrains IDEs.

## When to Use This Power

This power should be activated when:
- Building desktop applications with Compose for Desktop
- Creating IntelliJ-style UI components
- Working with Jewel components (buttons, text fields, tabs, etc.)
- Theming Compose Desktop applications
- Building IDE plugins with Compose UI

## Quick Start

### Add Dependencies

```kotlin
dependencies {
    implementation("org.jetbrains.jewel:jewel-int-ui-standalone:VERSION")
    implementation("org.jetbrains.jewel:jewel-int-ui-decorated-window:VERSION")
}
```

### Basic Application Setup

```kotlin
import androidx.compose.ui.window.application
import org.jetbrains.jewel.foundation.theme.JewelTheme
import org.jetbrains.jewel.intui.standalone.theme.IntUiTheme

fun main() = application {
    IntUiTheme(isDark = false) {
        // Your app content here
    }
}
```

## Available Components

### Buttons
- `OutlinedButton` - Standard outlined button
- `DefaultButton` - Primary action button
- `IconButton` - Button with icon only
- `ActionButton` - Button with optional tooltip
- `SplitButton` - Button with dropdown menu

### Text Input
- `TextField` - Single-line text input
- `TextArea` - Multi-line text input
- `EditableComboBox` - Editable dropdown

### Selection
- `Checkbox` / `TriStateCheckboxRow` - Checkbox with three states
- `RadioButton` - Radio button selection
- `ComboBox` / `ListComboBox` - Dropdown selection
- `SegmentedControl` - Segmented button group

### Navigation
- `TabStrip` - Tab navigation (Default and Editor styles)
- `Link` / `ExternalLink` / `DropdownLink` - Clickable links

### Feedback
- `HorizontalProgressBar` - Determinate progress
- `IndeterminateHorizontalProgressBar` - Indeterminate progress
- `CircularProgressIndicator` - Circular spinner
- `Tooltip` - Hover tooltips
- `Banner` - Information banners

### Layout
- `SplitLayout` - Resizable split panes
- `VerticallyScrollableContainer` - Scrollable container
- `GroupHeader` - Section headers

### Other
- `Icon` - SVG icon rendering with theming
- `Text` - Styled text component
- `Slider` - Value slider
- `Tree` - Tree view component

## When to Load Steering Files

- Learning component usage → `components.md`
- Understanding theming and styling → `theming.md`
- Looking for code examples → `examples.md`

## Theming

Jewel supports light and dark themes out of the box:

```kotlin
IntUiTheme(isDark = true) {
    // Dark theme content
}

IntUiTheme(isDark = false) {
    // Light theme content
}
```

Access theme values:
```kotlin
val colors = JewelTheme.globalColors
val textStyle = JewelTheme.defaultTextStyle
```

## Best Practices

1. **Use JewelTheme** - Access colors and styles through `JewelTheme`
2. **Follow IntelliJ patterns** - Match IntelliJ UI conventions
3. **Handle states** - Components support enabled, disabled, focused, hovered states
4. **Use proper outlines** - `Outline.Error` and `Outline.Warning` for validation
5. **Leverage icons** - Use `AllIconsKeys` for consistent iconography

## Resources

- [GitHub Repository](https://github.com/JetBrains/intellij-community/tree/master/platform/jewel)
- [IntelliJ Platform](https://www.jetbrains.com/opensource/idea)
