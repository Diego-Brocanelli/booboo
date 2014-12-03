# Shit Happens

## Purpose
An error handler for PHP that allows for the execution of handlers and formatters for viewing and managing errors in
development and production. Because "shit happens."

## Installation

This library requires PHP 5.4 or later.

It is recommended that you install this library using Composer.

```
$ composer require brandonsavage/shithappens
```

## Getting Started

### Instantiation

The main object that you need to instantiate is Savage\ShitHappens\Runner. This object takes care of setting the error
handler, as well as handling errors and exceptions. It takes optional arguments during construction for handlers and
formatters.

```php
<?php

$runner = new Savage\ShitHappens\Runner();
$runner->register(); // Registers the handlers
```

It's very important to call Runner::register() or the object won't register itself as PHP's error handler.

### Formatters are very important!

When you're developing, you want to view errors in the browser. In order to do this, you must provide a formatter.
Without a formatter, the system won't intelligently know how to display the errors. **If you do not set a formatter,
errors won't be formatted at all!**

The library ships with four formatters for your convenience:

* HtmlFormatter - Formats errors just like PHP's error formatting.
* JsonFormatter - Perfect for displaying errors to an API.
* CommandLineFormatter - Working with the command line? This will produce pretty command-line errors.
* NullFormatter - This formatter simply silences all errors.

Adding a formatter is easy:

```php
<?php

$runner->pushFormatter(new Savage\ShitHappens\Formatter\HtmlFormatter);
```

### Controlling which formater does the formatting

There may be times that you want certain formatters to handle the formatting for particular errors, and others to handle
the formatting for other error types. Formatters support this.

For example, if you want all errors of warning or higher to show in the browser, but errors that are below this level
to be ignored, you can configure the formatters to handle this scenario as such:

```php
<?php

$html = new Savage\ShitHappens\Formatter\HtmlFormatter;
$null = new Savage\ShitHappens\Formatter\NullFormatter;

$html->setErrorLimit(E_ERROR | E_WARNING | E_USER_ERROR | E_USER_WARNING);
$null->setErrorLimit(E_ALL);

$runner->pushFormatter($null);
$runner->pushFormatter($html);
```

### Formatters and handlers are a stack

Formatters and handlers are treated as a stack. This means that the last item in will be the first item out. This is
very important when dealing with formatters that only handle certain errors!

For example, in the example above, we have one formatter limited to errors and warnings, and the other formatting all
error types. If we insert the HTML handler first, it will be run last; this would cause the NullFormatter to format all
errors, and we would get no output.

### Handlers

Regardless of whether or not you want to format the error (or even output it to the screen), you may want to handle it
in some way, such as logging it. ShitHappens provides a way to handle errors, and provides a built-in PSR-3 compatible
logging handler.

You can implement the HandlerInterface to create your own handlers. Handlers are run regardless of whether or not
display_errors is true. Unlike formatters, you cannot direct handlers to ignore certain errors; it's assumed that you
can handle this with the services that handlers work through.