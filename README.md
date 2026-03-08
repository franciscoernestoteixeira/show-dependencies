# show-dependencies

A simple CLI tool to list the **installed `require`** and **`require-dev`** packages of a PHP/Composer project with their **actual installed versions** — read from `composer.lock` (preferred) or directly from `vendor/`.

It outputs:
- A human-readable table
- A JSON excerpt you can paste straight back into your `composer.json`

---

## Why?

When maintaining projects, `composer.json` often lists version **ranges** (like `^4.0`), but sometimes you need to pin the **exact versions that are actually installed** — for example, when `composer.lock` has been accidentally deleted or was never committed, and `composer install` would otherwise upgrade everything.

This tool reads `composer.lock` when available, or falls back to scanning `vendor/`, and matches only the packages you declared in your own `composer.json`.

---

## Requirements

| Requirement | Minimum version |
|---|---|
| PHP | 5.6 or higher (up to PHP 8.x) |
| Composer | Any recent version |

PHP and Composer must both be available on your system `PATH`.  
See the installation guides below if you need to set them up first.

---

## Installing PHP and Composer

### 🪟 Windows

**PHP**

1. Download the latest PHP zip for Windows from [https://windows.php.net/download](https://windows.php.net/download).  
   Choose the **Thread Safe** x64 zip.
2. Extract it to a permanent folder, for example `C:\php`.
3. Add `C:\php` to your system `PATH`:
   - Open **Start → Search → "Edit the system environment variables"**
   - Click **Environment Variables**
   - Under **System variables**, select `Path` → **Edit → New**
   - Add `C:\php` → **OK**

4. Open a new terminal and verify:
   ```
   php --version
   ```

**Composer**

Download and run the official installer from [https://getcomposer.org/download](https://getcomposer.org/download).  
The Windows installer (`Composer-Setup.exe`) handles everything automatically, including adding Composer to `PATH`.

Verify:
```
composer --version
```

---

### 🍎 macOS

**PHP**

macOS ships with PHP, but it may be outdated. The recommended approach is [Homebrew](https://brew.sh):

```bash
brew install php
```

Verify:
```bash
php --version
```

**Composer**

```bash
brew install composer
```

Or install manually:

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

Verify:
```bash
composer --version
```

---

### 🐧 Linux (Debian / Ubuntu)

**PHP**

```bash
sudo apt update
sudo apt install php php-cli
```

For other distributions, use the equivalent package manager (`yum`, `dnf`, `pacman`, etc.).

Verify:
```bash
php --version
```

**Composer**

```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
php -r "unlink('composer-setup.php');"
sudo mv composer.phar /usr/local/bin/composer
```

Verify:
```bash
composer --version
```

---

## Installing show-dependencies

Once PHP and Composer are ready, install this tool globally with:

```bash
composer global require franciscoernestoteixeira/show-dependencies
```

This places the `show-dependencies` binary into Composer's global `bin` directory.  
You need to add that directory to your `PATH` so the command is available everywhere.

---

## Adding the Composer global bin to PATH

### 🪟 Windows

Composer's global bin directory on Windows is typically:

```
C:\Users\<YourUsername>\AppData\Roaming\Composer\vendor\bin
```

To find the exact path on your machine:

```
composer global config bin-dir --absolute
```

Then add that path to your system `PATH` (same steps as adding `C:\php` above).

After adding it, open a **new** terminal and verify:

```
show-dependencies
```

> **Tip:** In PowerShell you can add it temporarily for the current session:
> ```powershell
> $env:PATH += ";" + (composer global config bin-dir --absolute)
> ```

---

### 🍎 macOS

Find the directory:

```bash
composer global config bin-dir --absolute
```

It is usually `~/.composer/vendor/bin` or `~/.config/composer/vendor/bin`.

Add it permanently by appending to your shell profile. For **zsh** (default on macOS):

```bash
echo 'export PATH="$PATH:$(composer global config bin-dir --absolute 2>/dev/null)"' >> ~/.zshrc
source ~/.zshrc
```

For **bash**:

```bash
echo 'export PATH="$PATH:$(composer global config bin-dir --absolute 2>/dev/null)"' >> ~/.bash_profile
source ~/.bash_profile
```

Verify:

```bash
show-dependencies
```

---

### 🐧 Linux

Find the directory:

```bash
composer global config bin-dir --absolute
```

It is usually `~/.composer/vendor/bin` or `~/.config/composer/vendor/bin`.

Add it permanently to your shell profile:

```bash
echo 'export PATH="$PATH:$(composer global config bin-dir --absolute 2>/dev/null)"' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
show-dependencies
```

---

## Usage

Navigate to the root of any PHP/Composer project — the folder that contains your `composer.json` — and run:

```bash
# Linux / macOS
show-dependencies

# Windows (Command Prompt or PowerShell)
show-dependencies
```

The tool will automatically look for `composer.lock` first, then fall back to scanning `vendor/`.

> You do **not** need to run it from inside the `vendor/` directory. Always run it from the **project root**.

---

## Example

Suppose your `composer.json` declares:

```json
{
  "require": {
    "php": "^8.2",
    "ext-bcmath": "*",
    "ext-gd": "*",
    "laravel/framework": "^11.0",
    "guzzlehttp/guzzle": "^7.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^10.0"
  }
}
```

And your project actually has installed:

- `laravel/framework 11.28.1`
- `guzzlehttp/guzzle 7.9.2`
- `phpunit/phpunit 10.5.36`

Running `show-dependencies` from the project root produces:

```
// Source (resolved packages): composer.lock

name                  version
--------------------  --------------------
php                   ^8.2  (declared)
ext-bcmath            *  (declared)
ext-gd                *  (declared)
guzzlehttp/guzzle     7.9.2
laravel/framework     11.28.1
phpunit/phpunit       10.5.36

// --- Paste into your composer.json ---
{
    "require": {
        "php": "^8.2",
        "ext-bcmath": "*",
        "ext-gd": "*",
        "guzzlehttp/guzzle": "7.9.2",
        "laravel/framework": "11.28.1"
    },
    "require-dev": {
        "phpunit/phpunit": "10.5.36"
    }
}
```

Platform requirements (`php`, `ext-*`, `lib-*`, `hhvm`) appear first in the table marked as `(declared)` and are carried through verbatim to the JSON excerpt — their original constraint is the correct value to preserve.

---

## How it works

1. Reads `composer.json` in the current directory for declared `require` / `require-dev` package names.
2. **Separates platform constraints** (`php`, `ext-*`, `lib-*`, `hhvm`) from real packages. Platform entries are kept verbatim — their original declared constraint is the correct value to preserve, since they have no `vendor/` entry to resolve against.
3. If `composer.lock` exists, reads exact versions from it for all non-platform packages — the most reliable source.
4. Supplements with a `vendor/` scan for any package the lock did not cover, or when no lock file exists at all.
5. When the same package appears at multiple depths in `vendor/`, the shallowest (closest to root) copy wins.
6. Strips a leading `v` from version strings (e.g. `v2.3.0` → `2.3.0`).
7. Prints the table: platform entries first (in their original declared order, marked `(declared)`), then resolved packages sorted alphabetically.
8. Prints the JSON excerpt in the same order, ready to paste back into `composer.json`.

---

## Features

- Lists **only declared dependencies** (`require` and `require-dev`)
- **Prefers `composer.lock`** for precision; falls back to `vendor/` gracefully
- **Preserves platform constraints** (`php`, `ext-*`, `lib-*`, `hhvm`) verbatim with their original declared value — they are not resolved or dropped
- Strips leading `v` prefix from version strings
- Outputs a **sorted table**
- Prints a **JSON excerpt** ready to paste into `composer.json`
- Reports packages declared but **not found**
- **Zero dependencies** — pure PHP, no external packages required
- **Works on Windows, macOS, and Linux**
- **Compatible with PHP 5.6 through 8.x**

---

## Troubleshooting

**`show-dependencies: command not found`**  
The Composer global bin directory is not on your `PATH`. Follow the [Adding the Composer global bin to PATH](#adding-the-composer-global-bin-to-path) section above for your operating system.

**`composer.json not found in the current directory`**  
You are not in the project root. Use `cd` to navigate to the folder that contains your `composer.json` before running the tool.

**Some packages show as "not found"**  
The package is declared in `composer.json` but is not present in `composer.lock` or `vendor/`. Run `composer install` first to install all dependencies, then re-run `show-dependencies`.

**Version shows as a range or `*` instead of a real number**  
This can happen if a package's own `composer.json` inside `vendor/` does not declare a `version` field (some development checkouts omit it). Use `composer.lock` — it always contains the resolved version — by running `composer install` before using this tool.

---

## Running locally before publishing

You do not need to publish to Packagist to test the tool. There are two approaches, from simplest to most complete.

---

### Method 1 — Call PHP directly (quickest)

This runs the script immediately without any installation step. Use this to verify the output is correct during development.

**1. Clone or download the `show-dependencies` repository to your machine.**

Let's say you saved it to:

| OS | Example path |
|---|---|
| Linux / macOS | `/home/you/projects/show-dependencies` |
| Windows | `C:\projects\show-dependencies` |

**2. Open a terminal and navigate to the project you want to inspect** (not the `show-dependencies` folder — a real project that has its own `composer.json`):

```bash
# Linux / macOS
cd /home/you/projects/my-laravel-app

# Windows
cd C:\projects\my-laravel-app
```

**3. Run the script by calling PHP with the full path to the binary:**

```bash
# Linux / macOS
php /home/you/projects/show-dependencies/bin/show-dependencies

# Windows (Command Prompt)
php C:\projects\show-dependencies\bin\show-dependencies

# Windows (PowerShell)
php C:\projects\show-dependencies\bin\show-dependencies
```

You should see the table and JSON output immediately.  
No `composer install`, no PATH changes, no installation required.

---

### Method 2 — Install globally via a local path (full end-to-end test)

This is the closest simulation to how end users will install it from Packagist. Composer loads the package directly from your local folder instead of downloading it from the internet.

**1. Tell Composer's global config to look for packages in your local folder.**

```bash
composer global config repositories.show-dependencies \
  '{"type":"path","url":"/home/you/projects/show-dependencies","options":{"symlink":true}}'
```

On Windows (Command Prompt), replace single quotes with double quotes and escape the inner ones:

```
composer global config repositories.show-dependencies "{\"type\":\"path\",\"url\":\"C:\\projects\\show-dependencies\",\"options\":{\"symlink\":true}}"
```

The `"symlink": true` option means Composer links to your local folder instead of copying it, so any edits you make to the script are reflected immediately without reinstalling.

**2. Install it globally as usual:**

```bash
composer global require franciscoernestoteixeira/show-dependencies:@dev
```

The `@dev` flag is required when installing from a local path that has no version tag yet.

**3. Make sure the global bin directory is on your `PATH`** (see the [Adding the Composer global bin to PATH](#adding-the-composer-global-bin-to-path) section above).

**4. Test it from any project:**

```bash
cd /home/you/projects/my-laravel-app
show-dependencies
```

**5. When you are done testing, remove the local repository entry:**

```bash
composer global config --unset repositories.show-dependencies
```

And to remove the installed tool:

```bash
composer global remove franciscoernestoteixeira/show-dependencies
```

---

### Which method should I use?

| Situation | Recommended method |
|---|---|
| Quick smoke test / first run | Method 1 — direct PHP call |
| Testing that the Composer binary registration works | Method 2 — path repository |
| Testing PATH setup and the `show-dependencies` command name | Method 2 — path repository |
| Iterating on the script code | Method 1 — no reinstall needed |

---

## How to publish on Packagist

1. Push the project to a **public** repository on GitHub, GitLab, or Bitbucket.

2. Go to [https://packagist.org](https://packagist.org), log in (or create a free account).

3. Click **Submit**, paste in your repository URL, and click **Check**.  
   Packagist will auto-detect `composer.json` and register the package.

4. Set up a **webhook** so Packagist refreshes automatically on every push:
   - GitHub repo → **Settings → Webhooks → Add webhook**
   - Payload URL: `https://packagist.org/api/github?username=YOUR_PACKAGIST_USERNAME`
   - Content type: `application/json`
   - Secret: your Packagist API token (from your Packagist profile page)
   - Events: **Just the push event**

5. Tag your first stable release:

   ```bash
   git tag 1.0.0
   git push origin 1.0.0
   ```

   Packagist picks up the tag and makes `1.0.0` available as a stable version within a few minutes.

---

## License

MIT © [Francisco Ernesto Teixeira](https://github.com/franciscoernestoteixeira)
