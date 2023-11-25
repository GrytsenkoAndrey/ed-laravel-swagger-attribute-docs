# How to document your Laravel API with Swagger and PHP attributes

Part of writing reliable API’s is writing good API documentation. Your docs is how the client knows how to interact with your API.

In this Article I will show you how to document your Laravel API with swagger.

![image](https://github.com/GrytsenkoAndrey/ed-laravel-swagger-attribute-docs/assets/63291871/f6fadaed-d580-4418-b435-fba91b34591a)

## INSTALLATION

For this we are going to use the laravel-swagger package. It makes it easy to set up swagger-php and Swagger-ui in our laravel app. Swagger-php converts our attributes to openapi schema that swagger UI can interpret the schema into beautiful docs.

For this article, I am running Laravel 10 on PHP 8.1. let’s start with installing the laravel-swagger package by running:

```
composer require darkaonline/l5-swagger
```

after installation completes, publish the config and view files with:

```
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"
```

once these commands complete you should see a l5-swagger.php in your config directory. It look like this:

```
 [
        /*
        |--------------------------------------------------------------------------
        | Edit to set the api's title
        |--------------------------------------------------------------------------
        */

        'title' => 'L5 Swagger UI',
    ],

    'routes' => [
        /*
        |--------------------------------------------------------------------------
        | Route for accessing api documentation interface
        |--------------------------------------------------------------------------
        */

        'api' => 'api/documentation',

        // ...

    ],

    // ... many more options

    /*
    |--------------------------------------------------------------------------
    | Uncomment to add constants which can be used in annotations
    |--------------------------------------------------------------------------
     */
    'constants' => [
        'L5_SWAGGER_CONST_HOST' => env('L5_SWAGGER_CONST_HOST', 'http://my-default-host.com'),
    ],
];
```

change the title to the name of your api, and then in the L5_SWAGGER_CONST_HOST section change the default to your api’s base url. Alternatively you can set the environment variable instead so your base url can be dynamic.

## Documenting your routes

Documenting a route requires adding attributes above your controller class. before we do that there are some meta data about your api you have to set

![image](https://github.com/GrytsenkoAndrey/ed-laravel-swagger-attribute-docs/assets/63291871/c473dc70-2c1d-48b4-91bb-b2c8f17d3972)


In the above picture, things like your api name, version number etc. To add that go to your laravel base controller located at app/Http/Controllers/Controller.php and add the following:

```
use OpenApi\Attributes as OA;

#[
    OA\Info(version: "1.0.0", description: "petshop api", title: "Petshop-api Documentation"),
    OA\Server(url: 'http://localhost:8088', description: "local server"),
    OA\Server(url: 'http://staging.example.com', description: "staging server"),
    OA\Server(url: 'http://example.com', description: "production server"),
    OA\SecurityScheme( securityScheme: 'bearerAuth', type: "http", name: "Authorization", in: "header", scheme: "bearer"),
]
class Controller extends BaseController
```

Let’s break this down:

- In the first line we add our API version number, description and title.
- Second line we write out our servers, you can add multiple server urls for staging, production etc. This let’s consumers of your api switch between the base_url they intend to use.
- Then in the security scheme, you write the way you protect your routes. In this case I am using bearer jwt.


## Documenting GET endpoints

Documenting a GET endpoint looks like:

```
use OpenApi\Attributes as OA;
 
#[OA\Get(
        path: "/api/v1/admin/user-listing",
        summary: "List all non-admin users",
        security: [
            [
                'bearerAuth' => []
            ]
        ],
        tags: ["Admin"],
        responses: [
            new OA\Response(response: Response::HTTP_OK, description: "users retrieved success"),
            new OA\Response(response: Response::HTTP_UNAUTHORIZED, description: "Unauthorized"),
            new OA\Response(response: Response::HTTP_NOT_FOUND, description: "not found"),
            new OA\Response(response: Response::HTTP_INTERNAL_SERVER_ERROR, description: "Server Error")
        ]
    )]
     public function index(): \Illuminate\Http\Response
    {
```

- First, we import the OpenApi\Attributes namespace. it provides the GET, POST, PUT etc. http verbs
- Write the Api route for this controller
- write what it does for summary.
- if our route is to be protected we add the security section else you can omit it.
- tags is important if you want to group endpoints so all endpoints under admin can be given the admin tag.
- then for responses you document all the possible responses that endpoint can return.


## Documenting an endpoint with a request body

Document POST/PUT/PATCH with a request body looks like:

```
use OpenApi\Attributes as OA;   

#[OA\Post(
        path: "/api/v1/admin/create",
        summary: "Create an Admin Account",
        requestBody: new OA\RequestBody(required: true,
                content: new OA\MediaType(mediaType: "application/x-www-form-urlencoded",
                schema: new OA\Schema(required: ["first_name", "last_name", "email", "password", "password_confirmation"],
                        properties: [
                            new OA\Property(property: 'first_name', description: "User first name", type: "string"),
                            new OA\Property(property: 'last_name', description: "User last name", type: "string"),
                            new OA\Property(property: 'email', description: "User email", type: "string"),
                            new OA\Property(property: 'password', description: "User password", type: "string"),
                            new OA\Property(property: 'password_confirmation', description: "User password confirmation", type: "string"),
                ))),
        tags: ["Admin"],
        responses: [
            new OA\Response(response: Response::HTTP_CREATED, description: "Register Successfully"),
            new OA\Response(response: Response::HTTP_UNPROCESSABLE_ENTITY, description: "Unprocessable entity"),
            new OA\Response(response: Response::HTTP_BAD_REQUEST, description: "Bad Request"),
            new OA\Response(response: Response::HTTP_INTERNAL_SERVER_ERROR, description: "Server Error")
        ]
    )]
```

The only change is the requestBody parameter where you document each field and state if they are required or not.

## Auth routes

Auth config [here](https://github.com/GrytsenkoAndrey/adwisep-subs-mgmt.local/blob/master/config/auth.php#L44)

Auth route is [here](https://github.com/GrytsenkoAndrey/adwisep-subs-mgmt.local/blob/master/app/Http/Controllers/Api/V1/ApiController.php#L18)


## CONCLUSION

This article just covers what you need to get started with swagger in your laravel api. If you want me to cover more high level stuff, drop a comment .

For further reading:

- [swagger-php docs](http://zircote.github.io/swagger-php/)
- [laravel-swagger docs](https://github.com/DarkaOnLine/L5-Swagger)
- For a guide on documenting stuff like Request, Resource classes [read here](https://blog.quickadminpanel.com/laravel-api-documentation-with-openapiswagger/)
- If you want to know more [about PHP attributes here](https://stitcher.io/blog/attributes-in-php-8)
