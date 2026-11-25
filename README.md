# ta24bblog

Laravel 13 coursework project with Vite, Tailwind CSS and a MariaDB database.
This guide explains how to install the project on a new Windows computer and run it locally.

Author: Jegor Onoprijuk, TA-24B.

The assignment calls the repository `mm24blog`; this group's project is `ta24bblog`, so use the repository URL below.
The current application opens the Laravel welcome page. This guide documents the existing project setup.

## Requirements

- Windows 11 with PowerShell and Windows Package Manager (`winget`).
- Git and Visual Studio Code.
- **PHP 8.4.1 or newer within PHP 8.x**. Although `composer.json` allows PHP 8.3, the development packages in `composer.lock` require at least PHP 8.4.1.
- Composer 2, Bun, and a current Node.js LTS release. Bun installs the frontend dependencies; Node.js is also needed by the installed Vite executable.
- Docker Desktop running Linux containers for MariaDB.
- An internet connection to download tools, packages and the database image.

## Installation in 11 steps

Run the commands in **PowerShell**. After installing a tool, close and reopen the terminal so it picks up the updated `PATH`. Do not type the Markdown backticks into the terminal.

### 1. Install VS Code, Git, PHP and Docker Desktop

If these tools are not installed, run:

```powershell
winget install --id Microsoft.VisualStudioCode --exact
winget install --id Git.Git --exact
winget install --id PHP.PHP.8.4 --exact
winget install --id Docker.DockerDesktop --exact
```

Complete the installers. Start Docker Desktop, complete its WSL 2 setup if prompted, and wait until the engine is running. Use Linux containers.

