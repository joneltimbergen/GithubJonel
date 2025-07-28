# Electric Vehicle Charging Infrastructure API Documentation

## Overview

This project provides comprehensive APIs for managing electric vehicle charging infrastructure, applications, and related services. The API follows RESTful principles and provides endpoints for charging station management, application categories, and user management.

## Table of Contents

- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [API Endpoints](#api-endpoints)
- [Components](#components)
- [Functions](#functions)
- [Error Handling](#error-handling)
- [Examples](#examples)
- [Development](#development)

## Quick Start

### Prerequisites

- Node.js 16.x or higher
- npm or yarn
- API key (contact administrators)

### Installation

```bash
npm install
npm start
```

### Basic Usage

```javascript
const api = require('./src/api');

// Initialize API client
const client = new api.Client({
  apiKey: 'your-api-key',
  baseUrl: 'https://api.laadpaalnodig.nl'
});

// Get applications by category
const applications = await client.applications.getByCategory('category-id');
```

## Authentication

All API requests require authentication using an API key. Include your API key in the request headers:

```
Authorization: Bearer YOUR_API_KEY
```

## API Endpoints

### Applications API

#### Get Applications by Category

**Endpoint:** `GET /applications/{categoryId}`

Retrieves all applications for a specific category.

**Parameters:**
- `categoryId` (string, required): The unique identifier for the category

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "app-123",
      "name": "Charging Station App",
      "category": "residential",
      "status": "active",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  ],
  "meta": {
    "total": 1,
    "page": 1,
    "limit": 10
  }
}
```

**Example:**
```bash
curl -X GET "https://api.laadpaalnodig.nl/api/applications/residential" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

#### Create Application

**Endpoint:** `POST /applications`

Creates a new charging station application.

**Request Body:**
```json
{
  "name": "New Charging Station",
  "category": "commercial",
  "location": {
    "address": "Main Street 123",
    "city": "Amsterdam",
    "postalCode": "1000AA",
    "coordinates": {
      "lat": 52.3676,
      "lng": 4.9041
    }
  },
  "specifications": {
    "power": 22,
    "connectorType": "Type2",
    "count": 2
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "app-456",
    "name": "New Charging Station",
    "status": "pending",
    "createdAt": "2024-01-15T11:00:00Z"
  }
}
```

### Categories API

#### List Categories

**Endpoint:** `GET /categories`

Retrieves all available application categories.

**Parameters:**
- `page` (integer, optional): Page number (default: 1)
- `limit` (integer, optional): Items per page (default: 10)
- `search` (string, optional): Search term for filtering

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "residential",
      "name": "Residential Charging",
      "description": "Home charging solutions",
      "applicationCount": 150
    },
    {
      "id": "commercial",
      "name": "Commercial Charging",
      "description": "Business and public charging",
      "applicationCount": 89
    }
  ]
}
```

#### Get Category Details

**Endpoint:** `GET /categories/{categoryId}`

Retrieves detailed information about a specific category.

**Parameters:**
- `categoryId` (string, required): The category identifier

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "residential",
    "name": "Residential Charging",
    "description": "Comprehensive home charging solutions",
    "requirements": [
      "Property ownership verification",
      "Electrical capacity assessment",
      "Local permit requirements"
    ],
    "processingTime": "5-10 business days",
    "applicationCount": 150,
    "statistics": {
      "approved": 135,
      "pending": 10,
      "rejected": 5
    }
  }
}
```

### Users API

#### Get User Profile

**Endpoint:** `GET /users/profile`

Retrieves the current user's profile information.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user-789",
    "email": "user@example.com",
    "name": "John Doe",
    "role": "applicant",
    "verified": true,
    "createdAt": "2023-12-01T09:00:00Z",
    "applications": [
      {
        "id": "app-123",
        "status": "approved",
        "category": "residential"
      }
    ]
  }
}
```

#### Update User Profile

**Endpoint:** `PUT /users/profile`

Updates the current user's profile information.

**Request Body:**
```json
{
  "name": "John Smith",
  "phone": "+31612345678",
  "preferences": {
    "notifications": true,
    "language": "nl"
  }
}
```

### Charging Stations API

#### List Charging Stations

**Endpoint:** `GET /charging-stations`

Retrieves a list of charging stations with filtering options.

**Parameters:**
- `location` (string, optional): Filter by location (city, postal code)
- `radius` (number, optional): Search radius in kilometers
- `available` (boolean, optional): Filter by availability
- `connectorType` (string, optional): Filter by connector type
- `minPower` (number, optional): Minimum power rating

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "station-001",
      "name": "City Center Charging Hub",
      "location": {
        "address": "Central Square 1",
        "city": "Amsterdam",
        "coordinates": {
          "lat": 52.3676,
          "lng": 4.9041
        }
      },
      "connectors": [
        {
          "id": "conn-001",
          "type": "Type2",
          "power": 22,
          "status": "available"
        }
      ],
      "pricing": {
        "perKwh": 0.35,
        "currency": "EUR"
      },
      "operator": "ChargePoint NL"
    }
  ]
}
```

