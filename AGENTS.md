# TYPO3 + DDEV Project

## Project Overview
- **Framework**: TYPO3 CMS 13 LTS
- **Development Environment**: DDEV
- **PHP Version**: 8.2
- **Database**: MySQL 8.0
- **CSS Framework**: Bootstrap 5 (latest stable)
- **Build Tool**: Vite
- **Code Quality**: PHPStan, PHPCS

## TYPO3 Documentation
- **Core API**: https://docs.typo3.org/m/typo3/reference-coreapi/13.4/en-us/
- **Extension Development**: https://docs.typo3.org/m/typo3/reference-coreapi/13.4/en-us/ExtensionArchitecture/
- **TypoScript**: https://docs.typo3.org/m/typo3/reference-typoscript/13.4/en-us/

## Project Initialization

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
# Install TYPO3 (or use existing composer.json and: ddev composer install)
ddev composer create-project typo3/cms-base-distribution .

# Start DDEV
ddev start

# TYPO3 setup (non-interactive, no user input required)
# Standard: Apache. DB and admin credentials match DDEV defaults.
ddev typo3 setup \
  --server-type=apache \
  --driver=mysqli \
  --host=db --port=3306 --dbname=db --username=db --password=db \
  --admin-username=admin \
  --admin-user-password=admin \
  --admin-email=admin@example.com \
  --create-site=https://<project-name>.ddev.site/ \
  --project-name="TYPO3 Project" \
  --no-interaction

# Optional: interactive setup (waits for user input; first prompt: web server = Apache/IIS/Other)
# ddev typo3 setup

# Vite & dependencies
ddev exec npm install

# Open in browser
ddev launch
```

**Note:** The plain command `ddev typo3 setup` runs interactively and waits for input (first question: web server type). For scripts and automation, use the non-interactive form above. Default web server is set to **Apache**.

## Important Commands

### DDEV
```bash
ddev start          # Start containers
ddev stop           # Stop containers
ddev restart        # Restart containers
ddev ssh            # Shell into container
ddev logs           # Show logs
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

## Project Structure

```
project/
├── .ddev/
│   └── config.yaml
├── config/
│   └── system/
│       └── settings.php
├── packages/
│   ├── theme/                          # Main theme extension
│   │   ├── Classes/
│   │   ├── Configuration/
│   │   │   ├── TCA/
│   │   │   ├── TSconfig/
│   │   │   └── TypoScript/
│   │   ├── Resources/
│   │   │   ├── Private/
│   │   │   │   ├── Js/               # Source JS (compiled)
│   │   │   │   ├── Layouts/
│   │   │   │   ├── Partials/
│   │   │   │   ├── Scss/             # Bootstrap SCSS + custom
│   │   │   │   └── Templates/
│   │   │   └── Public/
│   │   │       ├── Css/              # Compiled CSS
│   │   │       ├── Js/               # Compiled JS
│   │   │       └── Images/
│   │   ├── ext_emconf.php
│   │   ├── ext_localconf.php
│   │   ├── ext_tables.php
│   │   └── composer.json
│   │
│   └── [further-extensions]/          # Add new extensions here
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

## Vite Configuration

### vite.config.js
```javascript
import { defineConfig } from 'vite';
import { resolve } from 'path';
import { readdirSync, statSync } from 'fs';

// Auto-discover all packages
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
# Development server (with HMR)
ddev exec npm run dev

# Production build
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
// Bootstrap 5 import
@import "bootstrap/scss/bootstrap";

// Custom variables
$primary-color: #0080ff;

// Custom styles
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

// Module imports
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

## Create New Extension

### 1. Set Up Structure (TYPO3 Standard)
```bash
mkdir -p packages/my_extension/{Classes/{Controller,Domain/{Model,Repository},ViewHelpers},Configuration/{TCA,TypoScript},Resources/{Private/{Layouts,Partials,Templates,Js,Scss},Public/{Css,Js}}}
cd packages/my_extension
```

### 2. composer.json (TYPO3 Standard)
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

### 3. ext_emconf.php
```php
<?php
$EM_CONF[$_EXTKEY] = [
    'title' => 'My Extension',
    'description' => 'Description',
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

### 4. ext_localconf.php (Register Plugin)
```php
<?php
defined('TYPO3') or die();

