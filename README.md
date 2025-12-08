# Deploying Laravel on Sevalla using Docker or Nixpacks

- [Architecture](#architecture)
- [Steps](#steps)
  - [1. Prepare your repository](#1-prepare-your-repository)
  - [2. Create Sevalla resources](#2-create-sevalla-resources)
  - [3. Configure the Sevalla app](#3-configure-the-sevalla-app)
    - [A. Create a process to run DB migrations](#a-create-a-process-to-run-db-migrations)
    - [B. Allow internal connections between the app and database](#b-allow-internal-connections-between-the-app-and-database)
    - [C. Set environment variables](#c-set-environment-variables)
    - [D. Start the scheduler](#d-start-the-scheduler)
    - [E. Start your default queue worker or Laravel Horizon](#e-start-your-default-queue-worker-or-laravel-horizon)
    - [F. Choose the way you want to build your application](#f-choose-the-way-you-want-to-build-your-application)
  - [4. Deploy 🚀](#4-deploy-)

Sevalla works either with Docker or Nixpacks.

This repository includes a [Dockerfile](/Dockerfile) that packages a Laravel application and runs it using the `serversideup/php:8.5-frankenphp` base image.

You can also [choose Nixpacks](#f-choose-the-way-you-want-to-build-your-application) in just a few clicks.

## Architecture

On Sevalla, every app has a **default web process** that serves HTTP requests.

If you prefer to build using the Dockerfile:
- The app is built from the repository’s `Dockerfile` and the web process runs **FrankenPHP** (Caddy-based) from the serversideup image, serving `/public` and running PHP.

If you prefer to build using Nixpacks:
- The app is built from your repository's code. Nixpacks figures out the best way to build your project.

## Steps

### 1. Prepare your repository

Copy this repository’s `Dockerfile` into the **root** of your Laravel project. Or just clone this repository if you are starting from scratch.

### 2. Create Sevalla resources

1. [Create a **database**](https://app.sevalla.com/databases).

2. [Create a **new application**](https://app.sevalla.com/apps/new) and connect your repository (don't deploy it yet).

### 3. Configure the Sevalla app

#### A. Create a process to run DB migrations

1. Go to **App → Processes** and create a **Job** process.
2. Set the start command to:

   ```bash
   php artisan migrate --force
   ```

<img width="540" src="https://github.com/user-attachments/assets/7af80896-c431-4cd4-b5f0-5034b2c65d23" />

#### B. Allow internal connections between the app and database

1. Go to **App → Networking** and scroll to **Connected services**.
2. Click **Add connection**, select the database you created, and enable **Add environment variables to the application** in the modal.

#### C. Set environment variables

Set the following in **App → Environment variables**. Fill in any empty values for your setup.

**Notes:**
- Set `DB_CONNECTION` with the value matching the database you created in step **B**. E.g., `mysql` or `pgsql`.
- `DB_URL` is automatically added if you completed step **B**.
- **Set `APP_URL` and `ASSET_URL` to your Sevalla app URL (e.g., `https://your-app.sevalla.app` or your custom domain).**
- Ensure `APP_KEY` is set (e.g., via `php artisan key:generate`).
- In production, keep `APP_DEBUG` to `false`.

#### D. Start the scheduler

1. Go to **App → Processes → Create process → Background worker**.
2. Set the custom start command to `php artisan schedule:work`. (Or `php artisan horizon` if you're using [Laravel Horizon](https://laravel.com/docs/horizon).)

<img width="540" height="1152" src="https://github.com/user-attachments/assets/78224eac-66d0-4a49-b128-4087a31b37b5" />

#### E. Start your default queue worker or Laravel Horizon

1. Go to **App → Processes → Create process → Background worker**.
2. Set the custom start command to `php artisan queue:work`.

#### F. Choose the way you want to build your application

Go to **App → Settings → Build** and change **Build environment** to either **Dockerfile** or **Nixpacks**.

<img width="473" src="https://github.com/user-attachments/assets/b074529e-3f51-471d-aa89-9a585dda2e5a" />

### 4. Deploy 🚀

Trigger a new deployment from Sevalla. Once deployed, your Laravel app and Nginx will run inside the web process under supervisord.