Install the current **LTS Windows Installer** from [the official Node.js download page](https://nodejs.org/en/download). Keep the option that adds Node.js to `PATH`.

In a new PowerShell window, check:

```powershell
git --version
code --version
php -v
node --version
npm --version
docker --version
docker compose version
```

Check that PHP is at least 8.4.1. If several PHP installations exist, use `Get-Command php` to see which executable PowerShell is using.

### 2. Install Bun

Use the Windows command from [Bun's installation guide](https://bun.sh/docs/installation):

```powershell
powershell -c "irm bun.sh/install.ps1 | iex"
```

Open a new terminal and check:

```powershell
bun --version
```

### 3. Enable PHP extensions

Find the configuration used by the command-line PHP:

```powershell
php --ini
Get-Command php
```

If `Loaded Configuration File` is `(none)`, create `php.ini` next to `php.exe` from the supplied development template:

```powershell
$phpDirectory = Split-Path (Get-Command php).Source
Copy-Item "$phpDirectory\php.ini-development" "$phpDirectory\php.ini"
code "$phpDirectory\php.ini"
```

Only copy the template when there is no existing configuration. Otherwise, open the **Loaded Configuration File** shown by `php --ini`.

In that file, remove the leading semicolon from these settings, or add each setting once if it is missing:

```ini
extension_dir = "ext"
extension=curl
extension=fileinfo
extension=mbstring
extension=openssl
extension=pdo_mysql
extension=zip
```

`pdo_mysql` connects PHP to MariaDB. `mbstring`, `openssl` and `fileinfo` support Laravel. `curl` and `zip` support package downloads and extraction. The PHP distribution also needs its standard XML, DOM, PDO, tokenizer and other core modules; Composer checks the complete package requirements in step 6.

Save the file and confirm that the configuration and extensions load without startup warnings:

```powershell
php --ini
php -m
```

If PHP cannot find an extension, set `extension_dir` to the absolute path of the `ext` folder next to your `php.exe`.

### 4. Install Composer

Download **Composer-Setup.exe** from [the official Composer download page](https://getcomposer.org/download/). Run the installer and select the same `php.exe` identified in step 3. Keep the option that makes Composer available on `PATH`.

Open a new terminal and check:

```powershell
composer --version
```

### 5. Clone the repository

Open a terminal in the folder where you keep your projects:

```powershell
git clone https://github.com/bigbossmcnugget/ta24bblog.git
cd ta24bblog
code .
```

Run all commands below from this `ta24bblog` folder, which contains `artisan`, `composer.json` and `docker-compose.yml`.

### 6. Install PHP dependencies

```powershell
composer install
composer check-platform-reqs
```

`composer install` restores the versions from `composer.lock` into `vendor`, including the development dependencies. Both commands must finish successfully. If Composer reports a missing extension or an incompatible PHP version, fix that before continuing. Do not use `--ignore-platform-reqs` to hide the problem.

### 7. Install frontend dependencies

```powershell
bun install --frozen-lockfile
```

This is `bun install` with a check that the committed `bun.lock` remains unchanged. It installs Vite, Tailwind CSS and the other JavaScript packages into `node_modules`.

### 8. Create the environment file

A clone does not contain the private `.env` file. Make a local copy of the example:

```powershell
Copy-Item .env.example .env
```

The example already matches `docker-compose.yml`:

```dotenv
APP_URL=http://localhost:8000
DB_CONNECTION=mariadb
DB_HOST=127.0.0.1
DB_PORT=33061
DB_DATABASE=blog
DB_USERNAME=root
DB_PASSWORD=example
```

Docker publishes the database's internal port `3306` as host port `33061`. Leave the other example values in place, including the database-backed sessions, cache and queue. These credentials are the repository's local development defaults. Keep `.env` out of Git.

### 9. Generate the application key

```powershell
php artisan key:generate
```

This writes a new `APP_KEY` into your local `.env`. Generate it once during initial setup; keep the same key for subsequent runs.

### 10. Start MariaDB and create the tables

With Docker Desktop running, start the database:

```powershell
docker compose up -d db
docker compose ps
docker compose logs --tail=30 db
```

On the first run, Docker downloads the MariaDB image, creates the `blog` database and initialises its named volume. Wait until the logs show the database is ready for connections. Initialisation can take longer than a few seconds.

Then create the application's tables:

```powershell
php artisan migrate
php artisan migrate:status
```

The users, cache and jobs migrations should show `DONE` during migration and `Ran` in the status output. No SQL dump or manual table creation is required for this repository. The Compose volume preserves the database when the container is stopped.

### 11. Start the application

```powershell
composer run dev
```

Leave the terminal open. This runs the PHP web server, database queue listener and Vite. On systems with PHP's `pcntl` extension, it also starts the Pail log viewer; native Windows automatically skips that viewer.

Open **[http://localhost:8000](http://localhost:8000)** in your browser. You should see the Laravel welcome page. Vite normally uses port `5173` for assets; its address is printed in the terminal and is not the main application URL.

Press **Ctrl+C** to stop the development processes. If PHP selects a different port because `8000` is occupied, use the application address printed by the server.

## Starting the project next time

Start Docker Desktop, open the project folder, and run:

```powershell
docker compose up -d db
composer run dev
```

When finished, press Ctrl+C and stop this project's database container:

```powershell
docker compose stop db
```

Do not use `docker compose down -v` for an ordinary restart: it deletes this project's database volume.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| A command is not found | Open a new terminal and check that the tool was added to `PATH`. |
| Composer rejects the PHP version | Run `php -v` and `Get-Command php`. This lock file requires PHP 8.4.1 or newer within PHP 8.x. |
| A PHP extension is missing | Use `php --ini`, edit that loaded configuration, then run `php -m` and `composer check-platform-reqs`. |
| `could not find driver` | Enable `pdo_mysql` in the PHP configuration used by the terminal. |
| Connection refused on migration | Start Docker Desktop and the `db` service; check its logs and `.env` port `33061`. |
| Database access denied | Check the username/password against Compose. Existing volumes keep the credentials set when they were first initialised. |
| Application encryption key is missing | Complete steps 8 and 9. |
| A session, cache or jobs table is missing | Run `php artisan migrate`; these features use the database in the example configuration. |
| Node.js is not found or Vite will not start | Install the current Node.js LTS release, reopen the terminal, and check `node --version`. |
| Assets or the Vite manifest are missing | Keep `composer run dev` running, or run `bun run build` to generate production assets. |
| An environment change is not picked up | Run `php artisan config:clear` and restart the development processes. |

## Validation

On 25 November 2026, a separate clean clone was checked on macOS with PHP 8.5.10, Composer 2.11.2, Bun 1.4.5 and Node.js 26.13.0. Dependency installation, platform requirements, frontend compilation, application key generation and all three migrations succeeded. `composer run dev` started the application and Vite; their HTTP responses were checked.

For this repeat check, MariaDB 13.0.2 ran as an isolated local server using the example's host, port, database and credentials. Port `18080` was used for the PHP server because another application occupied `8000`. The Windows installers and Docker Desktop steps were not executed during this macOS check.
