Nette SafeStream: Atomic Operations
===================================

[![Downloads this Month](https://img.shields.io/packagist/dm/nette/safe-stream.svg)](https://packagist.org/packages/nette/safe-stream)
[![Build Status](https://travis-ci.org/nette/safe-stream.svg?branch=master)](https://travis-ci.org/nette/safe-stream)
[![Coverage Status](https://coveralls.io/repos/github/nette/safe-stream/badge.svg?branch=master)](https://coveralls.io/github/nette/safe-stream?branch=master)
[![Latest Stable Version](https://poser.pugx.org/nette/safe-stream/v/stable)](https://github.com/nette/safe-stream/releases)
[![License](https://img.shields.io/badge/license-New%20BSD-blue.svg)](https://github.com/nette/safe-stream/blob/master/license.md)


Introduction
------------

The Nette SafeStram protocol for file manipulation guarantees atomicity and isolation of every file operation. 

Documentation can be found on the [website](https://doc.nette.org/safestream).

Why is it actually good? Let's start with a simple example, where we repeatedly write the same string to the file and then read it:

```php
$s = str_repeat('Long String', 10000);

$counter = 1000;
while ($counter--) {
	file_put_contents('file', $s);       // write it
	$read = file_get_contents('file');   // read it
	if ($s !== $read) {                    // check it
		echo 'Strings are different!';
	}
}
```

It may seem that the `echo 'Strings are different!'` command can't ever get executed. The opposite is true. Try to run this script in
two browsers simultaneously. The error occurs almost immediately.

It's because the code is not safe when performed repeatedly at the same time (ie, in multiple threads). And that is nothing unusual
on the Internet, where several people often connect to one website at the same time. Therefore, it's very important to ensure
that your application can handle multiple threads at once - that it's *thread-safe*, because native PHP functions are not.
Otherwise, you can expect data loss and strange errors occuring.

How to ensure, that functions like file_get_contets or `fwrite` behave atomically? The SafeStream protocol offers a secure solution,
so we can atomically manipulate files through standard PHP functions. To register this protocol install SafeStream via Composer.
After that, you just need prefix the filename with `nette.safe://`:

```php
$handle = fopen('nette.safe://test.txt', 'x'); // prefix the filename with nette.safe://

fwrite($handle, 'Nette Framework'); // for now, the data is written into a temporary file

fclose($handle); // and only now the file is renamed to test.txt
```

You can of course use all the familiar functions, such as:

```php
file_put_contents('nette.safe://test.txt', $content);

$ini = parse_ini_file('nette.safe://autoload.ini');
```

SafeStream guarantees:

- **Atomicity**: The file is written either as a whole or not written at all.
- **Isolation**: No one can start to read a file that is not yet fully written.

If you write to an existing file in the '`a`' mode (append), SafeStream creates it's copy and only after successfully writing it
renames it to the original name. Write in this mode is therefore more resource-consuming than in other modes.


Installation
------------

The recommended way to install is via Composer:

```
composer require nette/safe-stream
```

It requires PHP version 5.3 and supports PHP up to 7.2.
