# Jewel Theming Guide

## Theme Setup

### Basic Theme Application

```kotlin
import androidx.compose.ui.window.application
import org.jetbrains.jewel.intui.standalone.theme.IntUiTheme

fun main() = application {
    Window(onCloseRequest = ::exitApplication) {
        IntUiTheme(isDark = false) {
            // Your app content
            App()
        }
    }
}
```

### Dynamic Theme Switching

```kotlin
var isDark by remember { mutableStateOf(false) }

IntUiTheme(isDark = isDark) {
    Column {
        // Theme toggle
        TriStateCheckboxRow(
            text = "Dark mode",
            state = if (isDark) ToggleableState.On else ToggleableState.Off,
            onClick = { isDark = !isDark }
        )
        
        // App content
        App()
    }
}
```

## Accessing Theme Values

### Global Colors

```kotlin
import org.jetbrains.jewel.foundation.theme.JewelTheme

@Composable
fun MyComponent() {
    val colors = JewelTheme.globalColors
    
    // Text colors
    val normalText = colors.text.normal
    val infoText = colors.text.info
    val errorText = colors.text.error
    val warningText = colors.text.warning
    
    // Background colors
    val background = colors.paneBackground
    
    Text("Info text", color = colors.text.info)
}
```

### Component Styles

```kotlin
import org.jetbrains.jewel.foundation.theme.JewelTheme
import org.jetbrains.jewel.ui.theme.*

@Composable
fun MyComponent() {
    // Access various component styles
    val buttonStyle = JewelTheme.defaultButtonStyle
    val textFieldStyle = JewelTheme.textFieldStyle
    val tabStyle = JewelTheme.defaultTabStyle
    val editorTabStyle = JewelTheme.editorTabStyle
    val linkStyle = JewelTheme.linkStyle
    val iconButtonStyle = JewelTheme.iconButtonStyle
    val transparentIconButtonStyle = JewelTheme.transparentIconButtonStyle
    val splitButtonStyle = JewelTheme.outlinedSplitButtonStyle
}
```

### Text Styles

```kotlin
@Composable
fun MyComponent() {
    val defaultTextStyle = JewelTheme.defaultTextStyle
    
    Text(
        text = "Styled text",
        style = defaultTextStyle
    )
}
```

## Custom Styling

### Link Style Customization

```kotlin
import org.jetbrains.jewel.ui.component.styling.LinkUnderlineBehavior

@Composable
fun AlwaysUnderlinedLink() {
    val alwaysUnderline = JewelTheme.linkStyle.copy(
        underlineBehavior = LinkUnderlineBehavior.ShowAlways
    )
    
    Link(
        text = "Always underlined",
        onClick = {},
        style = alwaysUnderline
    )
}
```

### Icon Button Without Background

```kotlin
@Composable
fun TransparentIconButton() {
    IconButton(
        style = JewelTheme.transparentIconButtonStyle,
        onClick = {}
    ) {
        Icon(key = AllIconsKeys.Actions.AddFile, contentDescription = "Add")
    }
}
```

## Component States

### State-Aware Icons

```kotlin
import org.jetbrains.jewel.ui.painter.hints.Stateful
import org.jetbrains.jewel.ui.painter.hints.Selected
import org.jetbrains.jewel.ui.painter.hints.Stroke

@Composable
fun StatefulIcon() {
    IconButton(onClick = {}) { state ->
        Icon(
            key = AllIconsKeys.Actions.Find,
            contentDescription = "Find",
            hint = Stateful(state)  // Icon adapts to button state
        )
    }
}

// Selected state hint
SelectableIconButton(selected = isSelected, onClick = {}) { state ->
    val tint by LocalIconButtonStyle.current.colors.selectableForegroundFor(state)
    Icon(
        key = AllIconsKeys.Actions.MatchCase,
        contentDescription = "Match case",
        hints = arrayOf(Selected(isSelected), Stroke(tint))
    )
}
```

### Disabled Appearance

```kotlin
import org.jetbrains.jewel.ui.disabledAppearance

@Composable
fun DisabledItem() {
    SimpleListItem(
        modifier = Modifier.disabledAppearance(),
        text = "Disabled item",
        selected = false,
        active = false
    )
}
```

## Outline States

```kotlin
import org.jetbrains.jewel.ui.Outline

// Normal (no outline)
TextField(state = state)

// Error outline
TextField(
    state = state,
    outline = Outline.Error
)

// Warning outline
TextField(
    state = state,
    outline = Outline.Warning
)

// Also works with checkboxes
TriStateCheckboxRow(
    text = "Error checkbox",
    state = checked,
    onClick = {},
    outline = Outline.Error
)
```

## Icon Badges

```kotlin
import org.jetbrains.jewel.ui.painter.badge.DotBadgeShape
import org.jetbrains.jewel.ui.painter.hints.Badge

Icon(
    key = AllIconsKeys.Nodes.ConfigFolder,
    contentDescription = "Config",
    hint = Badge(Color.Red, DotBadgeShape.Default)
)
```

## Tab Content Alpha

```kotlin
import org.jetbrains.jewel.ui.component.tabContentAlpha

@Composable
fun TabWithIcon() {
    TabData.Editor(
        selected = isSelected,
        content = { tabState ->
            Icon(
                key = AllIconsKeys.Actions.Find,
                contentDescription = null,
                modifier = Modifier
                    .size(16.dp)
                    .tabContentAlpha(state = tabState)  // Fades when inactive
            )
        },
        onClick = {}
    )
}
```

## Conditional Modifiers

```kotlin
import org.jetbrains.jewel.foundation.modifier.thenIf

Box(
    modifier = Modifier
        .size(12.dp)
        .thenIf(isHovered) {
            drawWithCache {
                onDrawBehind {
                    drawCircle(color = Color.Magenta.copy(alpha = 0.4f))
                }
            }
        }
)
```
