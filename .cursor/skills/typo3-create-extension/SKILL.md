---
name: typo3-create-extension
description: Create a new TYPO3 extension with standard structure, composer.json, ext_emconf.php, ext_localconf.php and plugin registration. Use when the user asks to create a new extension, add an extension, or scaffold a TYPO3 extension (Extbase/plugin).
---

# Create New TYPO3 Extension

Follow this workflow for a TYPO3 13 extension (Extbase plugin) in `packages/<ext_key>/`.

## 1. Folder structure

```bash
mkdir -p packages/my_extension/{Classes/{Controller,Domain/{Model,Repository},ViewHelpers},Configuration/{TCA,TypoScript},Resources/{Private/{Layouts,Partials,Templates,Js,Scss},Public/{Css,Js}}}
```

Use lowercase_with_underscores for the extension key (e.g. `my_extension`). Replace with the actual key.

## 2. composer.json

- **name**: `vendor/extension-name` (kebab-case)
- **type**: `typo3-cms-extension`
- **extra.typo3/cms.extension-key**: same as folder name (e.g. `my_extension`)
- **autoload.psr-4**: `Vendor\\ExtensionName\\` → `Classes/`
- **require**: `typo3/cms-core`, `typo3/cms-extbase`, `typo3/cms-fluid` (^13.4)

Example:

```json
{
  "name": "vendor/my-extension",
  "type": "typo3-cms-extension",
  "description": "Extension description",
  "require": {
    "typo3/cms-core": "^13.4",
    "typo3/cms-extbase": "^13.4",
    "typo3/cms-fluid": "^13.4"
  },
  "autoload": {
    "psr-4": {
      "Vendor\\MyExtension\\": "Classes/"
    }
  },
  "extra": {
    "typo3/cms": {
      "extension-key": "my_extension"
    }
  }
}
```

## 3. ext_emconf.php

Set `$_EXTKEY`, title, description, category, author, version, and `constraints.depends.typo3` (e.g. `13.4.0-13.4.99`).

## 4. ext_localconf.php – plugin registration

```php
\TYPO3\CMS\Extbase\Utility\ExtensionUtility::configurePlugin(
    'MyExtension',           // extension name (CamelCase)
    'Pi1',                  // plugin name
    [\Vendor\MyExtension\Controller\MainController::class => 'list, show'],
    [\Vendor\MyExtension\Controller\MainController::class => '']
);
```

## 5. Installation (path repo)

Root `composer.json` must have `"repositories": [{"type": "path", "url": "packages/*"}]`. Then:

```bash
ddev composer require vendor/my-extension:@dev
ddev typo3 extension:activate my_extension
ddev typo3 cache:flush
```

## 6. Vite (if this project uses Vite)

Adding `Resources/Private/Js/main.js` and/or `Resources/Private/Scss/main.scss` makes the extension auto-discovered by the project’s Vite config. Build with `ddev exec npm run build`.

## Conventions

- Extension key: lowercase_with_underscores
- Composer name: vendor/extension-name (kebab-case)
- Namespace: Vendor\ExtensionName\ (CamelCase, no underscores)