## Components

### API Client Component

The main API client for interacting with all endpoints.

```javascript
class APIClient {
  constructor(config) {
    this.apiKey = config.apiKey;
    this.baseUrl = config.baseUrl;
    this.timeout = config.timeout || 30000;
  }

  async request(method, endpoint, data = null) {
    // Implementation details
  }

  // Category methods
  get categories() {
    return new CategoryService(this);
  }

  // Application methods
  get applications() {
    return new ApplicationService(this);
  }

  // User methods
  get users() {
    return new UserService(this);
  }

  // Charging station methods
  get chargingStations() {
    return new ChargingStationService(this);
  }
}
```

### Category Service Component

Handles all category-related operations.

```javascript
class CategoryService {
  constructor(client) {
    this.client = client;
  }

  async list(params = {}) {
    return this.client.request('GET', '/categories', params);
  }

  async get(categoryId) {
    return this.client.request('GET', `/categories/${categoryId}`);
  }

  async getApplications(categoryId, params = {}) {
    return this.client.request('GET', `/categories/${categoryId}/applications`, params);
  }
}
```

### Application Service Component

Manages charging station applications.

```javascript
class ApplicationService {
  constructor(client) {
    this.client = client;
  }

  async create(applicationData) {
    return this.client.request('POST', '/applications', applicationData);
  }

  async get(applicationId) {
    return this.client.request('GET', `/applications/${applicationId}`);
  }

  async update(applicationId, updateData) {
    return this.client.request('PUT', `/applications/${applicationId}`, updateData);
  }

  async delete(applicationId) {
    return this.client.request('DELETE', `/applications/${applicationId}`);
  }

  async getByCategory(categoryId) {
    return this.client.request('GET', `/applications?category=${categoryId}`);
  }
}
```

### Validation Component

Provides data validation utilities.

```javascript
class ValidationUtils {
  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
  }

  static validatePostalCode(postalCode, country = 'NL') {
    const patterns = {
      NL: /^\d{4}\s?[A-Z]{2}$/i,
      DE: /^\d{5}$/,
      BE: /^\d{4}$/
    };
    return patterns[country]?.test(postalCode) || false;
  }

  static validateCoordinates(lat, lng) {
    return lat >= -90 && lat <= 90 && lng >= -180 && lng <= 180;
  }

  static validateApplicationData(data) {
    const errors = [];
    
    if (!data.name || data.name.trim().length < 2) {
      errors.push('Name must be at least 2 characters long');
    }
    
    if (!data.category) {
      errors.push('Category is required');
    }
    
    if (!data.location || !data.location.address) {
      errors.push('Location address is required');
    }
    
    return {
      isValid: errors.length === 0,
      errors
    };
  }
}
```

## Functions

### Utility Functions

#### formatResponse

Standardizes API response format.

```javascript
/**
 * Formats API responses in a consistent structure
 * @param {boolean} success - Whether the operation was successful
 * @param {any} data - The response data
 * @param {Object} meta - Additional metadata
 * @param {Array} errors - Any error messages
 * @returns {Object} Formatted response object
 */
function formatResponse(success, data = null, meta = {}, errors = []) {
  return {
    success,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      ...meta
    },
    errors
  };
}
```

#### parseLocation

Parses and validates location data.

```javascript
/**
 * Parses location string into structured data
 * @param {string} locationString - Location in various formats
 * @returns {Object} Parsed location with coordinates if available
 */
function parseLocation(locationString) {
  // Implementation for parsing addresses, postal codes, coordinates
  const result = {
    original: locationString,
    parsed: {},
    coordinates: null
  };
  
  // Postal code detection
  const postalCodeMatch = locationString.match(/\d{4}\s?[A-Z]{2}/i);
  if (postalCodeMatch) {
    result.parsed.postalCode = postalCodeMatch[0].replace(/\s/, '');
  }
  
  return result;
}
```

