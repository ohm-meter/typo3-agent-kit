# TYPO3 Agent Kit

**Boilerplate only in Git.** Runtime and generated artifacts (vendor, node_modules, public/, config with credentials, etc.) are not committed. After cloning, a few commands give you a runnable TYPO3 13 LTS setup.

- TYPO3 CMS 13 LTS  
- DDEV, PHP 8.2, MySQL 8  
- Vite (auto-discovery for extensions in `packages/`)  
- Bootstrap 5, PHPStan, ECS  
- Theme extension “theme” as minimal frontend boilerplate  

Detailed commands and structure: **AGENTS.md** (including for AI assistants).

---

## Quick start (runnable boilerplate)

Requires [DDEV](https://ddev.com/get-started/) to be installed.

```bash
# 1. Clone repo
git clone <repo-url> my-project && cd my-project

# 2. Install dependencies (creates vendor/ and public/)
ddev composer install

# 3. Start DDEV (creates containers, DB, .ddev runtime data)
ddev start

# 4. Set up TYPO3 (DB + config/system/settings.php)
ddev typo3 setup

# 5. Activate theme
ddev typo3 extension:activate theme

# 6. Build frontend assets (Vite)
ddev exec npm install
ddev exec npm run build

# 7. Open site
ddev launch
```

Backend: `ddev launch` → “TYPO3 Backend” link or `https://<project>.ddev.site/typo3`.

Optional: set project name with  
`ddev config --project-name=my-project`

---

## What’s in Git (boilerplate)

| In repo (boilerplate)     | Not in repo (generated / ignored)        |
|---------------------------|-------------------------------------------|
| `composer.json`, `package.json` | `vendor/`, `node_modules/`              |
| `vite.config.js`, `phpstan.neon` | `public/` (web root)                    |
| `packages/theme/` (minimal) | `config/system/settings.php`, `.env`   |
| `.ddev/config.yaml`       | `var/`, `fileadmin/user_upload/`, build output |
| `config/` (empty, .gitkeep) | DDEV runtime (docker-compose, DB dumps)  |
| `AGENTS.md`, `README.md`  |                                           |

---

## Useful commands

```bash
ddev typo3 cache:flush
ddev typo3 extension:activate <ext_key>
ddev exec npm run dev    # Vite with HMR
ddev exec npm run build  # Production build
ddev composer require vendor/package
```

---

## License

See [LICENSE](LICENSE).
