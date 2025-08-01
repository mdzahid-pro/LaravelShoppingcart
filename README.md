# Attention
Due to the difficulty experienced by certain individuals and companies in comprehending the [copyleft provision of the MIT license](https://github.com/bumbummen99/LaravelShoppingcart/blob/1c50fb564826b6ee748d2d9afad7447b6884166e/LICENSE#L12), as evidenced by the unauthorized relicensing of my work and that of others, I have opted to terminate all further development and assistance for this repository. Given that this package is intended for use in e-commerce projects, it is expected that the necessary resources would be available for the development and support of your own solutions.


## LaravelShoppingcart
![CI Code Checks](https://github.com/bumbummen99/LaravelShoppingcart/workflows/CI%20Code%20Checks/badge.svg?branch=master)
[![codecov](https://codecov.io/gh/bumbummen99/LaravelShoppingcart/branch/master/graph/badge.svg)](https://codecov.io/gh/bumbummen99/LaravelShoppingcart)
[![StyleCI](https://styleci.io/repos/152610878/shield?branch=master)](https://styleci.io/repos/152610878)
[![Total Downloads](https://poser.pugx.org/bumbummen99/shoppingcart/downloads.png)](https://packagist.org/packages/bumbummen99/shoppingcart)
[![Latest Stable Version](https://poser.pugx.org/bumbummen99/shoppingcart/v/stable)](https://packagist.org/packages/bumbummen99/shoppingcart)
[![Latest Unstable Version](https://poser.pugx.org/bumbummen99/shoppingcart/v/unstable)](https://packagist.org/packages/bumbummen99/shoppingcart)
[![License](https://poser.pugx.org/bumbummen99/shoppingcart/license)](https://packagist.org/packages/bumbummen99/shoppingcart)

This is a fork of [Crinsane's LaravelShoppingcart](https://github.com/Crinsane/LaravelShoppingcart) extended with minor features compatible with Laravel 8-12. An example integration can be [found here](https://github.com/bumbummen99/LaravelShoppingcartDemo).

## Installation

Install the [package](https://packagist.org/packages/bumbummen99/shoppingcart) through [Composer](http://getcomposer.org/). 

Run the Composer require command from the Terminal:

    composer require mdzahid-pro/shoppingcart

Now you're ready to start using the shoppingcart in your application.

**As of version 2 of this package it's possibly to use dependency injection to inject an instance of the Cart class into your controller or other class**

You definitely should publish the `config` file and take a look at it.

    php artisan vendor:publish --provider="Gloudemans\Shoppingcart\ShoppingcartServiceProvider" --tag="config"

This will give you a `cart.php` config file in which you can make changes to the packages behaivor.

## Updates

As of version **4.2.0** this package does, when being used with PostgreSQL, encode the cart content to base64 before storing into database due to an [issue with saving values including zero bytes](https://github.com/bumbummen99/LaravelShoppingcart/pull/167). Please consider clearing your cart table in case you are upgrading using PostgreSQL from a version **<4.2.0**.

## Table of Contents
Look at one of the following topics to learn more about LaravelShoppingcart

* [Important note](#important-note)
* [Usage](#usage)
* [Collections](#collections)
* [Instances](#instances)
* [Models](#models)
* [Database](#database)
* [Calculators](#calculators)
* [Exceptions](#exceptions)
* [Events](#events)
* [Example](#example)
* [Contributors](#contributors)

## Important note

As all the shopping cart that calculate prices including taxes and discount, also this module could be affected by the "totals rounding issue" ([*](https://stackoverflow.com/questions/13529580/magento-tax-rounding-issue)) due to the decimal precision used for prices and for the results.
In order to avoid (or at least minimize) this issue, in the Laravel shoppingcart package the totals are calculated using the method **"per Row"** and returned already rounded based on the number format set as default in the config file (cart.php).
Due to this **WE DISCOURAGE TO SET HIGH PRECISION AS DEFAULT AND TO FORMAT THE OUTPUT RESULT USING LESS DECIMAL** Doing this can lead to the rounding issue.

The base price (product price) is left not rounded.

## Usage

The shoppingcart gives you the following methods to use:

### Cart::add()

Adding an item to the cart is really simple, you just use the `add()` method, which accepts a variety of parameters.

In its most basic form you can specify the id, name, quantity, price and weight of the product you'd like to add to the cart.

```php
Cart::add('293ad', 'Product 1', 1, 9.99, 550);
```

As an optional fifth parameter you can pass it options, so you can add multiple items with the same id, but with (for instance) a different size.

```php
Cart::add('293ad', 'Product 1', 1, 9.99, 550, ['size' => 'large']);
```

**The `add()` method will return an CartItem instance of the item you just added to the cart.**

Maybe you prefer to add the item using an array? As long as the array contains the required keys, you can pass it to the method. The options key is optional.

```php
Cart::add(['id' => '293ad', 'name' => 'Product 1', 'qty' => 1, 'price' => 9.99, 'weight' => 550, 'options' => ['size' => 'large']]);
```

New in version 2 of the package is the possibility to work with the [Buyable](#buyable) interface. The way this works is that you have a model implement the [Buyable](#buyable) interface, which will make you implement a few methods so the package knows how to get the id, name and price from your model. 
This way you can just pass the `add()` method a model and the quantity and it will automatically add it to the cart. 

**As an added bonus it will automatically associate the model with the CartItem**

```php
Cart::add($product, 1, ['size' => 'large']);
```
As an optional third parameter you can add options.
```php
Cart::add($product, 1, ['size' => 'large']);
```

Finally, you can also add multipe items to the cart at once.
You can just pass the `add()` method an array of arrays, or an array of Buyables and they will be added to the cart. 

**When adding multiple items to the cart, the `add()` method will return an array of CartItems.**

```php
Cart::add([
  ['id' => '293ad', 'name' => 'Product 1', 'qty' => 1, 'price' => 10.00, 'weight' => 550],
  ['id' => '4832k', 'name' => 'Product 2', 'qty' => 1, 'price' => 10.00, 'weight' => 550, 'options' => ['size' => 'large']]
]);

Cart::add([$product1, $product2]);

```

### Cart::update()

To update an item in the cart, you'll first need the rowId of the item.
Next you can use the `update()` method to update it.

If you simply want to update the quantity, you'll pass the update method the rowId and the new quantity:

```php
$rowId = 'da39a3ee5e6b4b0d3255bfef95601890afd80709';

Cart::update($rowId, 2); // Will update the quantity
```

If you would like to update options of an item inside the cart, 

```php
$rowId = 'da39a3ee5e6b4b0d3255bfef95601890afd80709';

Cart::update($rowId, ['options'  => ['size' => 'small']]); // Will update the size option with new value
```



If you want to update more attributes of the item, you can either pass the update method an array or a `Buyable` as the second parameter. This way you can update all information of the item with the given rowId.

```php
Cart::update($rowId, ['name' => 'Product 1']); // Will update the name

Cart::update($rowId, $product); // Will update the id, name and price

```

### Cart::remove()

To remove an item for the cart, you'll again need the rowId. This rowId you simply pass to the `remove()` method and it will remove the item from the cart.

```php
$rowId = 'da39a3ee5e6b4b0d3255bfef95601890afd80709';

Cart::remove($rowId);
```

### Cart::get()

If you want to get an item from the cart using its rowId, you can simply call the `get()` method on the cart and pass it the rowId.

```php
$rowId = 'da39a3ee5e6b4b0d3255bfef95601890afd80709';

Cart::get($rowId);
```

### Cart::content()

Of course you also want to get the carts content. This is where you'll use the `content` method. This method will return a Collection of CartItems which you can iterate over and show the content to your customers.

```php
Cart::content();
```

This method will return the content of the current cart instance, if you want the content of another instance, simply chain the calls.

```php
Cart::instance('wishlist')->content();
```

### Cart::destroy()

If you want to completely remove the content of a cart, you can call the destroy method on the cart. This will remove all CartItems from the cart for the current cart instance.

```php
Cart::destroy();
```

### Cart::weight()

The `weight()` method can be used to get the weight total of all items in the cart, given their weight and quantity.

```php
Cart::weight();
```

The method will automatically format the result, which you can tweak using the three optional parameters

```php
Cart::weight($decimals, $decimalSeperator, $thousandSeperator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the total property `$cart->weight`**

### Cart::total()

The `total()` method can be used to get the calculated total of all items in the cart, given there price and quantity.

```php
Cart::total();
```

The method will automatically format the result, which you can tweak using the three optional parameters

```php
Cart::total($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the total property `$cart->total`**

### Cart::tax()

The `tax()` method can be used to get the calculated amount of tax for all items in the cart, given there price and quantity.

```php
Cart::tax();
```

The method will automatically format the result, which you can tweak using the three optional parameters

```php
Cart::tax($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the tax property `$cart->tax`**

### Cart::subtotal()

The `subtotal()` method can be used to get the total of all items in the cart, minus the total amount of tax. 

```php
Cart::subtotal();
```

The method will automatically format the result, which you can tweak using the three optional parameters

```php
Cart::subtotal($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the subtotal property `$cart->subtotal`**

### Cart::discount()

The `discount()` method can be used to get the total discount of all items in the cart. 

```php
Cart::discount();
```

The method will automatically format the result, which you can tweak using the three optional parameters

```php
Cart::discount($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the subtotal property `$cart->discount`**

### Cart::initial()

The `initial()` method can be used to get the total price of all items in the cart before applying discount and taxes. 

It could be deprecated in the future. **When rounded could be affected by the rounding issue**, use it carefully or use [Cart::priceTotal()](#Cart::priceTotal())

```php
Cart::initial();
```

The method will automatically format the result, which you can tweak using the three optional parameters. 

```php
Cart::initial($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

### Cart::priceTotal()

The `priceTotal()` method can be used to get the total price of all items in the cart before applying discount and taxes. 

```php
Cart::priceTotal();
```

The method return the result rounded based on the default number format, but you can tweak using the three optional parameters

```php
Cart::priceTotal($decimals, $decimalSeparator, $thousandSeparator);
```

You can set the default number format in the config file.

**If you're not using the Facade, but use dependency injection in your (for instance) Controller, you can also simply get the subtotal property `$cart->initial`**

### Cart::count()

If you want to know how many items there are in your cart, you can use the `count()` method. This method will return the total number of items in the cart. So if you've added 2 books and 1 shirt, it will return 3 items.

```php
Cart::count();
$cart->count();
```

### Cart::search()

To find an item in the cart, you can use the `search()` method.

**This method was changed on version 2**

Behind the scenes, the method simply uses the filter method of the Laravel Collection class. This means you must pass it a Closure in which you'll specify you search terms.

If you for instance want to find all items with an id of 1:

```php
$cart->search(function ($cartItem, $rowId) {
	return $cartItem->id === 1;
});
```

As you can see the Closure will receive two parameters. The first is the CartItem to perform the check against. The second parameter is the rowId of this CartItem.

**The method will return a Collection containing all CartItems that where found**

This way of searching gives you total control over the search process and gives you the ability to create very precise and specific searches.

### Cart::setTax($rowId, $taxRate)

You can use the `setTax()` method to change the tax rate that applies to the CartItem. This will overwrite the value set in the config file.

```php
Cart::setTax($rowId, 21);
$cart->setTax($rowId, 21);
```

### Cart::setGlobalTax($taxRate)

You can use the `setGlobalTax()` method to change the tax rate for all items in the cart. New items will receive the setGlobalTax as well.

```php
Cart::setGlobalTax(21);
$cart->setGlobalTax(21);
```

### Cart::setGlobalDiscount($discountRate)

You can use the `setGlobalDiscount()` method to change the discount rate for all items in the cart. New items will receive the discount as well.

```php
Cart::setGlobalDiscount(50);
$cart->setGlobalDiscount(50);
```

### Cart::setDiscount($rowId, $taxRate)

You can use the `setDiscount()` method to change the discount rate that applies a CartItem. Keep in mind that this value will be changed if you set the global discount for the Cart afterwards.

```php
Cart::setDiscount($rowId, 21);
$cart->setDiscount($rowId, 21);
```

### Buyable

For the convenience of faster adding items to cart and their automatic association, your model has to implement the `Buyable` interface. You can use the `CanBeBought` trait to implement the required methods but keep in mind that these will use predefined fields on your model for the required values.
```php
<?php
namespace App\Models;

use Gloudemans\Shoppingcart\Contracts\Buyable;
use Illuminate\Database\Eloquent\Model;

class Product extends Model implements Buyable {
    use Gloudemans\Shoppingcart\CanBeBought;
}
```

If the trait does not work for on the model or you wan't to map the fields manually the model has to implement the `Buyable` interface methods. To do so, it must implement such functions:

```php
    public function getBuyableIdentifier(){
        return $this->id;
    }
    public function getBuyableDescription(){
        return $this->name;
    }
    public function getBuyablePrice(){
        return $this->price;
    }
    public function getBuyableWeight(){
        return $this->weight;
    }
```

Example:

```php
<?php
namespace App\Models;

use Gloudemans\Shoppingcart\Contracts\Buyable;
use Illuminate\Database\Eloquent\Model;

class Product extends Model implements Buyable {
    public function getBuyableIdentifier($options = null) {
        return $this->id;
    }
    public function getBuyableDescription($options = null) {
        return $this->name;
    }
    public function getBuyablePrice($options = null) {
        return $this->price;
    }
    public function getBuyableWeight($options = null) {
        return $this->weight;
    }
}
```

## Collections

On multiple instances the Cart will return to you a Collection. This is just a simple Laravel Collection, so all methods you can call on a Laravel Collection are also available on the result.

As an example, you can quicky get the number of unique products in a cart:

```php
Cart::content()->count();
```

Or you can group the content by the id of the products:

```php
Cart::content()->groupBy('id');
```

## Instances

The packages supports multiple instances of the cart. The way this works is like this:

You can set the current instance of the cart by calling `Cart::instance('newInstance')`. From this moment, the active instance of the cart will be `newInstance`, so when you add, remove or get the content of the cart, you're work with the `newInstance` instance of the cart.
If you want to switch instances, you just call `Cart::instance('otherInstance')` again, and you're working with the `otherInstance` again.

So a little example:

```php
Cart::instance('shopping')->add('192ao12', 'Product 1', 1, 9.99, 550);

// Get the content of the 'shopping' cart
Cart::content();

Cart::instance('wishlist')->add('sdjk922', 'Product 2', 1, 19.95, 550, ['size' => 'medium']);

// Get the content of the 'wishlist' cart
Cart::content();

// If you want to get the content of the 'shopping' cart again
Cart::instance('shopping')->content();

// And the count of the 'wishlist' cart again
Cart::instance('wishlist')->count();
```

You can also use the `InstanceIdentifier` Contract to extend a desired Model to assign / create a Cart instance for it. This also allows to directly set the global discount.
```
<?php

namespace App;
...
use Illuminate\Foundation\Auth\User as Authenticatable;
use Gloudemans\Shoppingcart\Contracts\InstanceIdentifier;

class User extends Authenticatable implements InstanceIdentifier
{
	...

	/**
     * Get the unique identifier to load the Cart from
     *
     * @return int|string
     */
    public function getInstanceIdentifier($options = null)
    {
        return $this->email;
    }

    /**
     * Get the unique identifier to load the Cart from
     *
     * @return int|string
     */
    public function getInstanceGlobalDiscount($options = null)
    {
        return $this->discountRate ?: 0;
    }
}

// Inside Controller
$user = \Auth::user();
$cart = Cart::instance($user);



```

**N.B. Keep in mind that the cart stays in the last set instance for as long as you don't set a different one during script execution.**

**N.B.2 The default cart instance is called `default`, so when you're not using instances,`Cart::content();` is the same as `Cart::instance('default')->content()`.**

## Models

Because it can be very convenient to be able to directly access a model from a CartItem is it possible to associate a model with the items in the cart. Let's say you have a `Product` model in your application. With the `associate()` method, you can tell the cart that an item in the cart, is associated to the `Product` model. 

That way you can access your model right from the `CartItem`!

The model can be accessed via the `model` property on the CartItem.

**If your model implements the `Buyable` interface and you used your model to add the item to the cart, it will associate automatically.**

Here is an example:

```php

// First we'll add the item to the cart.
$cartItem = Cart::add('293ad', 'Product 1', 1, 9.99, 550, ['size' => 'large']);

// Next we associate a model with the item.
Cart::associate($cartItem->rowId, 'Product');

// Or even easier, call the associate method on the CartItem!
$cartItem->associate('Product');

// You can even make it a one-liner
Cart::add('293ad', 'Product 1', 1, 9.99, 550, ['size' => 'large'])->associate('Product');

// Now, when iterating over the content of the cart, you can access the model.
foreach(Cart::content() as $row) {
	echo 'You have ' . $row->qty . ' items of ' . $row->model->name . ' with description: "' . $row->model->description . '" in your cart.';
}
```
## Database

- [Config](#configuration)
- [Storing the cart](#storing-the-cart)
- [Restoring the cart](#restoring-the-cart)

### Configuration
To save cart into the database so you can retrieve it later, the package needs to know which database connection to use and what the name of the table is.
By default the package will use the default database connection and use a table named `shoppingcart`. You can change that in the configuration.

To make your life easy, the package also includes a ready to use `migration` which you can publish by running:

    php artisan vendor:publish --provider="Gloudemans\Shoppingcart\ShoppingcartServiceProvider" --tag="migrations"
    
This will place a `shoppingcart` table's migration file into `database/migrations` directory. Now all you have to do is run `php artisan migrate` to migrate your database.

### Storing the cart    
To store your cart instance into the database, you have to call the `store($identifier) ` method. Where `$identifier` is a random key, for instance the id or username of the user.

    Cart::store('username');
    
    // To store a cart instance named 'wishlist'
    Cart::instance('wishlist')->store('username');

### Restoring the cart
If you want to retrieve the cart from the database and restore it, all you have to do is call the  `restore($identifier)` where `$identifier` is the key you specified for the `store` method.
 
    Cart::restore('username');
    
    // To restore a cart instance named 'wishlist'
    Cart::instance('wishlist')->restore('username');

### Merge the cart
If you want to merge the cart with another one from the database, all you have to do is call the  `merge($identifier)` where `$identifier` is the key you specified for the `store` method. You can also define if you want to keep the discount and tax rates of the items and if you want to dispatch "cart.added" events.
     
    // Merge the contents of 'savedcart' into 'username'.
    Cart::instance('username')->merge('savedcart', $keepDiscount, $keepTaxrate, $dispatchAdd, 'savedcartinstance');

### Erasing the cart
If you want to erase the cart from the database, all you have to do is call the  `erase($identifier)` where `$identifier` is the key you specified for the `store` method.
 
    Cart::erase('username');
    
    // To erase a cart switching to an instance named 'wishlist'
    Cart::instance('wishlist')->erase('username');

## Calculators

The calculation logic for the package is implemented and defined in `Calculator` classes. These implement the `Gloudemans\Shoppingcart\Contracts\Calculator` Contract and and determine how the prices are calculated and rounded. The calculators can be configured in the confugration file. This is the default calculator:
```php
<?php

namespace Gloudemans\Shoppingcart\Calculation;

use Gloudemans\Shoppingcart\CartItem;
use Gloudemans\Shoppingcart\Contracts\Calculator;

class DefaultCalculator implements Calculator
{
    public static function getAttribute(string $attribute, CartItem $cartItem)
    {
        $decimals = config('cart.format.decimals', 2);

        switch ($attribute) {
            case 'discount':
                return $cartItem->price * ($cartItem->getDiscountRate() / 100);
            case 'tax':
                return round($cartItem->priceTarget * ($cartItem->taxRate / 100), $decimals);
            case 'priceTax':
                return round($cartItem->priceTarget + $cartItem->tax, $decimals);
            case 'discountTotal':
                return round($cartItem->discount * $cartItem->qty, $decimals);
            case 'priceTotal':
                return round($cartItem->price * $cartItem->qty, $decimals);
            case 'subtotal':
                return max(round($cartItem->priceTotal - $cartItem->discountTotal, $decimals), 0);
            case 'priceTarget':
                return round(($cartItem->priceTotal - $cartItem->discountTotal) / $cartItem->qty, $decimals);
            case 'taxTotal':
                return round($cartItem->subtotal * ($cartItem->taxRate / 100), $decimals);
            case 'total':
                return round($cartItem->subtotal + $cartItem->taxTotal, $decimals);
            default:
                return;
        }
    }
}

```

## Exceptions

The Cart package will throw exceptions if something goes wrong. This way it's easier to debug your code using the Cart package or to handle the error based on the type of exceptions. The Cart packages can throw the following exceptions:

| Exception                    | Reason                                                                             |
| ---------------------------- | ---------------------------------------------------------------------------------- |
| *CartAlreadyStoredException* | When trying to store a cart that was already stored using the specified identifier |
| *InvalidRowIDException*      | When the rowId that got passed doesn't exists in the current cart instance         |
| *UnknownModelException*      | When you try to associate an none existing model to a CartItem.                    |

## Events

The cart also has events build in. There are five events available for you to listen for.

| Event         | Fired                                    | Parameter                             |
| ------------- | ---------------------------------------- | ------------------------------------- |
| cart.adding   | When adding an item to the cart.         | The `CartItem` that is being added.   |
| cart.updating | When updating an item to the cart.       | The `CartItem` that is being updated. |
| cart.removing | When removing an item to the cart.       | The `CartItem` that is being removed. |
| cart.added    | When an item was added to the cart.      | The `CartItem` that was added.        |
| cart.updated  | When an item was updated to the cart.    | The `CartItem` that was updated.      |
| cart.removed  | When an item was removed from the cart.  | The `CartItem` that was removed.      |
| cart.merged   | When the content of a cart is merged     | -                                     |
| cart.stored   | When the content of a cart was stored.   | -                                     |
| cart.restored | When the content of a cart was restored. | -                                     |
| cart.erased   | When the content of a cart was erased.   | -                                     |

## Example

Below is a little example of how to list the cart content in a table:

```php

// Add some items in your Controller.
Cart::add('192ao12', 'Product 1', 1, 9.99);
Cart::add('1239ad0', 'Product 2', 2, 5.95, ['size' => 'large']);

// Display the content in a View.
<table>
   	<thead>
       	<tr>
           	<th>Product</th>
           	<th>Qty</th>
           	<th>Price</th>
           	<th>Subtotal</th>
       	</tr>
   	</thead>

   	<tbody>

   		<?php foreach(Cart::content() as $row) :?>

       		<tr>
           		<td>
               		<p><strong><?php echo $row->name; ?></strong></p>
               		<p><?php echo ($row->options->has('size') ? $row->options->size : ''); ?></p>
           		</td>
           		<td><input type="text" value="<?php echo $row->qty; ?>"></td>
           		<td>$<?php echo $row->price; ?></td>
           		<td>$<?php echo $row->total; ?></td>
       		</tr>

	   	<?php endforeach;?>

   	</tbody>
   	
   	<tfoot>
   		<tr>
   			<td colspan="2">&nbsp;</td>
   			<td>Subtotal</td>
   			<td><?php echo Cart::subtotal(); ?></td>
   		</tr>
   		<tr>
   			<td colspan="2">&nbsp;</td>
   			<td>Tax</td>
   			<td><?php echo Cart::tax(); ?></td>
   		</tr>
   		<tr>
   			<td colspan="2">&nbsp;</td>
   			<td>Total</td>
   			<td><?php echo Cart::total(); ?></td>
   		</tr>
   	</tfoot>
</table>
```

## Collaborators

<!-- readme: collaborators -start -->
<table>
<tr>
    <td align="center">
        <a href="https://github.com/bumbummen99">
            <img src="https://avatars.githubusercontent.com/u/4533331?v=4" width="100;" alt="bumbummen99"/>
            <br />
            <sub><b>Patrick</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/Sartoric">
            <img src="https://avatars.githubusercontent.com/u/6607379?v=4" width="100;" alt="Sartoric"/>
            <br />
            <sub><b>Sartoric</b></sub>
        </a>
    </td></tr>
</table>
<!-- readme: collaborators -end -->

## Contributors
<!-- readme: contributors -start -->
<table>
<tr>
    <td align="center">
        <a href="https://github.com/bumbummen99">
            <img src="https://avatars.githubusercontent.com/u/4533331?v=4" width="100;" alt="bumbummen99"/>
            <br />
            <sub><b>Patrick</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/Crinsane">
            <img src="https://avatars.githubusercontent.com/u/1297781?v=4" width="100;" alt="Crinsane"/>
            <br />
            <sub><b>Rob Gloudemans</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/Norris1z">
            <img src="https://avatars.githubusercontent.com/u/18237132?v=4" width="100;" alt="Norris1z"/>
            <br />
            <sub><b>Norris Oduro</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/olegbespalov">
            <img src="https://avatars.githubusercontent.com/u/5425600?v=4" width="100;" alt="olegbespalov"/>
            <br />
            <sub><b>Oleg Bespalov</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/cwprogger">
            <img src="https://avatars.githubusercontent.com/u/11742147?v=4" width="100;" alt="cwprogger"/>
            <br />
            <sub><b>Andrew Savchenko</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/ChrisThompsonTLDR">
            <img src="https://avatars.githubusercontent.com/u/348801?v=4" width="100;" alt="ChrisThompsonTLDR"/>
            <br />
            <sub><b>Chris Thompson</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/Jam-Iko">
            <img src="https://avatars.githubusercontent.com/u/44161368?v=4" width="100;" alt="Jam-Iko"/>
            <br />
            <sub><b>Jam-Iko</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/mattusik">
            <img src="https://avatars.githubusercontent.com/u/1252223?v=4" width="100;" alt="mattusik"/>
            <br />
            <sub><b>Matus Rohal</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/rakibabu">
            <img src="https://avatars.githubusercontent.com/u/14089150?v=4" width="100;" alt="rakibabu"/>
            <br />
            <sub><b>Rakhal Imming</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/tiotobing">
            <img src="https://avatars.githubusercontent.com/u/33707075?v=4" width="100;" alt="tiotobing"/>
            <br />
            <sub><b>Tiotobing</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/Sartoric">
            <img src="https://avatars.githubusercontent.com/u/6607379?v=4" width="100;" alt="Sartoric"/>
            <br />
            <sub><b>Sartoric</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/macbookandrew">
            <img src="https://avatars.githubusercontent.com/u/784333?v=4" width="100;" alt="macbookandrew"/>
            <br />
            <sub><b>Andrew Minion</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/dtwebuk">
            <img src="https://avatars.githubusercontent.com/u/6045378?v=4" width="100;" alt="dtwebuk"/>
            <br />
            <sub><b>Daniel Tomlinson</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/tkaw220">
            <img src="https://avatars.githubusercontent.com/u/694289?v=4" width="100;" alt="tkaw220"/>
            <br />
            <sub><b>Edwin Aw</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/manojo123">
            <img src="https://avatars.githubusercontent.com/u/20805943?v=4" width="100;" alt="manojo123"/>
            <br />
            <sub><b>Jorge Moura</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/jorgejavierleon">
            <img src="https://avatars.githubusercontent.com/u/7950376?v=4" width="100;" alt="jorgejavierleon"/>
            <br />
            <sub><b>Jorge Javier León</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/geisi">
            <img src="https://avatars.githubusercontent.com/u/10728579?v=4" width="100;" alt="geisi"/>
            <br />
            <sub><b>Tim Geisendörfer</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/adamgoose">
            <img src="https://avatars.githubusercontent.com/u/611068?v=4" width="100;" alt="adamgoose"/>
            <br />
            <sub><b>Adam Engebretson</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/andcl">
            <img src="https://avatars.githubusercontent.com/u/8470427?v=4" width="100;" alt="andcl"/>
            <br />
            <sub><b>Andrés</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/ganyicz">
            <img src="https://avatars.githubusercontent.com/u/3823354?v=4" width="100;" alt="ganyicz"/>
            <br />
            <sub><b>Filip Ganyicz</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/guysolamour">
            <img src="https://avatars.githubusercontent.com/u/22590722?v=4" width="100;" alt="guysolamour"/>
            <br />
            <sub><b>Guy-roland ASSALE</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/jackmcdade">
            <img src="https://avatars.githubusercontent.com/u/44739?v=4" width="100;" alt="jackmcdade"/>
            <br />
            <sub><b>Jack McDade</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/jeremyvaught">
            <img src="https://avatars.githubusercontent.com/u/302304?v=4" width="100;" alt="jeremyvaught"/>
            <br />
            <sub><b>Jeremy Vaught</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/jmarkese">
            <img src="https://avatars.githubusercontent.com/u/1827586?v=4" width="100;" alt="jmarkese"/>
            <br />
            <sub><b>John Markese</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/nexxai">
            <img src="https://avatars.githubusercontent.com/u/4316564?v=4" width="100;" alt="nexxai"/>
            <br />
            <sub><b>JT Smith</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/mrabbani">
            <img src="https://avatars.githubusercontent.com/u/4253979?v=4" width="100;" alt="mrabbani"/>
            <br />
            <sub><b>Mahbub Rabbani</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/mauriciv">
            <img src="https://avatars.githubusercontent.com/u/12043163?v=4" width="100;" alt="mauriciv"/>
            <br />
            <sub><b>Mauricio Vera</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/xpundel">
            <img src="https://avatars.githubusercontent.com/u/1384653?v=4" width="100;" alt="xpundel"/>
            <br />
            <sub><b>Mikhail Lisnyak</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/absemetov">
            <img src="https://avatars.githubusercontent.com/u/735924?v=4" width="100;" alt="absemetov"/>
            <br />
            <sub><b>Nadir Absemetov</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/nielsiano">
            <img src="https://avatars.githubusercontent.com/u/947684?v=4" width="100;" alt="nielsiano"/>
            <br />
            <sub><b>Niels Stampe</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/4ilo">
            <img src="https://avatars.githubusercontent.com/u/15938739?v=4" width="100;" alt="4ilo"/>
            <br />
            <sub><b>Olivier</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/PazkaL">
            <img src="https://avatars.githubusercontent.com/u/1322192?v=4" width="100;" alt="PazkaL"/>
            <br />
            <sub><b>Pascal Kousbroek</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/quintenbuis">
            <img src="https://avatars.githubusercontent.com/u/36452184?v=4" width="100;" alt="quintenbuis"/>
            <br />
            <sub><b>Quinten Buis</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/publiux">
            <img src="https://avatars.githubusercontent.com/u/2847188?v=4" width="100;" alt="publiux"/>
            <br />
            <sub><b>Raul Ruiz</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/royduin">
            <img src="https://avatars.githubusercontent.com/u/1703233?v=4" width="100;" alt="royduin"/>
            <br />
            <sub><b>Roy Duineveld</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/CaddyDz">
            <img src="https://avatars.githubusercontent.com/u/13698160?v=4" width="100;" alt="CaddyDz"/>
            <br />
            <sub><b>Salim Djerbouh</b></sub>
        </a>
    </td></tr>
<tr>
    <td align="center">
        <a href="https://github.com/pendalff">
            <img src="https://avatars.githubusercontent.com/u/236587?v=4" width="100;" alt="pendalff"/>
            <br />
            <sub><b>Fukalov Sem</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/sobhanatar">
            <img src="https://avatars.githubusercontent.com/u/1507325?v=4" width="100;" alt="sobhanatar"/>
            <br />
            <sub><b>Sobhan Atar</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/mightyteja">
            <img src="https://avatars.githubusercontent.com/u/2662727?v=4" width="100;" alt="mightyteja"/>
            <br />
            <sub><b>Teja Babu S</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/kekenec">
            <img src="https://avatars.githubusercontent.com/u/11806874?v=4" width="100;" alt="kekenec"/>
            <br />
            <sub><b>Kekenec</b></sub>
        </a>
    </td>
    <td align="center">
        <a href="https://github.com/sasin91">
            <img src="https://avatars.githubusercontent.com/u/808722?v=4" width="100;" alt="sasin91"/>
            <br />
            <sub><b>Sasin91</b></sub>
        </a>
    </td></tr>
</table>
<!-- readme: contributors -end -->
