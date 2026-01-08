# Jewel Examples

## Complete Application Example

```kotlin
import androidx.compose.foundation.layout.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application
import org.jetbrains.jewel.foundation.theme.JewelTheme
import org.jetbrains.jewel.intui.standalone.theme.IntUiTheme
import org.jetbrains.jewel.ui.component.*

fun main() = application {
    var isDark by remember { mutableStateOf(false) }
    
    Window(
        onCloseRequest = ::exitApplication,
        title = "Jewel Demo"
    ) {
        IntUiTheme(isDark = isDark) {
            Column(
                modifier = Modifier.fillMaxSize().padding(16.dp),
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                // Theme toggle
                Row(
                    horizontalArrangement = Arrangement.spacedBy(8.dp),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    Text("Theme:")
                    DropdownLink(if (isDark) "Dark" else "Light") {
                        selectableItem(selected = !isDark, onClick = { isDark = false }) {
                            Text("Light")
                        }
                        selectableItem(selected = isDark, onClick = { isDark = true }) {
                            Text("Dark")
                        }
                    }
                }
                
                // Main content
                MainContent()
            }
        }
    }
}

@Composable
fun MainContent() {
    var text by remember { mutableStateOf("") }
    var selectedTab by remember { mutableIntStateOf(0) }
    
    Column(verticalArrangement = Arrangement.spacedBy(16.dp)) {
        // Text input section
        GroupHeader("Text Input")
        TextField(
            state = rememberTextFieldState(text),
            placeholder = { Text("Enter something...") },
            modifier = Modifier.width(300.dp)
        )
        
        // Buttons section
        GroupHeader("Actions")
        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            OutlinedButton(onClick = {}) { Text("Cancel") }
            DefaultButton(onClick = {}) { Text("Submit") }
        }
        
        // Tabs section
        GroupHeader("Tabs")
        val tabs = listOf("Overview", "Details", "Settings").mapIndexed { index, title ->
            TabData.Default(
                selected = index == selectedTab,
                content = { state -> SimpleTabContent(label = title, state = state) },
                onClick = { selectedTab = index }
            )
        }
        TabStrip(tabs = tabs, style = JewelTheme.defaultTabStyle)
    }
}
```

## Form Example

```kotlin
@Composable
fun SettingsForm() {
    var username by remember { mutableStateOf("") }
    var email by remember { mutableStateOf("") }
    var notifications by remember { mutableStateOf(ToggleableState.On) }
    var theme by remember { mutableIntStateOf(0) }
    
    Column(
        modifier = Modifier.padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        GroupHeader("User Settings")
        
        // Username field
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text("Username:", modifier = Modifier.width(100.dp))
            TextField(
                state = rememberTextFieldState(username),
                placeholder = { Text("Enter username") },
                modifier = Modifier.width(200.dp)
            )
        }
        
        // Email field with validation
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text("Email:", modifier = Modifier.width(100.dp))
            TextField(
                state = rememberTextFieldState(email),
                placeholder = { Text("Enter email") },
                outline = if (email.isNotEmpty() && !email.contains("@")) 
                    Outline.Error else Outline.None,
                modifier = Modifier.width(200.dp)
            )
        }
        
        // Theme selection
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Text("Theme:", modifier = Modifier.width(100.dp))
            ListComboBox(
                items = listOf("Light", "Dark", "System"),
                selectedIndex = theme,
                onSelectedItemChange = { theme = it },
                modifier = Modifier.width(200.dp),
                itemKeys = { _, item -> item }
            )
        }
        
        // Notifications checkbox
        TriStateCheckboxRow(
            text = "Enable notifications",
            state = notifications,
            onClick = {
                notifications = if (notifications == ToggleableState.On) 
                    ToggleableState.Off else ToggleableState.On
            }
        )
        
        // Action buttons
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.End
        ) {
            OutlinedButton(onClick = {}) { Text("Cancel") }
            Spacer(Modifier.width(8.dp))
            DefaultButton(onClick = {}) { Text("Save") }
        }
    }
}
```

## File Browser Example

