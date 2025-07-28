# API Reference

## Base URL

```
Production: https://api.laadpaalnodig.nl/v1
Staging: https://staging-api.laadpaalnodig.nl/v1
```

## Authentication

All API requests require authentication using a Bearer token:

```http
Authorization: Bearer YOUR_API_TOKEN
```

## Content Types

- **Request Content-Type**: `application/json`
- **Response Content-Type**: `application/json`

## Standard Response Format

All API responses follow this consistent structure:

```json
{
  "success": boolean,
  "data": any,
  "meta": {
    "timestamp": "ISO 8601 datetime",
    "version": "API version",
    "requestId": "Unique request identifier"
  },
  "errors": [
    {
      "code": "ERROR_CODE",
      "message": "Human readable error message",
      "field": "Field name (for validation errors)"
    }
  ]
}
```

## Applications API

### List Applications

```http
GET /applications
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 10, max: 100) |
| `category` | string | No | Filter by category ID |
| `status` | string | No | Filter by status: `pending`, `approved`, `rejected`, `installed` |
| `location` | string | No | Filter by city or postal code |
| `dateFrom` | string | No | Filter applications created after this date (ISO 8601) |
| `dateTo` | string | No | Filter applications created before this date (ISO 8601) |
| `sortBy` | string | No | Sort field: `createdAt`, `updatedAt`, `name` (default: `createdAt`) |
| `sortOrder` | string | No | Sort order: `asc`, `desc` (default: `desc`) |

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "app_01h123abc456def789",
      "name": "Residential Charging Station",
      "category": {
        "id": "residential",
        "name": "Residential Charging"
      },
      "status": "pending",
      "applicant": {
        "id": "user_01h123xyz789",
        "name": "John Doe",
        "email": "john@example.com"
      },
      "location": {
        "address": "Hoofdstraat 123",
        "city": "Amsterdam",
        "postalCode": "1000AA",
        "country": "NL",
        "coordinates": {
          "lat": 52.3676,
          "lng": 4.9041
        }
      },
      "specifications": {
        "power": 11,
        "connectorType": "Type2",
        "connectorCount": 1,
        "phase": "single"
      },
      "documents": [
        {
          "id": "doc_01h123",
          "type": "electrical_plan",
          "name": "Electrical Installation Plan.pdf",
          "url": "https://docs.example.com/doc_01h123.pdf",
          "uploadedAt": "2024-01-15T10:00:00Z"
        }
      ],
      "timeline": {
        "submitted": "2024-01-15T09:00:00Z",
        "underReview": null,
        "approved": null,
        "installed": null
      },
      "estimatedInstallationDate": "2024-02-15",
      "createdAt": "2024-01-15T09:00:00Z",
      "updatedAt": "2024-01-15T09:00:00Z"
    }
  ],
  "meta": {
    "timestamp": "2024-01-15T12:00:00Z",
    "version": "1.0",
    "requestId": "req_01h123456789",
    "pagination": {
      "total": 156,
      "page": 1,
      "limit": 10,
      "totalPages": 16,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

### Get Application Details

```http
GET /applications/{applicationId}
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `applicationId` | string | Yes | Unique application identifier |

**Response:**

