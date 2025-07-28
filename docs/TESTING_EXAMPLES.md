# Testing & Examples Documentation

## Overview

This document provides comprehensive testing guidelines, examples, and best practices for working with the Charging Station API. It includes unit tests, integration tests, and practical examples for common use cases.

## Table of Contents

- [Testing Setup](#testing-setup)
- [Unit Testing](#unit-testing)
- [Integration Testing](#integration-testing)
- [Mock Data](#mock-data)
- [Test Cases](#test-cases)
- [Performance Testing](#performance-testing)
- [Example Applications](#example-applications)

---

## Testing Setup

### Environment Configuration

```bash
# .env.test
API_KEY=test_api_key_12345
API_BASE_URL=https://staging-api.laadpaalnodig.nl/v1
API_TIMEOUT=30000
```

### Test Dependencies

**Node.js/JavaScript:**
```bash
npm install --save-dev jest supertest nock
```

**Python:**
```bash
pip install pytest pytest-asyncio responses factory-boy
```

**PHP:**
```bash
composer require --dev phpunit/phpunit guzzlehttp/guzzle mockery/mockery
```

---

## Unit Testing

### JavaScript/Node.js Tests

**Setup (Jest):**

```javascript
// tests/setup.js
const { LaadpaalAPI } = require('../src/index');
const nock = require('nock');

// Mock API base URL
const API_BASE_URL = 'https://api.laadpaalnodig.nl';

// Create API client for testing
const createTestClient = (options = {}) => {
  return new LaadpaalAPI({
    apiKey: 'test_key_12345',
    baseUrl: API_BASE_URL,
    ...options
  });
};

// Clean up nock after each test
afterEach(() => {
  nock.cleanAll();
});

module.exports = { createTestClient, API_BASE_URL };
```

**Application Tests:**

```javascript
// tests/applications.test.js
const nock = require('nock');
const { createTestClient, API_BASE_URL } = require('./setup');

describe('Applications API', () => {
  let api;

  beforeEach(() => {
    api = createTestClient();
  });

  describe('list()', () => {
    it('should fetch applications with default parameters', async () => {
      const mockResponse = {
        success: true,
        data: [
          {
            id: 'app_123',
            name: 'Test Application',
            status: 'pending',
            category: 'residential'
          }
        ],
        meta: {
          pagination: {
            total: 1,
            page: 1,
            limit: 10,
            hasNext: false,
            hasPrev: false
          }
        }
      };

      nock(API_BASE_URL)
        .get('/v1/applications')
        .query({ page: 1, limit: 10 })
        .reply(200, mockResponse);

      const result = await api.applications.list();

      expect(result.success).toBe(true);
      expect(result.data).toHaveLength(1);
      expect(result.data[0].id).toBe('app_123');
    });

    it('should handle filtering parameters', async () => {
      nock(API_BASE_URL)
        .get('/v1/applications')
        .query({
          page: 1,
          limit: 20,
          status: 'pending',
          category: 'residential'
        })
        .reply(200, { success: true, data: [], meta: {} });

      await api.applications.list({
        page: 1,
        limit: 20,
        status: 'pending',
        category: 'residential'
      });

      expect(nock.isDone()).toBe(true);
    });

    it('should handle API errors', async () => {
      nock(API_BASE_URL)
        .get('/v1/applications')
        .query(true)
        .reply(500, {
          success: false,
          errors: [{ code: 'INTERNAL_ERROR', message: 'Server error' }]
        });

      await expect(api.applications.list()).rejects.toThrow('Server error');
    });
  });

  describe('get()', () => {
    it('should fetch application by ID', async () => {
      const mockApplication = {
        success: true,
        data: {
          id: 'app_123',
          name: 'Test Application',
          status: 'pending',
          location: {
            city: 'Amsterdam',
            postalCode: '1000AA'
          }
        }
      };

      nock(API_BASE_URL)
        .get('/v1/applications/app_123')
        .reply(200, mockApplication);

      const result = await api.applications.get('app_123');

      expect(result.data.id).toBe('app_123');
      expect(result.data.location.city).toBe('Amsterdam');
    });

    it('should handle not found error', async () => {
      nock(API_BASE_URL)
        .get('/v1/applications/invalid_id')
        .reply(404, {
          success: false,
          errors: [{ code: 'NOT_FOUND', message: 'Application not found' }]
        });

      await expect(api.applications.get('invalid_id'))
        .rejects.toThrow('Application not found');
    });
  });

  describe('create()', () => {
    const validApplicationData = {
      name: 'Test Charging Station',
      category: 'residential',
      location: {
        address: 'Test Street 123',
        city: 'Amsterdam',
        postalCode: '1000AA',
        coordinates: { lat: 52.3676, lng: 4.9041 }
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
    };

    it('should create application with valid data', async () => {
      const mockResponse = {
        success: true,
        data: {
          id: 'app_new123',
          ...validApplicationData,
          status: 'pending',
          createdAt: '2024-01-15T09:00:00Z'
        }
      };

      nock(API_BASE_URL)
        .post('/v1/applications', validApplicationData)
        .reply(201, mockResponse);

      const result = await api.applications.create(validApplicationData);

      expect(result.data.id).toBe('app_new123');
      expect(result.data.status).toBe('pending');
    });

    it('should handle validation errors', async () => {
      const invalidData = { name: 'Test' }; // Missing required fields

      nock(API_BASE_URL)
        .post('/v1/applications', invalidData)
        .reply(422, {
          success: false,
          errors: [
            { code: 'VALIDATION_ERROR', message: 'Category is required', field: 'category' },
            { code: 'VALIDATION_ERROR', message: 'Location is required', field: 'location' }
          ]
        });

      await expect(api.applications.create(invalidData))
        .rejects.toThrow('Validation failed');
    });
  });
});
```

**Validation Tests:**

```javascript
// tests/validation.test.js
const { ValidationUtils } = require('../src/utils/validation');

describe('ValidationUtils', () => {
  describe('validateEmail()', () => {
    it('should validate correct email addresses', () => {
      expect(ValidationUtils.validateEmail('test@example.com')).toBe(true);
      expect(ValidationUtils.validateEmail('user.name@domain.co.uk')).toBe(true);
    });

    it('should reject invalid email addresses', () => {
      expect(ValidationUtils.validateEmail('invalid')).toBe(false);
      expect(ValidationUtils.validateEmail('test@')).toBe(false);
      expect(ValidationUtils.validateEmail('@example.com')).toBe(false);
    });
  });

  describe('validatePostalCode()', () => {
    it('should validate Dutch postal codes', () => {
      expect(ValidationUtils.validatePostalCode('1000AA', 'NL')).toBe(true);
      expect(ValidationUtils.validatePostalCode('1000 AA', 'NL')).toBe(true);
      expect(ValidationUtils.validatePostalCode('9999ZZ', 'NL')).toBe(true);
    });

    it('should reject invalid Dutch postal codes', () => {
      expect(ValidationUtils.validatePostalCode('100AA', 'NL')).toBe(false);
      expect(ValidationUtils.validatePostalCode('10000AA', 'NL')).toBe(false);
      expect(ValidationUtils.validatePostalCode('1000A', 'NL')).toBe(false);
    });
  });

  describe('validateApplicationData()', () => {
    const validData = {
      name: 'Test Application',
      category: 'residential',
      location: { address: 'Test Street 123' }
    };

    it('should validate correct application data', () => {
      const result = ValidationUtils.validateApplicationData(validData);
      expect(result.isValid).toBe(true);
      expect(result.errors).toHaveLength(0);
    });

    it('should detect missing required fields', () => {
      const result = ValidationUtils.validateApplicationData({});
      expect(result.isValid).toBe(false);
      expect(result.errors).toContain('Name must be at least 2 characters long');
      expect(result.errors).toContain('Category is required');
    });
  });
});
```

### Python Tests

**Setup (pytest):**

```python
# tests/conftest.py
import pytest
import responses
from laadpaal_api import LaadpaalAPI

@pytest.fixture
def api_client():
    return LaadpaalAPI(
        api_key='test_key_12345',
        base_url='https://api.laadpaalnodig.nl/v1',
        environment='test'
    )

@pytest.fixture
def mock_responses():
    with responses.RequestsMock() as rsps:
        yield rsps
```

**Application Tests:**

```python
# tests/test_applications.py
import pytest
import responses
from laadpaal_api.exceptions import NotFoundError, ValidationError

class TestApplicationsAPI:
    
    def test_list_applications(self, api_client, mock_responses):
        mock_data = {
            'success': True,
            'data': [
                {
                    'id': 'app_123',
                    'name': 'Test Application',
                    'status': 'pending',
                    'category': 'residential'
                }
            ],
            'meta': {'pagination': {'total': 1, 'page': 1}}
        }
        
        mock_responses.add(
            responses.GET,
            'https://api.laadpaalnodig.nl/v1/applications',
            json=mock_data,
            status=200
        )
        
        result = api_client.applications.list(status='pending')
        
        assert result.success is True
        assert len(result.data) == 1
        assert result.data[0]['id'] == 'app_123'

    def test_get_application(self, api_client, mock_responses):
        mock_data = {
            'success': True,
            'data': {
                'id': 'app_123',
                'name': 'Test Application',
                'status': 'pending'
            }
        }
        
        mock_responses.add(
            responses.GET,
            'https://api.laadpaalnodig.nl/v1/applications/app_123',
            json=mock_data,
            status=200
        )
        
        result = api_client.applications.get('app_123')
        assert result.data['id'] == 'app_123'

    def test_get_application_not_found(self, api_client, mock_responses):
        mock_responses.add(
            responses.GET,
            'https://api.laadpaalnodig.nl/v1/applications/invalid_id',
            json={
                'success': False,
                'errors': [{'code': 'NOT_FOUND', 'message': 'Application not found'}]
            },
            status=404
        )
        
        with pytest.raises(NotFoundError):
            api_client.applications.get('invalid_id')

    def test_create_application(self, api_client, mock_responses):
        application_data = {
            'name': 'Test Charging Station',
            'category': 'residential',
            'location': {
                'address': 'Test Street 123',
                'city': 'Amsterdam',
                'postal_code': '1000AA'
            }
        }
        
        mock_response = {
            'success': True,
            'data': {
                'id': 'app_new123',
                **application_data,
                'status': 'pending'
            }
        }
        
        mock_responses.add(
            responses.POST,
            'https://api.laadpaalnodig.nl/v1/applications',
            json=mock_response,
            status=201
        )
        
        result = api_client.applications.create(application_data)
        assert result.data['id'] == 'app_new123'
        assert result.data['status'] == 'pending'

    @pytest.mark.parametrize('invalid_data,expected_error', [
        ({}, 'Name is required'),
        ({'name': 'Test'}, 'Category is required'),
        ({'name': 'Test', 'category': 'residential'}, 'Location is required')
    ])
    def test_create_application_validation_errors(self, api_client, mock_responses, invalid_data, expected_error):
        mock_responses.add(
            responses.POST,
            'https://api.laadpaalnodig.nl/v1/applications',
            json={
                'success': False,
                'errors': [{'code': 'VALIDATION_ERROR', 'message': expected_error}]
            },
            status=422
        )
        
        with pytest.raises(ValidationError):
            api_client.applications.create(invalid_data)
```

**Async Tests:**

```python
# tests/test_async_api.py
import pytest
import aioresponses
from laadpaal_api import AsyncLaadpaalAPI

@pytest.mark.asyncio
class TestAsyncAPI:
    
    async def test_async_list_applications(self):
        api = AsyncLaadpaalAPI(api_key='test_key')
        
        mock_data = {
            'success': True,
            'data': [{'id': 'app_123', 'name': 'Test'}],
            'meta': {}
        }
        
        with aioresponses.aioresponses() as m:
            m.get(
                'https://api.laadpaalnodig.nl/v1/applications',
                payload=mock_data
            )
            
            result = await api.applications.list()
            assert result.success is True
            assert len(result.data) == 1
        
        await api.close()
```

### PHP Tests

**Setup (PHPUnit):**

```php
<?php
// tests/TestCase.php
use PHPUnit\Framework\TestCase as BaseTestCase;
use GuzzleHttp\Client;
use GuzzleHttp\Handler\MockHandler;
use GuzzleHttp\HandlerStack;
use GuzzleHttp\Psr7\Response;
use LaadpaalNodig\ApiClient\LaadpaalAPI;

abstract class TestCase extends BaseTestCase
{
    protected function createMockClient(array $responses = [])
    {
        $mock = new MockHandler($responses);
        $handlerStack = HandlerStack::create($mock);
        $httpClient = new Client(['handler' => $handlerStack]);
        
        return new LaadpaalAPI([
            'api_key' => 'test_key_12345',
            'http_client' => $httpClient
        ]);
    }
}
```

**Application Tests:**

```php
<?php
// tests/ApplicationsTest.php
use GuzzleHttp\Psr7\Response;

class ApplicationsTest extends TestCase
{
    public function testListApplications()
    {
        $mockResponse = new Response(200, [], json_encode([
            'success' => true,
            'data' => [
                [
                    'id' => 'app_123',
                    'name' => 'Test Application',
                    'status' => 'pending'
                ]
            ],
            'meta' => ['pagination' => ['total' => 1]]
        ]));
        
        $api = $this->createMockClient([$mockResponse]);
        $result = $api->applications()->list(['status' => 'pending']);
        
        $this->assertTrue($result->success);
        $this->assertCount(1, $result->data);
        $this->assertEquals('app_123', $result->data[0]->id);
    }
    
    public function testGetApplication()
    {
        $mockResponse = new Response(200, [], json_encode([
            'success' => true,
            'data' => [
                'id' => 'app_123',
                'name' => 'Test Application',
                'status' => 'pending'
            ]
        ]));
        
        $api = $this->createMockClient([$mockResponse]);
        $result = $api->applications()->get('app_123');
        
        $this->assertEquals('app_123', $result->data->id);
    }
    
    public function testCreateApplication()
    {
        $applicationData = [
            'name' => 'Test Charging Station',
            'category' => 'residential',
            'location' => [
                'address' => 'Test Street 123',
                'city' => 'Amsterdam',
                'postal_code' => '1000AA'
            ]
        ];
        
        $mockResponse = new Response(201, [], json_encode([
            'success' => true,
            'data' => array_merge($applicationData, [
                'id' => 'app_new123',
                'status' => 'pending'
            ])
        ]));
        
        $api = $this->createMockClient([$mockResponse]);
        $result = $api->applications()->create($applicationData);
        
        $this->assertEquals('app_new123', $result->data->id);
        $this->assertEquals('pending', $result->data->status);
    }
}
```

---

## Integration Testing

### End-to-End Test Suite

```javascript
// tests/integration/e2e.test.js
const { LaadpaalAPI } = require('../../src/index');

describe('End-to-End API Tests', () => {
  let api;
  let testApplicationId;

  beforeAll(() => {
    api = new LaadpaalAPI({
      apiKey: process.env.TEST_API_KEY,
      baseUrl: process.env.TEST_API_URL || 'https://staging-api.laadpaalnodig.nl/v1'
    });
  });

  describe('Full Application Lifecycle', () => {
    it('should create, update, and retrieve application', async () => {
      // Create application
      const applicationData = {
        name: 'E2E Test Charging Station',
        category: 'residential',
        location: {
          address: 'Integration Test Street 123',
          city: 'Amsterdam',
          postalCode: '1000AA',
          coordinates: { lat: 52.3676, lng: 4.9041 }
        },
        specifications: {
          power: 11,
          connectorType: 'Type2',
          connectorCount: 1
        },
        contact: {
          name: 'Test User',
          email: 'test@example.com',
          phone: '+31612345678'
        }
      };

      const createResponse = await api.applications.create(applicationData);
      expect(createResponse.success).toBe(true);
      expect(createResponse.data.id).toBeDefined();
      
      testApplicationId = createResponse.data.id;

      // Retrieve application
      const getResponse = await api.applications.get(testApplicationId);
      expect(getResponse.data.name).toBe(applicationData.name);
      expect(getResponse.data.status).toBe('pending');

      // Update application
      const updateData = {
        specifications: {
          power: 22,
          connectorCount: 2
        }
      };

      const updateResponse = await api.applications.update(testApplicationId, updateData);
      expect(updateResponse.data.specifications.power).toBe(22);

      // List applications (should include our test application)
      const listResponse = await api.applications.list({
        search: 'E2E Test'
      });
      
      const ourApp = listResponse.data.find(app => app.id === testApplicationId);
      expect(ourApp).toBeDefined();
    }, 30000);

    afterAll(async () => {
      // Clean up test data
      if (testApplicationId) {
        try {
          await api.applications.delete(testApplicationId);
        } catch (error) {
          console.warn('Failed to clean up test application:', error.message);
        }
      }
    });
  });

  describe('API Error Handling', () => {
    it('should handle rate limiting gracefully', async () => {
      const promises = Array(20).fill().map(() => 
        api.applications.list({ limit: 1 })
      );

      // Some requests might be rate limited
      const results = await Promise.allSettled(promises);
      const fulfilled = results.filter(r => r.status === 'fulfilled');
      const rejected = results.filter(r => r.status === 'rejected');

      expect(fulfilled.length).toBeGreaterThan(0);
      
      if (rejected.length > 0) {
        // Check that rate limit errors are properly handled
        const rateLimitErrors = rejected.filter(r => 
          r.reason.message.includes('rate limit')
        );
        expect(rateLimitErrors.length).toBeGreaterThan(0);
      }
    });
  });
});
```

### Performance Tests

```javascript
// tests/performance/load.test.js
const { performance } = require('perf_hooks');

describe('API Performance Tests', () => {
  let api;

  beforeAll(() => {
    api = new LaadpaalAPI({
      apiKey: process.env.TEST_API_KEY,
      baseUrl: process.env.TEST_API_URL
    });
  });

  it('should handle concurrent requests efficiently', async () => {
    const startTime = performance.now();
    const concurrentRequests = 10;

    const promises = Array(concurrentRequests).fill().map((_, index) =>
      api.applications.list({ page: index + 1, limit: 5 })
    );

    const results = await Promise.all(promises);
    const endTime = performance.now();
    const duration = endTime - startTime;

    // All requests should succeed
    results.forEach(result => {
      expect(result.success).toBe(true);
    });

    // Should complete within reasonable time (adjust based on your requirements)
    expect(duration).toBeLessThan(5000); // 5 seconds
    
    console.log(`Completed ${concurrentRequests} concurrent requests in ${duration}ms`);
  });

  it('should respond within acceptable time limits', async () => {
    const measurements = [];

    for (let i = 0; i < 5; i++) {
      const startTime = performance.now();
      await api.applications.list({ limit: 10 });
      const endTime = performance.now();
      measurements.push(endTime - startTime);
    }

    const avgResponseTime = measurements.reduce((a, b) => a + b) / measurements.length;
    
    expect(avgResponseTime).toBeLessThan(1000); // Should respond within 1 second on average
    console.log(`Average response time: ${avgResponseTime}ms`);
  });
});
```

---

## Mock Data

### Test Data Factories

```javascript
// tests/factories/application.js
const { faker } = require('@faker-js/faker');

class ApplicationFactory {
  static create(overrides = {}) {
    return {
      id: faker.string.uuid(),
      name: faker.company.name() + ' Charging Station',
      description: faker.lorem.sentence(),
      category: faker.helpers.arrayElement(['residential', 'commercial']),
      status: faker.helpers.arrayElement(['pending', 'under_review', 'approved', 'rejected']),
      location: {
        address: faker.location.streetAddress(),
        city: faker.location.city(),
        postalCode: faker.location.zipCode('####AA'),
        country: 'NL',
        coordinates: {
          lat: parseFloat(faker.location.latitude({ min: 50, max: 54 })),
          lng: parseFloat(faker.location.longitude({ min: 3, max: 8 }))
        }
      },
      specifications: {
        power: faker.helpers.arrayElement([3.7, 7.4, 11, 22]),
        connectorType: faker.helpers.arrayElement(['Type1', 'Type2']),
        connectorCount: faker.number.int({ min: 1, max: 4 })
      },
      contact: {
        name: faker.person.fullName(),
        email: faker.internet.email(),
        phone: faker.phone.number('+316########')
      },
      createdAt: faker.date.recent().toISOString(),
      updatedAt: faker.date.recent().toISOString(),
      ...overrides
    };
  }

  static createList(count = 5, overrides = {}) {
    return Array(count).fill().map(() => this.create(overrides));
  }

  static pending() {
    return this.create({ status: 'pending' });
  }

  static approved() {
    return this.create({ status: 'approved' });
  }

  static residential() {
    return this.create({ category: 'residential' });
  }

  static commercial() {
    return this.create({ 
      category: 'commercial',
      specifications: {
        power: faker.helpers.arrayElement([22, 50, 150]),
        connectorType: faker.helpers.arrayElement(['Type2', 'CCS']),
        connectorCount: faker.number.int({ min: 2, max: 8 })
      }
    });
  }
}

module.exports = ApplicationFactory;
```

```python
# tests/factories.py
import factory
from faker import Faker
from datetime import datetime
import random

fake = Faker()

class ApplicationFactory(factory.Factory):
    class Meta:
        model = dict
    
    id = factory.LazyFunction(lambda: fake.uuid4())
    name = factory.LazyAttribute(lambda obj: f"{fake.company()} Charging Station")
    description = factory.LazyFunction(lambda: fake.sentence())
    category = factory.LazyFunction(lambda: random.choice(['residential', 'commercial']))
    status = factory.LazyFunction(lambda: random.choice(['pending', 'under_review', 'approved']))
    
    location = factory.LazyFunction(lambda: {
        'address': fake.street_address(),
        'city': fake.city(),
        'postal_code': fake.postcode(),
        'country': 'NL',
        'coordinates': {
            'lat': float(fake.latitude()),
            'lng': float(fake.longitude())
        }
    })
    
    specifications = factory.LazyFunction(lambda: {
        'power': random.choice([3.7, 7.4, 11, 22]),
        'connector_type': random.choice(['Type1', 'Type2']),
        'connector_count': random.randint(1, 4)
    })
    
    contact = factory.LazyFunction(lambda: {
        'name': fake.name(),
        'email': fake.email(),
        'phone': fake.phone_number()
    })
    
    created_at = factory.LazyFunction(lambda: fake.date_time().isoformat())
    updated_at = factory.LazyFunction(lambda: fake.date_time().isoformat())

class ChargingStationFactory(factory.Factory):
    class Meta:
        model = dict
    
    id = factory.LazyFunction(lambda: fake.uuid4())
    name = factory.LazyAttribute(lambda obj: f"{fake.company()} Station")
    status = 'operational'
    
    location = factory.LazyFunction(lambda: {
        'address': fake.street_address(),
        'city': fake.city(),
        'coordinates': {
            'lat': float(fake.latitude()),
            'lng': float(fake.longitude())
        }
    })
    
    connectors = factory.LazyFunction(lambda: [
        {
            'id': fake.uuid4(),
            'type': random.choice(['Type2', 'CCS', 'CHAdeMO']),
            'power': random.choice([22, 50, 150]),
            'status': random.choice(['available', 'charging', 'offline'])
        }
        for _ in range(random.randint(1, 4))
    ])
```

---

## Test Cases

### Comprehensive Test Scenarios

```javascript
// tests/scenarios/application-workflow.test.js
const ApplicationFactory = require('../factories/application');

describe('Application Workflow Scenarios', () => {
  describe('Residential Application Flow', () => {
    it('should handle complete residential application process', async () => {
      // 1. User creates residential application
      const applicationData = ApplicationFactory.residential();
      delete applicationData.id; // Remove ID for creation
      
      const createResponse = await api.applications.create(applicationData);
      expect(createResponse.data.status).toBe('pending');
      
      const appId = createResponse.data.id;
      
      // 2. Upload required documents
      const documents = [
        { type: 'electrical_plan', file: 'electrical-plan.pdf' },
        { type: 'property_deed', file: 'property-deed.pdf' }
      ];
      
      for (const doc of documents) {
        const uploadResponse = await api.applications.uploadDocument(appId, {
          type: doc.type,
          file: Buffer.from('mock file content'),
          filename: doc.file
        });
        expect(uploadResponse.success).toBe(true);
      }
      
      // 3. Check application status
      const statusResponse = await api.applications.get(appId);
      expect(statusResponse.data.documents).toHaveLength(2);
      
      // 4. Simulate approval process
      // This would typically be done by admin/reviewer
      // await api.applications.updateStatus(appId, 'approved');
      
      // 5. Verify final state
      const finalResponse = await api.applications.get(appId);
      expect(finalResponse.data.specifications.power).toBe(applicationData.specifications.power);
    });
  });

  describe('Commercial Application Flow', () => {
    it('should handle complex commercial application', async () => {
      const commercialApp = ApplicationFactory.commercial();
      delete commercialApp.id;
      
      // Commercial applications require additional fields
      commercialApp.business = {
        name: 'Test Business BV',
        kvkNumber: '12345678',
        vatNumber: 'NL123456789B01'
      };
      
      commercialApp.specifications.power = 150; // High power for commercial
      commercialApp.specifications.connectorCount = 6;
      
      const response = await api.applications.create(commercialApp);
      expect(response.data.category).toBe('commercial');
      expect(response.data.specifications.power).toBe(150);
    });
  });

  describe('Error Scenarios', () => {
    it('should handle invalid location data', async () => {
      const invalidApp = ApplicationFactory.create({
        location: {
          coordinates: { lat: 999, lng: 999 } // Invalid coordinates
        }
      });
      
      await expect(api.applications.create(invalidApp))
        .rejects.toThrow('Invalid coordinates');
    });

    it('should handle duplicate applications', async () => {
      const appData = ApplicationFactory.create();
      delete appData.id;
      
      // Create first application
      await api.applications.create(appData);
      
      // Try to create duplicate (same location)
      await expect(api.applications.create(appData))
        .rejects.toThrow('Application already exists for this location');
    });
  });
});
```

### Edge Case Testing

```javascript
// tests/edge-cases.test.js
describe('Edge Cases', () => {
  describe('Large Dataset Handling', () => {
    it('should handle pagination with large datasets', async () => {
      const allApplications = [];
      let page = 1;
      let hasMore = true;
      
      while (hasMore && page <= 100) { // Safety limit
        const response = await api.applications.list({
          page,
          limit: 100
        });
        
        allApplications.push(...response.data);
        hasMore = response.meta.pagination.hasNext;
        page++;
      }
      
      expect(allApplications.length).toBeGreaterThan(0);
      // Verify no duplicates
      const ids = allApplications.map(app => app.id);
      const uniqueIds = [...new Set(ids)];
      expect(ids.length).toBe(uniqueIds.length);
    });
  });

  describe('Network Issues', () => {
    it('should handle connection timeouts', async () => {
      const slowApi = new LaadpaalAPI({
        apiKey: 'test_key',
        timeout: 100 // Very short timeout
      });
      
      // Mock a slow response
      nock(API_BASE_URL)
        .get('/v1/applications')
        .delay(200) // Longer than timeout
        .reply(200, { success: true, data: [] });
      
      await expect(slowApi.applications.list())
        .rejects.toThrow('timeout');
    });
  });

  describe('Special Characters and Encoding', () => {
    it('should handle special characters in names and addresses', async () => {
      const specialCharApp = ApplicationFactory.create({
        name: 'Charging Station Café & Résumé',
        location: {
          address: 'Straße der Einführung 123',
          city: 'München'
        },
        contact: {
          name: 'José María García-López',
          email: 'josé@café.com'
        }
      });
      
      delete specialCharApp.id;
      
      const response = await api.applications.create(specialCharApp);
      expect(response.data.name).toBe(specialCharApp.name);
      expect(response.data.contact.name).toBe(specialCharApp.contact.name);
    });
  });
});
```

---

## Example Applications

### Complete Application Examples

#### 1. Residential Charging Station Application

```javascript
// examples/residential-application.js
const { LaadpaalAPI, ValidationUtils } = require('../src/index');

async function submitResidentialApplication() {
  const api = new LaadpaalAPI({
    apiKey: process.env.API_KEY,
    environment: 'production'
  });

  try {
    // Step 1: Validate user input
    const applicationData = {
      name: 'Home Charging Station - Tesla Model 3',
      description: 'Installation of wall-mounted charging station for daily use',
      category: 'residential',
      location: {
        address: 'Keizersgracht 123',
        city: 'Amsterdam',
        postalCode: '1015 CJ',
        country: 'NL',
        coordinates: {
          lat: 52.3738,
          lng: 4.8842
        },
        propertyType: 'canal_house',
        parkingType: 'street_parking'
      },
      specifications: {
        power: 11,
        voltage: 230,
        phase: 'single',
        connectorType: 'Type2',
        connectorCount: 1,
        smartCharging: true,
        loadBalancing: false,
        weatherProtection: 'IP54'
      },
      technical: {
        meterboxCapacity: 3,
        availableCapacity: 15,
        cableLength: 5,
        installationMethod: 'wall_mounted',
        earthingSystem: 'TN-S'
      },
      contact: {
        name: 'Emma van der Berg',
        email: 'emma.vandenberg@email.nl',
        phone: '+31612345678',
        preferredContactMethod: 'email'
      },
      preferences: {
        installationTimeframe: 'within_month',
        preferredInstaller: null,
        notes: 'Prefer installation on weekdays after 9 AM'
      }
    };

    // Validate application data
    const validation = ValidationUtils.validateApplicationData(applicationData);
    if (!validation.isValid) {
      throw new Error(`Validation failed: ${validation.errors.join(', ')}`);
    }

    // Step 2: Submit application
    console.log('Submitting residential application...');
    const application = await api.applications.create(applicationData);
    console.log('Application created:', application.data.id);

    // Step 3: Upload required documents
    const documents = [
      { type: 'property_deed', path: './documents/property-deed.pdf' },
      { type: 'electrical_plan', path: './documents/electrical-plan.pdf' },
      { type: 'site_photo', path: './documents/parking-area.jpg' }
    ];

    for (const doc of documents) {
      console.log(`Uploading ${doc.type}...`);
      const fs = require('fs');
      const fileStream = fs.createReadStream(doc.path);
      
      await api.applications.uploadDocument(application.data.id, {
        file: fileStream,
        type: doc.type,
        description: `${doc.type.replace('_', ' ')} for residential application`
      });
    }

    // Step 4: Monitor application status
    console.log('Monitoring application status...');
    let currentStatus = 'pending';
    
    while (currentStatus !== 'approved' && currentStatus !== 'rejected') {
      await new Promise(resolve => setTimeout(resolve, 30000)); // Wait 30 seconds
      
      const statusUpdate = await api.applications.get(application.data.id);
      const newStatus = statusUpdate.data.status;
      
      if (newStatus !== currentStatus) {
        console.log(`Status changed: ${currentStatus} → ${newStatus}`);
        currentStatus = newStatus;
        
        if (newStatus === 'approved') {
          console.log('🎉 Application approved!');
          console.log('Estimated installation date:', statusUpdate.data.estimatedDates.installation);
        } else if (newStatus === 'rejected') {
          console.log('❌ Application rejected');
          console.log('Reason:', statusUpdate.data.rejectionReason);
        }
      }
    }

    return application.data;

  } catch (error) {
    console.error('Application submission failed:', error.message);
    throw error;
  }
}

// Usage
if (require.main === module) {
  submitResidentialApplication()
    .then(app => console.log('Final application:', app))
    .catch(error => console.error('Error:', error));
}

module.exports = { submitResidentialApplication };
```

#### 2. Commercial Charging Hub Application

```javascript
// examples/commercial-hub-application.js
async function submitCommercialHubApplication() {
  const api = new LaadpaalAPI({
    apiKey: process.env.API_KEY,
    environment: 'production'
  });

  const hubApplication = {
    name: 'Shopping Center Fast Charging Hub',
    description: 'High-power charging hub for shopping center visitors',
    category: 'commercial',
    location: {
      address: 'Amstelveenseweg 500',
      city: 'Amsterdam',
      postalCode: '1081 KL',
      country: 'NL',
      coordinates: { lat: 52.3376, lng: 4.8635 },
      propertyType: 'commercial_building',
      parkingType: 'private_parking_lot',
      accessibility: 'public'
    },
    specifications: {
      power: 350, // High-power DC charging
      connectorType: 'CCS',
      connectorCount: 8,
      dcFastCharging: true,
      smartCharging: true,
      loadBalancing: true,
      paymentMethods: ['card', 'app', 'subscription'],
      operatingHours: '24/7'
    },
    business: {
      name: 'Shopping Center BV',
      kvkNumber: '12345678',
      vatNumber: 'NL123456789B01',
      contactPerson: 'Jan Jansen',
      businessType: 'retail'
    },
    technical: {
      gridConnection: 'high_voltage',
      transformerCapacity: 630,
      availableCapacity: 400,
      backup_power: true,
      monitoring: 'remote_24_7'
    },
    environmental: {
      renewableEnergy: true,
      solarPanels: true,
      batteryStorage: true,
      carbonNeutral: true
    },
    expectedUsage: {
      dailySessions: 150,
      peakHours: ['12:00-14:00', '17:00-19:00'],
      averageSessionDuration: 30,
      targetMarket: 'shopping_center_visitors'
    }
  };

  try {
    console.log('Submitting commercial hub application...');
    const application = await api.applications.create(hubApplication);
    
    // Upload comprehensive documentation
    const commercialDocuments = [
      { type: 'business_registration', path: './docs/kvk-extract.pdf' },
      { type: 'site_plan', path: './docs/site-layout.pdf' },
      { type: 'electrical_plan', path: './docs/electrical-infrastructure.pdf' },
      { type: 'environmental_impact', path: './docs/environmental-assessment.pdf' },
      { type: 'parking_permit', path: './docs/parking-permits.pdf' },
      { type: 'insurance_certificate', path: './docs/insurance.pdf' }
    ];

    for (const doc of commercialDocuments) {
      await api.applications.uploadDocument(application.data.id, {
        file: fs.createReadStream(doc.path),
        type: doc.type
      });
    }

    return application.data;
  } catch (error) {
    console.error('Commercial application failed:', error);
    throw error;
  }
}
```

#### 3. Fleet Management Integration

```javascript
// examples/fleet-management.js
class FleetChargingManager {
  constructor(apiKey) {
    this.api = new LaadpaalAPI({ apiKey });
    this.fleetVehicles = new Map();
    this.chargingSessions = new Map();
  }

  async registerFleet(fleetData) {
    // Register company fleet
    const fleetApplication = {
      name: `${fleetData.companyName} Fleet Charging`,
      category: 'commercial',
      location: fleetData.depotLocation,
      specifications: {
        power: 22,
        connectorType: 'Type2',
        connectorCount: fleetData.vehicleCount,
        smartCharging: true,
        loadBalancing: true,
        scheduledCharging: true
      },
      fleetDetails: {
        companyName: fleetData.companyName,
        vehicleCount: fleetData.vehicleCount,
        vehicleTypes: fleetData.vehicleTypes,
        operatingSchedule: fleetData.schedule,
        chargingRequirements: fleetData.requirements
      }
    };

    return await this.api.applications.create(fleetApplication);
  }

  async optimizeChargingSchedule(fleetId, vehicles) {
    // Implement smart charging logic
    const schedule = vehicles.map(vehicle => ({
      vehicleId: vehicle.id,
      arrivalTime: vehicle.returnTime,
      departureTime: vehicle.nextDepartureTime,
      currentBattery: vehicle.batteryLevel,
      requiredBattery: vehicle.targetBatteryLevel,
      chargingPower: this.calculateOptimalPower(vehicle),
      priority: vehicle.priority
    }));

    // Sort by priority and time constraints
    schedule.sort((a, b) => {
      if (a.priority !== b.priority) return b.priority - a.priority;
      return new Date(a.departureTime) - new Date(b.departureTime);
    });

    return schedule;
  }

  async monitorChargingSessions() {
    // Real-time monitoring of charging sessions
    const activeSessions = await this.api.chargingSessions.listActive();
    
    for (const session of activeSessions.data) {
      if (session.status === 'charging') {
        const progress = this.calculateChargingProgress(session);
        
        if (progress.estimatedCompletion < new Date()) {
          await this.notifyVehicleReady(session.vehicleId);
        }
      }
    }
  }

  calculateOptimalPower(vehicle) {
    const timeAvailable = new Date(vehicle.nextDepartureTime) - new Date(vehicle.returnTime);
    const energyNeeded = (vehicle.targetBatteryLevel - vehicle.batteryLevel) * vehicle.batteryCapacity / 100;
    const requiredPower = energyNeeded / (timeAvailable / 3600000); // Convert to hours
    
    return Math.min(requiredPower, vehicle.maxChargingPower, 22); // Limit to 22kW
  }
}

// Usage example
async function manageDeliveryFleet() {
  const fleetManager = new FleetChargingManager(process.env.API_KEY);
  
  const fleetData = {
    companyName: 'Express Delivery Services',
    vehicleCount: 25,
    vehicleTypes: ['van', 'truck'],
    depotLocation: {
      address: 'Industrial Estate 45',
      city: 'Rotterdam',
      postalCode: '3000 AB'
    },
    schedule: {
      operatingHours: '06:00-22:00',
      peakHours: ['07:00-09:00', '17:00-19:00'],
      chargingWindow: '22:00-06:00'
    }
  };

  try {
    const fleetApplication = await fleetManager.registerFleet(fleetData);
    console.log('Fleet registered:', fleetApplication.id);
    
    // Start monitoring
    setInterval(async () => {
      await fleetManager.monitorChargingSessions();
    }, 60000); // Check every minute
    
  } catch (error) {
    console.error('Fleet management error:', error);
  }
}
```

### Testing Best Practices Summary

1. **Test Pyramid**: Focus on unit tests (70%), integration tests (20%), and E2E tests (10%)
2. **Mocking**: Use mocks for external dependencies and APIs
3. **Test Data**: Use factories for consistent, realistic test data
4. **Performance**: Include performance benchmarks in your test suite
5. **Error Scenarios**: Test all error conditions and edge cases
6. **CI/CD**: Run tests automatically on every commit and deployment
7. **Environment Separation**: Use separate environments for testing
8. **Documentation**: Keep test documentation up to date with examples

This comprehensive testing framework ensures reliable, maintainable code and provides excellent examples for API usage patterns.