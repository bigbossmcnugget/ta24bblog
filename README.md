# ta24bblog

A blog built with [Laravel](https://laravel.com) (PHP), Vite and Tailwind CSS, with a MariaDB database that runs in Docker.
This README is a complete install guide: from an empty Windows computer to the site running in the browser.

Author: Jegor Onoprijuk (TA-24B). The guide was tested step by step on a clean clone (see "Tested" at the bottom).

## Install guide

### 1. Clone the project

```bash
git clone https://github.com/bigbossmcnugget/ta24bblog.git
cd ta24bblog
```

You need Git for this. If `git` is not installed yet, do step 2 first and then come back here.

### 2. Install the tools

Open **PowerShell** and install VS Code, Git, PHP and Docker (the database runs in Docker):

```powershell
winget install Microsoft.VisualStudioCode Git.Git PHP.PHP.8.4 Docker.DockerDesktop
```

Close PowerShell and open it again so the new programs are found. Check the versions:

```powershell
git --version
php -v
docker --version
```

PHP must be version 8.3 or newer (the project requires `^8.3`). Start Docker Desktop once and wait until it says it is running.

### 3. Install Bun

Run the install command from the [Bun home page](https://bun.sh):

```powershell
powershell -c "irm bun.sh/install.ps1 | iex"
```

Open a new terminal and check it:

```powershell
bun --version
```

### 4. Install Composer

Download and run the installer from [getcomposer.org/download](https://getcomposer.org/download/) (the `Composer-Setup.exe` link). When it asks for the PHP location, choose the PHP you installed in step 2. Open a new terminal and check it:

```powershell
composer --version
```

### 5. Enable the PHP extensions in `php.ini`

The PHP from winget has no `php.ini` yet. Find the PHP folder with `php --ini` (or look in `C:\Users\<you>\AppData\Local\Microsoft\WinGet\Packages\PHP.PHP.8.4_...`), copy `php.ini-development` to `php.ini`, open `php.ini` in VS Code and remove the `;` at the start of these lines (use Ctrl+F to find them):

```ini
extension_dir = "ext"
extension=curl
extension=fileinfo
extension=mbstring
extension=openssl
extension=pdo_mysql
extension=zip
```

What they are for: `mbstring`, `openssl`, `fileinfo` are required by Laravel, `pdo_mysql` connects to the MariaDB database, `curl` and `zip` let Composer download packages. Check that they are loaded:

```powershell
php -m
```

### 6. `composer install`

In the project folder:

```bash
composer install
```

This downloads every PHP package that the project already lists in `composer.lock` into the `vendor` folder. (`install` restores what is listed, `require` adds something new.)

### 7. `bun install`

```bash
bun install
```

This downloads the JavaScript packages (Vite, Tailwind CSS and so on) into the `node_modules` folder.

### 8. Create the `.env` file

The `.env` file is not in git, so a fresh clone does not have it. Create it from the example:

```bash
cp .env.example .env
```

On Windows PowerShell `cp` works too (it is an alias of `Copy-Item`). You can also copy the file in VS Code and rename the copy to `.env`.

The default values in `.env.example` already match the Docker database (database `blog`, user `root`, password `example`, host `127.0.0.1`, port `33061`), so nothing needs to be edited.

### 9. Generate the application key

A fresh `.env` has an empty `APP_KEY` and the app will not start without it:

```bash
php artisan key:generate
```

### 10. Start the database and run the migrations

Start the MariaDB container from `docker-compose.yml` (Docker Desktop must be running) and wait about 10 seconds for it to start:

```bash
docker compose up -d
```

Then create the database tables:

```bash
php artisan migrate
```

You should see `DONE` for the users, cache and jobs migrations. If `migrate` says it cannot connect, wait a few more seconds and run it again.

### 11. Run the project

```bash
composer run dev
```

This single command starts everything at once: the PHP server, Vite, the queue worker and the log viewer. Open [http://localhost:8000](http://localhost:8000) in the browser. Stop it with `Ctrl+C`.

## Next time

After the first setup you only need:

```bash
docker compose up -d
composer run dev
```

and `docker compose stop` when you are done.

## Troubleshooting

| Problem | Fix |
|---|---|
| `git`, `php`, `composer` or `bun` is not found | Close the terminal and open a new one, so that the new `PATH` is loaded. |
| `composer install` says an extension is missing | Check step 5 and run `php -m`. |
| `migrate` cannot connect to the database | Check that Docker Desktop is running, `docker compose ps` shows the `db` container as Up, and wait a few seconds. |
| `No application encryption key has been specified` | Run step 9 (`php artisan key:generate`). |
| Port `8000` or `5173` is already in use | Stop the other program that uses it, then run `composer run dev` again. |

## Tested

All steps above were run from a fresh `git clone` with PHP 8.5, Composer 2.11 and Bun 1.4 on macOS: `composer install`, `bun install`, `.env` and `php artisan key:generate`, `docker compose up -d`, `php artisan migrate` (all 3 migrations `DONE`), and `composer run dev` (the site answers at `http://localhost:8000`, Vite at `http://localhost:5173`). The Windows-specific commands (`winget`, `php.ini` and the Composer installer) are the Windows equivalents of the same steps.
