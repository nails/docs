# Helper

`Nails\Admin\Helper` is the usual way to load admin views and emit shared chrome. All of its methods are static.

Views are resolved relative to the current controller. See [Controllers → Views](../controllers/#views).

| Method | Purpose |
| ------ | ------- |
| `loadView($sView, $bLoadStructure = true, $bReturnView = false)` | Load a controller view, with the admin header/footer unless `$bLoadStructure` is false. The view directory is inferred from the controller name. |
| `loadInlineView($sView, $aData = [], $bReturnView = false)` | Load a view without the structure. |
| `tabs($aTabs, $sGroup = '')` | [Tab](tabs.md) markup. |
| `floatingControls($aConfig = [])` | Sticky [save bar](../forms.md#floating-save-bar). Pass `unsaved_changes` => true to opt into [Unsaved Changes](../javascript/unsaved-changes.md). |
| `addModal($sTitle, $sBody, $bIsOpen = true)` | Queue a [modal](../javascript/modal.md) to render in the footer. |
| `loadCsv($mData, $sFilename = '', $bHeaderRow = true)` | Stream a CSV download. |
| `addHeaderButton($sUrl, $sLabel, $sContext = null, $sConfirmTitle = null, $sConfirmBody = null)` | Add a button to the page header. |
| `dynamicTable($sKey, $aFields, $aData = [])` | Render a [dynamic table](../javascript/dynamic-table.md). |
| `loadUserCell($mUser)`, `loadDateCell($sDate)`, `loadDateTimeCell($sDateTime)`, `loadBoolCell($mValue)` | Render a consistent `<td>` for common value types in index tables. |
| `loadSearch($oSearch)` / `loadPagination($oPagination)` | Render the search bar and pagination used on index screens. |
