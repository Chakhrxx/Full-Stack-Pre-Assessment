## Back-end Questions

**1. Assuming the system currently has three microservices: Customer API, Master Data API,
and Transaction Data API, there is a new feature that requires data from all three
microservices to be displayed in near real-time. The current technology stack includes
REST APIs and an RDBMS database. How would you design a new API for this feature?**

- **1 Using REST APIs:**
  - Create a new Aggregator API that will receive requests from the client, fetch data from all three microservices via their respective REST APIs, and combine the data into a single response.
  - This new API should perform the aggregation of data and respond to the client within a reasonable timeframe (close to real-time).
- **2 Handling Data and Speed:**

  - To ensure the data is as fresh as possible in near real-time, consider using caching for data that doesn’t change frequently, or store commonly used data in memory to reduce the time spent fetching data from the databases or microservices.
  - If the data updates frequently or in real-time, consider implementing a Message Queue or Event-Driven Architecture. This would allow each microservice to update data asynchronously, and the Aggregator API can consume updates from the queue.

- **3 API Design:**

  - Design a new endpoint, such as /aggregated-data, that fetches data from the Customer API, Master Data API, and Transaction Data API, and combines the results into a single response.
  - Use Parallel Requests to fetch data from all three microservices simultaneously, reducing the wait time for responses.

- **4 Load Management:**
  - If many users are expected to request data in near real-time, consider scaling the system to handle high request frequency.
  - Implement monitoring and alerting to track system health and respond to any issues promptly.

**2. Assuming the team has started planning a new project, the project manager asks you for a
performance test strategy plan for this release. How would you recommend proceeding to
the project manager?**

- **1. Understand the Requirements**
  - Identify Performance Goals: Start by understanding the specific performance requirements, such as the maximum number of users, response time expectations, throughput, and scalability goals.
  - Key Metrics: Define the performance metrics that are most important for the project, such as:
    - Response time (average and 90th percentile)
    - Throughput (requests per second)
    - Error rates (acceptable failure thresholds)
    - Resource utilization (CPU, memory, disk, network)
    - Scalability and system limits
- **2. Define the Testing Scope**

  - Types of Performance Testing: Determine the necessary types of tests:
    - Load Testing: To evaluate system performance under expected traffic loads.
    - Stress Testing: To identify the system’s breaking point under extreme conditions.
    - Scalability Testing: To test how well the system scales with increased load.
    - Endurance Testing: To ensure the system can handle sustained load over time.

- **3. Plan for Test Environment and Tools**

  - Test Environment: Set up a test environment that mirrors production as closely as possible, including hardware, network configurations, and databases.
  - Tools: Decide which performance testing tools will be used (e.g., JMeter, Gatling, LoadRunner, etc.), considering factors such as scalability, ease of integration, and compatibility with the project’s technology stack.

- **4. Test Data and Scenarios**
  - Test Data: Use realistic data sets to simulate real-world scenarios, ensuring that the data is representative of production workloads.
  - Test Scenarios: Define and document the key use cases and traffic patterns that need to be tested, including peak load and stress conditions.
- **5. Execution and Monitoring**
  - Execution Plan: Schedule the performance tests at various stages, including during development, pre-production, and before release.
  - Monitoring: Implement monitoring tools to track system performance during testing (e.g., server CPU, memory, response times, and network throughput).
- **6. Analyze Results and Iterate**

  - After the tests are completed, analyze the results to identify bottlenecks and areas for improvement.
  - Work with the development team to address performance issues, then retest if necessary.

- **7. Reporting**
  - Prepare a detailed performance testing report summarizing the results, issues encountered, and recommended improvements.

**3. Design and develop two APIs using NestJS and Postgres with the following
specifications:**

1. Create a Multilingual Product API: Develop an API that allows for the creation of products, each with attributes for name and description that support multiple languages.
2. Multilingual Product Search API: Implement an API that enables searching for products by name in any language and returns results in a paginated format.

**Additional Requirements:**

- Validation: Outline how you will validate data inputs in both APIs to ensure data integrity.
- Database Design: Describe the database schema and the approach you will use to handle multilingual support for product information.
- Testing Strategy: Explain your strategy for testing these APIs, including how you will handle unit tests, integration tests, and any end-to-end testing considerations.

