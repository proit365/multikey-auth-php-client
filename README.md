# PHP Client SDK

`proit365/multikey-auth` provides PHP request signing against an MKAP v1
server. It validates client exports, obtains a challenge, derives the proof
internally, and returns the target API's normal HTTP response.

For transport requirements, configuration, request behavior, and security
details, see the [PHP Client SDK guide in Markdown](docs/guide.md) or
[HTML](docs/guide.html).

## Installation

Install [`proit365/multikey-auth` from Packagist](https://packagist.org/packages/proit365/multikey-auth). Its Core dependency is also available on Packagist:

```shell
composer require proit365/multikey-auth:^1.0
```

The package requires PHP `^8.2`; Core requires the OpenSSL PHP extension.

## Usage

```php
use ProIT365\MultiKeyAuth\Client\ClientConfig;
use ProIT365\MultiKeyAuth\Client\ExportLoader;
use ProIT365\MultiKeyAuth\Client\PhpMultiKeyClient;
use ProIT365\MultiKeyAuth\Core\Key\IdentifierGenerator;

$keyPool = (new ExportLoader())->fromJson($exportJson);
$client = new PhpMultiKeyClient(
    new ClientConfig(
        'https://api.example.test',
        'CLIENT_001',
        1,
        6,
        IdentifierGenerator::DEFAULT_ALPHABET
    ),
    $keyPool,
    $httpTransport
);

$response = $client->request('POST', '/api/orders', $exactBodyBytes);
```

Provide an implementation of `HttpTransportInterface`. The transport must
preserve the supplied HTTP method, request target, body bytes, and headers, and
must not follow redirects. Redirecting a signed request can change the origin or
request target outside the proof binding.

The base URI must be HTTPS by default and contain no path, query, credentials,
or fragment. HTTP is allowed only when `requireHttps` is explicitly disabled
for development or testing.

`request()` accepts the exact body string to send. It does not serialize
application payloads; serialize once in the application and pass those bytes
unchanged. Challenge endpoint errors throw `ChallengeRequestFailedException`.
Malformed or oversized challenge responses throw
`InvalidChallengeResponseException`. API responses, including non-2xx
responses, are returned as `HttpResponse`.
In particular, an API `401` is returned unchanged. The SDK does not replay
requests automatically because a `401` can also be an application response
after the server has already authorized the request.

The SDK keeps validated secrets in memory for the lifetime of the loaded pool.
Protect the export and process memory appropriately. A complete export
compromise allows client impersonation.

## Validation status

The historically published v1.0.2 client completed a live challenge-response request against
the Laravel test server and received HTTP 200. This loopback integration test
does not validate a production HTTPS deployment.

## Source and support

Project website: [https://www.proit365.com](https://www.proit365.com).

The monorepo is the development source of truth:
[github.com/proit365/multikey-auth](https://github.com/proit365/multikey-auth).
Report issues through its [issue tracker](https://github.com/proit365/multikey-auth/issues).
