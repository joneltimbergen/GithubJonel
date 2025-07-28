# SDK Documentation

## Overview

The Charging Station API SDK provides easy-to-use client libraries for multiple programming languages. This document covers installation, configuration, and usage of the SDK.

## Supported Languages

- [JavaScript/Node.js](#javascript--nodejs)
- [Python](#python)
- [PHP](#php)
- [Java](#java)
- [C#/.NET](#c--net)
- [Go](#go)

---

## JavaScript / Node.js

### Installation

```bash
npm install @laadpaalnodig/api-client
```

### Quick Start

```javascript
const { LaadpaalAPI } = require('@laadpaalnodig/api-client');

const api = new LaadpaalAPI({
  apiKey: 'your-api-key',
  environment: 'production' // or 'staging'
});

// Get applications
const applications = await api.applications.list({
  status: 'pending',
  limit: 10
});

console.log(applications.data);
```

### Configuration

```javascript
const api = new LaadpaalAPI({
  apiKey: 'your-api-key',
  environment: 'production',
  timeout: 30000,
  retries: 3,
  baseUrl: 'https://custom-api.example.com', // optional override
  userAgent: 'MyApp/1.0.0'
});
```

### Applications API

#### List Applications

```javascript
const applications = await api.applications.list({
  page: 1,
  limit: 20,
  status: 'pending',
  category: 'residential',
  location: 'Amsterdam',
  dateFrom: '2024-01-01',
  dateTo: '2024-12-31',
  sortBy: 'createdAt',
  sortOrder: 'desc'
});
```

#### Get Application Details

```javascript
const application = await api.applications.get('app_01h123abc456def789');
console.log(application.data.status);
```

#### Create Application

```javascript
const newApplication = await api.applications.create({
  name: 'Home Charging Station',
  category: 'residential',
  location: {
    address: 'Hoofdstraat 123',
    city: 'Amsterdam',
    postalCode: '1000AA',
    coordinates: {
      lat: 52.3676,
      lng: 4.9041
    }
  },
  specifications: {
    power: 11,
    connectorType: 'Type2',
    connectorCount: 1
  },
  contact: {
    name: 'John Doe',
    email: 'john@example.com',
    phone: '+31612345678'
  }
});
```

#### Update Application

```javascript
const updatedApplication = await api.applications.update('app_01h123', {
  specifications: {
    power: 22,
    connectorCount: 2
  }
});
```

#### Upload Document

```javascript
const fs = require('fs');

const document = await api.applications.uploadDocument('app_01h123', {
  file: fs.createReadStream('./electrical-plan.pdf'),
  type: 'electrical_plan',
  description: 'Electrical installation plan'
});
```

### Categories API

```javascript
// List all categories
const categories = await api.categories.list();

// Get category details
const category = await api.categories.get('residential');

// Get applications for category
const categoryApplications = await api.categories.getApplications('residential', {
  status: 'approved'
});
```

### Charging Stations API

```javascript
// Find nearby stations
const stations = await api.chargingStations.list({
  lat: 52.3676,
  lng: 4.9041,
  radius: 10,
  available: true,
  connectorType: 'Type2'
});

// Get station details
const station = await api.chargingStations.get('station_01h123');
```

### User Management

```javascript
// Get current user profile
const profile = await api.users.getProfile();

// Update profile
const updatedProfile = await api.users.updateProfile({
  name: 'John Smith',
  phone: '+31687654321',
  preferences: {
    language: 'en',
    notifications: { email: true, sms: false }
  }
});
```

### Error Handling

```javascript
try {
  const application = await api.applications.get('invalid-id');
} catch (error) {
  if (error instanceof api.NotFoundError) {
    console.log('Application not found');
  } else if (error instanceof api.ValidationError) {
    console.log('Validation errors:', error.errors);
  } else if (error instanceof api.RateLimitError) {
    console.log('Rate limit exceeded. Retry after:', error.retryAfter);
  } else {
    console.log('API error:', error.message);
  }
}
```

### Webhooks

```javascript
// Create webhook
const webhook = await api.webhooks.create({
  url: 'https://your-app.com/webhooks',
  events: ['application.status_changed', 'installation.completed'],
  secret: 'your-webhook-secret'
});

// Verify webhook signature (Express.js middleware)
const { verifyWebhook } = require('@laadpaalnodig/api-client');

app.use('/webhooks', verifyWebhook('your-webhook-secret'), (req, res) => {
  const event = req.body;
  console.log('Webhook event:', event.type, event.data);
  res.status(200).send('OK');
});
```

### Async Iterators

```javascript
// Iterate through all applications
for await (const application of api.applications.iterate()) {
  console.log(application.name, application.status);
}

// With filters
for await (const station of api.chargingStations.iterate({ available: true })) {
  console.log(station.name, station.location.city);
}
```

---

## Python

### Installation

```bash
pip install laadpaal-api-client
```

### Quick Start

```python
from laadpaal_api import LaadpaalAPI

api = LaadpaalAPI(
    api_key='your-api-key',
    environment='production'
)

# Get applications
applications = api.applications.list(status='pending', limit=10)
print(applications.data)
```

### Configuration

```python
api = LaadpaalAPI(
    api_key='your-api-key',
    environment='production',
    timeout=30,
    retries=3,
    base_url='https://custom-api.example.com',  # optional override
    user_agent='MyApp/1.0.0'
)
```

### Applications API

```python
# List applications with filters
applications = api.applications.list(
    page=1,
    limit=20,
    status='pending',
    category='residential',
    location='Amsterdam',
    date_from='2024-01-01',
    date_to='2024-12-31',
    sort_by='created_at',
    sort_order='desc'
)

# Get application details
application = api.applications.get('app_01h123abc456def789')

# Create application
new_application = api.applications.create({
    'name': 'Home Charging Station',
    'category': 'residential',
    'location': {
        'address': 'Hoofdstraat 123',
        'city': 'Amsterdam',
        'postal_code': '1000AA',
        'coordinates': {'lat': 52.3676, 'lng': 4.9041}
    },
    'specifications': {
        'power': 11,
        'connector_type': 'Type2',
        'connector_count': 1
    },
    'contact': {
        'name': 'John Doe',
        'email': 'john@example.com',
        'phone': '+31612345678'
    }
})

# Upload document
with open('electrical-plan.pdf', 'rb') as f:
    document = api.applications.upload_document(
        'app_01h123',
        file=f,
        type='electrical_plan',
        description='Electrical installation plan'
    )
```

### Error Handling

```python
from laadpaal_api.exceptions import (
    NotFoundError, ValidationError, RateLimitError, APIError
)

try:
    application = api.applications.get('invalid-id')
except NotFoundError:
    print('Application not found')
except ValidationError as e:
    print('Validation errors:', e.errors)
except RateLimitError as e:
    print(f'Rate limit exceeded. Retry after: {e.retry_after}')
except APIError as e:
    print(f'API error: {e.message}')
```

### Async Support

```python
import asyncio
from laadpaal_api import AsyncLaadpaalAPI

async def main():
    api = AsyncLaadpaalAPI(api_key='your-api-key')
    
    applications = await api.applications.list(status='pending')
    print(applications.data)
    
    await api.close()

asyncio.run(main())
```

### Data Classes

```python
from laadpaal_api.models import Application, Location, Specifications

# Type-safe application creation
application_data = Application(
    name='Home Charging Station',
    category='residential',
    location=Location(
        address='Hoofdstraat 123',
        city='Amsterdam',
        postal_code='1000AA',
        coordinates={'lat': 52.3676, 'lng': 4.9041}
    ),
    specifications=Specifications(
        power=11,
        connector_type='Type2',
        connector_count=1
    )
)

result = api.applications.create(application_data)
```

---

## PHP

### Installation

```bash
composer require laadpaalnodig/api-client
```

### Quick Start

```php
<?php
require_once 'vendor/autoload.php';

use LaadpaalNodig\ApiClient\LaadpaalAPI;

$api = new LaadpaalAPI([
    'api_key' => 'your-api-key',
    'environment' => 'production'
]);

// Get applications
$applications = $api->applications()->list([
    'status' => 'pending',
    'limit' => 10
]);

print_r($applications->data);
```

### Configuration

```php
$api = new LaadpaalAPI([
    'api_key' => 'your-api-key',
    'environment' => 'production',
    'timeout' => 30,
    'retries' => 3,
    'base_url' => 'https://custom-api.example.com', // optional
    'user_agent' => 'MyApp/1.0.0'
]);
```

### Applications API

```php
// List applications
$applications = $api->applications()->list([
    'page' => 1,
    'limit' => 20,
    'status' => 'pending',
    'category' => 'residential',
    'location' => 'Amsterdam',
    'date_from' => '2024-01-01',
    'date_to' => '2024-12-31',
    'sort_by' => 'created_at',
    'sort_order' => 'desc'
]);

// Get application
$application = $api->applications()->get('app_01h123abc456def789');

// Create application
$newApplication = $api->applications()->create([
    'name' => 'Home Charging Station',
    'category' => 'residential',
    'location' => [
        'address' => 'Hoofdstraat 123',
        'city' => 'Amsterdam',
        'postal_code' => '1000AA',
        'coordinates' => ['lat' => 52.3676, 'lng' => 4.9041]
    ],
    'specifications' => [
        'power' => 11,
        'connector_type' => 'Type2',
        'connector_count' => 1
    ],
    'contact' => [
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'phone' => '+31612345678'
    ]
]);

// Upload document
$document = $api->applications()->uploadDocument('app_01h123', [
    'file' => new CURLFile('electrical-plan.pdf'),
    'type' => 'electrical_plan',
    'description' => 'Electrical installation plan'
]);
```

### Error Handling

```php
use LaadpaalNodig\ApiClient\Exceptions\{
    NotFoundException, ValidationException, RateLimitException, ApiException
};

try {
    $application = $api->applications()->get('invalid-id');
} catch (NotFoundException $e) {
    echo 'Application not found';
} catch (ValidationException $e) {
    echo 'Validation errors: ' . print_r($e->getErrors(), true);
} catch (RateLimitException $e) {
    echo 'Rate limit exceeded. Retry after: ' . $e->getRetryAfter();
} catch (ApiException $e) {
    echo 'API error: ' . $e->getMessage();
}
```

---

## Java

### Installation

Maven:
```xml
<dependency>
    <groupId>nl.laadpaalnodig</groupId>
    <artifactId>api-client</artifactId>
    <version>1.0.0</version>
</dependency>
```

Gradle:
```gradle
implementation 'nl.laadpaalnodig:api-client:1.0.0'
```

### Quick Start

```java
import nl.laadpaalnodig.client.LaadpaalAPI;
import nl.laadpaalnodig.client.models.*;

LaadpaalAPI api = new LaadpaalAPI.Builder()
    .apiKey("your-api-key")
    .environment(Environment.PRODUCTION)
    .build();

// Get applications
ApplicationList applications = api.applications()
    .list(ApplicationListRequest.builder()
        .status("pending")
        .limit(10)
        .build());

System.out.println(applications.getData());
```

### Configuration

```java
LaadpaalAPI api = new LaadpaalAPI.Builder()
    .apiKey("your-api-key")
    .environment(Environment.PRODUCTION)
    .timeout(Duration.ofSeconds(30))
    .retries(3)
    .baseUrl("https://custom-api.example.com") // optional
    .userAgent("MyApp/1.0.0")
    .build();
```

### Applications API

```java
// List applications
ApplicationList applications = api.applications()
    .list(ApplicationListRequest.builder()
        .page(1)
        .limit(20)
        .status("pending")
        .category("residential")
        .location("Amsterdam")
        .dateFrom(LocalDate.of(2024, 1, 1))
        .dateTo(LocalDate.of(2024, 12, 31))
        .sortBy("createdAt")
        .sortOrder("desc")
        .build());

// Get application
Application application = api.applications().get("app_01h123abc456def789");

// Create application
Application newApplication = api.applications()
    .create(CreateApplicationRequest.builder()
        .name("Home Charging Station")
        .category("residential")
        .location(Location.builder()
            .address("Hoofdstraat 123")
            .city("Amsterdam")
            .postalCode("1000AA")
            .coordinates(Coordinates.builder()
                .lat(52.3676)
                .lng(4.9041)
                .build())
            .build())
        .specifications(Specifications.builder()
            .power(11)
            .connectorType("Type2")
            .connectorCount(1)
            .build())
        .contact(Contact.builder()
            .name("John Doe")
            .email("john@example.com")
            .phone("+31612345678")
            .build())
        .build());
```

### Error Handling

```java
import nl.laadpaalnodig.client.exceptions.*;

try {
    Application application = api.applications().get("invalid-id");
} catch (NotFoundException e) {
    System.out.println("Application not found");
} catch (ValidationException e) {
    System.out.println("Validation errors: " + e.getErrors());
} catch (RateLimitException e) {
    System.out.println("Rate limit exceeded. Retry after: " + e.getRetryAfter());
} catch (ApiException e) {
    System.out.println("API error: " + e.getMessage());
}
```

### Async Support

```java
import java.util.concurrent.CompletableFuture;

// Async API client
AsyncLaadpaalAPI asyncApi = new AsyncLaadpaalAPI.Builder()
    .apiKey("your-api-key")
    .build();

CompletableFuture<ApplicationList> futureApplications = 
    asyncApi.applications().listAsync(ApplicationListRequest.builder()
        .status("pending")
        .build());

futureApplications.thenAccept(applications -> {
    System.out.println("Received " + applications.getData().size() + " applications");
});
```

---

## C# / .NET

### Installation

```bash
dotnet add package LaadpaalNodig.ApiClient
```

### Quick Start

```csharp
using LaadpaalNodig.ApiClient;
using LaadpaalNodig.ApiClient.Models;

var api = new LaadpaalAPI(new LaadpaalAPIConfiguration
{
    ApiKey = "your-api-key",
    Environment = ApiEnvironment.Production
});

// Get applications
var applications = await api.Applications.ListAsync(new ApplicationListRequest
{
    Status = "pending",
    Limit = 10
});

Console.WriteLine($"Found {applications.Data.Count} applications");
```

### Configuration

```csharp
var api = new LaadpaalAPI(new LaadpaalAPIConfiguration
{
    ApiKey = "your-api-key",
    Environment = ApiEnvironment.Production,
    Timeout = TimeSpan.FromSeconds(30),
    Retries = 3,
    BaseUrl = "https://custom-api.example.com", // optional
    UserAgent = "MyApp/1.0.0"
});
```

### Applications API

```csharp
// List applications
var applications = await api.Applications.ListAsync(new ApplicationListRequest
{
    Page = 1,
    Limit = 20,
    Status = "pending",
    Category = "residential",
    Location = "Amsterdam",
    DateFrom = new DateTime(2024, 1, 1),
    DateTo = new DateTime(2024, 12, 31),
    SortBy = "createdAt",
    SortOrder = "desc"
});

// Get application
var application = await api.Applications.GetAsync("app_01h123abc456def789");

// Create application
var newApplication = await api.Applications.CreateAsync(new CreateApplicationRequest
{
    Name = "Home Charging Station",
    Category = "residential",
    Location = new Location
    {
        Address = "Hoofdstraat 123",
        City = "Amsterdam",
        PostalCode = "1000AA",
        Coordinates = new Coordinates { Lat = 52.3676, Lng = 4.9041 }
    },
    Specifications = new Specifications
    {
        Power = 11,
        ConnectorType = "Type2",
        ConnectorCount = 1
    },
    Contact = new Contact
    {
        Name = "John Doe",
        Email = "john@example.com",
        Phone = "+31612345678"
    }
});

// Upload document
using var fileStream = File.OpenRead("electrical-plan.pdf");
var document = await api.Applications.UploadDocumentAsync("app_01h123", new UploadDocumentRequest
{
    File = fileStream,
    Type = "electrical_plan",
    Description = "Electrical installation plan"
});
```

### Error Handling

```csharp
using LaadpaalNodig.ApiClient.Exceptions;

try
{
    var application = await api.Applications.GetAsync("invalid-id");
}
catch (NotFoundException)
{
    Console.WriteLine("Application not found");
}
catch (ValidationException ex)
{
    Console.WriteLine($"Validation errors: {string.Join(", ", ex.Errors.Select(e => e.Message))}");
}
catch (RateLimitException ex)
{
    Console.WriteLine($"Rate limit exceeded. Retry after: {ex.RetryAfter}");
}
catch (ApiException ex)
{
    Console.WriteLine($"API error: {ex.Message}");
}
```

---

## Go

### Installation

```bash
go get github.com/laadpaalnodig/api-client-go
```

### Quick Start

```go
package main

import (
    "context"
    "fmt"
    "log"
    
    "github.com/laadpaalnodig/api-client-go"
)

func main() {
    client := laadpaal.NewClient("your-api-key", laadpaal.Production)
    
    // Get applications
    applications, err := client.Applications.List(context.Background(), &laadpaal.ApplicationListParams{
        Status: "pending",
        Limit:  10,
    })
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("Found %d applications\n", len(applications.Data))
}
```

### Configuration

```go
client := laadpaal.NewClient("your-api-key", laadpaal.Production, &laadpaal.Config{
    Timeout:   30 * time.Second,
    Retries:   3,
    BaseURL:   "https://custom-api.example.com", // optional
    UserAgent: "MyApp/1.0.0",
})
```

### Applications API

```go
// List applications
applications, err := client.Applications.List(ctx, &laadpaal.ApplicationListParams{
    Page:      1,
    Limit:     20,
    Status:    "pending",
    Category:  "residential",
    Location:  "Amsterdam",
    DateFrom:  time.Date(2024, 1, 1, 0, 0, 0, 0, time.UTC),
    DateTo:    time.Date(2024, 12, 31, 0, 0, 0, 0, time.UTC),
    SortBy:    "createdAt",
    SortOrder: "desc",
})

// Get application
application, err := client.Applications.Get(ctx, "app_01h123abc456def789")

// Create application
newApplication, err := client.Applications.Create(ctx, &laadpaal.CreateApplicationParams{
    Name:     "Home Charging Station",
    Category: "residential",
    Location: &laadpaal.Location{
        Address:    "Hoofdstraat 123",
        City:       "Amsterdam",
        PostalCode: "1000AA",
        Coordinates: &laadpaal.Coordinates{
            Lat: 52.3676,
            Lng: 4.9041,
        },
    },
    Specifications: &laadpaal.Specifications{
        Power:          11,
        ConnectorType:  "Type2",
        ConnectorCount: 1,
    },
    Contact: &laadpaal.Contact{
        Name:  "John Doe",
        Email: "john@example.com",
        Phone: "+31612345678",
    },
})

// Upload document
file, _ := os.Open("electrical-plan.pdf")
defer file.Close()

document, err := client.Applications.UploadDocument(ctx, "app_01h123", &laadpaal.UploadDocumentParams{
    File:        file,
    Type:        "electrical_plan",
    Description: "Electrical installation plan",
})
```

### Error Handling

```go
import "github.com/laadpaalnodig/api-client-go/errors"

application, err := client.Applications.Get(ctx, "invalid-id")
if err != nil {
    switch e := err.(type) {
    case *errors.NotFoundError:
        fmt.Println("Application not found")
    case *errors.ValidationError:
        fmt.Printf("Validation errors: %+v\n", e.Errors)
    case *errors.RateLimitError:
        fmt.Printf("Rate limit exceeded. Retry after: %v\n", e.RetryAfter)
    default:
        fmt.Printf("API error: %v\n", err)
    }
}
```

## Common Patterns

### Pagination Helper

```javascript
// JavaScript - Auto-paginate through all results
async function getAllApplications(api, filters = {}) {
    const allApplications = [];
    let page = 1;
    let hasMore = true;
    
    while (hasMore) {
        const response = await api.applications.list({
            ...filters,
            page,
            limit: 100
        });
        
        allApplications.push(...response.data);
        hasMore = response.meta.pagination.hasNext;
        page++;
    }
    
    return allApplications;
}
```

### Retry Logic

```python
# Python - Exponential backoff retry
import time
import random

def api_call_with_retry(api_func, max_retries=3):
    for attempt in range(max_retries + 1):
        try:
            return api_func()
        except RateLimitError as e:
            if attempt == max_retries:
                raise
            wait_time = (2 ** attempt) + random.uniform(0, 1)
            time.sleep(wait_time)
        except Exception as e:
            if attempt == max_retries:
                raise
            time.sleep(1)
```

### Webhook Verification

```php
// PHP - Verify webhook signature
function verifyWebhookSignature($payload, $signature, $secret) {
    $expectedSignature = 'sha256=' . hash_hmac('sha256', $payload, $secret);
    return hash_equals($expectedSignature, $signature);
}

if (verifyWebhookSignature($rawPayload, $_SERVER['HTTP_X_SIGNATURE'], $webhookSecret)) {
    $event = json_decode($rawPayload, true);
    // Process webhook event
}
```

## Best Practices

1. **API Key Security**
   - Store API keys in environment variables
   - Use different keys for development and production
   - Rotate keys regularly

2. **Error Handling**
   - Always handle API errors gracefully
   - Implement exponential backoff for rate limits
   - Log errors for debugging

3. **Performance**
   - Use pagination for large datasets
   - Cache responses when appropriate
   - Use async/await for concurrent requests

4. **Testing**
   - Use staging environment for testing
   - Mock API responses in unit tests
   - Test error scenarios

5. **Monitoring**
   - Monitor API usage and rate limits
   - Set up alerts for failed requests
   - Track application performance metrics