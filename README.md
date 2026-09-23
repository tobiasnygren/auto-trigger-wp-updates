# auto-trigger-wp-updates

> **This is an example repository.** It is a plain [Roots Bedrock](https://roots.io/bedrock/) WordPress project used to experiment with automatic dependency pull requests via Dependabot. It is not a production site.

## What it demonstrates

Dependabot checks the Composer dependencies daily and opens pull requests for:

- **WordPress core** (`roots/wordpress`) – minor and patch releases only; major versions (e.g. 7.x) are ignored.
- **Plugins and themes** (`wp-plugin/*`, `wp-theme/*`) – grouped into a single PR.
- **Other Composer packages** – one PR per package.

## How to add Dependabot to your own repo

### 1. Add the config file

Create `.github/dependabot.yml` on the default branch:

```yaml
version: 2
updates:
  - package-ecosystem: composer
    directory: "/"
    schedule:
      interval: daily
    open-pull-requests-limit: 10
    ignore:
      - dependency-name: "roots/wordpress"
        update-types: ["version-update:semver-major"]
    groups:
      wordpress-plugins:
        patterns: ["wp-plugin/*", "wp-theme/*"]
```

Dependabot only reads this file from the **default branch**, so nothing happens until it is merged.

### 2. Make sure Composer can resolve your dependencies

Dependabot resolves updates using the PHP version from your `composer.json`. If it's lower than what your locked packages need, every update check silently fails. Pin it explicitly:

```json
"require": {
  "php": ">=8.3"
},
"config": {
  "platform": {
    "php": "8.3"
  }
}
```

Then run `composer update --lock` and commit the updated `composer.lock`.

### 3. Custom Composer repositories

Public repositories declared in `composer.json` (like `repo.wp-packages.org` here) work without extra config. Only add a `registries` entry in `dependabot.yml` for **private** repositories – it requires credentials and fails validation without them.

### 4. Trigger and monitor

- Go to **Insights → Dependency graph → Dependabot** in the GitHub repo.
- Click **Recent update jobs → Check for updates** to run it manually.
- Use **view logs** on a job to debug. Look for `ERROR` lines such as "requirements could not be resolved".

## Gotchas we hit

| Problem | Fix |
|---|---|
| `registries/... did not match one or more of the required schemas` | Remove the `composer-repository` registry for public repos (it needs credentials). |
| `Dependabot couldn't find a <anything>.yml` | Remove the `github-actions` ecosystem until the repo has workflows. |
| Jobs succeed but "No PRs affected" | Check the logs – usually a PHP version mismatch (see step 2). |
| "Cooldown was not applied" warning | Harmless; the registry doesn't publish release dates. |