```json
{
  "success": true,
  "data": {
    "id": "app_01h123abc456def789",
    "name": "Residential Charging Station",
    "description": "Installation of home charging station for Tesla Model 3",
    "category": {
      "id": "residential",
      "name": "Residential Charging",
      "requirements": [
        "Property ownership verification",
        "Electrical capacity assessment"
      ]
    },
    "status": "under_review",
    "statusHistory": [
      {
        "status": "submitted",
        "timestamp": "2024-01-15T09:00:00Z",
        "note": "Application submitted"
      },
      {
        "status": "under_review",
        "timestamp": "2024-01-16T14:30:00Z",
        "note": "Technical review initiated",
        "reviewer": "tech_reviewer_01"
      }
    ],
    "applicant": {
      "id": "user_01h123xyz789",
      "name": "John Doe",
      "email": "john@example.com",
      "phone": "+31612345678",
      "company": null,
      "verified": true
    },
    "location": {
      "address": "Hoofdstraat 123",
      "city": "Amsterdam",
      "postalCode": "1000AA",
      "country": "NL",
      "region": "Noord-Holland",
      "coordinates": {
        "lat": 52.3676,
        "lng": 4.9041
      },
      "propertyType": "single_family_home",
      "parkingType": "private_driveway"
    },
    "specifications": {
      "power": 11,
      "voltage": 230,
      "phase": "single",
      "connectorType": "Type2",
      "connectorCount": 1,
      "smartCharging": true,
      "loadBalancing": false,
      "weatherProtection": "IP54"
    },
    "technical": {
      "meterboxCapacity": 3,
      "availableCapacity": 11,
      "cableLength": 7.5,
      "installationMethod": "wall_mounted",
      "earthingSystem": "TN-S"
    },
    "costs": {
      "applicationFee": 50.00,
      "installationCost": 1200.00,
      "materialCost": 800.00,
      "totalCost": 2050.00,
      "currency": "EUR",
      "taxIncluded": true
    },
    "documents": [
      {
        "id": "doc_01h123",
        "type": "electrical_plan",
        "name": "Electrical Installation Plan.pdf",
        "description": "Detailed electrical installation plan",
        "url": "https://docs.example.com/doc_01h123.pdf",
        "fileSize": 1048576,
        "mimeType": "application/pdf",
        "uploadedAt": "2024-01-15T10:00:00Z",
        "status": "approved"
      }
    ],
    "permits": [
      {
        "type": "building_permit",
        "status": "not_required",
        "reason": "Minor electrical work"
      },
      {
        "type": "electrical_permit",
        "status": "required",
        "applicationDate": "2024-01-16T00:00:00Z",
        "expectedApprovalDate": "2024-01-30T00:00:00Z"
      }
    ],
    "timeline": {
      "submitted": "2024-01-15T09:00:00Z",
      "underReview": "2024-01-16T14:30:00Z",
      "approved": null,
      "installationScheduled": null,
      "installed": null,
      "completed": null
    },
    "estimatedDates": {
      "approval": "2024-01-25T00:00:00Z",
      "installation": "2024-02-15T00:00:00Z",
      "completion": "2024-02-15T00:00:00Z"
    },
    "assignedTo": {
      "reviewer": {
        "id": "reviewer_01",
        "name": "Alice Johnson",
        "role": "technical_reviewer"
      },
      "installer": null
    },
    "notes": [
      {
        "id": "note_01",
        "author": "reviewer_01",
        "content": "Electrical capacity looks sufficient. Awaiting building permit verification.",
        "createdAt": "2024-01-16T15:00:00Z",
        "visibility": "internal"
      }
    ],
    "createdAt": "2024-01-15T09:00:00Z",
    "updatedAt": "2024-01-16T15:00:00Z"
  }
}
```

### Create Application

```http
POST /applications
```

**Request Body:**

```json
{
  "name": "Home Charging Station Installation",
  "description": "Installation of 11kW charging station for daily commute vehicle",
  "category": "residential",
  "location": {
    "address": "Hoofdstraat 123",
    "city": "Amsterdam",
    "postalCode": "1000AA",
    "country": "NL",
    "coordinates": {
      "lat": 52.3676,
      "lng": 4.9041
    },
    "propertyType": "single_family_home",
    "parkingType": "private_driveway"
  },
  "specifications": {
    "power": 11,
    "connectorType": "Type2",
    "connectorCount": 1,
    "smartCharging": true,
    "loadBalancing": false
  },
  "technical": {
    "meterboxCapacity": 3,
    "availableCapacity": 11,
    "cableLength": 7.5,
    "installationMethod": "wall_mounted"
  },
  "contact": {
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "+31612345678",
    "preferredContactMethod": "email"
  },
  "preferences": {
    "installationTimeframe": "within_month",
    "preferredInstaller": null,
    "notes": "Installation should be completed before end of February"
  }
}
```

### Update Application

```http
PUT /applications/{applicationId}
```

**Request Body:** (Partial update supported)

```json
{
  "specifications": {
    "power": 22,
    "connectorCount": 2
  },
  "preferences": {
    "installationTimeframe": "within_two_weeks"
  }
}
```

### Upload Document

```http
POST /applications/{applicationId}/documents
```

**Content-Type:** `multipart/form-data`

**Form Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | file | Yes | Document file (PDF, JPG, PNG) |
| `type` | string | Yes | Document type (see document types below) |
| `description` | string | No | Document description |

**Document Types:**
- `electrical_plan`
- `property_deed`
- `building_permit`
- `electrical_permit`
- `site_photo`
- `meter_photo`
- `other`

### Delete Application

```http
DELETE /applications/{applicationId}
```

**Note:** Applications can only be deleted if status is `draft` or `pending`.

## Categories API

### List Categories

