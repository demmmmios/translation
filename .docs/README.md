# Translation

## Content
- [Setup](README.md#setup)
- [Configuration](README.md#configuration)
	- [Locale resolvers](#locale-resolvers)
- [Examples](README.md#examples)
	- [Presenter](#presenter)
	- [Model](#model)
	- [Latte](#latte)
	- [Neon](#neon)
	- [Parameters in messages](#parameters-in-messages)
- [Loaders](#loaders)
	- [File loaders](#file-loaders)
	- [Database loaders](#database-loaders)
		- [Doctrine](#doctrine)
		- [Nette Database](#nette-database)
		- [Nextras Dbal](#nextras-dbal)
- [Features](#features)
	- [Wrappers](#wrappers)
	- [TranslationProviderInterface](#translationproviderinterface)

## Setup

Require package:

```bash
composer require contributte/translation
```

Register extension:

```neon
extensions:
	translation: Contributte\Translation\DI\TranslationExtension
```

## Configuration

Basic configuration:

```neon
translation:
	locales:
		whitelist: [en, cs, sk]
		default: en
		fallback: [en]
	dirs:
		- %appDir%/lang
	returnOriginalMessage: true # to not translate undefined messages, default is true
```
Note: The `fallback` configuration values should always reflect the **locale code** used in the file you wish to _fallback_ to (e.g. `en_US` for `messages.en_US.neon`, or `en` for `messages.en.neon`).

### Locale resolvers

This configuration instructs the extension how to resolve the locale and the order in which it will do so:

```neon
translation:
	localeResolvers:
		- Contributte\Translation\LocalesResolvers\Router
```

Available resolvers:

- Contributte\Translation\LocalesResolvers\Router
- Contributte\Translation\LocalesResolvers\Header (HTTP header)
- Contributte\Translation\LocalesResolvers\Parameter (Get parameter)
- Contributte\Translation\LocalesResolvers\Session

By default the `Router`, `Parameter` and `Session` resolvers expect the name of the parameter/key to be `locale`.

## Examples

### Presenter

```php
<?php declare(strict_types = 1);

namespace App;

use Nette;
use Contributte;

class BasePresenter extends Nette\Application\UI\Presenter
{

	/** @var Nette\Localization\ITranslator @inject */
	public $translator;

	/** @var Contributte\Translation\LocalesResolvers\Session @inject */
	public $translatorSessionResolver;


	public function handleChangeLocale(string $locale): void
	{
		$this->translatorSessionResolver->setLocale($locale);
		$this->redirect('this');
	}


	public function renderDefault(): void
	{
		$this->translator->translate('domain.message');
		$prefixedTranslator = $this->translator->createPrefixedTranslator('domain');
		$prefixedTranslator->translate('message');
	}

}
```

### Model

```php
<?php declare(strict_types = 1);

namespace App\Model;

use Nette;

class Model
{

	/** @var Nette\Localization\ITranslator */
	private $translator;


	public function __construct(Nette\Localization\ITranslator $translator)
	{
		$this->translator = $translator;
	}

}
```

### Latte

How to use on frontend.

```latte
{_domain.message}
{_domain.message, $count}
{_domain.message, [name => "Ales"]}

{translator domain}
	{_message}
	{_message, $count}
	{_message, [name => "Ales"]}
{/translator}

{var $myMessage = 'domain.message'}
{$myMessage|translate}
```

### Neon

File name format:

```
        locale
          |
         /--\
messages.en_US.neon
\______/       \__/
   |            |
 domain     extension
```

File content format:

```neon
prefix:
	for: "message" # messages.prefix.for
```

### Parameters in messages

Sometimes it is convenient to include a dynamic parameter in the translation - as seen in the Latte examples above.

Neon:

```neon
user.name.taken: "Sorry, the username %name% is already taken, please try a different one."
```

Latte:

```latte
{_user.name.taken, [name => "Ales"]}
```

Presenter/Model:

```php
$this->translator->translate('user.name.taken', [name => 'Ales']);
```

**Note**: When passing parameters to the translator, the parameter names must not be enclosed in `%` characters. This is done by `Contributte/Translation` automatically.

## Loaders

By default the extension will look for `.neon` files.

### File loaders

```neon
array: Symfony\Component\Translation\Loader\ArrayLoader
csv: Symfony\Component\Translation\Loader\CsvFileLoader
dat: Symfony\Component\Translation\Loader\IcuDatFileLoader
res: Symfony\Component\Translation\Loader\IcuResFileLoader
ini: Symfony\Component\Translation\Loader\IniFileLoader
json: Symfony\Component\Translation\Loader\JsonFileLoader
mo: Symfony\Component\Translation\Loader\MoFileLoader
php: Symfony\Component\Translation\Loader\PhpFileLoader
po: Symfony\Component\Translation\Loader\PoFileLoader
ts: Symfony\Component\Translation\Loader\QtFileLoader
xlf: Symfony\Component\Translation\Loader\XliffFileLoader
yml: Symfony\Component\Translation\Loader\YamlFileLoader
```

### Database loaders

Package includes database loaders for **[Doctrine 2](https://www.doctrine-project.org/)** and **[Nette Database 3](https://doc.nette.org/cs/3.0/database)** and **[Nextras Dbal](https://nextras.org/dbal/docs/main/)**.

#### Doctrine

You must create a file with specific format in scanned dirs such as **messages.en_US.doctrine**. All parameters are optional, but the file has to exist.

```neon
table: "My\Entity" # if you specify the entity key, "messages" from file name will be ignored
id: "id" # id column name, default is "id"
locale: "locale" # locale column name, default is "locale"
message: "message" # message column name, default is "message"
```

Add loader to translation configuration:

```neon
translation:
	loaders:
		doctrine: Contributte\Translation\Loaders\Doctrine
```

Entity example:

```php
<?php declare(strict_types = 1);

namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="messages")
 */
class Messages
{

	/**
	 * @ORM\Id
	 * @ORM\Column(type="integer", nullable=false)
	 * @ORM\GeneratedValue
	 */
	public $messageId;

	/**
	 * @ORM\Column(type="string", nullable=false)
	 */
	public $id;

	/**
	 * @ORM\Column(type="string", nullable=false)
	 */
	public $locale;

	/**
	 * @ORM\Column(type="string", nullable=false)
	 */
	public $message;

}
```

#### Nette Database

You must create a file with specific format in scanned dirs such as **messages.en_US.nettedatabase**. All parameters are optional, but the file has to exist.

```neon
table: "my_table" # if you specify table key, "messages" from file name will be ignored
id: "id" # id column name, default is "id"
locale: "locale" # locale column name, default is "locale"
message: "message" # message column name, default is "message"
```

Add loader to translation configuration:

```neon
translation:
	loaders:
		nettedatabase: Contributte\Translation\Loaders\NetteDatabase
```

#### Nextras Dbal

You must create a file with specific format in scanned dirs such as **messages.en_US.nextrasdbal**. All parameters are optional, but the file has to exist.

```neon
table: "my_table" # if you specify table key, "messages" from file name will be ignored
id: "id" # id column name, default is "id"
locale: "locale" # locale column name, default is "locale"
message: "message" # message column name, default is "message"
```

Add loader to translation configuration:

```neon
translation:
	loaders:
		nextrasdbal: Contributte\Translation\Loaders\NextrasDbal
```

DB table example:

```sql
CREATE TABLE `messages` (
    `id` varchar(191) NOT NULL,
    `locale` char(5) NOT NULL,
    `message` varchar(191) NOT NULL,
    UNIQUE KEY `id` (`id`),
    KEY `locale` (`locale`),
    KEY `message` (`message`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

## Features

### Wrappers

It is possible to pass pluralization to components without pre-translation and avoiding double translation.

```php
$form = new Nette\Application\UI\Form;

$form->addText('mail', 'form.mail.label')
	->setOption('description', new Contributte\Translation\Wrappers\Message('form.mail.description', [...]);
```

Or pass the not translatable texts:

```php
$form->addSelect('country', 'form.country.label')
	->setItems([
		new Contributte\Translation\Wrappers\NotTranslate('Czech republic'),
		new Contributte\Translation\Wrappers\NotTranslate('Slovak republic'),
	]);
```

### TranslationProviderInterface

It is possible to pass additional translation resources from your compiler extensions by implementing the `TranslationProviderInterface`.

```php
use Nette\DI\CompilerExtension;
use Contributte\Translation\DI\TranslationProviderInterface;

class MyExtension extends CompilerExtension implements TranslationProviderInterface
{

 	public function getTranslationResources(): array
	{
		return [
			__DIR__ . '/../lang/',
		];
	}

}
```