\TYPO3\CMS\Extbase\Utility\ExtensionUtility::configurePlugin(
    'MyExtension',
    'Pi1',
    [
        \Vendor\MyExtension\Controller\MainController::class => 'list, show',
    ],
    [
        \Vendor\MyExtension\Controller\MainController::class => '',
    ]
);
```

### 5. Installation
```bash
# Add repository (if not already in composer.json)
ddev composer config repositories.my-extension path packages/my_extension

# Install extension
ddev composer require vendor/my-extension:@dev

# Activate extension
ddev typo3 extension:activate my_extension

# Flush cache
ddev typo3 cache:flush
```

### 6. Vite Integration (Automatic)
- Create `Resources/Private/Js/main.js` → auto-discovered
- Create `Resources/Private/Scss/main.scss` → compiled automatically
- Build: `ddev exec npm run build`

## Code Quality & Testing

### PHPStan Configuration (phpstan.neon)
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
        # TYPO3-specific ignores
        - '#Cannot access property .* on TYPO3\\CMS\\.*#'
```

### Run PHPStan
```bash
# Analyse full project
ddev exec vendor/bin/phpstan analyse

# Single extension only
ddev exec vendor/bin/phpstan analyse packages/theme/

# With level
ddev exec vendor/bin/phpstan analyse -l 8 packages/
```

### ECS (Easy Coding Standard)
```bash
# Code style check
ddev exec vendor/bin/ecs check packages/

# Auto-fix
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

### Extension Development (per TYPO3 Docs)
- **Namespace**: `Vendor\ExtensionName\`
- **Extension Key**: lowercase_with_underscores
- **Composer Name**: vendor/extension-name
- **Classes**: PSR-4 autoloading in `Classes/`
- **Configuration**: TCA in `Configuration/TCA/`, TypoScript in `Configuration/TypoScript/`
- **Templates**: Fluid templates in `Resources/Private/`
- **Assets**: Compiled in `Resources/Public/`

### Controller (Extbase)
```php
<?php
namespace Vendor\MyExtension\Controller;

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

### Repository
```php
<?php
namespace Vendor\MyExtension\Domain\Repository;

use TYPO3\CMS\Extbase\Persistence\Repository;

class ItemRepository extends Repository
{
    // Custom queries here
}
```

## Configuration

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
- Admin user: (see .env or setup)

## Development Workflow

### 1. Develop New Feature
```bash
# Create branch
git checkout -b feature/my-feature

# Start Vite dev server (HMR)
ddev exec npm run dev

# Edit code in packages/[extension]/
# - PHP: Classes/
# - JS: Resources/Private/Js/
# - SCSS: Resources/Private/Scss/
# - Templates: Resources/Private/Templates/

# Flush cache
ddev typo3 cache:flush

# Test
ddev launch
```

### 2. Code Quality Check
```bash
# PHPStan
ddev composer test:phpstan

# Code style
ddev composer test:ecs

# Run all tests
ddev composer test
```

### 3. Production Build
```bash
# Compile assets
ddev exec npm run build

# Commit
git add .
git commit -m "feat: new feature"
git push
```

### 4. Deployment
```bash
# Composer production install
composer install --no-dev --optimize-autoloader

# Assets build
npm run build

# Database update
vendor/bin/typo3 database:updateschema

# Flush cache
vendor/bin/typo3 cache:flush
```

## Troubleshooting

### 404 – No site configuration found
If the browser shows **404** with *"No site configuration found"*, TYPO3 has no site config for the current host. Create `config/sites/<identifier>/config.yaml` (e.g. `config/sites/main/config.yaml`) with at least:

- **rootPageId**: UID of the root page (usually `1` if setup was run)
- **base**: Full base URL of the site, e.g. `https://typo3-agent-kit.ddev.site/`
- **languages**: One language with `languageId: 0`, `base: /`, `enabled: true`

Example minimal `config/sites/main/config.yaml`:

```yaml
rootPageId: 1
base: 'https://typo3-agent-kit.ddev.site/'
baseVariants: []
languages:
  - title: Deutsch
    enabled: true
    base: /
    locale: de_DE.UTF-8
    languageId: 0
    websiteTitle: ''
    navigationTitle: Deutsch
    hreflang: de-DE
    direction: ltr
    typo3Language: de
    flag: de
errorHandling: []
routes: []
```