```http
GET /categories
```

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "residential",
      "name": "Residential Charging",
      "description": "Home charging solutions for private properties",
      "icon": "home",
      "requirements": [
        "Property ownership verification",
        "Electrical capacity assessment",
        "Local permit compliance"
      ],
      "averageProcessingTime": "7-14 days",
      "applicationCount": 156,
      "eligibilityCriteria": [
        "Must own the property or have landlord permission",
        "Electrical system must support required load",
        "Installation site must be accessible"
      ],
      "supportedSpecifications": {
        "powerRange": {
          "min": 3.7,
          "max": 22,
          "unit": "kW"
        },
        "connectorTypes": ["Type1", "Type2"],
        "phases": ["single", "three"]
      }
    },
    {
      "id": "commercial",
      "name": "Commercial Charging",
      "description": "Business and public charging infrastructure",
      "icon": "building",
      "requirements": [
        "Business registration",
        "Site accessibility plan",
        "Insurance coverage",
        "Environmental impact assessment"
      ],
      "averageProcessingTime": "14-30 days",
      "applicationCount": 89,
      "eligibilityCriteria": [
        "Valid business registration",
        "Appropriate zoning for commercial use",
        "Public accessibility compliance"
      ],
      "supportedSpecifications": {
        "powerRange": {
          "min": 7.4,
          "max": 350,
          "unit": "kW"
        },
        "connectorTypes": ["Type2", "CCS", "CHAdeMO"],
        "phases": ["three"]
      }
    }
  ]
}
```

### Get Category Details

```http
GET /categories/{categoryId}
```

**Response:** (Same as individual category object above with additional statistics)

```json
{
  "success": true,
  "data": {
    "id": "residential",
    "name": "Residential Charging",
    "description": "Home charging solutions for private properties",
    "icon": "home",
    "requirements": [...],
    "statistics": {
      "totalApplications": 156,
      "monthlyApplications": 23,
      "statusBreakdown": {
        "pending": 12,
        "under_review": 8,
        "approved": 89,
        "rejected": 3,
        "installed": 44
      },
      "averageApprovalTime": 9.5,
      "satisfactionRating": 4.7
    },
    "pricing": {
      "applicationFee": 50.00,
      "averageInstallationCost": 1200.00,
      "currency": "EUR"
    },
    "installers": [
      {
        "id": "installer_01",
        "name": "EV Install Pro",
        "rating": 4.8,
        "serviceAreas": ["Amsterdam", "Utrecht", "Den Haag"],
        "certifications": ["NEN3140", "VCA"]
      }
    ]
  }
}
```

## Users API

### Get Current User Profile

```http
GET /users/profile
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": "user_01h123xyz789",
    "email": "john@example.com",
    "name": "John Doe",
    "role": "applicant",
    "verified": true,
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "phone": "+31612345678",
      "company": null,
      "title": null,
      "dateOfBirth": "1985-06-15",
      "nationality": "NL"
    },
    "address": {
      "street": "Hoofdstraat 123",
      "city": "Amsterdam",
      "postalCode": "1000AA",
      "country": "NL"
    },
    "preferences": {
      "language": "nl",
      "notifications": {
        "email": true,
        "sms": false,
        "push": true
      },
      "marketing": false
    },
    "verification": {
      "email": {
        "verified": true,
        "verifiedAt": "2023-12-01T10:00:00Z"
      },
      "phone": {
        "verified": true,
        "verifiedAt": "2023-12-01T10:30:00Z"
      },
      "identity": {
        "verified": true,
        "verifiedAt": "2023-12-01T11:00:00Z",
        "method": "digid"
      }
    },
    "applications": [
      {
        "id": "app_01h123abc456def789",
        "name": "Home Charging Station",
        "status": "approved",
        "category": "residential",
        "createdAt": "2024-01-15T09:00:00Z"
      }
    ],
    "statistics": {
      "totalApplications": 1,
      "approvedApplications": 1,
      "installedStations": 1
    },
    "createdAt": "2023-12-01T09:00:00Z",
    "updatedAt": "2024-01-15T09:00:00Z"
  }
}
```

### Update User Profile

```http
PUT /users/profile
```

**Request Body:**

```json
{
  "profile": {
    "firstName": "John",
    "lastName": "Smith",
    "phone": "+31687654321"
  },
  "preferences": {
    "language": "en",
    "notifications": {
      "email": true,
      "sms": true
    }
  }
}
```

## Charging Stations API

### List Charging Stations

```http
GET /charging-stations
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `lat` | number | No | Latitude for proximity search |
| `lng` | number | No | Longitude for proximity search |
| `radius` | number | No | Search radius in kilometers (default: 10) |
| `location` | string | No | City, postal code, or address |
| `available` | boolean | No | Filter by availability |
| `connectorType` | string | No | Filter by connector type |
| `minPower` | number | No | Minimum power rating (kW) |
| `maxPower` | number | No | Maximum power rating (kW) |
| `operator` | string | No | Filter by operator ID |
| `accessType` | string | No | `public`, `private`, `semi_public` |
| `paymentMethods` | string | No | Comma-separated list: `card`, `app`, `subscription` |
| `amenities` | string | No | Comma-separated list: `parking`, `restroom`, `food`, `wifi` |

