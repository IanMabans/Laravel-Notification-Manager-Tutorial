# Laravel Notification Manager Tutorial

This repository contains a step-by-step guide for creating a notification manager in Laravel that supports sending emails and SMS messages.

---

## Table of Contents

- [Introduction](#introduction)
- [Step 1: Set Up the Configuration File](#step-1-set-up-the-configuration-file)
- [Step 2: Create the Notification Manager](#step-2-create-the-notification-manager)
- [Step 3: Implement the Drivers](#step-3-implement-the-drivers)
- [Step 4: Register the Manager in the Service Container](#step-4-register-the-manager-in-the-service-container)
- [Step 5: Test the System](#step-5-test-the-system)


---

## Introduction

In this tutorial, we’ll walk through creating a notification manager in Laravel that supports sending emails and SMS messages. This system uses **Managers** and **Factories**, which are powerful tools for managing different drivers (services) dynamically based on configuration.

---

## Step 1: Set Up the Configuration File

The first step is to define the configuration for your notification system. Create a new file called `config/notification.php`:

```php
return [
    'default_driver' => env('NOTIFICATION_DEFAULT_DRIVER', 'email'),

    'drivers' => [
        'email' => [
            'from_address' => env('MAIL_FROM_ADDRESS'),
            'to_address' => env('MAIL_TO_ADDRESS'),
        ],
        'sms' => [
            'sender_id' => env('SMS_SENDER_ID'),
            'api_token' => env('SMS_API_TOKEN'),
            'api_url' => env('SMS_API_URL'),
            'to_number' => env('SMS_TO_NUMBER'),
        ],
    ],
];
```
Add the following to your .env file (It can vary)
```
NOTIFICATION_DEFAULT_DRIVER=email
MAIL_FROM_ADDRESS=hello@example.com
MAIL_TO_ADDRESS=recipient@example.com
SMS_SENDER_ID=YourSenderId
SMS_API_TOKEN=YourApiToken
SMS_API_URL=https://api.yoursmsprovider.com/send
SMS_TO_NUMBER=+0987654321

```
## Step 2: Create the Notification Manager
The NotificationManager handles the creation and management of drivers (email, SMS, etc.) based on the configuration.

Create a new file at app/Notification/NotificationManager.php

```php
<?php

namespace App\Notification;

use Illuminate\Support\Manager;

class NotificationManager extends Manager
{
    public function createEmailDriver()
    {
        return new EmailNotifier();
    }

    public function createSmsDriver()
    {
        return new SmsNotifier();
    }

    public function getDefaultDriver()
    {
        return config('notification.default_driver');
    }
}
```
## Step 3: Implement the Drivers

Now, let's create the EmailNotifier and SmsNotifier classes.

EmailNotifier
This class sends emails using Laravel's Mail facade.

Create app/Notification/EmailNotifier.php:

```php
<?php

namespace App\Notification;

use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Mail;

class EmailNotifier
{
    protected $config;

    public function __construct()
    {
        $this->config = config('notification.drivers.email');
    }

    public function send($message)
    {
        try {
            Mail::send([], [], function ($mail) use ($message) {
                $mail->to($this->config['to_address'])
                    ->subject('Test Email')
                    ->setBody("<html><body>$message</body></html>", 'text/html');
            });

            Log::info("Email sent successfully: {$message}");
        } catch (\Exception $e) {
            Log::error('Error while sending email: ' . $e->getMessage());
        }
    }
}
```
SmsNotifier
This class sends SMS messages via an external API.

Create app/Notification/SmsNotifier.php:

```php
<?php

namespace App\Notification;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class SmsNotifier
{
    protected $config;

    public function __construct()
    {
        $this->config = config('notification.drivers.sms');
    }

    public function send($message)
    {
        try {
            $data = [
                'phone' => $this->config['to_number'],
                'senderid' => $this->config['sender_id'],
                'message' => $message,
            ];

            $headers = [
                'Authorization' => 'Bearer ' . $this->config['api_token'],
                'Content-Type' => 'application/json',
            ];

            $response = Http::withHeaders($headers)->post($this->config['api_url'], $data);

            if ($response->successful()) {
                Log::info('SMS sent successfully: ' . $message);
            } else {
                Log::error('Failed to send SMS: ' . $response->status() . ' - ' . $response->body());
            }
        } catch (\Exception $e) {
            Log::error('Error while sending SMS: ' . $e->getMessage());
        }
    }
}
```
## Step 4: Register the Manager in the Service Container
To make the NotificationManager available throughout your application, register it in the AppServiceProvider.

Open app/Providers/AppServiceProvider.php and update the register method:


```php
$this->app->singleton('notification', function ($app) {
    return new \App\Notification\NotificationManager($app);
});
```
## Test the System
Add test routes to verify that the notification system works.

Update routes/web.php:

```php
use App\Notification\NotificationManager;

Route::get('/send-email', function () {
    app('notification')->driver('email')->send('This is a test email!');
    return 'Email sent!';
});

Route::get('/send-sms', function () {
    app('notification')->driver('sms')->send('This is a test SMS!');
    return 'SMS sent!';
});
```
Access these routes in your browser:

/send-email
/send-sms
Check the logs (storage/logs/laravel.log) for confirmation.

