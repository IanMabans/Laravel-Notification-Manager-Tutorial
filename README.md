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
- [Best Practices](#best-practices)
- [Conclusion](#conclusion)

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
