# Jewel Components Reference

## Buttons

### OutlinedButton & DefaultButton

```kotlin
import org.jetbrains.jewel.ui.component.OutlinedButton
import org.jetbrains.jewel.ui.component.DefaultButton

// Outlined button (secondary action)
OutlinedButton(onClick = { /* action */ }) {
    Text("Outlined")
}

// Default button (primary action)
DefaultButton(onClick = { /* action */ }) {
    Text("Default")
}

// Disabled state
OutlinedButton(onClick = {}, enabled = false) {
    Text("Disabled")
}
```

### IconButton

```kotlin
import org.jetbrains.jewel.ui.component.IconButton
import org.jetbrains.jewel.ui.component.Icon
import org.jetbrains.jewel.ui.icons.AllIconsKeys

// Basic icon button
IconButton(onClick = { /* action */ }) {
    Icon(key = AllIconsKeys.Actions.AddFile, contentDescription = "Add file")
}

// Non-focusable icon button
IconButton(onClick = {}, focusable = false) {
    Icon(key = AllIconsKeys.Actions.AddFile, contentDescription = "Add file")
}

// Selectable icon button
SelectableIconButton(onClick = { /* action */ }, selected = isSelected) { state ->
    Icon(
        key = AllIconsKeys.Actions.MatchCase,
        contentDescription = "Match case",
        hints = arrayOf(Selected(isSelected))
    )
}

// Toggleable icon button
var checked by remember { mutableStateOf(false) }
ToggleableIconButton(onValueChange = { checked = it }, value = checked) { state ->
    Icon(key = AllIconsKeys.Actions.MatchCase, contentDescription = "Toggle")
}
```

### SplitButton

```kotlin
import org.jetbrains.jewel.ui.component.OutlinedSplitButton
import org.jetbrains.jewel.ui.component.DefaultSplitButton

OutlinedSplitButton(
    onClick = { /* main action */ },
    secondaryOnClick = { /* dropdown clicked */ },
    content = { Text("Split button") },
    menuContent = {
        selectableItem(selected = false, onClick = { /* item clicked */ }) {
            Text("Menu Item 1")
        }
        separator()
        selectableItem(selected = false, onClick = { /* item clicked */ }) {
            Text("Menu Item 2")
        }
    }
)
```

### ActionButton

```kotlin
import org.jetbrains.jewel.ui.component.ActionButton

// With tooltip
ActionButton(onClick = {}, tooltip = { Text("Tooltip text") }) {
    Text("Hover me!")
}

// Without tooltip
ActionButton(onClick = {}) {
    Text("Do something")
}
```

## Text Input

### TextField

```kotlin
import androidx.compose.foundation.text.input.rememberTextFieldState
import org.jetbrains.jewel.ui.component.TextField
import org.jetbrains.jewel.ui.Outline

val state = rememberTextFieldState("Initial value")

// Basic text field
TextField(
    state = state,
    modifier = Modifier.width(200.dp)
)

// With placeholder
TextField(
    state = rememberTextFieldState(""),
    placeholder = { Text("Enter text...") },
    modifier = Modifier.width(200.dp)
)

// With error outline
TextField(
    state = state,
    outline = Outline.Error,
    placeholder = { Text("Error state") },
    modifier = Modifier.width(200.dp)
)

// With warning outline
TextField(
    state = state,
    outline = Outline.Warning,
    placeholder = { Text("Warning state") },
    modifier = Modifier.width(200.dp)
)

// Disabled
TextField(
    state = state,
    enabled = false,
    modifier = Modifier.width(200.dp)
)

// Read-only
TextField(
    state = state,
    readOnly = true,
    modifier = Modifier.width(200.dp)
)

// With leading icon
TextField(
    state = state,
    leadingIcon = {
        Icon(
            key = AllIconsKeys.Actions.Find,
            contentDescription = "Search",
            modifier = Modifier.size(16.dp)
        )
    },
    modifier = Modifier.width(200.dp)
)

// With trailing button (clear button)
TextField(
    state = state,
    trailingIcon = {
        if (state.text.isNotEmpty()) {
            IconButton(onClick = { state.setTextAndPlaceCursorAtEnd("") }) {
                Icon(AllIconsKeys.General.Close, contentDescription = "Clear")
            }
        }
    },
    modifier = Modifier.width(200.dp)
)

// Undecorated (no border)
TextField(
    state = state,
    undecorated = true,
    modifier = Modifier.width(200.dp)
)
```

## Checkboxes

```kotlin
import org.jetbrains.jewel.ui.component.TriStateCheckboxRow
import androidx.compose.ui.state.ToggleableState

var checked by remember { mutableStateOf(ToggleableState.Off) }

TriStateCheckboxRow(
    text = "Checkbox label",
    state = checked,
    onClick = {
        checked = when (checked) {
            ToggleableState.On -> ToggleableState.Off
            ToggleableState.Off -> ToggleableState.Indeterminate
            ToggleableState.Indeterminate -> ToggleableState.On
        }
    }
)

// With error outline
TriStateCheckboxRow(
    text = "Error",
    state = checked,
    onClick = { /* toggle */ },
    outline = Outline.Error
)

// Disabled
TriStateCheckboxRow(
    text = "Disabled",
    state = checked,
    onClick = {},
    enabled = false
)
```

## ComboBox / Dropdown