**Response:**

```json
{
  "success": true,
  "data": [
    {
      "id": "station_01h123",
      "name": "Amsterdam Central Station",
      "operator": {
        "id": "op_fastcharge",
        "name": "FastCharge Netherlands",
        "logo": "https://cdn.example.com/logos/fastcharge.png"
      },
      "location": {
        "address": "Stationsplein 1",
        "city": "Amsterdam",
        "postalCode": "1012AB",
        "country": "NL",
        "coordinates": {
          "lat": 52.3791,
          "lng": 4.9003
        }
      },
      "accessType": "public",
      "status": "operational",
      "connectors": [
        {
          "id": "conn_01",
          "type": "Type2",
          "power": 22,
          "voltage": 400,
          "current": 32,
          "phase": "three",
          "status": "available",
          "lastStatusUpdate": "2024-01-15T11:45:00Z"
        },
        {
          "id": "conn_02",
          "type": "CCS",
          "power": 50,
          "voltage": 400,
          "current": 125,
          "phase": "dc",
          "status": "charging",
          "lastStatusUpdate": "2024-01-15T11:30:00Z"
        }
      ],
      "pricing": {
        "structure": "per_kwh",
        "rates": [
          {
            "connectorType": "Type2",
            "rate": 0.35,
            "currency": "EUR",
            "unit": "kWh"
          },
          {
            "connectorType": "CCS",
            "rate": 0.55,
            "currency": "EUR",
            "unit": "kWh"
          }
        ],
        "startingFee": 0.35,
        "parkingFee": {
          "rate": 2.50,
          "unit": "hour",
          "freeMinutes": 15
        }
      },
      "paymentMethods": ["card", "app", "subscription"],
      "amenities": ["parking", "restroom", "food", "wifi"],
      "openingHours": {
        "monday": "00:00-23:59",
        "tuesday": "00:00-23:59",
        "wednesday": "00:00-23:59",
        "thursday": "00:00-23:59",
        "friday": "00:00-23:59",
        "saturday": "00:00-23:59",
        "sunday": "00:00-23:59"
      },
      "distance": 2.3,
      "lastUpdate": "2024-01-15T11:45:00Z"
    }
  ],
  "meta": {
    "searchCenter": {
      "lat": 52.3676,
      "lng": 4.9041
    },
    "radiusKm": 10,
    "totalResults": 47
  }
}
```

### Get Charging Station Details

```http
GET /charging-stations/{stationId}
```

**Response:**

