# Laravel 5 HAL+JSON Transformer Package


[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/nilportugues/laravel5-hal-json-transformer/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/nilportugues/laravel5-hal-json-transformer/?branch=master) [![SensioLabsInsight](https://insight.sensiolabs.com/projects/93029d8e-7052-42e0-a7db-fabbd2e566d5/mini.png?)](https://insight.sensiolabs.com/projects/93029d8e-7052-42e0-a7db-fabbd2e566d5) 
[![Latest Stable Version](https://poser.pugx.org/nilportugues/laravel5-haljson/v/stable?)](https://packagist.org/packages/nilportugues/laravel5-haljson) 
[![Total Downloads](https://poser.pugx.org/nilportugues/laravel5-haljson/downloads?)](https://packagist.org/packages/nilportugues/laravel5-haljson) 
[![License](https://poser.pugx.org/nilportugues/laravel5-haljson/license?)](https://packagist.org/packages/nilportugues/laravel5-haljson) 



## Installation

Use [Composer](https://getcomposer.org) to install the package:

```
$ composer require nilportugues/laravel5-haljson
```


## Laravel 5 / Lumen Configuration

**Step 1: Add the Service Provider**

Open up `bootstrap/app.php`and add the following lines before the `return $app;` statement:

```php
$app->register('NilPortugues\Laravel5\HalJsonSerializer\Laravel5HalJsonSerializerServiceProvider');
$app['config']->set('haljson_mapping', include('haljson.php'));
```

**Step 2: Add the mapping**

Create a `haljson.php` file in `bootstrap/` directory. This file should return an array returning all the class mappings.

An example as follows:


**Step 3: Usage**

For instance, lets say the following object has been fetched from a Repository , lets say `PostRepository` - this being implemented in Eloquent or whatever your flavour is:

```php
use Acme\Domain\Dummy\Post;
use Acme\Domain\Dummy\ValueObject\PostId;
use Acme\Domain\Dummy\User;
use Acme\Domain\Dummy\ValueObject\UserId;
use Acme\Domain\Dummy\Comment;
use Acme\Domain\Dummy\ValueObject\CommentId;

//$postId = 9;
//PostRepository::findById($postId); 

$post = new Post(
  new PostId(9),
  'Hello World',
  'Your first post',
  new User(
      new UserId(1),
      'Post Author'
  ),
  [
      new Comment(
          new CommentId(1000),
          'Have no fear, sers, your king is safe.',
          new User(new UserId(2), 'Barristan Selmy'),
          [
              'created_at' => (new \DateTime('2015/07/18 12:13:00'))->format('c'),
              'accepted_at' => (new \DateTime('2015/07/19 00:00:00'))->format('c'),
          ]
      ),
  ]
);
```

And a series of mappings, placed in `bootstrap/haljson.php`, that require to use *named routes* so we can use the `route()` helper function:

```php
<?php
//bootstrap/haljson.php
return [
    [
        'class' => 'Acme\Domain\Dummy\Post',
        'alias' => 'Message',
        'aliased_properties' => [
            'author' => 'author',
            'title' => 'headline',
            'content' => 'body',
        ],
        'hide_properties' => [

        ],
        'id_properties' => [
            'postId',
        ],
        'urls' => [
            'self' => route('get_post'),
            'comments' => route('get_post_comments'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
    [
        'class' => 'Acme\Domain\Dummy\ValueObject\PostId',
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'postId',
        ],
        'urls' => [
            'self' => 'self' => route('get_post'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
    [
        'class' => 'Acme\Domain\Dummy\User',
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'userId',
        ],
        'urls' => [
            'self' => route('get_user'),
            'friends' => route('get_user_friends'),
            'comments' => route('get_user_comments'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
    [
        'class' => 'Acme\Domain\Dummy\ValueObject\UserId',
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'userId',
        ],
        'urls' => [
            'self' => route('get_user'),
            'friends' => route('get_user_friends'),
            'comments' => route('get_user_comments'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
    [
        'class' => 'Acme\Domain\Dummy\Comment',
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'commentId',
        ],
        'urls' => [
            'self' => route('get_comment'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
    [
        'class' => 'Acme\Domain\Dummy\ValueObject\CommentId',
        'alias' => '',
        'aliased_properties' => [],
        'hide_properties' => [],
        'id_properties' => [
            'commentId',
        ],
        'urls' => [
            'self' => route('get_comment'),
        ],
        'curies' => [
            'name' => 'example',
            'href' => route("get_example_curie_rel"),
        ]
    ],
];

```

The named routes belong to the `app/Http/routes.php`. Here's a sample for the routes provided mapping:

```php
$app->get(
  '/docs/rels/{rel}',
  ['as' => 'get_example_curie_rel', 'uses' => 'ExampleCurieController@getRelAction']
);

$app->get(
  '/post/{postId}',
  ['as' => 'get_post', 'uses' => 'PostController@getPostAction']
);

$app->get(
  '/post/{postId}/comments',
  ['as' => 'get_post_comments', 'uses' => 'CommentsController@getPostCommentsAction']
);

//...
``` 

All of this set up allows you to easily use the `Serializer` service as follows:

```php
<?php

namespace App\Http\Controllers;

use Acme\Domain\Dummy\PostRepository;
use NilPortugues\Api\HalJson\Http\Message\Response;
use NilPortugues\Serializer\Serializer;
use Symfony\Bridge\PsrHttpMessage\Factory\HttpFoundationFactory;


class PostController extends \Laravel\Lumen\Routing\Controller
{
    /**
     * @var PostRepository
     */
    private $postRepository;

    /**
     * @param PostRepository $postRepository
     * @param Serializer $halJsonSerializer
     */
    public function __construct(PostRepository $postRepository, Serializer $halJsonSerializer)
    {
        $this->postRepository = $postRepository;
        $this->halJsonSerializer = $halJsonSerializer;
    }

    /**
     * @param int $postId
     *
     * @return \Symfony\Component\HttpFoundation\Response
     */
    public function getPostAction($postId)
    {
        $post = $this->postRepository->findById($postId);
        $json = $this->halJsonSerializer->serialize($post);

        return (new HttpFoundationFactory())->createResponse(new Response($json));
    }
}
```

**Output:**

```
HTTP/1.1 200 OK
Cache-Control: private, max-age=0, must-revalidate
Content-type: application/hal+json
```

```json
{
    "post_id": 9,
    "headline": "Hello World",
    "body": "Your first post",
    "_embedded": {
        "author": {
            "user_id": 1,
            "name": "Post Author",
            "_links": {
                "self": {
                    "href": "http://example.com/users/1"
                },
                "example:friends": {
                    "href": "http://example.com/users/1/friends"
                },
                "example:comments": {
                    "href": "http://example.com/users/1/comments"
                }
            }
        },
        "comments": [
            {
                "comment_id": 1000,
                "dates": {
                    "created_at": "2015-08-13T22:47:45+02:00",
                    "accepted_at": "2015-08-13T23:22:45+02:00"
                },
                "comment": "Have no fear, sers, your king is safe.",
                "_embedded": {
                    "user": {
                        "user_id": 2,
                        "name": "Barristan Selmy",
                        "_links": {
                            "self": {
                                "href": "http://example.com/users/2"
                            },
                            "example:friends": {
                                "href": "http://example.com/users/2/friends"
                            },
                            "example:comments": {
                                "href": "http://example.com/users/2/comments"
                            }
                        }
                    }
                },
                "_links": {
                    "example:user": {
                        "href": "http://example.com/users/2"
                    },
                    "self": {
                        "href": "http://example.com/comments/1000"
                    }
                }
            }
        ]
    },
    "_links": {
        "curies": [
            {
                "name": "example",
                "href": route("get_example_curie_rel"),
                "templated": true
            }
        ],
        "self": {
            "href": "http://example.com/posts/9"
        },
        "next": {
            "href": "http://example.com/posts/10"
        },
        "example:author": {
            "href": "http://example.com/users/1"
        },
        "example:comments": {
            "href": "http://example.com/posts/9/comments"
        }
    },
    "_meta": {
        "author": [
            {
                "name": "Nil Portugués Calderó",
                "email": "contact@nilportugues.com"
            }
        ]
    }
}
```

#### Response objects

The following PSR-7 Response objects providing the right headers and HTTP status codes are available:

- `NilPortugues\Api\HalJson\Http\Message\ErrorResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourceCreatedResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourceDeletedResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourceNotFoundResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourcePatchErrorResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourcePostErrorResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourceProcessingResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\ResourceUpdatedResponse($json)`
- `NilPortugues\Api\HalJson\Http\Message\Response($json)`
- `NilPortugues\Api\HalJson\Http\Message\UnsupportedActionResponse($json)`

Due to the current lack of support for PSR-7 Requests and Responses in Laravel, it is also recommended to install the package `symfony/psr-http-message-bridge`that will bridge between the PHP standard and the Response object used by Laravel automatically, as seen in the Controller example code provided.


<br>
## Quality

To run the PHPUnit tests at the command line, go to the tests directory and issue phpunit.

This library attempts to comply with [PSR-1](http://www.php-fig.org/psr/psr-1/), [PSR-2](http://www.php-fig.org/psr/psr-2/), [PSR-4](http://www.php-fig.org/psr/psr-4/) and [PSR-7](http://www.php-fig.org/psr/psr-7/).

If you notice compliance oversights, please send a patch via [Pull Request](https://github.com/nilportugues/laravel5-haljson-transformer/pulls).


<br>
## Contribute

Contributions to the package are always welcome!

* Report any bugs or issues you find on the [issue tracker](https://github.com/nilportugues/laravel5-haljson-transformer/issues/new).
* You can grab the source code at the package's [Git repository](https://github.com/nilportugues/laravel5-haljson-transformer).


<br>
## Support

Get in touch with me using one of the following means:

 - Emailing me at <contact@nilportugues.com>
 - Opening an [Issue](https://github.com/nilportugues/laravel5-haljson-transformer/issues/new)
 - Using Gitter: [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/nilportugues/laravel5-haljson-transformer?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)


<br>
## Authors

* [Nil Portugués Calderó](http://nilportugues.com)
* [The Community Contributors](https://github.com/nilportugues/laravel5-haljson-transformer/graphs/contributors)


## License
The code base is licensed under the [MIT license](LICENSE).
