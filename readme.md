# Location

[![Travis CI](https://img.shields.io/travis/stevebauman/location.svg?style=flat-square)](https://travis-ci.org/stevebauman/location)
[![Scrutinizer Code Quality](https://img.shields.io/scrutinizer/g/stevebauman/location.svg?style=flat-square)](https://scrutinizer-ci.com/g/stevebauman/location/?branch=master)
[![Latest Stable Version](https://img.shields.io/packagist/v/stevebauman/location.svg?style=flat-square)](https://packagist.org/packages/stevebauman/location)
[![Total Downloads](https://img.shields.io/packagist/dt/stevebauman/location.svg?style=flat-square)](https://packagist.org/packages/stevebauman/location)
[![License](https://img.shields.io/packagist/l/stevebauman/location.svg?style=flat-square)](https://packagist.org/packages/stevebauman/location)

Retrieve a users location from their IP address using external web services, or through a flat-file database hosted on your server.

## Requirements

- Laravel 5.*
- PHP 5.6 or greater
- cURL extension enabled

## Installation

Add Location to your `composer.json` file:

```json
"stevebauman/location": "3.0.*"
```

Then run `composer update` on your project source.

> **Note**: If you're using Laravel 5.5, you can skip the
> registration of the service provider and the facade,
> as they are registered automatically.

Add the service provider in `config/app.php`:

```php
Stevebauman\Location\LocationServiceProvider::class,
```

Add the alias in your `config/app.php` file:'
```php
'Location' => Stevebauman\Location\Facades\Location::class,
```

Publish the config file:

```bash
php artisan vendor:publish --provider="Stevebauman\Location\LocationServiceProvider"
```

## Usage

#### Retrieving a users location:

```php
$position = Location::get();

// Returns instance of Stevebauman\Location\Position
```

#### Retrieving a location with a specific IP address:

```php
$position = Location::get('192.168.1.1');
```

## Drivers

#### Available Drivers

Available drivers at the moment are:

- [IpInfo](https://ipinfo.io/) - Default
- [FreeGeoIp](https://freegeoip.net/)
- [GeoPlugin](http://www.geoplugin.com/)
- [MaxMind](https://www.maxmind.com/en/home)

#### Setting Up MaxMind

To setup MaxMind to retrieve the users location from your own server, download the database file here:

http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz

Extract the downloaded file, and place the GeoLite2-City.mmdb file
into your `database` directory, and create the folder `maxmind`.

You should end up with a folder path of: `my-laravel-app/database/maxmind/GeoLite2-City.mmdb`.

Then, set your default driver to `Stevebauman\Location\Drivers\MaxMind::class`, and you're all set!

> **Note**: Keep in mind, you'll need to update this file continuously to retrieve the most current information.

#### Fallback drivers

In the config file, you can specify as many fallback drivers as you wish. It's recommended to install
the MaxMind database `.mmdb` file so your always retrieving some generic location information from the user.

If an exception occurs trying to grab a driver (such as a 404 error if the
providers API changes), it will automatically use the next driver in line.

#### Creating your own drivers

To create your own driver, simply create a class in your application, and extend the abstract Driver:

```php
namespace App\Location\Drivers;

use Illuminate\Support\Fluent;
use Stevebauman\Location\Position;
use Stevebauman\Location\Drivers\Driver;

class MyDriver extends Driver
{
    public function url()
    {
        return '';
    }

    protected function hydrate(Position $position, Fluent $location)
    {
        $position->countryCode = $location->country_code;

        return $position;
    }

    protected function process($ip)
    {
        try {
            $response = json_decode(file_get_contents($this->url().$ip), true);

            return new Fluent($response);
        } catch (\Exception $e) {
            return false;
        }
    }
}
```

Then, insert your driver class name into the configuration file:

```php
/*
|--------------------------------------------------------------------------
| Driver
|--------------------------------------------------------------------------
|
| The default driver you would like to use for location retrieval.
|
*/

'driver' => App\Locations\MyDriver::class,
```
