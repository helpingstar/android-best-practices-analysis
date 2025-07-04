## fun CreationScreen

```kotlin
@Composable
fun CreationScreen(
    fileName: String? = null,
    creationViewModel: CreationViewModel = hiltViewModel(),
    isMedium: Boolean = isAtLeastMedium(),
    onCameraPressed: () -> Unit = {},
    onBackPressed: () -> Unit,
    onAboutPressed: () -> Unit,
)
````

## fun EditScreen

```kotlin
@Composable
fun EditScreen(
    snackbarHostState: SnackbarHostState,
    isExpanded: Boolean,
    onCameraPressed: () -> Unit,
    onBackPressed: () -> Unit,
    onAboutPressed: () -> Unit,
    uiState: CreationState,
    onChooseImageClicked: (PickVisualMedia.VisualMediaType) -> Unit,
    onPromptOptionSelected: (PromptType) -> Unit,
    onUndoPressed: () -> Unit,
    onPromptGenerationPressed: () -> Unit,
    onBotColorSelected: (BotColor) -> Unit,
    onStartClicked: () -> Unit,
)
```

## fun ColumnScope.BottomButtons

```kotlin
@Composable
private fun ColumnScope.BottomButtons(
    onButtonColorClicked: () -> Unit,
    uiState: CreationState,
    onStartClicked: () -> Unit,
    modifier: Modifier = Modifier,
)
```

## fun TransformButton

```kotlin
@Composable
private fun TransformButton(
    modifier: Modifier = Modifier,
    buttonText: String = stringResource(CreationR.string.transform_button),
    onClicked: () -> Unit = {},
)
```

## fun BotColorPickerBottomSheet

```kotlin
@Composable
private fun BotColorPickerBottomSheet(
    showColorPickerBottomSheet: Boolean,
    dismissBottomSheet: () -> Unit,
    onColorChanged: (BotColor) -> Unit,
    listBotColors: List<BotColor>,
    selectedBotColor: BotColor,
)
```

## fun MainCreationPane

```kotlin
@Composable
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
private fun MainCreationPane(
    uiState: CreationState,
    modifier: Modifier = Modifier,
    onCameraPressed: () -> Unit,
    onChooseImageClicked: () -> Unit = {},
    onUndoPressed: () -> Unit = {},
    onPromptGenerationPressed: () -> Unit,
    onSelectedPromptOptionChanged: (PromptType) -> Unit,
)
```

## fun ImagePreview

```kotlin
@Composable
fun ImagePreview(
    uri: Uri,
    onUndoPressed: () -> Unit,
    onChooseImagePressed: () -> Unit,
    modifier: Modifier = Modifier,
)
```

## fun TextPrompt

```kotlin
@Composable
fun TextPrompt(
    textFieldState: TextFieldState,
    promptGenerationInProgress: Boolean,
    modifier: Modifier = Modifier,
    generatedPrompt: String? = null,
    onPromptGenerationPressed: () -> Unit,
)
```

## fun HelpMeWriteButton

```kotlin
@Composable
private fun HelpMeWriteButton(
    promptGenerationInProgress: Boolean,
    onPromptGenerationPressed: () -> Unit,
)
```

## fun PromptTypeToolbar

```kotlin
@Composable
@OptIn(ExperimentalMaterial3ExpressiveApi::class)
fun PromptTypeToolbar(
    selectedOption: PromptType,
    modifier: Modifier = Modifier,
    onOptionSelected: (PromptType) -> Unit,
)
```

## fun UploadEmptyState

```kotlin
@Composable
private fun UploadEmptyState(
    onCameraPressed: () -> Unit,
    onChooseImagePress: () -> Unit,
    modifier: Modifier = Modifier,
)
```

## fun TakePhotoButton

```kotlin
@Composable
private fun TakePhotoButton(onCameraPressed: () -> Unit)
```

## fun TextPromptGenerationPreview

```kotlin
@Preview(showBackground = true)
@Composable
private fun TextPromptGenerationPreview()
```

## fun TextPromptGenerationInProgressPreview

```kotlin
@Preview(showBackground = true)
@Composable
private fun TextPromptGenerationInProgressPreview()
```

## fun UploadEmptyPreview

```kotlin
@LargeScreensPreview
@Preview
@Composable
private fun UploadEmptyPreview()
```

