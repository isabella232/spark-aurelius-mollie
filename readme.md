# Laravel Spark, Mollie edition

[spark.laravel.com](https://spark.laravel.com) | [mollie.com](https://www.mollie.com)

## Support

If you'd like to have a chat, [join us on the dedicated Discord channel](https://discord.gg/tnTvNmS).

Bugs and feature requests will be tracked [here](https://github.com/laravel/spark-aurelius-mollie/issues) in the repository.
Feel free to open a ticket.

## Installation

Create a new Laravel project using the [Laravel installer](https://laravel.com/docs/installation):

    laravel new my-project

Next, add the following repository to your `composer.json` file:

    "repositories":[
        {
            "type": "vcs",
            "url": "git@github.com:laravel/spark-aurelius-mollie.git"
        }
    ]

You should also add the following dependency to your `composer.json` file's `require` section:

    "laravel/spark-aurelius-mollie": "^2.0"

*Note: Spark's installer will wire up everything for you, including Cashier and the VAT calculator. No need to follow their installation instructions.*

Next, run the `composer update` command. You may be prompted for a GitHub token in order to install the private Spark
repository. Composer will provide a link where you can create this token.

Once the dependencies are installed, add the following service providers to your `app.php` configuration file:

    Laravel\Spark\Providers\SparkServiceProvider::class,
    Laravel\Cashier\CashierServiceProvider::class,

Next, run the install command:

    php artisan spark:install --force --mollie

or for team billing:

    php artisan spark:install --force --mollie --team-billing

Set the `MOLLIE_KEY` and database settings in the `.env` file.
You can obtain test and live keys in the [Mollie dashboard](http://mollie.com/dashboard).

Once Spark is installed, add the following provider to your `app.php` configuration file:

    App\Providers\SparkServiceProvider::class,

Finally, you are ready to run the `npm install`, `npm run dev`, and `php artisan migrate` commands.
Once these commands have completed, you are ready to enjoy Spark!

### Linking The Storage Directory

Once Spark is installed, you should link the `public/storage` directory to your `storage/app/public` directory.
Otherwise, user profile photos stored on the local disk will not be available:

    php artisan storage:link
    
### Schedule Cashier

Schedule a periodic job in `\App\Console\Kernel` to execute `Cashier::run()`.

```php
    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('cashier:run')
            ->daily() // run as often as you like (Daily, monthly, every minute, ...)
            ->withoutOverlapping(); // make sure to include this
    }
```

## Configuring billing plans

1. Set up your subscription plans as described in the
[laravel/cashier-mollie documentation](https://github.com/laravel/cashier-mollie).
2. Then configure the SparkServiceProvider as described in the
[Spark documentation](https://spark.laravel.com/docs/9.0/billing).

## Upgrading to Spark Mollie v2

Spark Mollie v2 provides compatibility with Laravel 8.

Upgrading from v1 only takes five minutes.

Start with updating `composer.json` to use Spark Mollie v2:
    
    "spark-aurelius-mollie": "^2.0
    
Then, move the Team and User models into the `App\Models` namespace.
Alternatively, if you would like to keep your model classes in the `App` namespace, you may use the `useUserModel` and
`useTeamModel` methods in the register method of your `SparkServiceProvider`:

```php
Spark::useUserModel('App\User');

Spark::useTeamModel('App\Team');
```

Finally, in `App\Providers\SparkServiceProvider`, rename the `booted` method into `boot` and call the parent method:

```php
    public function boot()
    {
        parent::boot(); // Don't forget to call the parent boot method

        Spark::useMollie()
            ->trialDays(10)
            ->defaultBillableCountry('NL')
            ->collectEuropeanVat('NL');

        Spark::freePlan()
            ->features([
                'First', 'Second', 'Third'
            ]);

        Spark::plan('Basic', 'example-1')
            ->price(10)
            ->features([
                'First', 'Second', 'Third'
            ]);
    }
```

## Local testing

You can use `valet share` (a ngrok wrapper) to make your local setup reachable for Mollie's webhook calls.

Make sure to use the ngrok generated url both in your `.env` file (`APP_URL`) and in your browser.

## Note on mandated payments in test mode

A mandate for subsequent payments is obtained from the first payment (where the customer proceeds through Mollie's checkout).

The subsequent mandated payments are triggered by Spark. Once the payment has status paid, Spark's webhook will be called and an invoice will become available. Depending on how long the clearance takes, this takes anywhere between 1 second and 48 hours.

**In production**, Spark's webhook will be called once the payment has been paid (or has failed).

**In test mode** this is not the case. The payment status will not get updated automatically. You can do so manually in your browser using:

```php
$url = mollie()->payments()->get("the_payment_id_here")->_links->changePaymentState->href;
```

More information is available in the [Mollie docs](https://docs.mollie.com/guides/testing).

## Running on Laravel Vapor

Running on Vapor requires you to override:

1. the profile picture upload features (for both teams and users)
1. the invoice pdf generation and download

Here's an easy package to get you started: [sandervanhooft/vaporize-spark-mollie](https://github.com/sandervanhooft/vaporize-spark-mollie).
