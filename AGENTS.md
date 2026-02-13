# TYPO3 + DDEV Projekt

## Projekt-Übersicht
- **Framework**: TYPO3 CMS 13 LTS
- **Development Environment**: DDEV
- **PHP Version**: 8.2
- **Database**: MySQL 8.0
- **CSS Framework**: Bootstrap 5 (latest stable)
- **Build Tool**: Vite
- **Code Quality**: PHPStan, PHPCS

## TYPO3 Dokumentation
- **Core API**: https://docs.typo3.org/m/typo3/reference-coreapi/13.4/en-us/
- **Extension Development**: https://docs.typo3.org/m/typo3/reference-coreapi/13.4/en-us/ExtensionArchitecture/
- **TypoScript**: https://docs.typo3.org/m/typo3/reference-typoscript/13.4/en-us/

## Projekt-Initialisierung

### 1. Composer Setup (TYPO3 13 LTS)
```json
{
  "name": "vendor/project-name",
  "description": "TYPO3 13 LTS Project",
  "type": "project",
  "require": {
    "typo3/cms-core": "^13.4",
    "typo3/cms-backend": "^13.4",
    "typo3/cms-frontend": "^13.4",
    "typo3/cms-extbase": "^13.4",
    "typo3/cms-fluid": "^13.4",
    "typo3/cms-install": "^13.4"
  },
  "require-dev": {
    "phpstan/phpstan": "^1.10",
    "phpstan/extension-installer": "^1.3",
    "saschaegerer/phpstan-typo3": "^1.10",
    "symplify/easy-coding-standard": "^12.0"
  },
  "repositories": [
    {
      "type": "path",
      "url": "packages/*"
    }
  ],
  "autoload": {
    "psr-4": {
      "Vendor\\Theme\\": "packages/theme/Classes/"
    }
  },
  "extra": {
    "typo3/cms": {
      "web-dir": "public"
    }
  }
}
```

### 2. Installation
```bash
# TYPO3 installieren
ddev composer create-project typo3/cms-base-distribution .

# DDEV starten
ddev start

# TYPO3 Setup
ddev typo3 setup

# Vite & Dependencies
ddev exec npm install

# Browser öffnen
ddev launch
```

## Wichtige Befehle

### DDEV
```bash
ddev start          # Container starten
ddev stop           # Container stoppen
ddev restart        # Container neustarten
ddev ssh            # In Container einloggen
ddev logs           # Logs anzeigen
```

### TYPO3
```bash
ddev typo3          # TYPO3 CLI
ddev typo3 cache:flush
ddev typo3 extension:activate <ext_key>
ddev typo3 database:updateschema
```

### Composer
```bash
ddev composer require <package>
ddev composer update
```

### Database
```bash
ddev export-db --file=backup.sql.gz
ddev import-db --file=backup.sql.gz
ddev mysql          # MySQL CLI
```

## Projekt-Struktur

```
project/
├── .ddev/
│   └── config.yaml
├── config/
│   └── system/
│       └── settings.php
├── packages/
│   ├── theme/                          # Haupt-Theme Extension
│   │   ├── Classes/
│   │   ├── Configuration/
│   │   │   ├── TCA/
│   │   │   ├── TSconfig/
│   │   │   └── TypoScript/
│   │   ├── Resources/
│   │   │   ├── Private/
│   │   │   │   ├── Js/               # Source JS (wird compiliert)
│   │   │   │   ├── Layouts/
│   │   │   │   ├── Partials/
│   │   │   │   ├── Scss/             # Bootstrap SCSS + Custom
│   │   │   │   └── Templates/
│   │   │   └── Public/
│   │   │       ├── Css/              # Compiliertes CSS
│   │   │       ├── Js/               # Compiliertes JS
│   │   │       └── Images/
│   │   ├── ext_emconf.php
│   │   ├── ext_localconf.php
│   │   ├── ext_tables.php
│   │   └── composer.json
│   │
│   └── [weitere-extensions]/         # Neue Extensions hier
│       ├── Classes/
│       │   ├── Controller/
│       │   ├── Domain/
│       │   │   ├── Model/
│       │   │   └── Repository/
│       │   └── ViewHelpers/
│       ├── Configuration/
│       │   ├── TCA/
│       │   └── TypoScript/
│       ├── Resources/
│       │   ├── Private/
│       │   └── Public/
│       ├── ext_emconf.php
│       ├── ext_localconf.php
│       ├── ext_tables.php
│       └── composer.json
│
├── public/
│   ├── typo3/
│   ├── typo3conf/
│   ├── fileadmin/
│   └── index.php
├── var/
├── vendor/
├── node_modules/
├── vite.config.js
├── package.json
├── composer.json
├── phpstan.neon
└── claude.md
```