#### calculateDistance

Calculates distance between two coordinates.

```javascript
/**
 * Calculates the distance between two geographical points
 * @param {number} lat1 - Latitude of first point
 * @param {number} lng1 - Longitude of first point
 * @param {number} lat2 - Latitude of second point
 * @param {number} lng2 - Longitude of second point
 * @returns {number} Distance in kilometers
 */
function calculateDistance(lat1, lng1, lat2, lng2) {
  const R = 6371; // Earth's radius in kilometers
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLng = (lng2 - lng1) * Math.PI / 180;
  
  const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
    Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
    Math.sin(dLng / 2) * Math.sin(dLng / 2);
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  return R * c;
}
```

### Data Processing Functions

#### filterApplications

Filters applications based on criteria.

```javascript
/**
 * Filters applications based on multiple criteria
 * @param {Array} applications - Array of application objects
 * @param {Object} filters - Filter criteria
 * @returns {Array} Filtered applications
 */
function filterApplications(applications, filters = {}) {
  return applications.filter(app => {
    if (filters.category && app.category !== filters.category) {
      return false;
    }
    
    if (filters.status && app.status !== filters.status) {
      return false;
    }
    
    if (filters.location && !app.location.city.toLowerCase().includes(filters.location.toLowerCase())) {
      return false;
    }
    
    if (filters.dateFrom && new Date(app.createdAt) < new Date(filters.dateFrom)) {
      return false;
    }
    
    if (filters.dateTo && new Date(app.createdAt) > new Date(filters.dateTo)) {
      return false;
    }
    
    return true;
  });
}
```

#### aggregateStatistics

Generates statistical summaries.

```javascript
/**
 * Aggregates statistics from application data
 * @param {Array} applications - Array of application objects
 * @returns {Object} Statistical summary
 */
function aggregateStatistics(applications) {
  const stats = {
    total: applications.length,
    byStatus: {},
    byCategory: {},
    byMonth: {},
    averageProcessingTime: 0
  };
  
  applications.forEach(app => {
    // Count by status
    stats.byStatus[app.status] = (stats.byStatus[app.status] || 0) + 1;
    
    // Count by category
    stats.byCategory[app.category] = (stats.byCategory[app.category] || 0) + 1;
    
    // Count by month
    const month = new Date(app.createdAt).toISOString().slice(0, 7);
    stats.byMonth[month] = (stats.byMonth[month] || 0) + 1;
  });
  
  return stats;
}
```

## Error Handling

### Error Codes

| Code | Description | HTTP Status |
|------|-------------|-------------|
| `INVALID_API_KEY` | API key is missing or invalid | 401 |
| `RESOURCE_NOT_FOUND` | Requested resource does not exist | 404 |
| `VALIDATION_ERROR` | Request data validation failed | 422 |
| `RATE_LIMIT_EXCEEDED` | Too many requests | 429 |
| `INTERNAL_ERROR` | Internal server error | 500 |

### Error Response Format

```json
{
  "success": false,
  "data": null,
  "meta": {
    "timestamp": "2024-01-15T12:00:00Z"
  },
  "errors": [
    {
      "code": "VALIDATION_ERROR",
      "message": "Invalid email format",
      "field": "email"
    }
  ]
}
```

### Error Handling Function

```javascript
/**
 * Handles and formats errors consistently
 * @param {Error} error - The error object
 * @param {string} context - Context where error occurred
 * @returns {Object} Formatted error response
 */
function handleError(error, context = 'unknown') {
  const errorResponse = {
    code: error.code || 'INTERNAL_ERROR',
    message: error.message || 'An unexpected error occurred',
    context
  };
  
  // Log error for debugging
  console.error(`Error in ${context}:`, error);
  
  return formatResponse(false, null, {}, [errorResponse]);
}
```

## Examples

### Complete Application Submission Workflow

