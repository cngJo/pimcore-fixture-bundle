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

To use fixtures in tests, a few preparations must be made.

Currently, the FixtureFactory still has to be instantiated manually. The easiest way to do this is with a project-specific kernel base class.

```php
// pimcore/tests/Functional/Foundation/BaseKernelTestCase.php
<?php declare(strict_types=1);

namespace Tests\Functional\Foundation;

use Neusta\Pimcore\FixtureBundle\Factory\FixtureFactory;
use Neusta\Pimcore\FixtureBundle\Factory\FixtureInstantiator\FixtureInstantiatorForAll;
use Neusta\Pimcore\FixtureBundle\Factory\FixtureInstantiator\FixtureInstantiatorForParametrizedConstructors;
use Neusta\Pimcore\FixtureBundle\Fixture;
use Pimcore\Test\KernelTestCase;

class BaseKernelTestCase extends KernelTestCase
{
    /** @param list<class-string<Fixture>> $fixtures */
    protected function importFixtures(array $fixtures): void
    {
        $instantiators = [
            new FixtureInstantiatorForParametrizedConstructors(static::getContainer()),
            new FixtureInstantiatorForAll(),
        ];

        (new FixtureFactory([], $instantiators))->createFixtures($fixtures);
    }

    protected function setUp(): void
    {
        static::bootKernel();
    }

    protected function tearDown(): void
    {
        \Pimcore\Cache::clearAll();
        \Pimcore::collectGarbage();

        parent::tearDown();
    }
}
```

Use the base class as follows. For depending fixtures, use the public properties (not in the example).

```php
// pimcore/tests/Functional/MyCustomTest.php
<?php declare(strict_types=1);

namespace Tests\Functional;

use Pimcore\Model\DataObject;
use Tests\Fixtures\ProductFixture;
use Tests\Functional\Foundation\BaseKernelTestCase;

class MyCustomTest extends BaseKernelTestCase
{
    /** @test */
    public function test_must_import_fixtures(): void
    {
        $this->importFixtures([
            ProductFixture::class,
        ]);
        
        $productFixture = DataObject::getByPath('/product-1');
        
        self::assertNotNull($productFixture);
    }
}
```

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
