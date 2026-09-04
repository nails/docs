---
description: The Logger service is responsible for writing to log files.
---

# Logging

Logs in Nails are stored at `application/logs`. By default Nails is fairly quiet when it comes to logging, allowing you to write meaningful messages relevant to your application's needs.

## Loading the logger

The Nails Logger can be loaded using the [Factory](../key-concepts/factory/):

```php
use Nails\Common\Factory;
use Nails\Common\Service\Logger;

/** @var Logger $oLogger **/
$oLogger = Factory::service('Logger');
```

## Writing to the log

Write lines to the logs using the logger's `line($sLine = '')` method. This will write a new timestamped line to the end of the log file.

```php
$oLogger->line('This will be written to the log');
$oLogger->line('Another line to write');
```

Log files have the date in the title, so logs naturally distribute across days, making it easier to find the logs you need and for log rotation.

## Alternative logs

The Logger service is dedicated to writing to the general app log. If you need to write to another log file then you can create a new instance of the Logger factory and customise it to your needs (internally, the Logger service does this).

```php
use Nails\Common\Factory\Logger;

/** @var \DateTime **/
$oNow = Factory::factory('DateTime');
/** @var Logger **/
$oLogger = Factory::factory('Logger');

// Set a custom file
$oLogger->setFile('custom-' . $oNow->format('Y-m-d') . '.php');

// Set a custom log directory
$oLogger->setDir('/path/to/dir/');

// Write to the log, which now exists at
// /path/to/dir/custom-0000-00-00.php
$oLogger->line('Hello world');
```

{% hint style="warning" %}
When using custom files it is your responsibility to title the logs, including any timestamp and file extension.
{% endhint %}