Use your actual DDEV hostname in `base`. After adding or changing the file, flush cache.

### 404 – Root page missing (empty `pages` table)
If the site config exists but the frontend still shows **404** or the root page is missing, the `pages` table may be empty. Use TYPO3’s built-in setup instead of manual SQL.

**Preferred: TYPO3 setup with `--create-site` (creates root page + site config)**  
Run setup **once** when the database is still empty (or after resetting the DB). TYPO3 will create the root page (“Home”), a basic TypoScript record, and `config/sites/main/config.yaml`:

```bash
# Optional: reset DB if it already has data but no pages
# ddev delete --omit-snapshot && ddev start

ddev typo3 setup \
  --server-type=apache --driver=mysqli --host=db --port=3306 --dbname=db --username=db --password=db \
  --admin-username=admin --admin-user-password=admin --admin-email=admin@example.com \
  --create-site=https://typo3-agent-kit.ddev.site/ \
  --project-name="TYPO3 Agent Kit" \
  --no-interaction
```

Replace the URL with your DDEV hostname. If the DB already contains tables, setup may refuse to run; then use one of the options below.

**Alternative 1 – Backend (Site Management):**  
Backend → **Site Management** → **Sites** → **Create new site**. Follow the wizard (root page + site configuration are created).

**Alternative 2 – Backend (Page tree):**  
Backend → **Web** → **Page** → right‑click root level → **New** → enter title → in the new page under **Behaviour** enable **Use as Root Page** → Save. Then add a matching site in **Site Management** → **Sites** with this page as root.

**Check if root page exists:**
```bash
ddev mysql -e "SELECT uid, pid, title, doktype, is_siteroot FROM pages WHERE uid = 1;"
```
If a row is returned, the root page exists and the site config’s `rootPageId` (e.g. 1) must match. Then flush cache: `ddev exec vendor/bin/typo3 cache:flush`.

### 503 – Trusted Hosts (DDEV)
If the browser shows **503** and the message *"The current host header value does not match the configured trusted hosts pattern"* (e.g. host `typo3-agent-kit.ddev.site`), add the trusted hosts pattern for DDEV in `config/system/settings.php` inside the `SYS` array:

```php
'SYS' => [
    'trustedHostsPattern' => '.*\.ddev\.site(\.local)?.*',
    // ... rest of SYS config
],
```

This allows any `*.ddev.site` (and `*.ddev.site.local`) host. After changing, flush cache: `ddev exec vendor/bin/typo3 cache:flush` or use Backend → Admin Tools → Flush cache.

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

## Commit Message Rules (TYPO3)

Based on the [TYPO3 Commit Message rules](https://docs.typo3.org/m/typo3/guide-contributionworkflow/main/en-us/Appendix/CommitMessage.html).

### Summary Line (First Line)

* Start with a **keyword**: `[BUGFIX]`, `[FEATURE]`, `[TASK]`, `[DOCS]`
* For breaking changes: put `[!!!]` before the keyword, e.g. `[!!!][FEATURE]`
* Keep the line under 52 characters (max 72)
* Use **imperative mood** (“Add feature”, not “Added feature”).  
  Check: “If applied, this commit will **&lt;Subject&gt;**” must read naturally
* Capitalize after the keyword

Examples:
```
[TASK] Add initial TYPO3 Agent Kit boilerplate
[FEATURE] Add option to hide BE search box in list module
[BUGFIX] Fix backend edit URL in admin panel
[DOCS] Add documentation for version 9.1
```

### Body (Description)

* What is changed or added – brief and to the point
* Use `*` for bullet points and a hanging indent
* Wrap lines at 72 characters

### Relationships (Optional; Required for TYPO3 Forge Patches)

* `Resolves: #12345` – closes a Forge ticket
* `Related: #12340` – links another ticket
* `Releases: main, 13.4` – target branches

In your own repos without Forge, Resolves/Releases can be omitted.

## Useful Links
- DDEV Docs: https://ddev.readthedocs.io
- TYPO3 Docs: https://docs.typo3.org
- Project Repository: https://github.com/ohm-meter/typo3-agent-kit

## Notes
- (Project-specific notes here)
