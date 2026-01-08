---
title: PHP Swagger OpenAPI Documentation
deprecated: false
hidden: false
metadata:
  robots: index
---
# OpenAPI Documentation System

This project uses a hybrid approach to generate OpenAPI/Swagger documentation, combining:

1. **PHP 8 Attributes** - Modern inline documentation using `swagger-php`
2. **JSON Definition Files** - Legacy `.json` files alongside API classes
3. **Schema-based Generation** - Auto-generated CRUD endpoints from `schema.json`

## Quick Start

Generate and publish the API documentation:

```bash
admin updatedocs
```

This command:

1. Parses validator types and converts them to OpenAPI schemas
2. Reads schema definitions from `sites/home/core.json`
3. Scans all API class files for OpenAPI attributes
4. Merges legacy `.json` definition files
5. Uploads the combined spec to S3
6. Syncs with ReadMe.io (if configured)

## PHP 8 Attribute Documentation

### Basic Setup

Add the OpenApi namespace at the top of your API class file:

```php
<?php
use OpenApi\Attributes as OA;
```

### Documenting a Class

Use the `#[OA\Tag]` attribute to define a tag for grouping endpoints:

```php
#[OA\Tag(name: "Wallet", description: "Actualize Wallet operations")]
class ACTUALIZE_WALLET {
    // ...
}
```

### Documenting Endpoints

#### GET Request

```php
#[OA\Get(
    path: "/v1/actualize_wallet/transactions",
    summary: "Get wallet transactions",
    description: "Returns a paginated list of wallet transactions.",
    tags: ["Wallet"],
    security: [["bearerAuth" => []]],
    parameters: [
        new OA\Parameter(
            name: "type",
            in: "query",
            required: false,
            schema: new OA\Schema(type: "string"),
            description: "Filter by transaction type"
        )
    ],
    responses: [
        new OA\Response(response: 200, description: "List of transactions"),
        new OA\Response(response: 401, description: "Unauthorized")
    ]
)]
public static function transactions($r) { }
```

#### POST Request with Body

```php
#[OA\Post(
    path: "/v1/actualize_wallet/pass/create",
    summary: "Create a wallet pass",
    description: "Creates a digital wallet pass for a ticket.",
    tags: ["Wallet", "Pass"],
    security: [["bearerAuth" => []]],
    requestBody: new OA\RequestBody(
        required: true,
        content: new OA\JsonContent(
            required: ["id"],
            properties: [
                new OA\Property(property: "id", type: "string", description: "The ticket ID")
            ]
        )
    ),
    responses: [
        new OA\Response(response: 200, description: "Pass created"),
        new OA\Response(response: 404, description: "Ticket not found")
    ]
)]
public static function ensurePass($r) { }
```

#### DELETE Request

```php
#[OA\Delete(
    path: "/v1/actualize_wallet/pass/invalidate",
    summary: "Invalidate a wallet pass",
    tags: ["Wallet"],
    security: [["bearerAuth" => []]],
    requestBody: new OA\RequestBody(
        required: true,
        content: new OA\JsonContent(
            required: ["id"],
            properties: [
                new OA\Property(property: "id", type: "string")
            ]
        )
    ),
    responses: [
        new OA\Response(response: 200, description: "Pass invalidated")
    ]
)]
public static function invalidatePass($r) { }
```

## Hiding Endpoints from Documentation

To exclude an endpoint from the generated docs, add `x: ["internal" => true]`:

```php
#[OA\Post(
    path: "/v1/actualize_wallet/buy/sol_stripe",
    x: ["internal" => true],  // <-- This hides the endpoint
    summary: "Buy SOL via Stripe",
    // ... rest of definition
)]
public static function buySolStripe($r) { }
```

## Defining Reusable Schemas

Define schemas that can be referenced across multiple endpoints:

```php
#[OA\Schema(
    schema: "WalletTransaction",
    description: "A wallet transaction record",
    properties: [
        new OA\Property(property: "id", type: "string", description: "Transaction ID"),
        new OA\Property(property: "type", type: "string", description: "Transaction type"),
        new OA\Property(property: "amount", type: "number", description: "Amount"),
        new OA\Property(property: "currency", type: "string", description: "Currency code")
    ]
)]
class WalletTransaction {}
```

Reference it in responses:

```php
responses: [
    new OA\Response(
        response: 200,
        description: "List of transactions",
        content: new OA\JsonContent(
            properties: [
                new OA\Property(property: "success", type: "boolean"),
                new OA\Property(
                    property: "data",
                    type: "array",
                    items: new OA\Items(ref: "#/components/schemas/WalletTransaction")
                )
            ]
        )
    )
]
```

## File Structure

```
api/
├── generate-openapi.php     # Scans PHP files and generates OpenAPI JSON
├── class/
│   ├── actualize_wallet.php # API class with OA attributes
│   ├── event.php
│   └── event.json           # Legacy JSON definitions (optional)
classes/
├── admin.php                # Contains updateDocs() function
├── validator.php            # Type definitions mapped to OpenAPI types
sites/home/
└── core.json                # Core API path definitions
```

## Type Mappings

The system maps validator types to OpenAPI types:

| OpenAPI Type | Validator Types                                                                 |
| ------------ | ------------------------------------------------------------------------------- |
| `string`     | string, string_unsafe, html, email, password, url, id, text, hexcolor, textarea |
| `number`     | float, latlng                                                                   |
| `integer`    | int, timestamp                                                                  |
| `boolean`    | bool                                                                            |
| `array`      | array, multiimage, links, tag                                                   |
| `object`     | point, link, attachments, image, object, geotext, geocode, coords               |

## Standalone OpenAPI Generation

To generate the OpenAPI spec without publishing:

```bash
# Output JSON to stdout
php api/generate-openapi.php

# Output YAML
php api/generate-openapi.php --yaml

# Save to file
php api/generate-openapi.php > openapi.json
```

## Output Locations

After running `php admin updatedocs`:

* **Local file**: `/var/www/phi/_manage/data/api_schema.json`
* **S3**: `https://s3.amazonaws.com/one-earth/source/{env}/api_schema.json`
* **ReadMe.io**: Synced automatically if configured

## Troubleshooting

### Duplicate JSON Output

If the generated JSON contains two concatenated objects, check that no included PHP files are executing code that outputs JSON. The `generate-openapi.php` script includes files to load classes for reflection - ensure they only contain class definitions.

### Missing Endpoints

1. Ensure the file is in `api/` or `api/class/`
2. Check the file isn't in the exclude list in `generate-openapi.php`
3. Verify the OA attribute syntax is correct

### Validation Errors

Test your spec with:

```bash
npx @redocly/cli lint openapi.json
```
