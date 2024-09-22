# License Server

The **License Server** is a Java-based web server that manages the issuance of software licenses for multiple products. It provides secure endpoints for issuing and requesting licenses, leveraging hashed customer identifiers sent by the **Integrator** component after payment confirmation. The raw customer ID is used for internal queries and validation, allowing the License Server to track licenses tied to specific customers.

## Key Features

- **License Issuance**: Licenses are issued after payment confirmation, tied to a customer’s unique identifier (e.g., a Shopify customer ID).
- **Customer ID Hashing and Shortening**: The customer’s ID is hashed by the **Integrator**, and both the raw and hashed IDs are sent to the License Server. The raw ID is stored for internal queries, while the hashed ID is used by customers to request licenses.
- **License Voucher Generation**: License vouchers are AES 256 encrypted tokens containing the license expiration date. These vouchers are securely requested by applications and stored locally.
- **License Storage**: Granted licenses are stored in simple text files in CSV format, one per product.
- **Secure Encryption**: AES 256 encryption with initialization vectors (IVs) ensures that license information is secure.

## Endpoints

The License Server provides two primary endpoints:

### 1. License Issuance Endpoint

**URL**: `/api/v1/licenses/grant`

This endpoint is responsible for issuing licenses once payment is confirmed. The **Integrator** sends the hashed customer ID (which will be given to the customer for input into their application) and the raw customer ID (used for internal license queries on the License Server). The License Server uses the raw ID to manage the licenses and customer queries while storing the hashed ID for customer interactions.

Granted licenses are stored in a CSV file format, where each product has its own file for tracking issued licenses. The CSV contains information such as:

- Hashed Customer ID
- Raw Customer ID
- Product ID
- Expiry Date
- License Limit

#### Request

**Method**: `POST`

**Request Body**:

```json
{
  "orderId": "12345",
  "productId": "67890",
  "customerId": "shopify_customer_id",
  "hashedCustomerId": "hashed_customer_id",
  "expiryDate": "2025-09-17",
  "licenseLimit": 3
}
```

#### Response

**Status Codes**:

- `200 OK`: License successfully issued and stored in the CSV file.
- `400 Bad Request`: Invalid request parameters.
- `500 Internal Server Error`: Server encountered an error.

---

### 2. License Voucher Request Endpoint

**URL**: `/api/v1/licenses/request-voucher`

This endpoint allows an application to request a new **license voucher**, using the hashed customer ID. The voucher contains an encrypted expiry date and is valid until that date, after which the application will need to request a new voucher.

#### Request

**Method**: `POST`

**Request Body**:

```json
{
  "licenseKey": "hashed_customer_id",
  "productId": "67890"
}
```

**Status Codes**:

- `200 OK`: License voucher successfully issued.
- `400 Bad Request`: Invalid request parameters.
- `500 Internal Server Error`: Server encountered an error.

