# DDEV Seq add-on

This is a DDEV add-on for [Seq](https://datalust.co/seq), a self-hosted search,
analysis, and alerting server built for structured logs and traces.

## Add-on installation

```
ddev add-on get https://github.com/happiness/ddev-seq
```

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

## Tracing

The following example of how to do your own tracing in your custom code is
taken from the [OpenTelemetry](https://www.drupal.org/project/opentelemetry)
module page.

```php
$this->openTelemetryTracer = \Drupal::service('opentelemetry.tracer');
$tracer = $this->openTelemetryTracer->getTracer();
$mainSpan = $tracer->spanBuilder('My custom operation')->startSpan();

// Make an external API call.
$apiCallSpan = $tracer->spanBuilder("Coindesk API call")->startSpan();
$data = json_decode(
  file_get_contents('https://api.coindesk.com/v1/bpi/currentprice.json')
);
$apiCallSpan->end();

// Set attributes and put an event to the main span.
$bots = 3;
$mainSpan->setAttribute('Bitcoin value', $data->bpi->USD->rate_float);
$mainSpan->addEvent('Starting bots tuning', ['bots available' => $bots]);

for ($i = 1; $i <= $bots; $i++) {
  $innerSpan = $tracer->spanBuilder("Tuning bot $i")->startSpan();
  // Do some internal business logic.
  usleep(rand(200000, 500000));
  $raised[$i] = rand(0, 100);
  $innerSpan->addEvent("Bot $i raised money!", ['amount' => $raised[$i]]);
  usleep(rand(100000, 200000));
  $innerSpan->end();
}

// Do some more stuff and finalize the main span.
usleep(rand(200000, 500000));
$mainSpan->addEvent('We got richer!', ['raised' => array_sum($raised)]);
$mainSpan->setAttribute('raised_details', $raised);
$mainSpan->end();

```