# explaining how the font system works using "Source Sans Pro" as an example

1. **Font Definition in Frontend**:

```rust
// frontend/src/state-providers/fonts.ts




const LOCAL_FONTS = [
  {
    family: "Source Sans Pro",
    variants: ["Regular", "Bold", "Italic"],
    files: {
      Regular: "path/to/SourceSansPro-Regular.ttf",
      Bold: "path/to/SourceSansPro-Bold.ttf",
      Italic: "path/to/SourceSansPro-Italic.ttf"
    }
  }
  // ...
];
```

2. **Creating Text with Source Sans Pro**:
When a user creates text with Source Sans Pro, the text tool initializes with these settings:

```rust
// editor/src/messages/tool/tool_messages/text_tool.rs




impl Default for TextOptions {
    fn default() -> Self {
        Self {
            font_size: 24.0,
            line_height_ratio: 1.2,
            character_spacing: 1.0,
            font_name: "Source Sans Pro".to_string(),  // 
            font_style: "Regular".to_string(),
            fill: ToolColorOptions::new_primary(),
        }
    }
}
```

3. **Font Loading Process**:
When the text tool needs Source Sans Pro, it triggers the font loading process:

```typescript
// frontend/src/state-providers/fonts.ts




editor.subscriptions.subscribeJsMessage(TriggerFontLoad, async (triggerFontLoad) => {

    const url = await getFontFileUrl("Source Sans Pro", "Regular");
    if (url) {
        try {
            const response = await fetch(url);
            if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
            const arrayBuffer = await response.arrayBuffer();
            // Send font data to backend
            editor.handle.onFontLoad(
                "Source Sans Pro",
                "Regular",
                url,
                new Uint8Array(arrayBuffer)
            );
        } catch (error: unknown) {
          
        }
    }
});
```

4. **Font Cache in Backend**:
backend stores the font data in its cache:

```rust
// node-graph/gcore/src/text/font_cache.rs




impl FontCache {
    pub fn insert(&mut self, font: Font, preview_url: String, data: Vec<u8>) {

        // font would be Font { font_family: "Source Sans Pro", font_style: "Regular" }
        if !data.is_empty() {
            self.font_file_data.insert(font.clone(), data);
            self.preview_urls.insert(font, preview_url);
        }
    }

    pub fn get<'a>(&'a self, font: &Font) -> Option<&'a Vec<u8>> {

        self.resolve_font(font).and_then(|font| {
            let data = self.font_file_data.get(font)?;
            if data.is_empty() { None } else { Some(data) }
        })
    }
}
```

5. **Text Rendering with Source Sans Pro**:
rendering text with Source Sans Pro:

```rust
// node-graph/gstd/src/text.rs




fn text<'i: 'n>(
    _: impl Ctx,
    editor: &'i WasmEditorApi,
    text: String,
    font_name: Font,  // Font { font_family: "Source Sans Pro", font_style: "Regular" }
    #[default(24.)] font_size: f64,
    #[default(1.2)] line_height_ratio: f64,
    #[default(1.)] character_spacing: f64,
    #[default(None)] max_width: Option<f64>,
    #[default(None)] max_height: Option<f64>,
) -> VectorDataTable {
    // Get Source Sans Pro font data from cache
    let buzz_face = editor.font_cache.get(&font_name).map(|data| load_face(data));

    let typesetting = TypesettingConfig {
        font_size,
        line_height_ratio,
        character_spacing,
        max_width,
        max_height,
    };

    // Convert text to vector paths using Source Sans Pro
    let result = VectorData::from_subpaths(to_path(&text, buzz_face, typesetting), false);
    VectorDataTable::new(result)
}
```

6. **Frontend Text Display**:
displaying the editable text box with Source Sans Pro:

```scss
// frontend/src/components/panels/Document.svelte




async function displayEditableTextbox(displayEditableTextbox: DisplayEditableTextbox) {
    showTextInput = true;
    await tick();

    if (!textInput) return;

    // Set up text input with Source Sans Pro
    textInput.style.fontSize = `${displayEditableTextbox.fontSize}px`;
    textInput.style.lineHeight = `${displayEditableTextbox.lineHeightRatio}`;
    textInput.style.color = displayEditableTextbox.color.toHexOptionalAlpha() || "transparent";

    // Load Source Sans Pro for the text input
    const newFont = new FontFace("text-font", `url(${displayEditableTextbox.url})`);
    window.document.fonts.add(newFont);
    textInput.style.fontFamily = "text-font";  // Using loaded Source Sans Pro

    // Set up text input for editing
    textInput.contentEditable = "true";
    textInput.style.transformOrigin = "0 0";
    textInput.style.width = displayEditableTextbox.maxWidth ? 
        `${displayEditableTextbox.maxWidth}px` : "max-content";
}
```

7. **Text Bounding Box Calculation**:
calculating text boundaries for Source Sans Pro:

```rust
// node-graph/gcore/src/text/to_path.rs




pub fn bounding_box(str: &str, buzz_face: Option<&rustybuzz::Face>, typesetting: TypesettingConfig, for_clipping_test: bool) -> DVec2 {
    // Get the loaded Source Sans Pro face
    let Some(buzz_face) = buzz_face else { return DVec2::ZERO };
    let space_glyph = buzz_face.glyph_index(' ');

    // Calculate scale and line height for Source Sans Pro
    let (scale, line_height, mut buffer) = font_properties(
        buzz_face, 
        typesetting.font_size, 
        typesetting.line_height_ratio
    );

    // Calculate bounds for the text using Source Sans Pro metrics
    let [mut text_cursor, mut bounds] = [DVec2::ZERO; 2];
    
    // ...
    
    bounds
}
```