# Multilingual Product API Design

## Database Design

To support multilingual content, I will create two tables:

- `products`: Stores core product information.
- `product_translations`: Stores translations of product names and descriptions in different languages.

**Database Schema:**

```sql
-- products table
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  sku VARCHAR(255) UNIQUE NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  category VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- product_translations table
CREATE TABLE product_translations (
  id SERIAL PRIMARY KEY,
  product_id INT REFERENCES products(id) ON DELETE CASCADE,
  language_code VARCHAR(10) NOT NULL,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  UNIQUE (product_id, language_code)
);
```

- `products`: Contains core product details (e.g., SKU, price, and category).
- `product_translations`: Contains product name and description in different languages (e.g., en, th)

## API Endpoints

- POST /products: Create a new product with translations for its name and description.

**Request Body (Example):**

```json
{
  "sku": "P12345",
  "price": 99.99,
  "category": "Electronics",
  "translations": [
    {
      "language_code": "en",
      "name": "Smartphone",
      "description": "Latest smartphone with advanced features"
    },
    {
      "language_code": "th",
      "name": "สมาร์ทโฟน",
      "description": "สมาร์ทโฟนล่าสุดที่มีคุณสมบัติขั้นสูง"
    }
  ]
}
```

**Validation:**

- Validate input fields using class-validator to ensure proper data formatting (e.g., numeric types for price, required fields for translations).

```ts
class CreateProductDto {
  @IsString()
  @IsNotEmpty()
  sku: string;

  @IsDecimal()
  price: number;

  @IsString()
  @IsNotEmpty()
  category: string;

  @IsArray()
  @ArrayNotEmpty()
  translations: ProductTranslationDto[];
}

class ProductTranslationDto {
  @IsString()
  @IsNotEmpty()
  language_code: string;

  @IsString()
  @IsNotEmpty()
  name: string;

  @IsString()
  description: string;
}
```

# Multilingual Product Search API

## Database Design for Search

The search feature will query the product_translations table based on the name field in the specified language and return paginated results.

**SQL Query Example:**

```sql
SELECT p.id, pt.name, pt.description, p.sku, p.price, p.category
FROM products p
JOIN product_translations pt ON p.id = pt.product_id
WHERE pt.language_code = 'en' AND pt.name ILIKE '%smartphone%'
ORDER BY pt.name
LIMIT 10 OFFSET 0;
```

- Pagination: I use LIMIT and OFFSET to paginate results.

## API Endpoints

- GET /products/search: Search for products by name in a specific language.

  - Request Parameters:
    - `query`: The search term (e.g., "smartphone").
    - `language_code`: The language code (e.g., "en" for English).
    - `page`: The page number (default: 1).
    - `page_size`: The number of results per page (default: 10).
  - Response:

    ```json
    {
      "results": [
        {
          "id": 1,
          "name": "Smartphone",
          "description": "Latest smartphone with advanced features",
          "sku": "P12345",
          "price": 99.99,
          "category": "Electronics"
        }
      ],
      "pagination": {
        "page": 1,
        "page_size": 10,
        "total_results": 50
      }
    }
    ```

  - Validation:

    - Validate that query and language_code are non-empty strings.
    - Validate that page and page_size are integers and within a reasonable range.

    ```ts
    class ProductSearchDto {
      @IsString()
      @IsNotEmpty()
      query: string;

      @IsString()
      @IsNotEmpty()
      language_code: string;

      @IsInt()
      @Min(1)
      page: number = 1;

      @IsInt()
      @Min(1)
      @Max(100)
      page_size: number = 10;
    }
    ```

# Testing Strategy

- Unit Testing: Use Jest for unit testing the services and controllers. Ensure that the API logic, such as product creation and search functionality, works as expected.
- Integration Testing: Test the interaction between the APIs and the PostgreSQL database using in-memory or dedicated test databases.
- End-to-End Testing: Use supertest to perform end-to-end testing for the API endpoints to verify that the API correctly handles real requests, such as product creation and searching with multilingual support.