### ListComboBox

```kotlin
import org.jetbrains.jewel.ui.component.ListComboBox

val items = listOf("Option 1", "Option 2", "Option 3")
var selectedIndex by remember { mutableIntStateOf(0) }

ListComboBox(
    items = items,
    selectedIndex = selectedIndex,
    onSelectedItemChange = { index -> selectedIndex = index },
    modifier = Modifier.width(200.dp),
    itemKeys = { _, item -> item }
)
```

### Custom Item Content

```kotlin
data class Language(val name: String, val icon: IconKey)

val languages = listOf(
    Language("Kotlin", AllIconsKeys.FileTypes.Kotlin),
    Language("Java", AllIconsKeys.FileTypes.Java)
)

ListComboBox(
    items = languages,
    selectedIndex = selectedIndex,
    onSelectedItemChange = { index -> selectedIndex = index },
    itemKeys = { index, _ -> index },
    itemContent = { item, isSelected, isActive ->
        SimpleListItem(
            text = item.name,
            selected = isSelected,
            active = isActive,
            icon = item.icon
        )
    }
)
```

### EditableListComboBox

```kotlin
import org.jetbrains.jewel.ui.component.EditableListComboBox

EditableListComboBox(
    items = items,
    selectedIndex = selectedIndex,
    onSelectedItemChange = { index -> selectedIndex = index },
    modifier = Modifier.width(200.dp)
)
```

## Tabs

### Default Tabs

```kotlin
import org.jetbrains.jewel.ui.component.TabStrip
import org.jetbrains.jewel.ui.component.TabData
import org.jetbrains.jewel.ui.component.SimpleTabContent

var selectedTabIndex by remember { mutableIntStateOf(0) }

val tabs = listOf("Tab 1", "Tab 2", "Tab 3").mapIndexed { index, title ->
    TabData.Default(
        selected = index == selectedTabIndex,
        content = { tabState ->
            SimpleTabContent(label = title, state = tabState)
        },
        onClose = { /* handle close */ },
        onClick = { selectedTabIndex = index }
    )
}

TabStrip(tabs = tabs, style = JewelTheme.defaultTabStyle)
```

### Editor Tabs

```kotlin
val editorTabs = files.mapIndexed { index, file ->
    TabData.Editor(
        selected = index == selectedTabIndex,
        content = { tabState ->
            SimpleTabContent(
                state = tabState,
                icon = { Icon(key = AllIconsKeys.FileTypes.Text, contentDescription = null) },
                label = { Text(file.name) }
            )
        },
        onClose = { /* handle close */ },
        onClick = { selectedTabIndex = index }
    )
}

TabStrip(tabs = editorTabs, style = JewelTheme.editorTabStyle)
```

## Links

```kotlin
import org.jetbrains.jewel.ui.component.Link
import org.jetbrains.jewel.ui.component.ExternalLink
import org.jetbrains.jewel.ui.component.DropdownLink

// Basic link
Link(text = "Click me", onClick = { /* action */ })

// External link (opens in browser)
ExternalLink(text = "GitHub", uri = "https://github.com")

// Dropdown link
DropdownLink("Select theme") {
    selectableItem(selected = isDark, onClick = { isDark = true }) {
        Text("Dark")
    }
    selectableItem(selected = !isDark, onClick = { isDark = false }) {
        Text("Light")
    }
}

// Disabled
Link(text = "Disabled", onClick = {}, enabled = false)
```

## Progress Indicators

```kotlin
import org.jetbrains.jewel.ui.component.HorizontalProgressBar
import org.jetbrains.jewel.ui.component.IndeterminateHorizontalProgressBar
import org.jetbrains.jewel.ui.component.CircularProgressIndicator
import org.jetbrains.jewel.ui.component.CircularProgressIndicatorBig

// Determinate progress (0.0 to 1.0)
HorizontalProgressBar(
    progress = 0.5f,
    modifier = Modifier.width(200.dp)
)

// Indeterminate progress
IndeterminateHorizontalProgressBar(
    modifier = Modifier.width(200.dp)
)

// Circular spinner (16x16)
CircularProgressIndicator()

// Large circular spinner (32x32)
CircularProgressIndicatorBig()
```

## Tooltips

```kotlin
import org.jetbrains.jewel.ui.component.Tooltip

Tooltip(tooltip = { Text("This is a tooltip") }) {
    Text("Hover over me")
}
```

## Icons

```kotlin
import org.jetbrains.jewel.ui.component.Icon
import org.jetbrains.jewel.ui.icons.AllIconsKeys

// Basic icon
Icon(
    key = AllIconsKeys.Actions.Find,
    contentDescription = "Search"
)

// With size
Icon(
    key = AllIconsKeys.Actions.Find,
    contentDescription = "Search",
    modifier = Modifier.size(16.dp)
)

// With tint
Icon(
    key = AllIconsKeys.Actions.Find,
    contentDescription = "Search",
    tint = Color.Red
)
```

## Layout Components

### GroupHeader

```kotlin
import org.jetbrains.jewel.ui.component.GroupHeader

GroupHeader("Section Title")
```

### VerticallyScrollableContainer

```kotlin
import org.jetbrains.jewel.ui.component.VerticallyScrollableContainer

VerticallyScrollableContainer(Modifier.fillMaxSize()) {
    Column {
        // Scrollable content
    }
}
```
