Laravel Prometheus Exporter
===========================

![CI](https://github.com/renoki-co/laravel-exporter-contracts/workflows/CI/badge.svg?branch=master)
[![codecov](https://codecov.io/gh/renoki-co/laravel-exporter-contracts/branch/master/graph/badge.svg)](https://codecov.io/gh/renoki-co/laravel-exporter-contracts/branch/master)
[![StyleCI](https://github.styleci.io/repos/410802300/shield?branch=master)](https://github.styleci.io/repos/410802300)
[![Latest Stable Version](https://poser.pugx.org/renoki-co/laravel-exporter-contracts/v/stable)](https://packagist.org/packages/renoki-co/laravel-exporter-contracts)
[![Total Downloads](https://poser.pugx.org/renoki-co/laravel-exporter-contracts/downloads)](https://packagist.org/packages/renoki-co/laravel-exporter-contracts)
[![Monthly Downloads](https://poser.pugx.org/renoki-co/laravel-exporter-contracts/d/monthly)](https://packagist.org/packages/renoki-co/laravel-exporter-contracts)
[![License](https://poser.pugx.org/renoki-co/laravel-exporter-contracts/license)](https://packagist.org/packages/renoki-co/laravel-exporter-contracts)

Base contracts implementation for Prometheus exports in Laravel.

## 🤝 Supporting

If you are using one or more Renoki Co. open-source packages in your production apps, in presentation demos, hobby projects, school projects or so, spread some kind words about our work or sponsor our work via Patreon. 📦

You will sometimes get exclusive content on tips about Laravel, AWS or Kubernetes on Patreon and some early-access to projects or packages.

[<img src="https://c5.patreon.com/external/logo/become_a_patron_button.png" height="41" width="175" />](https://www.patreon.com/bePatron?u=10965171)

## 🚀 Installation

You can install the package via composer:

```bash
composer require renoki-co/laravel-exporter-contracts
```

Publish the config:

```bash
$ php artisan vendor:publish --provider="RenokiCo\LaravelExporter\LaravelExporterServiceProvider" --tag="config"
```

## 🙌 Usage

All you have to do is to create a `\RenokiCo\LaravelExporter\Metric` class that defines how the values will update on each Prometheus call to scrap, and the definition of the collector.

By default, metrics are available on the `/exporter/group/metrics` endpoint and you can point Prometheus towards it for scraping. (i.e. `http://localhost/exporter/group/metrics`)

You can choose one of the following classes to extend:

- `\RenokiCo\LaravelExporter\GaugeMetric` for gauges
- `\RenokiCo\LaravelExporter\CounterMetric` for counters

For example, you can define gauges for users:

```php
use RenokiCo\LaravelExporter\GaugeMetric;

class DatabaseUsers extends GaugeMetric
{
    /**
     * Get the metric help.
     *
     * @return string
     */
    protected function help(): string
    {
        return 'Get the total amount of users.';
    }

    /**
     * Perform the update call on the collector.
     * Optional, as some metrics can be modified somewhere else.
     *
     * @return void
     */
    public function update(): void
    {
        $this->set(User::count());
    }
}
```

In your `AppServiceProvider`'s `boot()` method, register your metric:

```php
use RenokiCo\LaravelExporter\Exporter;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Exporter::register(DatabaseUsers::class);
    }

    // ...
}
```

You don't need to set the values. When the Prometheus scraper will make the request for metrics, the gauge will automatically be set.

## Labelling

You may label data by setting default values (i.e. server name, static data, etc.) and on-update.

```php
class DatabaseRecords extends GaugeMetric
{
    /**
     * Define the default labels with their values.
     *
     * @return array
     */
    protected function defaultLabels(): array
    {
        return [
            'static_label' => 'static-value',
        ];
    }

    /**
     * Get the metric allowed labels.
     *
     * @return array
     */
    protected function allowedLabels(): array
    {
        return [
            'model',
            'static_label',
        ];
    }

    /**
     * Perform the update call on the collector.
     * Optional, as some metrics can be modified somewhere else.
     *
     * @return void
     */
    public function update(): void
    {
        $models = [
            User::class,
            Post::class,
            Team::class,
        ];

        foreach ($models as $model) {
            $this->set(
                value: $model::count(),
                labels: ['model' => $model,
            ]);
        }
    }
}
```

## Grouping

If you wish to have separate endpoints for different metrics, consider specifying it in the `$showsOnGroup` property:

```php
class CustomMetrics extends Metric
{
    /**
     * The group this metric gets shown into.
     *
     * @var string|null
     */
    public static $showsOnGroup = 'metrics-reloaded';

    // ...
}
```

Under the hood, Laravel Exporter registers a route that allows you to scrape any group:

```php
Route::get('/exporter/group/{group?}', ...);
```

To scrape this specifc metric (and other metrics that are associated with this group), the endpoint is `/exporter/group/metrics-reloaded` (i.e. `http://localhost/exporter/group/metrics-reloaded`).

## 🐛 Testing

``` bash
vendor/bin/phpunit
```

## 🤝 Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## 🔒  Security

If you discover any security related issues, please email alex@renoki.org instead of using the issue tracker.

## 🎉 Credits

- [Alex Renoki](https://github.com/rennokki)
- [All Contributors](../../contributors)
