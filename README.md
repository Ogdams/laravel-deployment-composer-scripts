# Laravel `setup:live` — copy into your `composer.json`

This repository is **not** a Composer package. It exists so you can **copy one script** into your own Laravel project and stop typing the same production commands on every new server or deploy.

## What you do

1. Open your Laravel app’s **`composer.json`** (the real project, not this repo).
2. Inside the existing **`"scripts"`** object, add a **`setup:live`** entry (keep valid JSON: commas between keys).
3. From your Laravel project root, run:

```bash
composer run setup:live
```

(Composer 2 also allows `composer setup:live`.)

The script to copy lives in [`composer.json`](composer.json) under `"scripts"` → `"setup:live"`. Paste the whole array of commands there.

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
    ]
}
```

## What it runs (in order)

| Step | What it does |
|------|----------------|
| 1 | Production Composer install (`--no-dev`) |
| 2 | `npm ci` from your lockfile |
| 3 | `npm run build` (Vite/Mix/etc.) |
| 4 | Create `.env` from `.env.example` if missing |
| 5 | `php artisan key:generate` |
| 6 | `php artisan migrate --force` |
| 7 | `php artisan storage:link --force` |
| 8 | `optimize:clear`, then `optimize` |

Remove or reorder steps if your host or CI already does some of this.

## Prerequisites

Run from the Laravel root (where `artisan` lives): PHP, Composer, Node/npm, `.env.example`, and a `build` script in `package.json`.

## Security (public repo / real deploys)

Do not commit real `.env` or secrets. Use `.env.example` as a template only. For repeat deploys, think through whether you always want `key:generate` and `migrate --force` on every run.

## License

MIT — see [`LICENSE`](LICENSE).
