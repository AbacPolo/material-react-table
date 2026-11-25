# Migrating from Material UI V6 to V9

This fork of Material React Table V3 supports **Material UI V9 only**. This guide explains what changed compared to the original library (built for Material UI V6 and MUI X Date Pickers V7) and what you need to update in your own app.

> Material UI core has no V8. It went from V7.3 straight to V9 to align its version number with MUI X. MUI X Date Pickers did have a V8, so the pickers went V7 → V8 → V9.

- [1. Update your dependencies](#1-update-your-dependencies)
- [2. Update your `mui*Props` table options](#2-update-your-muiprops-table-options)
- [3. Date, time and date-time filters](#3-date-time-and-date-time-filters)
- [4. Theming and dark mode](#4-theming-and-dark-mode)
- [5. Other visible behavior changes](#5-other-visible-behavior-changes)
- [6. What changed inside the library](#6-what-changed-inside-the-library)
- [7. Upgrade checklist](#7-upgrade-checklist)
- [Official MUI migration guides](#official-mui-migration-guides)

## 1. Update your dependencies

| Package               | Before (V6) | Now (V9)  |
| --------------------- | ----------- | --------- |
| `@mui/material`       | `>=6`       | `^9.0.0`  |
| `@mui/icons-material` | `>=6`       | `^9.0.0`  |
| `@mui/x-date-pickers` | `>=7.15`    | `^9.0.0`  |
| `@emotion/react`      | `>=11.13`   | unchanged |
| `@emotion/styled`     | `>=11.13`   | unchanged |
| `react`, `react-dom`  | `>=18.0`    | unchanged |

```bash
npm install @mui/material@^9 @mui/icons-material@^9 @mui/x-date-pickers@^9
```

`@mui/x-date-pickers` V8 and older do not support Material UI V9, so the pickers must be upgraded together with Material UI.

Other requirements that come from Material UI itself:

- **Browsers:** the default targets rose to Chrome 117, Firefox 121, Safari 17 and Edge 121.
- **React 18 users:** Material UI depends on `react-is@19`, so force-resolve `react-is` to your React version (for example `react-is@18.3.1` for `react@18.3.1`) with npm `overrides`, Yarn `resolutions` or pnpm `overrides`.

## 2. Update your `mui*Props` table options

The `mui*Props` table and column options are typed with Material UI's own prop types (`TextFieldProps`, `CheckboxProps`, `MenuProps`, …). Material UI V9 **removed** the deprecated `*Props`/`components*` props, so any of them you pass through Material React Table must move to `slotProps`. With TypeScript you will get compile errors for each one.

### Text fields

Applies to `muiEditTextFieldProps`, `muiFilterTextFieldProps`, `muiSearchTextFieldProps` and the matching column options.

| Removed prop          | Replacement                |
| --------------------- | -------------------------- |
| `InputProps`          | `slotProps.input`          |
| `inputProps`          | `slotProps.htmlInput`      |
| `SelectProps`         | `slotProps.select`         |
| `InputLabelProps`     | `slotProps.inputLabel`     |
| `FormHelperTextProps` | `slotProps.formHelperText` |

```diff
 muiEditTextFieldProps={{
-  InputProps: { sx: { fontWeight: 'bold' } },
-  inputProps: { maxLength: 20 },
-  SelectProps: { native: true },
+  slotProps: {
+    input: { sx: { fontWeight: 'bold' } },
+    htmlInput: { maxLength: 20 },
+    select: { native: true },
+  },
 }}
```

```diff
 muiSearchTextFieldProps={{
-  InputLabelProps: { shrink: true },
+  slotProps: { inputLabel: { shrink: true } },
   label: 'Search',
 }}
```

The library still adds its own defaults (`autoComplete: 'off'`, the search and clear adornments, `disableUnderline` in table edit mode, `MenuProps: { disableScrollLock: true }` for selects) and merges your `slotProps` on top of them, as it did before with the old props.

### Checkboxes and radios

Applies to `muiSelectCheckboxProps` and `muiSelectAllCheckboxProps`.

| Removed prop | Replacement           |
| ------------ | --------------------- |
| `inputProps` | `slotProps.input`     |
| `inputRef`   | `slotProps.input.ref` |

### Other components you might customize

If you pass any of these to Material UI components (for example inside `renderTopToolbarCustomActions`, `renderDetailPanel` or the various `render*MenuItems` callbacks), they were removed too:

| Component                     | Removed prop(s)                                                                        | Replacement                                                                     |
| ----------------------------- | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `Menu`                        | `MenuListProps`, `PaperProps`, `TransitionProps`                                       | `slotProps.list`, `slotProps.paper`, `slotProps.transition`                     |
| `Dialog`                      | `PaperProps`, `TransitionComponent`, `TransitionProps`, `BackdropProps`                | `slotProps.paper`, `slots.transition`, `slotProps.transition`, `slotProps.backdrop` |
| `Popover`                     | `PaperProps`, `TransitionComponent`, `TransitionProps`                                 | `slotProps.paper`, `slots.transition`, `slotProps.transition`                   |
| `Tooltip`                     | `PopperProps`, `TransitionComponent`, `TransitionProps`, `components`, `componentsProps` | `slotProps.popper`, `slots.transition`, `slotProps.transition`, `slots`, `slotProps` |
| `FormControlLabel`            | `componentsProps`                                                                      | `slotProps`                                                                     |
| `Autocomplete`                | `ChipProps`, `ListboxProps`, `PaperComponent`, `PopperComponent`, `renderTags`         | `slotProps.chip`, `slotProps.listbox`, `slots.paper`, `slots.popper`, `renderValue` |
| `Autocomplete` `renderInput`  | `params.inputProps`, `params.InputProps`                                               | `params.slotProps.htmlInput`, `params.slotProps.input`                          |
| `ListItemText`                | `primaryTypographyProps`, `secondaryTypographyProps`                                   | `slotProps.primary`, `slotProps.secondary`                                      |
| `Dialog`, `Modal`             | `disableEscapeKeyDown`                                                                 | check `reason === 'escapeKeyDown'` in `onClose`                                 |
| `Typography`                  | `paragraph`                                                                            | `sx={{ marginBottom: '16px' }}`                                                 |

Material UI provides codemods for most of these: `npx @mui/codemod@latest deprecations/all <path>`.

### System props

System props (`mt`, `gap`, `alignItems`, `fontStyle`, `color`, …) were removed from `Box`, `Stack`, `Typography`, `Link` and `Grid`. Move them into `sx`. `Stack`'s own `direction` and `spacing` props still work.

```diff
-<Stack alignItems="center" direction="row" gap="16px">
+<Stack direction="row" sx={{ alignItems: 'center', gap: '16px' }}>
```

Codemod: `npx @mui/codemod@latest v9.0.0/system-props <path>`.

### Unchanged

`muiPaginationProps.SelectProps` is a Material React Table option (it configures the rows-per-page `Select`), not a Material UI prop, so it keeps working as before.

## 3. Date, time and date-time filters

Columns with `filterVariant` `'date'`, `'date-range'`, `'datetime'`, `'datetime-range'`, `'time'` or `'time-range'` use MUI X Date Pickers, which changed significantly.

### Prop types are no longer generic

`muiFilterDatePickerProps`, `muiFilterDateTimePickerProps` and `muiFilterTimePickerProps` now use `DatePickerProps`, `DateTimePickerProps` and `TimePickerProps` without a generic argument. If you typed your own props, drop the generic:

```diff
-const props: DatePickerProps<Dayjs> = { ... };
+const props: DatePickerProps = { ... };
```

### The field is a `PickersTextField`, not a `TextField`

Pickers V9 always render the accessible "sections" field (one `spinbutton` per month/day/year part) with a hidden `<input>`. As a result:

- `slotProps.textField` only accepts `PickersTextFieldProps`. Material UI `TextField`-only props such as `select`, `multiline` or `placeholder` are not supported.
- The legacy `InputProps`, `inputProps`, `InputLabelProps` and `FormHelperTextProps` were removed from the picker text field. Use `slotProps.textField.slotProps.input`, `.htmlInput`, `.inputLabel` and `.formHelperText`.
- `slotProps.textField.slotProps.htmlInput` targets the **hidden** input. Use it for `name` or `data-*` attributes, not for styling or labels.
- `enableAccessibleFieldDOMStructure` was removed. The old single-`<input>` field no longer exists.
- If you supply a custom `slots.textField`, it has to wrap `PickersTextField`.
- The `unstableFieldRef` prop is now `slotProps.field.fieldRef`.

```diff
 muiFilterDatePickerProps={{
   slotProps: {
     textField: {
-      InputProps: { sx: { fontSize: 12 } },
+      slotProps: { input: { sx: { fontSize: 12 } } },
     },
   },
 }}
```

### What you will see

- The filter shows the date format (`MM/DD/YYYY`) instead of the "Filter by …" placeholder.
- The picker's own **Clear** and calendar buttons appear in the field, as before.
- `onChange` is no longer called with an invalid date while a date is only partly typed. It fires once the value is complete or empty.
- `closeOnSelect` now defaults to `false` for `DateTimePicker`, `TimePicker` and the mobile pickers. They show **Cancel**/**OK** buttons, and the filter applies after **OK**. `DesktopDatePicker` still closes on selection. To restore the old behavior:

  ```tsx
  muiFilterDateTimePickerProps={{ closeOnSelect: true }}
  ```

- When a filter mode that needs no value is active (for example "Empty"), the picker is disabled while the mode chip is shown.
- Focusing a date filter from the column actions menu ("Filter by …") still works.

## 4. Theming and dark mode

### `mrtTheme` colors follow the active color scheme

If your Material UI theme uses `colorSchemes` (with or without `cssVariables`), `theme.palette` stays fixed to the default scheme and `theme.palette.mode` never changes when the user switches between light and dark. In the original library this left the table with light colors in dark mode.

The table now reads the active scheme with `useColorScheme()` and computes `baseBackgroundColor`, `menuBackgroundColor`, `matchHighlightColor`, `pinnedRowBackgroundColor`, `selectedRowBackgroundColor` and the outline colors from that scheme's palette. They update when the mode changes.

If you pass `mrtTheme` as a function, it now receives a theme whose `palette` is the **active** scheme's palette:

```tsx
mrtTheme={(theme) => ({
  baseBackgroundColor: theme.palette.background.paper, // active scheme
})}
```

### `nativeColor` themes are supported

With `createTheme({ cssVariables: { nativeColor: true } })`, palette colors can be any CSS color, such as `oklch(...)`. The standalone `alpha`, `darken` and `lighten` helpers from `@mui/material/styles` throw `Unsupported color` errors for those, which crashed the original library on every render. The table now uses the theme's own `theme.alpha`, `theme.lighten` and `theme.darken`, which produce `color-mix()` or relative-color CSS for native colors and the same `rgb()`/`rgba()` values as before for regular colors.

If your own `mrtTheme` or `sx` callbacks compute colors, do the same:

```diff
-import { alpha, lighten } from '@mui/material/styles';
 mrtTheme={(theme) => ({
-  baseBackgroundColor: lighten(theme.palette.background.default, 0.05),
-  selectedRowBackgroundColor: alpha(theme.palette.primary.main, 0.2),
+  baseBackgroundColor: theme.lighten(theme.palette.background.default, 0.05),
+  selectedRowBackgroundColor: theme.alpha(theme.palette.primary.main, 0.2),
 })}
```

`theme.alpha` also accepts CSS variables (`theme.vars.palette.primary.main`). `theme.lighten` and `theme.darken` only accept concrete colors unless `nativeColor` is enabled.

### Dark-mode styles use `theme.applyStyles`

Internal styles that depended on `theme.palette.mode === 'dark'` (pinned cells, row hover, sticky footer outline, sort icon color, search-match highlight text) now use `theme.applyStyles('dark', …)`, Material UI's recommended approach. Without `colorSchemes`, it behaves exactly like the old `palette.mode` check. With `colorSchemes`, it follows your `colorSchemeSelector` (media query, class or data attribute).

If you use `colorSchemeSelector: 'class'`, make sure no other element between `<html>` and the table carries a `light` or `dark` class, or it will override the CSS variables for everything inside it.

## 5. Other visible behavior changes

These come from Material UI V9 itself:

| Area                   | Change                                                                                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Menu items             | `ListItemIcon` default `min-width` dropped from 56px to 36px, so icons in the row, column and cell action menus sit closer to their labels.               |
| Select fields          | The label of `<TextField select />` (select filters, `editVariant: 'select'`) renders as a `<div>` instead of a `<label>`.                               |
| Menus                  | `MenuItem` throws an error when rendered outside a `Menu`/`MenuList`. Children with `role="menuitem"` that don't wrap a `MenuItem` are no longer supported in `render*MenuItems`. |
| Buttons                | Pressing Enter or Space on a button now dispatches a click that bubbles to parent elements. Check row `onClick` handlers if you put custom buttons in cells. |
| Slider                 | Uses pointer events. Use `onPointerDown` instead of `onMouseDown` in `muiFilterSliderProps`.                                                            |
| Icons                  | 23 legacy `*Outline` icons (for example `DeleteOutline`, `InfoOutline`) were removed from `@mui/icons-material`. Use the `*Outlined` variants if you pass them through the `icons` option. |

## 6. What changed inside the library

For reference, these are the internal changes made in this fork. None of them require action from you beyond the sections above.

| Area                    | Change                                                                                                                                                         | Files                                                                                             |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Text fields             | `InputProps`, `inputProps`, `SelectProps` → `slotProps.input`, `slotProps.htmlInput`, `slotProps.select`                                                       | `MRT_EditCellTextField`, `MRT_GlobalFilterTextField`, `MRT_FilterTextField`                       |
| Autocomplete filter     | `renderInput` reads `params.slotProps` instead of `params.inputProps`/`InputProps`                                                                             | `MRT_FilterTextField`                                                                             |
| Menus                   | `MenuListProps` → `slotProps.list`                                                                                                                             | `MRT_CellActionMenu`, `MRT_ColumnActionMenu`, `MRT_FilterOptionMenu`, `MRT_RowActionMenu`, `MRT_ShowHideColumnsMenu` |
| Show/hide columns       | `FormControlLabel` `componentsProps` → `slotProps`                                                                                                             | `MRT_ShowHideColumnsMenuItems`                                                                    |
| Row selection           | Checkbox/Radio `inputProps` → `slotProps.input`                                                                                                                | `MRT_SelectCheckbox`                                                                              |
| System props            | Moved into `sx`. The previous `<Box gap="xs">` was not valid CSS and was removed.                                                                              | `MRT_TablePagination`, `MRT_ToolbarAlertBanner`, `MRT_ToolbarDropZone`, `getMRT_RowExpandColumnDef` |
| Date pickers            | Generic-free prop types; picker-specific text field props without `placeholder` and hidden-input attributes; `disabled` while a filter chip is shown           | `types.ts`, `MRT_FilterTextField`                                                                 |
| Theme colors            | `getActiveColorSchemeTheme` + `useColorScheme()`; `theme.alpha`/`lighten`/`darken`; `theme.applyStyles('dark', …)`; `(theme.vars ?? theme).palette` for scheme-dependent tokens | `style.utils.ts`, `useMRT_TableOptions`, `MRT_TableBodyRow`, `MRT_TableBodyCellValue`, `MRT_TableFooter`, `MRT_TableHeadCellSortLabel`, `MRT_TableHeadCellResizeHandle`, `MRT_TableHeadRow`, `MRT_TableLoadingOverlay`, `MRT_BottomToolbar`, `MRT_ToolbarDropZone` |
| Peer dependencies       | `@mui/material`, `@mui/icons-material`, `@mui/x-date-pickers` → `^9.0.0`                                                                                       | `packages/material-react-table/package.json`                                                      |

## 7. Upgrade checklist

1. Upgrade `@mui/material`, `@mui/icons-material` and `@mui/x-date-pickers` to V9 together.
2. On React 18, add a `react-is` resolution matching your React version.
3. Run `npx @mui/codemod@latest deprecations/all <path>` and `npx @mui/codemod@latest v9.0.0/system-props <path>` on your app.
4. Run `npx @mui/x-codemod@latest v9.0.0/pickers/preset-safe <path>` if you use pickers outside the table.
5. Replace `InputProps`/`inputProps`/`SelectProps`/`InputLabelProps` in your `mui*TextFieldProps` with `slotProps`.
6. Replace `inputProps` in `muiSelectCheckboxProps`/`muiSelectAllCheckboxProps` with `slotProps.input`.
7. Drop generics from `DatePickerProps`/`DateTimePickerProps`/`TimePickerProps` and move picker text field props under `slotProps.textField.slotProps`.
8. Replace imported `alpha`/`darken`/`lighten` with `theme.alpha`/`theme.darken`/`theme.lighten` in your `mrtTheme` and `sx` callbacks.
9. Decide whether you want `closeOnSelect: true` on DateTime/Time filters.
10. Run your type-checker. Every remaining removed prop is reported as a TypeScript error.

## Official MUI migration guides

- [Material UI: Upgrade to V7](https://mui.com/material-ui/migration/upgrade-to-v7/)
- [Material UI: Upgrade to V9](https://mui.com/material-ui/migration/upgrade-to-v9/)
- [Material UI: Migrating from deprecated APIs](https://mui.com/material-ui/migration/migrating-from-deprecated-apis/)
- [MUI X Date Pickers: Migration from V7 to V8](https://mui.com/x/migration/migration-pickers-v7/)
- [MUI X Date Pickers: Migration from V8 to V9](https://mui.com/x/migration/migration-pickers-v8/)
