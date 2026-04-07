# Laravel production scripts — copy into your `composer.json`

This repository is **not** a Composer package. It exists so you can **copy two Composer scripts** into your own Laravel project: one for **first-time** production bootstrap and one for **every deploy after that**.

## What you do

1. Open your Laravel app’s **`composer.json`** (the real project, not this repo).
2. Inside the existing **`"scripts"`** object, add **`setup:live`** and **`deploy:live`** (keep valid JSON: commas between keys).
3. **First production bootstrap** — from your Laravel project root:

```bash
composer run setup:live
```

(Composer 2 also allows `composer setup:live`.)

4. **Every deploy after the app is already live** — same directory:

```bash
composer run deploy:live
```

(Composer 2 also allows `composer deploy:live`.)

**Assumption for `deploy:live`:** `.env` is already present and configured on the server (real secrets, database URL, `APP_KEY`, etc.). Do not rely on `deploy:live` to create or populate `.env`.

The exact command arrays to copy live in [`composer.json`](composer.json) under `"scripts"` → `"setup:live"` and `"deploy:live"`.

### Example shape in *your* file

Your `scripts` section might look like this after you merge (keep your existing keys like `post-autoload-dump`):

```json
"scripts": {
    "post-autoload-dump": [
        "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
        "@php artisan package:discover --ansi"
    ],
    "setup:live": [
        "composer install --optimize-autoloader --no-dev",
        "npm ci",
        "npm run build",
        "@php -r \"file_exists('.env') || copy('.env.example', '.env');\"",
        "@php artisan key:generate",
        "php artisan migrate --force",
        "php artisan storage:link --force",
        "php artisan optimize:clear",
        "php artisan optimize"
    ],
    "deploy:live": [
        "composer install --optimize-autoloader --no-dev",
        "npm ci",
        "npm run build",
        "php artisan migrate --force",
        "php artisan optimize:clear",
        "php artisan optimize"
    ]
}
```

## What each script runs (in order)

### `setup:live` (first bootstrap)

| Step | What it does |
|------|---------------|
| 1 | Production Composer install (`--no-dev`) |
| 2 | `npm ci` from your lockfile |
| 3 | `npm run build` (Vite/Mix/etc.) |
| 4 | Create `.env` from `.env.example` if missing |
| 5 | `php artisan key:generate` |
| 6 | `php artisan migrate --force` |
| 7 | `php artisan storage:link --force` |
| 8 | `optimize:clear`, then `optimize` |

### `deploy:live` (repeat deploys)

| Step | What it does |
|------|---------------|
| 1 | Production Composer install (`--no-dev`) |
| 2 | `npm ci` from your lockfile |
| 3 | `npm run build` (Vite/Mix/etc.) |
| 4 | `php artisan migrate --force` |
| 5 | `optimize:clear`, then `optimize` |

**Not run by `deploy:live` (compared to `setup:live`):** bootstrapping `.env` from `.env.example`, `key:generate`, and `storage:link`. Add those to your workflow only when you actually need them (for example, a brand-new server still uses `setup:live`).

Remove or reorder steps in *your* copy if your host or CI already does some of this.

## Prerequisites

Run from the Laravel root (where `artisan` lives): PHP, Composer, **Node/npm** (for both scripts as written), `.env.example` (needed for `setup:live` when `.env` is missing), and a `build` script in `package.json`. If your app has no frontend build, drop or replace the `npm ci` / `npm run build` lines in your project’s `composer.json`.

## Security and operations

Do not commit real `.env` or secrets. Use `.env.example` as a template only. **`deploy:live`** is meant for repeat deploys: it does **not** run `key:generate`. Still decide deliberately whether you want **`migrate --force`** on every deploy in your environment.

**Optional customizations:** After copying the scripts into your app, you can append commands your stack needs — for example `php artisan queue:restart` (or Horizon-specific steps) if workers must pick up new code. Keep those changes in your application repo; this snippet stays minimal on purpose.

## License

MIT — see [`LICENSE`](LICENSE).
