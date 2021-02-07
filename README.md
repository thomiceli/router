# thomas-miceli/router

```shell
$ composer require thomas-miceli/router
```

### Dependencies
* Nyholm's [PSR-7 implementation](https://github.com/nyholm/psr7) (to create HTTP OO messages) and [PSR-7 server](https://github.com/nyholm/psr7-server) (for server requests)
* [PHP-DI](https://php-di.org) for dependency injection 
* [PSR-15 interfaces](https://www.php-fig.org/psr/psr-15/)

### Working full example

```php
<?php
// index.php
use Psr\Http\Server\RequestHandlerInterface;
use ThomasMiceli\Router\Http\Request;
use ThomasMiceli\Router\Http\Response;
use ThomasMiceli\Router\Router;

require 'vendor/autoload.php';

$router = new Router();

$dependency = new stdClass();
$dependency->hello = 'hello';

$router->registerInstance($dependency);

$router
    ->get('/hello/{name}', function ($name, Request $request, Response $response, stdClass $myClass) {
        if ($request->getAttribute('middleware') === true) {
            $myClass->ok = 'middleware works';
        }
        $myClass->hello .= " $name";

        return $response->json($myClass);
        // or
        // $response->getBody()->write("$name");
        // return $response;

    })
    ->middleware(function (Request $request, RequestHandlerInterface $handler) {
        // executed second
        $request = $request->withAttribute('middleware', true);

        /** @var Response $res */
        $res = $handler->handle($request);
        $res->setHeader('after', 'true');
        return $res;
    })
    ->middleware(function (Request $request, RequestHandlerInterface $handler) {
        // executed first
        $request = $request->withAttribute('middleware', false);
        return $handler->handle($request);
    });

$router->run();

```