# Laravel CI/CD Pipeline with Jenkins & Docker

A ready-to-run, fully containerized CI/CD environment for Laravel applications. One `docker-compose up` gives you a Jenkins server, a MySQL database, phpMyAdmin, and a PHP 8.2 + Nginx application container, wired together with a Jenkins pipeline that installs dependencies and runs the Laravel test suite on every build.

## What's Inside

| Service | Image | Port | Purpose |
|---|---|---|---|
| **Jenkins** | `jenkins/jenkins:lts` | 8080, 50000 | CI server running the pipeline |
| **Laravel App** | `webdevops/php-nginx:8.2` | 8000 | Application runtime (PHP 8.2 + Nginx) |
| **MySQL** | `mysql:8.0` | 3306 | Application database |
| **phpMyAdmin** | `phpmyadmin/phpmyadmin` | 8081 | Database administration UI |
| **Composer** | `composer:latest` | - | Dependency installation (on-demand) |

## Pipeline Stages

The included `Jenkinsfile` defines a declarative pipeline:

1. **Clone Repository** - pulls the project source
2. **Install Dependencies** - `composer install` inside a dedicated container
3. **Run Tests** - executes `php artisan test` (PHPUnit) in the app container

Build status is reported on success/failure via post actions. The pipeline runs entirely inside Docker, so Jenkins agents need nothing preinstalled except Docker itself.

## Quick Start

```bash
git clone https://github.com/swizzen1/jenkins
cd jenkins

# Start the full stack
docker-compose up -d

# Jenkins UI
open http://localhost:8080

# Application
open http://localhost:8000

# phpMyAdmin
open http://localhost:8081
```

### First-time Jenkins setup

1. Grab the initial admin password:
   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```
2. Open `http://localhost:8080`, paste the password, and install the suggested plugins.
3. Create a new **Pipeline** job and point it at this repository - the `Jenkinsfile` is picked up automatically.

### Environment

Copy the environment file and generate the app key inside the container:

```bash
docker-compose run laravel cp .env.example .env
docker-compose run laravel php artisan key:generate
docker-compose run laravel php artisan migrate
```

Database credentials (matching `docker-compose.yml`):

```env
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=laravel_password
```

## Why This Setup

- **Reproducible builds**: every pipeline stage runs in a clean container, no "works on my machine"
- **Zero host dependencies**: PHP, Composer, and MySQL versions are pinned in `docker-compose.yml`
- **Easy to extend**: add stages (static analysis, code style, deployment) by editing the `Jenkinsfile`

## Extending the Pipeline

Typical additions for a production pipeline:

```groovy
stage('Static Analysis') {
    steps {
        sh 'docker-compose run laravel ./vendor/bin/phpstan analyse'
    }
}
stage('Code Style') {
    steps {
        sh 'docker-compose run laravel ./vendor/bin/pint --test'
    }
}
stage('Deploy') {
    when { branch 'main' }
    steps {
        // rsync / Envoyer / kubectl rollout, etc.
    }
}
```

## Requirements

- Docker & Docker Compose
- 2 GB+ free RAM (Jenkins + MySQL)

## License

Open-sourced under the [MIT license](https://opensource.org/licenses/MIT).
