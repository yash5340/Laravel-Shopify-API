Laravel / Shopify API Wrapper
===========================

An easy-to-use PHP package to communicate with [Shopify's API](http://docs.shopify.com/api) in Laravel.

## Installation
#### Make sure to download the .env file from the server before starting
#### Require rocket-code/shopify-cottonbabies in `composer.json`

Add `"rocket-code/shopify-cottonbabies": "~1.0"` in your "require" object.  

Example:

```
	"require": {
		"php": ">=5.6.4",
		"laravel/framework": "5.4.*",
		"laravel/tinker": "~1.0",
		"rocket-code/shopify-cottonbabies": "~1.0"
	}
```

Add to the end of `psr-4` section `"RocketCode\\Shopify\\": "Packages/shopify_api_wrapper/src"`

Add `"repositories"` section at the end of the composer.json file

Example:

```
"repositories": [
      {
        "type": "package",
        "package": {
          "name": "rocket-code/shopify-cottonbabies",
          "version": "1.0",
          "source": {
            "url": "https://bitbucket.org/cottonbabiesinc/laravel-shopify-api-wrapper.git",
            "type": "git",
            "reference": "master"
          }
        }
      }
    ]
```

#### Add the Service Provider
In `config/app.php`, add `RocketCode\Shopify\ShopifyServiceProvider::class,` to the end of the `providers` array.

#### Composer dump-autoload
SSH to the server, 
then run from the project root the following commands
`composer update`
`composer dump-autoload` 

#### Add the Middleware
In `app/http/kernel.php`, add `'shopify.webhook' => \RocketCode\Shopify\VerifyShopifyWebhook::class` to the end of the `$routeMiddleware` array.

### Setting up the shopify App
From the App Setup page add `domain/success` to the whitelisted redirection URL(s) section and save it

## Setting Up

Get the necessary 'ID', 'SECRET' from the partners.shopify.com for this app (create the app if necessary)

In `.env`, add these Four entries:
 
 * `SHOPIFY_APP_ID` with your Shopify App *API key*
 * `SHOPIFY_APP_SECRET` with your Shopify App *Secret* (which, for private apps, is not the same as the *Password*)
 * `SHOPIFY_EMAIL_NOTICE` with your email address to receive notices

 Example:
 
 ```
SHOPIFY_APP_ID=000102030405060708090a0b0c0d0e0f

SHOPIFY_APP_SECRET=101112131415161718191a1b1c1d1e1f

SHOPIFY_APP_REDIRECT=https://example.com/oauth

SHOPIFY_EMAIL_NOTICE=example@cottonbabies.com
 ```

## Quick Install

First make sure you run `php artisan migrate` from the server via SSH

Use the existing helper install functionality to install the application quickly and easily. If you need more specific control over the install process refer to the Manual Install section.

To install application on a store access /install and simply enter in the myshopify url.

NOTE: Make sure /success is in the Whitelisted redirection URL(s) in Shopify App Settings.


That's it! You're ready to make some API calls.

To begin, use `App::make()` to grab an instance of the `API` class.

```
$sh = App::make('ShopifyAPI');
```

#### Loading API Credentials
Simply pass an array with the following keys (and filled-in values) to prepare. Not all values need to be passed at once; you can call the `setup()` method as many times as you'd like; it will only accept the following 4 keys, and overwrite a values if it's already set.

```
$sh->setup(['API_KEY' => '', 'API_SECRET' => '', 'SHOP_DOMAIN' => '', 'ACCESS_TOKEN' => '']);
```
##### Shortcut:
Pass the setup array as the second argument in `App::make()`:

```
$sh = App::make('ShopifyAPI', ['API_KEY' => '', 'API_SECRET' => '', 'SHOP_DOMAIN' => '', 'ACCESS_TOKEN' => '']);
``` 

```

## Quick Install
Use the existing helper install functionality to install the application quickly and easily. If you need more specific control over the install process refer to the Manual Install section.

To install application on a store access /install and simply enter in the myshopify url.

NOTE: Make sure /success is in the Whitelisted redirection URL(s) in Shopify App Settings.

## Manual Install

### Finding the Install URL
After setting up with at least `SHOP_DOMAIN` & `API_KEY`, call `installURL()` with an array of permissions ([the app's Scope](docs.shopify.com/api/authentication/oauth#scopes)):

```
$sh->installURL(['permissions' => array('write_orders', 'write_products')]);
```

You may also pass a redirect URL per the `redirect_uri` parameter [as described by the Shopify API Docs](http://docs.shopify.com/api/authentication/oauth#asking-for-permission)

```
$sh->installURL(['permissions' => array('write_orders', 'write_products'), 'redirect' => 'http://myapp.com/success']);
```

### Authentication / Getting OAuth Access Token
In order to make Authenticated requests, [the Access Token must be passed as a header in each request](http://docs.shopify.com/api/authentication/oauth#making-authenticated-requests). This package will automatically do that for you, but you must first authenticate your app on each store (as the user installs it), and save the Access Token.

Once the user accesses the Install URL and clicks the Install button, they will be redirected back to your app with data in the Query String.

After setting up with at least `SHOP_DOMAIN`, `API_KEY`, & `API_SECRET`, call `getAccessToken()` with the code passed back in the URL. Laravel makes this easy:

```
$code = Input::get('code');
$sh = App::make('ShopifyAPI', ['API_KEY' => '', 'API_SECRET' => '', 'SHOP_DOMAIN' => '']);

try
{
	$accessToken = $sh->getAccessToken($code);
}
catch (Exception $e)
{
	echo '<pre>Error: ' . $e->getMessage() . '</pre>';
}

// Save $accessToken
```

#### Verifying OAuth Data
Shopify returns a hashed value to validate the data against. To validate (recommended before calling `getAccessToken()`), utilize `verifyRequest()`.

```
try
{
	$verify = $sh->verifyRequest(Input::all());
	if ($verify)
	{
		$code = Input::get('code');
		$accessToken = $sh->getAccessToken($code);
	}
	else
	{
		// Issue with data
	}

}
catch (Exception $e)
{
	echo '<pre>Error: ' . $e->getMessage() . '</pre>';
}


	
```

`verifyRequest()` returns `TRUE` when data is valid, otherwise `FALSE`. It throws an Exception in two cases: If the timestamp generated by Shopify and your server are more than an hour apart, or if the argument passed is not an array or URL-encoded string of key/values.

If you would like to skip the timestamp check (not recommended unless you cannot correct your server's time), you can pass `TRUE` as a second argument to `verifyRequest()` and timestamps will be ignored:

```
$verify = $sh->verifyRequest(Input::all(), TRUE);
```

## Private Apps
The API Wrapper does not distinguish between private and public apps. In order to utilize it with a private app, set up everything as you normally would, replacing the OAuth Access Token with the private app's Password.

## Calling the API
Once set up, simply pass the data you need to the `call()` method.

```
$result = $sh->call($args);
```

#### `call()` Parameters
The parameters listed below allow you to set required values for an API call as well as override additional default values.

* `METHOD`: The HTTP method to use for your API call. Different endpoints require different methods.
  * Default: `GET`
* `URL`: The URL of the API Endpoint to call.
  * Default: `/` (not an actual endpoint)
* `HEADERS`: An array of additional Headers to be sent
  * Default: Empty `array()`. Headers that are automatically sent include:
    * Accept 
    * Content-Type
    * charset
    * X-Shopify-Access-Token
* `CHARSET`: Change the charset if necessary
  * Default: `UTF-8`
* `DATA`: An array of data being sent with the call. For example, `$args['DATA'] = array('product' => $product);` For an [`/admin/products.json`](http://docs.shopify.com/api/product#create) product creation `POST`.
  * Default: Empty `array()`
* `RETURNARRAY`: Set this to `TRUE` to return data in `array()` format. `FALSE` will return a `stdClass` object.
  * Default: `FALSE`
* `ALLDATA`: Set this to `TRUE` if you would like all error and cURL info returned along with your API data (good for debugging). Data will be available in `$result->_ERROR` and `$result->_INFO`, or `$result['_ERROR']` and `$result['_INFO']`, depending if you are having it returned as an object or array. Recommended to be set to `FALSE` in production.
  * Default: `FALSE`
* `FAILONERROR`: The value passed to cURL's [CURLOPT_FAILONERROR](http://php.net/manual/en/function.curl-setopt.php) setting. `TRUE` will cause the API Wrapper to throw an Exception if the HTTP code is >= `400`. `FALSE` in combination with `ALLDATA` set to `TRUE` will give you more debug information.
  * Default: `TRUE`

## Adding properties to the call data
### $sh->addCallData($key, $value);
  Adds properties to the call data.

  Example:

  ``` 
  $sh->addCallData('METHOD', 'GET'); 
  ``` 


  The results would look like

  ``` 
	[
		METHOD: "GET"
	]
 ```
### $sh->addData($key, $value);
  Adds properties to the call data's "DATA" array.
  Example:

  ``` 
  $sh->addData('limit', 10); 
  ``` 

  The results would look like:

  ``` 
	[
		DATA: {
			limit: "10"
		}
	]
 ```
### $sh->buildChildData($key, $value, $child_resource = null);
  Adds properties inside the DATA array's $resource property.
  if $child\_resource variable is passed, it will add it inside the $resource array's $child\_resource property.
### $sh->commitChildData($child_resource);
  Commits the properties of the DATA array's $resource property.

  Example:

  ``` 
   $this->sh->addCallData('resource', "products");

   $this->sh->buildChildData("title", "Test Title");
   $this->sh->commitChildData("title");

   $this->sh->buildChildData("url", "https://image", "images"); 
   $this->sh->commitChildData("images");
  
  ```

  The results would look like:

  ``` 
	PLURAL_NAME: "products",  /* PLURAL_NAME and SINGULAR_NAME will be added automatically when resource property is added */
	SINGULAR_NAME: "product",
	resource: "products",
	DATA: [ 
		product: {
			title: "Test Title",
			images: {
				0: {
						url: "https://image",
					}
				}
			}
	 ]
 ```

### Some more examples:
Example 1: 


```
	$this->sh->addCallData('resource', "products");
	$this->sh->addCallData('URL', 'admin/' . "products");
	$this->sh->addCallData('METHOD', 'GET');

	$this->sh->addData('limit', 10);
```

Results: 

```
{
	PLURAL_NAME: "products",
	SINGULAR_NAME: "product",
	resource: "custom_collections",
	URL: "admin/custom_collections.json",
	METHOD: "GET",
	DATA: {
		limit: "10",
	}
}
```

Example 2:


```
	$this->sh->addCallData('resource', "products");
	$this->sh->addCallData('URL', 'admin/' . "products");
	$this->sh->addCallData('METHOD', 'GET');

	$this->sh->buildChildData("title", "Test Title");
	$this->sh->commitChildData("title");

	$this->sh->buildChildData("url", "http://image-link/", "images");
	$this->sh->buildChildData("title", "Image Title!", "images");
	$this->sh->commitChildData("images");
```

Results: 

```
{
	PLURAL_NAME: "products",
	SINGULAR_NAME: "product",
	resource: "custom_collections",
	URL: "admin/custom_collections.json",
	METHOD: "GET",
	DATA: {
		product: {
			title: "Test Title",
			images: {
				0: {
						url: "http://image-link/",
						title: "Image Title!"
				}
			}
		}
	}
}
```


## Some Examples
Assume that `$sh` has already been set up as documented above.

#### Listing Products
```
try
{

	$call = $sh->call(['URL' => 'products.json', 'METHOD' => 'GET', 'DATA' => ['limit' => 5, 'published_status' => 'any']]);
}
catch (Exception $e)
{
	$call = $e->getMessage();
}

echo '<pre>';
var_dump($call);
echo '</pre>';

```

`$call` will either contain a `stdClass` object with `products` or an Exception error message.

#### Creating a snippet from a Laravel View

```
$testData = ['name' => 'Foo', 'location' => 'Bar'];
$view = (string) View::make('snippet', $testData);

$themeID = 12345678;

try
{
	$call = $sh->call(['URL' => '/admin/themes/' . $themeID . '/assets.json', 'METHOD' => 'PUT', 'DATA' => ['asset' => ['key' => 'snippets/test.liquid', 'value' => $view] ] ]);
}
catch (Exception $e)
{
	$call = $e->getMessage();
}

echo '<pre>';
var_dump($call);
echo '</pre>';
```

#### Performing operations on multiple shops
The `setup()` method makes changing the current shop simple.

```
$apiKey = '123';
$apiSecret = '456';

$sh = App::make('ShopifyAPI', ['API_KEY' => $apiKey, 'API_SECRET' => $apiSecret]);

$shops = array(
		'my-shop.myshopify.com'		=> 'abc',
		'your-shop.myshopify.com'	=> 'def',
		'another.myshopify.com'		=> 'ghi'
		);
		
foreach($shops as $domain => $access)
{
	$sh->setup(['SHOP_DOMAIN' => $domain, 'ACCESS_TOKEN'' => $access]);
	// $sh->call(), etc

}

```


## Webhooks

### Setting up routes

Make sure to put the routes for saving the webhook files in the routes/api.php file

```
Route::post('webhook/orders/create', 'WebhookController@saveWebhooks');
Route::post('webhook/save', 'WebhookController@saveWebhooks');
```

### Creating Webhooks

To create a webhook, use the API class' "createWebhook" function

Example:

```

$result = $this->sh->createWebhook('products/update', '/api/webhook/save');

```

To get all the webhook:

```

$resource = 'webhooks';
$this->sh->addCallData('resource', $resource);
$this->sh->addCallData('URL', 'admin/' . $resource);

$result = $this->sh->listShopifyResources();

```