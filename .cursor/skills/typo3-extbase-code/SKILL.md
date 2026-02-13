---
name: typo3-extbase-code
description: Write TYPO3 Extbase code following TYPO3 extension standards: namespaces, extension keys, directory layout, Controller, Model, Repository patterns. Use when writing or generating PHP code for TYPO3 extensions, Controllers, Models, Repositories, or when reviewing TYPO3 extension code.
---

# TYPO3 Extension Standards & Extbase Code

Apply when writing or generating PHP for TYPO3 13 extensions (Extbase/Fluid).

## Conventions (per TYPO3 docs)

- **Namespace**: `Vendor\ExtensionName\` (CamelCase, one level per folder under `Classes/`)
- **Extension key**: lowercase_with_underscores (e.g. `my_extension`)
- **Composer name**: vendor/extension-name (kebab-case)
- **Paths**: PSR-4 in `Classes/`, TCA in `Configuration/TCA/`, TypoScript in `Configuration/TypoScript/`, Fluid in `Resources/Private/`, compiled assets in `Resources/Public/`

## Controller (Extbase)

- Extend `TYPO3\CMS\Extbase\Mvc\Controller\ActionController`
- Namespace: `Vendor\ExtensionName\Controller`
- Actions: `listAction()`, `showAction()` etc.; return `void`, use `$this->view->assign()` for variables
- Inject repositories via constructor or `@inject` (property injection deprecated in TYPO3 12+; prefer constructor injection)

```php
<?php
namespace Vendor\MyExtension\Controller;

use TYPO3\CMS\Extbase\Mvc\Controller\ActionController;

class MainController extends ActionController
{
    public function __construct(
        private readonly ItemRepository $itemRepository
    ) {}

    public function listAction(): void
    {
        $items = $this->itemRepository->findAll();
        $this->view->assign('items', $items);
    }
}
```

## Model

- Extend `TYPO3\CMS\Extbase\DomainObject\AbstractEntity`
- Namespace: `Vendor\ExtensionName\Domain\Model`
- Properties with getters/setters; use type hints and strict types

```php
<?php
namespace Vendor\MyExtension\Domain\Model;

use TYPO3\CMS\Extbase\DomainObject\AbstractEntity;

class Item extends AbstractEntity
{
    protected string $title = '';

    public function getTitle(): string
    {
        return $this->title;
    }

    public function setTitle(string $title): void
    {
        $this->title = $title;
    }
}
```

## Repository

- Extend `TYPO3\CMS\Extbase\Persistence\Repository`
- Namespace: `Vendor\ExtensionName\Domain\Repository`
- Default `findAll()`; add custom methods with QueryBuilder or `createQuery()` as needed

```php
<?php
namespace Vendor\MyExtension\Domain\Repository;

use TYPO3\CMS\Extbase\Persistence\Repository;

class ItemRepository extends Repository
{
    // Custom find methods here
}
```

## TCA & TypoScript

- Register TCA in `Configuration/TCA/`; include in `ext_tables.php` or via `Configuration/TCA/Overrides/...` if needed.
- Load TypoScript from `Configuration/TypoScript/` via `ext_localconf.php` (`addTypoScriptSetup` / `addTypoScriptConstants`) or static template.
