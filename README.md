Phlack
======

[![Build Status](https://travis-ci.org/mcrumm/phlack.svg?branch=master)](https://travis-ci.org/mcrumm/phlack) [![Scrutinizer Quality Score](https://scrutinizer-ci.com/g/mcrumm/phlack/badges/quality-score.png?s=aa61fc3e04fa4042c0988b9e6670938f65ca21e6)](https://scrutinizer-ci.com/g/mcrumm/phlack/) [![Code Coverage](https://scrutinizer-ci.com/g/mcrumm/phlack/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/mcrumm/phlack/?branch=master) [![Total Downloads](https://poser.pugx.org/mcrumm/phlack/downloads.png)](https://packagist.org/packages/mcrumm/phlack) [![Latest Stable Version](https://poser.pugx.org/mcrumm/phlack/v/stable.png)](https://packagist.org/packages/mcrumm/phlack) [![Latest Unstable Version](https://poser.pugx.org/mcrumm/phlack/v/unstable.png)](https://packagist.org/packages/mcrumm/phlack) [![License](https://poser.pugx.org/mcrumm/phlack/license.png)](https://packagist.org/packages/mcrumm/phlack)

Phlack eases the creation of [Slack Integrations](http://slack.com) in PHP.

**Update:** Phlack now contains a partial implementation of the [Slack API](http://api.slack.com). For details, see the [API Docs](#slack-api) section below.

## Installation

via [composer](https://packagist.org/packages/mcrumm/phlack):

```
composer require mcrumm/phlack
```

## Basic Usage

### Send a Message

```php
<?php
$phlack = new Crummy\Phlack\Phlack('https://my.webhook.url');

$response = $phlack->send('Hello, from Phlack!');

if (200 === $response['status']) {
    echo 'Success!';
}
```

## Advanced Usage

### Configuration Options

#### Legacy WebHook URLs

A previous version of Incoming Webhooks used a generic webhook path for every team URL. If your webhook URL starts with something like `myteam.slack.com`, give Phlack your team name and Incoming Webhook token, and it will do the rest:

```php
<?php
$phlack  = new Crummy\Phlack\Phlack([
    'username' => 'myteam',
    'token'    => 'my_webhook_token'
]);
```

#### via `factory()`:

If you prefer, you can instantiate Phlack via its static `factory()` method:

```php
<?php
$phlack = Crummy\Phlack\Phlack::factory($config);
```

#### via `new Phlack()`:

Besides a webhook url or an array configuration, Phlack will also accept a `PhlackClient` instance as a constructor argument:

```php
<?php
$client = new Crummy\Phlack\Bridge\Guzzle\PhlackClient('https://my.webhook.url');
$phlack = new Crummy\Phlack\Phlack($client);
```

>**Note:** The constructor and factory method both accept the same types of arguments: a string representing the webhook url, an array of client config options, or a `PhlackClient` object.

#### :heart: for Guzzle
The PhlackClient is simply a web service client implemented with [Guzzle](http://guzzlephp.org).  Examine its [service description](src/Crummy/Phlack/Bridge/Guzzle/Resources/slack.json) for more details.

### Creating Messages

A Phlack [Message](src/Crummy/Phlack/Message/Message.php) takes care of structuring the payload for Slack's Incoming Webhooks integration.

#### via `new Message()`;

```php
<?php
//...
use Crummy\Phlack\Message\Message;
$message = new Message('Hello, from phlack!');
echo 'The message payload: ' . PHP_EOL:
echo $message;
```

### via the `MessageBuilder`

A [MessageBuilder](src/Crummy/Phlack/Builder/MessageBuilder.php) is also provided:

```php
<?php
// ...
$messageBuilder = $phlack->getMessageBuilder();
$messageBuilder
  ->setText('I was created in the MessageBuilder')
  ->setChannel('testing')
  ->setIconEmoji('ghost');
$message = $messageBuilder->create();
```

### Sending Messages

Use Phlack's `send()` command to fire off the message:

```php
<?php
// ...
$response = $phlack->send($message);

if (200 != $response['status']) {
  die('FAIL! - ' . $response['text']);
}

echo 'The message was sent: ' . $message;
```

### Custom Message Parameters

Custom messages can be sent by using an array of [valid parameters](https://api.slack.com/incoming-webhooks):

```php
<?php
$phlack->send([
    'channel'      => '#random',
    'icon_emoji'   => ':taco:',
    'username'     => 'Phlack',
    'unfurl_links' => true,
    'text'         => 'I :heart: the <http://api.slack.com|Slack API>!',
]);
```

> Note: No input validation is performed on custom message parameters. You are responsible for formatting channels, emojis, and text data yourself.

#### Response

The [MessageResponse](src/Crummy/Phlack/Bridge/Guzzle/Response/MessageResponse.php) hash contains the `status`, `reason`, and `text` from the response.

Responses from the Incoming Webhooks Integration are very sparse. Success messages will simply return a `status` of `200`. Error messages will contain more details in the response `text` and `reason`.

### More Examples
See the [examples directory](examples/) for more use cases.

## Slack API

Programmatic access to the [Slack API](http://api.slack.com) is provided via the [ApiClient](src/Crummy/Phlack/Bridge/Guzzle/ApiClient.php).

**Note:** Currently, bearer token authentication is the only supported authentication method.  Contributions toward OAuth2 support would be greatly appreciated.

### Getting a Client

Get an [ApiClient](src/Crummy/Phlack/Bridge/Guzzle/ApiClient.php) object by instantiating it with a hash containing your API token, or passing a config hash to its `factory` method.

#### via `factory()`:
```php
<?php

use Crummy\Phlack\Bridge\Guzzle\ApiClient;

$slack = ApiClient::factory([ 'token' => 'my_bearer_token' ]);
```

#### via `new ApiClient()`:
```php
<?php

use Crummy\Phlack\Bridge\Guzzle\ApiClient;

$slack = new ApiClient([ 'token' => 'my_bearer_token' ]);
```

### Methods

The methods currently implemented are:

- [channels.list](https://api.slack.com/methods/channels.list)
- [chat.delete](https://api.slack.com/methods/chat.delete)
- [chat.postMessage](https://api.slack.com/methods/chat.postMessage)
- [chat.update](https://api.slack.com/methods/chat.update)
- [files.list](https://api.slack.com/methods/files.list)
- [groups.list](https://api.slack.com/methods/groups.list)
- [im.list](https://api.slack.com/methods/im.list)
- [users.list](https://api.slack.com/methods/users.list)

Consult the client's [service description](src/Crummy/Phlack/Bridge/Guzzle/Resources/slack_api.json) for information on the responses returned by the API methods.

#### Example: Listing all Channels

```php
<?php

use Crummy\Phlack\Bridge\Guzzle\ApiClient;

$config = [ 'token' => 'my_bearer_token' ];
$slack = new ApiClient($config);

echo 'Fetching Channels List...' . PHP_EOL;
$result = $slack->ListChannels();

if (!$result['ok']) {
    die('FAIL! Error was: ' . $result['error'] . PHP_EOL);
}

foreach ($result['channels'] as $channel) {
    printf('%s: %s' . PHP_EOL, $channel['name'], $channel['purpose']['value']);
}
```

### Iterators

#### Example: ListFilesIterator

The [ListFilesIterator](src/Crummy/Phlack/Bridge/Guzzle/Model/ListFilesIterator.php) eases the ability to iterate through multiple pages of data from the Slack API.  Using the iterator eliminates the need to manually call the API multiple times to retrieve all pages of the result set.

```php
<?php
//...
$iterator = $slack->getIterator('ListFiles');

$i = 0;
foreach ($iterator as $file) {
    $i++;
    echo $file['title'] . PHP_EOL;
}

echo PHP_EOL . 'Retrieved ' . $i . ' files.' . PHP_EOL;
```

A [complete example](examples/api/files_iterator.php) is available in the examples directory.

**Note**: The ListFilesIterator is not strictly necessary to page through file results, but it's certainly easier than the alternative.  [An example without the iterator](examples/api/files_list.php) is also available.

#### More Examples
See the [API examples directory](examples/api) for more use cases.

---
## Disclaimer

Any undocumented portion of this library should be considered *EXPERIMENTAL AT BEST*. Proceed with caution, and, as always, pull requests are welcome.

## Acknowledgements

1. The regex in the [LinkFormatter](src/Crummy/Phlack/Common/Formatter/LinkFormatter.php) was pulled directly from [StevenSloan](https://github.com/stevenosloan) and his [slack-notifier](https://github.com/stevenosloan/slack-notifier) project.