## Vite Konfiguration

### vite.config.js
```javascript
import { defineConfig } from 'vite';
import { resolve } from 'path';
import { readdirSync, statSync } from 'fs';

// Auto-discover alle packages
function discoverPackages() {
  const packagesDir = resolve(__dirname, 'packages');
  const entries = {};
  
  try {
    const packages = readdirSync(packagesDir);
    
    packages.forEach(pkg => {
      const pkgPath = resolve(packagesDir, pkg);
      if (statSync(pkgPath).isDirectory()) {
        const jsEntry = resolve(pkgPath, 'Resources/Private/Js/main.js');
        const scssEntry = resolve(pkgPath, 'Resources/Private/Scss/main.scss');
        
        try {
          if (statSync(jsEntry).isFile()) {
            entries[`${pkg}-js`] = jsEntry;
          }
        } catch {}
        
        try {
          if (statSync(scssEntry).isFile()) {
            entries[`${pkg}-css`] = scssEntry;
          }
        } catch {}
      }
    });
  } catch (e) {
    console.warn('Packages directory not found');
  }
  
  return entries;
}

export default defineConfig({
  build: {
    manifest: true,
    rollupOptions: {
      input: discoverPackages(),
      output: {
        entryFileNames: 'Js/[name].[hash].js',
        chunkFileNames: 'Js/[name].[hash].js',
        assetFileNames: (assetInfo) => {
          if (assetInfo.name.endsWith('.css')) {
            return 'Css/[name].[hash][extname]';
          }
          return 'Assets/[name].[hash][extname]';
        }
      }
    },
    outDir: 'public/_assets',
    emptyOutDir: true
  },
  server: {
    origin: 'http://localhost:5173',
    cors: true,
    strictPort: true,
    port: 5173,
    hmr: {
      host: 'localhost',
      protocol: 'ws'
    }
  },
  css: {
    devSourcemap: true
  }
});
```

### package.json
```json
{
  "name": "typo3-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "sass": "^1.69.0"
  },
  "dependencies": {
    "bootstrap": "^5.3.2"
  }
}
```

### Vite Development
```bash
# Development Server (mit HMR)
ddev exec npm run dev

# Production Build
ddev exec npm run build
```

## Theme Extension (packages/theme/)

### composer.json
```json
{
  "name": "vendor/theme",
  "type": "typo3-cms-extension",
  "description": "Site Theme Extension",
  "require": {
    "typo3/cms-core": "^13.4",
    "typo3/cms-frontend": "^13.4",
    "typo3/cms-fluid": "^13.4"
  },
  "autoload": {
    "psr-4": {
      "Vendor\\Theme\\": "Classes/"
    }
  },
  "extra": {
    "typo3/cms": {
      "extension-key": "theme"
    }
  }
}
```

### Resources/Private/Scss/main.scss
```scss
// Bootstrap 5 Import
@import "bootstrap/scss/bootstrap";

// Custom Variables
$primary-color: #0080ff;

// Custom Styles
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}
```

### Resources/Private/Js/main.js
```javascript
// Bootstrap JS
import 'bootstrap';

// Custom JS
console.log('Theme loaded');

// Module Imports
// import { myFunction } from './modules/example.js';
```

### ext_emconf.php
```php
<?php
$EM_CONF[$_EXTKEY] = [
    'title' => 'Site Theme',
    'description' => 'Main theme extension',
    'category' => 'fe',
    'author' => 'Your Name',
    'author_email' => 'email@example.com',
    'state' => 'stable',
    'version' => '1.0.0',
    'constraints' => [
        'depends' => [
            'typo3' => '13.4.0-13.4.99',
        ],
    ],
];
```

### ext_localconf.php
```php
<?php
defined('TYPO3') or die();

\TYPO3\CMS\Core\Utility\ExtensionManagementUtility::addPageTSConfig(
    '@import "EXT:theme/Configuration/TSconfig/Page.tsconfig"'
);
```

## Neue Extension erstellen

