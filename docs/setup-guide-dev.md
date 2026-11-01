# Development Environment Setup Guide

## Requirements

On Windows host machine, install the following:

- WSL 2 (Ubuntu)

- Docker Engine

- Docker Desktop

- Git

- Git Desktop

## WSL 2 Installation

To install WSL 2 on Windows, refer to Microsoft's [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install) documentation.

## Git Setup on WSL 2 Ubuntu

Open Ubuntu and run the command below to check if `git` is installed.

```
git --version
```

If `git` is not installed yet, refer to Microsoft's [Get started using Git on Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-git) documentation.

Once installed, set `user.name` and `user.email` matching your GitHub account.

```
git config --global user.name "<username>"
git config --global user.eemail "<email>"
```

## Docker Desktop Configuration

### Enable integration with WSL 2 distro.

Open docker desktop. Go to Settings > WSL Integration.

Enable the `Enable integration with my default WSL distro` option.

Then, select `Ubuntu` on `Enable integration with additional distros`.

## Project Setup

### Creating Project Structure

Open Ubuntu on Start Menu.

Create a folder on `/home/<user_name>` named `dcs` and navigate inside it.

```
mkdir dcs && cd dcs
```

Clone the repositories inside the `dcs` folder.

```
git clone https://github.com/Roml04/document-control-system.git
git clone https://github.com/Roml04/document-control-system-frontend.git
git clone https://github.com/Roml04/document-control-system-backend.git
```

The `dcs` folder should now contain all the repositories that looks like the following:

```
/dcs
├─ /document-control-system             # contains the files necessary for system orchestration
├─ /document-control-system-frontend    # contains the frontend's source code
├─ /document-control-system-backend     # contains the backend's source code
```

### Setting Up Environment Variables

On `/dcs/document-control-system`, create a copy of `.env.example` on the same directory and rename it to `.env`.

Inside `.env` uncomment the following variables.

| Variable                | Meaning                                                                   |
| ----------------------- | ------------------------------------------------------------------------- |
| `FRONTEND_PATH`         | The relative path to the frontend's source code                           |
| `BACKEND_PATH`          | The relative path to the backend's source code                            |
| `COMPOSE_FILE`          | Defines the production and development compose files to merge.            |
| `COMPOSE_PROJECT_NAME`  | The name used by Docker Compose to group and identify project resources.  |
| `USER_ID`               | The user ID on the WSL 2 Ubuntu user.                                     |
| `GROUP_ID`              | The group ID on the WSL 2 Ubuntu user.                                    |

> To avoid file permission issues, set `USER_ID` and `GROUP_ID` to match the user ID and group ID of your WSL 2 Ubuntu user on the host machine.

| Variable                        | Meaning                                                                                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ONLYOFFICE_JWT_SECRET`         | The secret key used to sign and verify JSON Web Tokens (JWT) for authenticating requests between the application and ONLYOFFICE Document Server.                           |

> This will be generated on the next step of the application setup process. Leave this empty for now.

On Ubuntu, run the command below to find out the `USER_ID` and `GROUP_ID` of the Ubuntu user.

```
id -u && id -g
```

On `/dcs/document-control-system-backend`, create a copy of `.env.example` and rename it to `.env.dev`.

| Variable  | Meaning                                                                                                           |
| --------- | ----------------------------------------------------------------------------------------------------------------- |
| `APP_KEY` | The Laravel application key used for encrypting data, securing sessions, and other encryption-related operations. |

> This will be generated later on the application setup process. Leave this empty for now.

| Variable            | Meaning                                                    |
| ------------------- | ---------------------------------------------------------- |
| `MAIL_FROM_ADDRESS` | A custom email address the mail notification is sent from. |

> Enclose the value in double quotation marks (`""`) to ensure values containing special characters are interpreted correctly.

| Variable         | Meaning                                                                                                                    |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `RESEND_API_KEY` | The API key generated by Resend, used to authenticate and authorize the application to send emails through the Resend API. |

> API keys are secret tokens used to authenticate your requests. They are unique to your account and must be kept confidential.

To create an API key, refer to [Resend's API Key Creation Guide.](https://resend.com/docs/create-an-api-key)

### Installing Project Dependencies

To install project dependencies, run the following commands:

```
docker compose build
docker compose run --rm frontend npm ci
docker compose up -d mysql backend
docker compose exec backend composer install
```

After composer has finished installing dependencies, generate the `APP_KEY`.

```
docker compose exec backend php artisan key:generate --show
```

Copy the value and paste it on `APP_KEY` in `document-control-system-backend/.env.dev`.

Then, run the following command to generate a JWT secret:

```
docker compose exec backend php -r "echo base64_encode(random_bytes(64)), PHP_EOL;"
```

This will return a 64 cryptographically secure random bytes which can be used on the `ONLYOFFICE_JWT_SECRET` variable in `document-control-system/.env`.

Recreate the containers to apply the changes on both `.env` and `.env.dev` files.

```
docker compose up -d
```

### Database Migration and Opening the App

Once the containers are running, create the database tables:

```
docker compose exec backend php artisan migrate
```

Open the app in your browser at [http://localhost:5173](http://localhost:5173).

Setup is complete. From now on, `docker compose up -d` is all you need to start the environment.
