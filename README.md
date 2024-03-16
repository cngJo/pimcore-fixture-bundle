# Pimcore Fixture Bundle

Provides a way to manage and execute the loading of data fixtures in Pimcore.

It can be useful for testing purposes, or for seeding a database with initial data.

## Installation

1. **Require the bundle**

   ```shell
   composer require teamneusta/pimcore-fixture-bundle
   ```

2. **Enable the bundle**

   Add the Bundle to your `config/bundles.php`:

   ```php
   Neusta\Pimcore\FixtureBundle\NeustaPimcoreFixtureBundle::class => ['test' => true],
   ```

## Usage

### Writing Fixtures

Data fixtures are PHP classes where you create objects and persist them to the database.

Imagine that you want to add some `Product` objects to your database.
To do this, create a fixture class and start adding products:

```php
use Neusta\Pimcore\FixtureBundle\Fixture;
use Pimcore\Model\DataObject\Product;

final class ProductFixture implements Fixture
{
    public function create(): void
    {
        for ($i = 1; $i <= 20; $i++) {
            $product = new Product();
            $product->setParentId(0);
            $product->setPublished(true);
            $product->setKey("Product {$i}");
            // ...
            
            $product->save();
        }
    }
}
```

### Loading Fixtures

Todo

### Accessing Services from the Fixtures

Sometimes you may need to access your application's services inside a fixture class.
You can use normal dependency injection for this:

> [!IMPORTANT]
> You need to create your `FixtureFactory` with the `FixtureInstantiatorForParametrizedConstructors` for this to work!

```php
final class SomeFixture implements Fixture
{
    public function __construct(
        private Something $something,
    ) {
    }

    public function create(): void
    {
        // ... use $this->something
    }
}
```

### Depending on Other Fixtures

In a fixture, you can depend on other fixtures.
Therefore, you have to reference them in your `create()` method as parameters.

> [!IMPORTANT]
> All parameters of the `create()` method in your fixtures may *only* reference other fixtures.
> Everything else is not allowed!

Referencing other fixtures ensures they are created before this one.

This also allows accessing some state of the other fixtures.

```php
final class SomeFixture implements Fixture
{
    public function create(OtherFixture $otherFixture): void
    {
        // do something with $otherFixture->someInformation
    }
}

final class OtherFixture implements Fixture
{
    public string $someInformation;

    public function create(): void
    {
        $this->someInformation = 'some information created in this fixture';
    }
}
```

## Contribution

Feel free to open issues for any bug, feature request, or other ideas.

Please remember to create an issue before creating large pull requests.

### Local Development

To develop on a local machine, the vendor dependencies are required.

```shell
bin/composer install
```

We use composer scripts for our main quality tools. They can be executed via the `bin/composer` file as well.

```shell
bin/composer cs:fix
bin/composer phpstan
bin/composer tests
```