### 1. Struktur anlegen (TYPO3 Standard)
```bash
mkdir -p packages/meine_extension/{Classes/{Controller,Domain/{Model,Repository},ViewHelpers},Configuration/{TCA,TypoScript},Resources/{Private/{Layouts,Partials,Templates,Js,Scss},Public/{Css,Js}}}
cd packages/meine_extension
```

### 2. composer.json (TYPO3 Standard)
```json
{
  "name": "vendor/meine-extension",
  "type": "typo3-cms-extension",
  "description": "Extension Beschreibung",
  "require": {
    "typo3/cms-core": "^13.4",
    "typo3/cms-extbase": "^13.4",
    "typo3/cms-fluid": "^13.4"
  },
  "autoload": {
    "psr-4": {
      "Vendor\\MeineExtension\\": "Classes/"
    }
  },
  "extra": {
    "typo3/cms": {
      "extension-key": "meine_extension"
    }
  }
}
```

### 3. ext_emconf.php
```php
<?php
$EM_CONF[$_EXTKEY] = [
    'title' => 'Meine Extension',
    'description' => 'Beschreibung',
    'category' => 'plugin',
    'author' => 'Author Name',
    'author_email' => 'email@example.com',
    'state' => 'stable',
    'version' => '1.0.0',
    'constraints' => [
        'depends' => [
            'typo3' => '13.4.0-13.4.99',
        ],
    ],
];
```

### 4. ext_localconf.php (Plugin registrieren)
```php
<?php
defined('TYPO3') or die();

\TYPO3\CMS\Extbase\Utility\ExtensionUtility::configurePlugin(
    'MeineExtension',
    'Pi1',
    [
        \Vendor\MeineExtension\Controller\MainController::class => 'list, show',
    ],
    [
        \Vendor\MeineExtension\Controller\MainController::class => '',
    ]
);
```

### 5. Installation
```bash
# Repository hinzufügen (falls noch nicht in composer.json)
ddev composer config repositories.meine-extension path packages/meine_extension

# Extension installieren
ddev composer require vendor/meine-extension:@dev

# Extension aktivieren
ddev typo3 extension:activate meine_extension

# Cache leeren
ddev typo3 cache:flush
```

### 6. Vite Integration (automatisch)
- `Resources/Private/Js/main.js` erstellen → wird automatisch gefunden
- `Resources/Private/Scss/main.scss` erstellen → wird automatisch compiliert
- Build: `ddev exec npm run build`

## Code Quality & Testing

### PHPStan Konfiguration (phpstan.neon)
```neon
includes:
    - vendor/saschaegerer/phpstan-typo3/extension.neon

parameters:
    level: 8
    paths:
        - packages/
    excludePaths:
        - packages/*/Resources/
        - packages/*/Tests/
    ignoreErrors:
        # TYPO3 spezifische Ignores
        - '#Cannot access property .* on TYPO3\\CMS\\.*#'
```

### PHPStan Ausführen
```bash
# Komplettes Projekt analysieren
ddev exec vendor/bin/phpstan analyse

# Nur eine Extension
ddev exec vendor/bin/phpstan analyse packages/theme/

# Mit Level
ddev exec vendor/bin/phpstan analyse -l 8 packages/
```

### ECS (Easy Coding Standard)
```bash
# Code Style Check
ddev exec vendor/bin/ecs check packages/

# Auto-Fix
ddev exec vendor/bin/ecs check packages/ --fix
```

### composer.json Scripts
```json
{
  "scripts": {
    "test:phpstan": "phpstan analyse",
    "test:ecs": "ecs check packages/",
    "fix:ecs": "ecs check packages/ --fix",
    "test": [
      "@test:phpstan",
      "@test:ecs"
    ]
  }
}
```

### Pre-Commit Hook (.git/hooks/pre-commit)
```bash
#!/bin/bash
ddev exec vendor/bin/phpstan analyse --no-progress
if [ $? -ne 0 ]; then
    echo "PHPStan failed. Commit aborted."
    exit 1
fi
```

## TYPO3 Standards & Best Practices

### Extension Development (nach TYPO3 Docs)
- **Namespace**: `Vendor\ExtensionName\`
- **Extension Key**: lowercase_with_underscores
- **Composer Name**: vendor/extension-name
- **Classes**: PSR-4 Autoloading in `Classes/`
- **Configuration**: TCA in `Configuration/TCA/`, TypoScript in `Configuration/TypoScript/`
- **Templates**: Fluid Templates in `Resources/Private/`
- **Assets**: Compiled in `Resources/Public/`

### Controller (Extbase)
```php
<?php
namespace Vendor\MeineExtension\Controller;

