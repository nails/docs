# Helper

`Nails\Admin\Helper` is the usual way to load admin views and emit shared chrome.

| Method | Purpose |
| ------ | ------- |
| `loadView($sView, $bLoadStructure = true, $bReturnView = false)` | Load a controller view, with the admin header/footer unless `$bLoadStructure` is false. The view directory is inferred from the controller name. |
| `loadInlineView($sView, $aData = [], $bReturnView = false)` | Load a view without the structure. |
| `tabs($aTabs, $sGroup = '')` | [Tab](tabs.md) markup. |
| `floatingControls($aConfig = [])` | Sticky [save bar](../forms.md#floating-save-bar). |
| `addModal($sTitle, $sBody, $bIsOpen = true)` | Queue a [modal](../javascript/modal.md) to render in the footer. |
| `loadCsv($mData, $sFilename = '', $bHeaderRow = true)` | Stream a CSV download. |