```javascript
const { APIClient, ValidationUtils } = require('./src/api');

async function submitApplication() {
  // Initialize client
  const client = new APIClient({
    apiKey: process.env.API_KEY,
    baseUrl: 'https://api.laadpaalnodig.nl'
  });
  
  try {
    // Step 1: Get available categories
    const categoriesResponse = await client.categories.list();
    console.log('Available categories:', categoriesResponse.data);
    
    // Step 2: Prepare application data
    const applicationData = {
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
        count: 1
      },
      contact: {
        name: 'John Doe',
        email: 'john@example.com',
        phone: '+31612345678'
      }
    };
    
    // Step 3: Validate data
    const validation = ValidationUtils.validateApplicationData(applicationData);
    if (!validation.isValid) {
      throw new Error(`Validation failed: ${validation.errors.join(', ')}`);
    }
    
    // Step 4: Submit application
    const response = await client.applications.create(applicationData);
    console.log('Application submitted:', response.data);
    
    // Step 5: Monitor status
    const applicationId = response.data.id;
    const status = await client.applications.get(applicationId);
    console.log('Current status:', status.data.status);
    
    return response.data;
    
  } catch (error) {
    console.error('Application submission failed:', error.message);
    throw error;
  }
}
```

### Searching Nearby Charging Stations

```javascript
async function findNearbyStations(userLocation, maxDistance = 10) {
  const client = new APIClient({
    apiKey: process.env.API_KEY,
    baseUrl: 'https://api.laadpaalnodig.nl'
  });
  
  try {
    // Get all charging stations
    const stationsResponse = await client.chargingStations.list({
      available: true
    });
    
    // Filter by distance
    const nearbyStations = stationsResponse.data.filter(station => {
      const distance = calculateDistance(
        userLocation.lat,
        userLocation.lng,
        station.location.coordinates.lat,
        station.location.coordinates.lng
      );
      return distance <= maxDistance;
    });
    
    // Sort by distance
    nearbyStations.sort((a, b) => {
      const distanceA = calculateDistance(
        userLocation.lat, userLocation.lng,
        a.location.coordinates.lat, a.location.coordinates.lng
      );
      const distanceB = calculateDistance(
        userLocation.lat, userLocation.lng,
        b.location.coordinates.lat, b.location.coordinates.lng
      );
      return distanceA - distanceB;
    });
    
    return nearbyStations;
    
  } catch (error) {
    console.error('Error finding stations:', error.message);
    throw error;
  }
}
```

### Real-time Status Updates

```javascript
const EventSource = require('eventsource');

class StatusMonitor {
  constructor(apiKey, applicationId) {
    this.apiKey = apiKey;
    this.applicationId = applicationId;
    this.eventSource = null;
  }
  
  start() {
    const url = `https://api.laadpaalnodig.nl/applications/${this.applicationId}/events`;
    
    this.eventSource = new EventSource(url, {
      headers: {
        'Authorization': `Bearer ${this.apiKey}`
      }
    });
    
    this.eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.handleStatusUpdate(data);
    };
    
    this.eventSource.onerror = (error) => {
      console.error('EventSource error:', error);
    };
  }
  
  handleStatusUpdate(data) {
    console.log('Status update received:', data);
    
    switch (data.status) {
      case 'under_review':
        console.log('Application is under review');
        break;
      case 'approved':
        console.log('Application approved! Installation can proceed.');
        break;
      case 'rejected':
        console.log('Application rejected:', data.reason);
        break;
      case 'installation_scheduled':
        console.log('Installation scheduled for:', data.scheduledDate);
        break;
    }
  }
  
  stop() {
    if (this.eventSource) {
      this.eventSource.close();
    }
  }
}
```

## Development

### Setting Up Development Environment

1. Clone the repository
2. Install dependencies: `npm install`
3. Copy environment variables: `cp .env.example .env`
4. Update `.env` with your configuration
5. Run tests: `npm test`
6. Start development server: `npm run dev`

### Running Tests

```bash
# Run all tests
npm test

# Run specific test suite
npm run test:unit
npm run test:integration

# Run tests with coverage
npm run test:coverage
```

### Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/new-feature`
3. Make your changes
4. Write/update tests
5. Ensure all tests pass
6. Submit a pull request

### API Versioning

The API uses semantic versioning. Include the version in your requests:

```
Accept: application/vnd.laadpaal.v1+json
```

### Rate Limiting

- 1000 requests per hour for authenticated users
- 100 requests per hour for unauthenticated requests
- Rate limit headers are included in responses

### Support

For questions and support:
- Email: api-support@laadpaalnodig.nl
- Documentation: https://docs.laadpaalnodig.nl
- Status Page: https://status.laadpaalnodig.nl

---

*Last updated: January 15, 2024*
*Version: 1.0.0*