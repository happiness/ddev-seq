# DDEV Seq add-on

This is a DDEV add-on for [Seq](https://datalust.co/seq), a self-hosted search,
analysis, and alerting server built for structured logs and traces.

## Add-on installation

```
ddev add-on get https://github.com/happiness/ddev-seq
```

## DDEV customizations

You need to install the PHP OpenTelemetry package. Edit the `./ddev/config.yml`
file in your project and add the following lines:

```
webimage_extra_packages: ["php${DDEV_PHP_VERSION}-opentelemetry"]
```

Then restart DDEV by executing `ddev restart`.

## Drupal integration

Install the modules [OpenTelemetry](https://www.drupal.org/project/opentelemetry) and
[OpenTelemetry Log](https://www.drupal.org/project/otlog). The *OpenTelemetry* module
comes with it's own log implementation but I have not managed to get that working
properly so for now I recommend using the *OpenTelemetry Log* module instead.

```
composer require drupal/opentelemetry drupal/otlog
```

Configure the *OpenTelemetry* module at `/admin/config/development/opentelemetry`
and add the **Endpoint** `http://seq/ingest/otlp`.

That's it. Open `http://seq.<DDEV_NAME>.ddev.site:8501/` to start exploring
your logs and traces.