```kotlin
@Composable
fun FileBrowser() {
    var selectedTab by remember { mutableIntStateOf(0) }
    var searchText by remember { mutableStateOf("") }
    
    Column(modifier = Modifier.fillMaxSize()) {
        // Toolbar
        Row(
            modifier = Modifier.fillMaxWidth().padding(8.dp),
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            IconButton(onClick = {}) {
                Icon(AllIconsKeys.Actions.Back, "Back")
            }
            IconButton(onClick = {}) {
                Icon(AllIconsKeys.Actions.Forward, "Forward")
            }
            IconButton(onClick = {}) {
                Icon(AllIconsKeys.Actions.Refresh, "Refresh")
            }
            
            Spacer(Modifier.weight(1f))
            
            TextField(
                state = rememberTextFieldState(searchText),
                placeholder = { Text("Search...") },
                leadingIcon = {
                    Icon(
                        AllIconsKeys.Actions.Find,
                        "Search",
                        modifier = Modifier.size(16.dp)
                    )
                },
                modifier = Modifier.width(200.dp)
            )
        }
        
        // Editor tabs
        val files = listOf("Main.kt", "App.kt", "Utils.kt")
        val tabs = files.mapIndexed { index, name ->
            TabData.Editor(
                selected = index == selectedTab,
                content = { state ->
                    SimpleTabContent(
                        state = state,
                        icon = {
                            Icon(
                                AllIconsKeys.FileTypes.Kotlin,
                                null,
                                modifier = Modifier.size(16.dp)
                            )
                        },
                        label = { Text(name) }
                    )
                },
                onClose = { /* handle close */ },
                onClick = { selectedTab = index }
            )
        }
        
        TabStrip(tabs = tabs, style = JewelTheme.editorTabStyle)
        
        // Content area
        Box(
            modifier = Modifier.fillMaxSize().padding(16.dp),
            contentAlignment = Alignment.Center
        ) {
            Text("Content for ${files[selectedTab]}")
        }
    }
}
```

## Progress Dialog Example

```kotlin
@Composable
fun ProgressDialog(
    title: String,
    progress: Float,
    onCancel: () -> Unit
) {
    Column(
        modifier = Modifier.width(400.dp).padding(16.dp),
        verticalArrangement = Arrangement.spacedBy(16.dp)
    ) {
        Text(title, style = JewelTheme.defaultTextStyle)
        
        Row(
            horizontalArrangement = Arrangement.spacedBy(8.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            if (progress < 0) {
                // Indeterminate
                IndeterminateHorizontalProgressBar(
                    modifier = Modifier.weight(1f)
                )
            } else {
                // Determinate
                HorizontalProgressBar(
                    progress = progress,
                    modifier = Modifier.weight(1f)
                )
                Text("${(progress * 100).toInt()}%")
            }
        }
        
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.End
        ) {
            OutlinedButton(onClick = onCancel) {
                Text("Cancel")
            }
        }
    }
}
```

## Split Layout Example

```kotlin
@Composable
fun SplitPaneDemo() {
    var splitRatio by remember { mutableStateOf(0.3f) }
    
    Row(modifier = Modifier.fillMaxSize()) {
        // Left panel
        Column(
            modifier = Modifier
                .fillMaxHeight()
                .weight(splitRatio)
                .padding(8.dp)
        ) {
            GroupHeader("Project")
            // Tree view or file list
            Text("File 1.kt")
            Text("File 2.kt")
            Text("File 3.kt")
        }
        
        // Divider (simplified - use actual SplitLayout component)
        Box(
            modifier = Modifier
                .fillMaxHeight()
                .width(4.dp)
                .background(JewelTheme.globalColors.borders.normal)
        )
        
        // Right panel
        Column(
            modifier = Modifier
                .fillMaxHeight()
                .weight(1f - splitRatio)
                .padding(8.dp)
        ) {
            GroupHeader("Editor")
            Text("Editor content here...")
        }
    }
}
```

## Menu Example

```kotlin
@Composable
fun MenuDemo() {
    var selected by remember { mutableStateOf("Option 1") }
    
    OutlinedSplitButton(
        onClick = { /* main action */ },
        content = { Text("File") },
        menuContent = {
            selectableItem(
                selected = false,
                onClick = { /* new file */ }
            ) {
                Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
                    Icon(AllIconsKeys.Actions.AddFile, null, Modifier.size(16.dp))
                    Text("New File")
                }
            }
            
            selectableItem(
                selected = false,
                onClick = { /* open */ }
            ) {
                Text("Open...")
            }
            
            separator()
            
            submenu(
                submenu = {
                    selectableItem(selected = false, onClick = {}) {
                        Text("Export as PDF")
                    }
                    selectableItem(selected = false, onClick = {}) {
                        Text("Export as HTML")
                    }
                },
                content = { Text("Export") }
            )
            
            separator()
            
            selectableItem(
                selected = false,
                onClick = { /* exit */ }
            ) {
                Text("Exit")
            }
        }
    )
}
```

## Loading State Example

```kotlin
@Composable
fun LoadingContent(isLoading: Boolean, content: @Composable () -> Unit) {
    Box(modifier = Modifier.fillMaxSize()) {
        if (isLoading) {
            Column(
                modifier = Modifier.align(Alignment.Center),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                CircularProgressIndicatorBig()
                Text("Loading...", color = JewelTheme.globalColors.text.info)
            }
        } else {
            content()
        }
    }
}
```