```json
{
  "success": true,
  "data": {
    "id": "station_01h123",
    "name": "Amsterdam Central Station",
    "description": "Fast charging hub at central train station",
    "operator": {
      "id": "op_fastcharge",
      "name": "FastCharge Netherlands",
      "logo": "https://cdn.example.com/logos/fastcharge.png",
      "contact": {
        "phone": "+31201234567",
        "email": "support@fastcharge.nl",
        "website": "https://fastcharge.nl"
      }
    },
    "location": {
      "address": "Stationsplein 1",
      "city": "Amsterdam",
      "postalCode": "1012AB",
      "country": "NL",
      "region": "Noord-Holland",
      "coordinates": {
        "lat": 52.3791,
        "lng": 4.9003
      },
      "parkingSpaces": 8,
      "accessInstructions": "Enter parking garage from Stationsplein side"
    },
    "accessType": "public",
    "status": "operational",
    "connectors": [
      {
        "id": "conn_01",
        "name": "Connector 1",
        "type": "Type2",
        "power": 22,
        "voltage": 400,
        "current": 32,
        "phase": "three",
        "status": "available",
        "capabilities": ["smart_charging", "load_balancing"],
        "lastStatusUpdate": "2024-01-15T11:45:00Z",
        "usageStatistics": {
          "sessionsToday": 5,
          "averageSessionDuration": 45,
          "utilizationRate": 0.68
        }
      }
    ],
    "capabilities": [
      "reservation",
      "remote_start",
      "smart_charging",
      "load_balancing",
      "renewable_energy"
    ],
    "pricing": {
      "structure": "per_kwh",
      "rates": [...],
      "tariffDetails": {
        "name": "Standard Public Charging",
        "validFrom": "2024-01-01T00:00:00Z",
        "validTo": "2024-12-31T23:59:59Z"
      }
    },
    "paymentMethods": ["card", "app", "subscription"],
    "amenities": [
      {
        "type": "parking",
        "description": "Free parking during charging"
      },
      {
        "type": "restroom",
        "description": "Public restrooms in station building"
      },
      {
        "type": "food",
        "description": "Restaurants and cafes in station"
      },
      {
        "type": "wifi",
        "description": "Free WiFi available"
      }
    ],
    "openingHours": {
      "monday": "00:00-23:59",
      "tuesday": "00:00-23:59",
      "wednesday": "00:00-23:59",
      "thursday": "00:00-23:59",
      "friday": "00:00-23:59",
      "saturday": "00:00-23:59",
      "sunday": "00:00-23:59",
      "timezone": "Europe/Amsterdam"
    },
    "photos": [
      {
        "id": "photo_01",
        "url": "https://cdn.example.com/stations/station_01h123_1.jpg",
        "description": "Main entrance view",
        "takenAt": "2024-01-10T14:00:00Z"
      }
    ],
    "reviews": {
      "averageRating": 4.3,
      "totalReviews": 127,
      "ratingBreakdown": {
        "5": 56,
        "4": 41,
        "3": 22,
        "2": 6,
        "1": 2
      }
    },
    "statistics": {
      "totalSessions": 1247,
      "totalEnergyDelivered": 15678.5,
      "averageSessionDuration": 52,
      "utilizationRate": 0.73,
      "uptime": 0.98
    },
    "createdAt": "2023-08-15T00:00:00Z",
    "lastUpdate": "2024-01-15T11:45:00Z"
  }
}
```

## Webhooks API

### List Webhooks

```http
GET /webhooks
```

### Create Webhook

```http
POST /webhooks
```

**Request Body:**

```json
{
  "url": "https://your-app.com/webhooks/charging-stations",
  "events": [
    "application.status_changed",
    "application.approved",
    "installation.scheduled",
    "installation.completed"
  ],
  "secret": "webhook_secret_key",
  "description": "Application status updates"
}
```

### Webhook Events

| Event | Description | Payload |
|-------|-------------|---------|
| `application.created` | New application submitted | Application object |
| `application.status_changed` | Application status updated | Application object with previous status |
| `application.approved` | Application approved | Application object |
| `application.rejected` | Application rejected | Application object with rejection reason |
| `installation.scheduled` | Installation date scheduled | Application object with installation details |
| `installation.completed` | Installation completed | Application object with completion details |
| `payment.processed` | Payment successfully processed | Payment object |
| `payment.failed` | Payment failed | Payment object with error details |

## Rate Limits

| Endpoint Category | Limit | Window |
|------------------|-------|---------|
| Authentication | 10 requests | 1 minute |
| Read operations | 1000 requests | 1 hour |
| Write operations | 100 requests | 1 hour |
| File uploads | 20 requests | 1 hour |

Rate limit headers:
- `X-RateLimit-Limit`: Request limit per time window
- `X-RateLimit-Remaining`: Remaining requests in current window
- `X-RateLimit-Reset`: Unix timestamp when the rate limit resets

## Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | `INVALID_REQUEST` | Request format is invalid |
| 401 | `UNAUTHORIZED` | Authentication required |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `CONFLICT` | Resource conflict |
| 422 | `VALIDATION_ERROR` | Input validation failed |
| 429 | `RATE_LIMITED` | Rate limit exceeded |
| 500 | `INTERNAL_ERROR` | Internal server error |
| 503 | `SERVICE_UNAVAILABLE` | Service temporarily unavailable |

## Pagination

All list endpoints support pagination using the following parameters:

- `page`: Page number (starts from 1)
- `limit`: Number of items per page (max 100)

Pagination metadata is included in the response:

```json
{
  "meta": {
    "pagination": {
      "total": 156,
      "page": 1,
      "limit": 10,
      "totalPages": 16,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

## Filtering and Sorting

Most list endpoints support filtering and sorting:

**Common Filter Parameters:**
- `search`: Free text search
- `status`: Filter by status
- `dateFrom`/`dateTo`: Date range filtering
- `category`: Filter by category

**Sorting:**
- `sortBy`: Field to sort by
- `sortOrder`: `asc` or `desc`

Example:
```http
GET /applications?status=pending&sortBy=createdAt&sortOrder=desc&page=2&limit=20
```