use TYPO3\CMS\Extbase\Mvc\Controller\ActionController;

class MainController extends ActionController
{
    public function listAction(): void
    {
        $items = $this->itemRepository->findAll();
        $this->view->assign('items', $items);
    }
}
```

### Model
```php
<?php
namespace Vendor\MeineExtension\Domain\Model;

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

### Repository
```php
<?php
namespace Vendor\MeineExtension\Domain\Repository;

use TYPO3\CMS\Extbase\Persistence\Repository;

class ItemRepository extends Repository
{
    // Custom queries hier
}
```

## Konfiguration

### DDEV (.ddev/config.yaml)
```yaml
name: project-name
type: typo3
php_version: "8.2"
webserver_type: nginx-fpm
database:
  type: mysql
  version: "8.0"
```

### TYPO3 Backend
- URL: https://project-name.ddev.site/typo3
- Admin User: (siehe .env oder setup)

## Development Workflow

### 1. Neue Feature entwickeln
```bash
# Branch erstellen
git checkout -b feature/neue-funktion

# Vite Dev Server starten (HMR)
ddev exec npm run dev

# Code ändern in packages/[extension]/
# - PHP: Classes/
# - JS: Resources/Private/Js/
# - SCSS: Resources/Private/Scss/
# - Templates: Resources/Private/Templates/

# Cache leeren
ddev typo3 cache:flush

# Testen
ddev launch
```

### 2. Code Quality Check
```bash
# PHPStan
ddev composer test:phpstan

# Code Style
ddev composer test:ecs

# Alles testen
ddev composer test
```

### 3. Production Build
```bash
# Assets compilieren
ddev exec npm run build

# Commit
git add .
git commit -m "feat: neue Funktion"
git push
```

### 4. Deployment
```bash
# Composer Production Install
composer install --no-dev --optimize-autoloader

# Assets Build
npm run build

# Database Update
vendor/bin/typo3 database:updateschema

# Cache leeren
vendor/bin/typo3 cache:flush
```

## Troubleshooting

### Cache Issues
```bash
ddev typo3 cache:flush
ddev restart
```

### Permission Problems
```bash
ddev exec chmod -R 775 var/ public/typo3temp/
```

### Database Problems
```bash
ddev typo3 database:updateschema
```

## Commit-Message-Regeln (TYPO3)

Angelehnt an die [TYPO3 Commit Message rules](https://docs.typo3.org/m/typo3/guide-contributionworkflow/main/en-us/Appendix/CommitMessage.html).

### Summary-Zeile (erste Zeile)

* Mit **Keyword** beginnen: `[BUGFIX]`, `[FEATURE]`, `[TASK]`, `[DOCS]`
* Bei Breaking Changes: `[!!!]` vor das Keyword setzen, z. B. `[!!!][FEATURE]`
* Zeile unter 52 Zeichen (max. 72)
* **Imperativ** formulieren („Add feature“, nicht „Added feature“)  
  Prüfregel: „If applied, this commit will **&lt;Subject&gt;**“ muss sinnvoll lesbar sein
* Nach dem Keyword mit Großbuchstabe starten

Beispiele:
```
[TASK] Add initial TYPO3 Agent Kit boilerplate
[FEATURE] Add option to hide BE search box in list module
[BUGFIX] Fix backend edit URL in admin panel
[DOCS] Add documentation for version 9.1
```

### Body (Beschreibung)

* Was wird geändert/hinzugefügt – kurz und sachlich
* Aufzählungen mit `*` und hängendem Einzug
* Zeilen nach 72 Zeichen umbrechen

### Relationships (optional, bei TYPO3-Forge-Patches Pflicht)

* `Resolves: #12345` – schließt ein Forge-Ticket
* `Related: #12340` – verknüpft weiteres Ticket
* `Releases: main, 13.4` – Ziel-Branches

In eigenen Repos ohne Forge können Resolves/Releases entfallen.

## Nützliche Links
- DDEV Docs: https://ddev.readthedocs.io
- TYPO3 Docs: https://docs.typo3.org
- Project Repository: https://github.com/ohm-meter/typo3-agent-kit

## Notizen
- (Projektspezifische Infos hier)
