# CLAUDE.md

This file provides guidance to Claude Code when working with the `hanafalah/laravel-stub` package.

## Overview

Laravel Stub is a code generation and stub templating system for Laravel applications. It provides a fluent interface for loading stub template files, performing variable replacements, and generating code files. The package is designed to streamline code scaffolding by allowing developers to define reusable templates with placeholder variables.

**Namespace:** `Hanafalah\LaravelStub`

**Dependency:** Requires `hanafalah/laravel-support` package.

## Package Structure

```
src/
  Commands/
    InstallMakeCommand.php    # Artisan command for package installation
  Contracts/
    Stub.php                  # Interface contract for stub operations
  Facades/
    Stub.php                  # Laravel facade for static access
  Providers/
    CommandServiceProvider.php # Registers artisan commands
  LaravelStub.php             # Main class with stub processing logic
  LaravelStubServiceProvider.php # Service provider registration
  helper.php                  # Global helper functions
assets/
  config/
    config.php                # Publishable configuration file
```

## Key Classes

### LaravelStub (Main Class)

The core class that handles stub file loading, variable replacement, and file generation.

**Key Properties:**
- `$path` - Path to the stub template file
- `$replaces` - Array of placeholder-to-value replacements
- `$__base_path` - Static base path for stub files (defaults to `stubs/` directory)

**Key Methods:**
- `init(string $path, array|callable $replaces)` - Create new instance with path and replacements
- `setPath(string $path)` - Set the stub template file path
- `setRepalcements(array $replaces)` - Set variable replacements
- `replace(array $replaces)` - Alias for setting replacements
- `getContents()` - Load stub file and perform all replacements
- `render()` - Alias for getContents()
- `saveTo(string $path, string $filename)` - Save generated content to a file
- `formatArrayAsStub(array $array, int $indentLevel)` - Format PHP array for stub output

### Contracts\Stub (Interface)

Defines the contract for stub implementations:
- `__construct($path, array $replaces)`
- `init($path, array $replaces)`

### Facades\Stub

Laravel facade providing static access to LaravelStub methods:
```php
use Hanafalah\LaravelStub\Facades\Stub;

Stub::init('/path/to/template.stub', ['VAR' => 'value']);
```

### LaravelStubServiceProvider

Registers the package with Laravel:
- Binds `LaravelStub` as the main class
- Binds `Contracts\Stub` interface to `LaravelStub` implementation
- Registers command service provider

## Configuration

Published to `config/laravel-stub.php` via `php artisan stub:install`.

**Key Configuration Options:**
```php
return [
    'libs' => [
        'model' => 'Models',
        'contract' => 'Contracts',
        'schema' => 'Schemas',
        'database' => 'Database',
        'data' => 'Data',
        'resource' => 'Resources',
        'migration' => '../assets/database/migrations'
    ],
    'stub' => [
        'open_separator'  => '{{',    // Opening delimiter for placeholders
        'close_separator' => '}}',    // Closing delimiter for placeholders
        'path'            => base_path('stubs'),  // Default stub directory
    ],
    'commands' => [
        InstallMakeCommand::class
    ],
];
```

## Helper Functions

Defined in `src/helper.php` and autoloaded:

- `stub_path($path = '')` - Returns full path to stubs directory with optional suffix
- `to_studly($name)` - Converts string to StudlyCase using Laravel's Str::studly()

## Usage Patterns

### Basic Stub Processing

```php
use Hanafalah\LaravelStub\LaravelStub;

// Initialize with path and replacements
$stub = (new LaravelStub)->init('/templates/model.stub', [
    'NAMESPACE' => 'App\\Models',
    'CLASS_NAME' => 'User',
    'TABLE' => 'users'
]);

// Get processed content
$content = $stub->render();

// Or save to file
$stub->saveTo(app_path('Models/'), 'User.php');
```

### Using the Facade

```php
use Hanafalah\LaravelStub\Facades\Stub;

$content = Stub::init('/templates/controller.stub', [
    'NAMESPACE' => 'App\\Http\\Controllers',
    'CLASS_NAME' => 'UserController'
])->render();
```

### Stub Template Format

Stub files use configurable delimiters (default `{{` and `}}`) for placeholders:

```php
<?php

namespace {{NAMESPACE}};

class {{CLASS_NAME}}
{
    protected $table = '{{TABLE}}';
}
```

### Nested Replacements

The replacement system supports nested arrays that recursively process:

```php
$stub->replace([
    'CLASS_NAME' => 'MyClass',
    'METHODS' => [
        'METHOD_1' => 'public function one() {}',
        'METHOD_2' => 'public function two() {}'
    ]
]);
```

### Callable Replacements

Replacement values can be callables for dynamic content:

```php
$stub->replace([
    'TIMESTAMP' => fn() => date('Y-m-d H:i:s'),
    'UUID' => fn() => \Illuminate\Support\Str::uuid()
]);
```

### Array Formatting

The `formatArrayAsStub()` method converts PHP arrays to formatted stub-friendly strings:

```php
$stub->formatArrayAsStub(['id', 'name', 'email'], 2);
// Output:
// [
//     'id',
//     'name',
//     'email',
// ]
```

## Artisan Commands

### stub:install

Publishes the package configuration file:

```bash
php artisan stub:install
```

This creates `config/laravel-stub.php` with default settings.

## Integration Notes

- The package auto-registers via Laravel's package discovery (defined in `composer.json` extra.laravel.providers)
- Uses `BaseServiceProvider` from `laravel-support` for registration patterns
- Default stub path is `{base_path}/stubs/` - create this directory to store templates
- String conversion to output is supported via `__toString()` magic method
