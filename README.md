# API Endpoints: **Products, Stock, Pricing and Decoration Pricing**

## Base URL & Auth

- **Base URL:** `https://api.legendlife.com.au/v1/`
- **Auth:** API key from your Legend Life account → *Integrations → API Data Feed*. Include it in requests as per your account instructions.

> The examples below omit auth headers for brevity.

---

## Public Products

### GET `/products`

Fetches public base products. Supports pagination and optional SKU/style filtering.

**Query params**

- `sku` *(string, optional)*: Provide a style code **or** a single SKU to filter.
- `page` *(int, optional)*: Page number (≥ 1).
- `pageSize` *(int, optional)*: Page size (1–100; default 20).

**Example**

```bash
curl "https://api.legendlife.com.au/v1/products?page=1&pageSize=20"
```

**200 Response (shape)**

```json
{
  "data": [
    {
      "styleCode": "4171",
      "name": "Rainier Softshell Jacket",
      "shortDescription": "...",
      "description": "...",
      "class": "APPAREL",
      "sizesAvailable": ["S","M","L"],
      "coloursAvailable": ["Black"],
      "brands": ["Legend"],
      "usages": ["Uniform"],
      "decoration": {
        "decorationTypes": [
          {
            "names": ["Embroidery"],
            "decorationPositions": [
              { "names": ["Left Chest"],
                "logoSizes": {
                  "maxWidthMm": 100,
                  "maxHeightMm": 100,
                  "maxDiameterMm": null
                }
              }
            ]
          }
        ]
      },
      "fabricTypes": ["Polyester"],
      "genders": ["Unisex"],
      "range": ["Rainier"],
      "bagCapacity": [],
      "bagFeatures": [],
      "capStructure": [],
      "umbrellaOpening": [],
      "categorisation": ["Outerwear"],
      "specifications": ["Water-resistant"],
      "cartonSpecifications": ["10 units/carton"],
      "images": {"baseImage": ["https://..."], "altImages": ["https://..."]},
      "url": "https://www.legendlife.com.au/...",
      "skus": [
        {
          "sku": "4171-BL.S",
          "name": "...",
          "shortDescription": "...",
          "description": "...",
          "searchColours": ["Black"],
          "colours": ["Black"],
          "sizes": ["S"],
          "image": ["https://..."]
        }
      ]
    }
  ],
  "currentPage": 1,
  "lastPage": 100,
  "perPage": 20,
  "total": 2000
}
```

**Notes**

- Decoration metadata (types, positions, logo size bounds) is included per style.
- SKU-level blocks list colour/size combinations and images.

---

## Customer Pricing & Stock

Pricing and customer stock views are style-based or SKU-based.

### GET `/customer/productStock`

Fetch **stock at locations** for a **single SKU**.

**Query params**

- `sku` *(string, required)*: e.g. `4171-BL`

**Example**

```bash
curl "https://api.legendlife.com.au/v1/customer/productStock?sku=4171-BL"
```

**200 Response (shape)**

```json
{
  "data": {
    "sku": "4171-BL",
    "name": "Rainier Softshell Jacket",
    "shortDescription": "...",
    "stock": [
      { "location": "QLD DC", "qty": "120" },
      { "location": "NSW DC", "qty": "45" }
    ]
  }
}
```

---

### GET `/customer/styleStock`

Fetch **all SKUs + location stock** for a **style**.

**Query params**

- `styleCode` *(string, required)*: e.g. `4171`

**Example**

```bash
curl "https://api.legendlife.com.au/v1/customer/styleStock?styleCode=4171"
```

**200 Response (shape)**

```json
{
  "data": {
    "styleCode": "4171",
    "name": "Rainier Softshell Jacket",
    "shortDescription": "...",
    "rrpPrice": 129.9,
    "stock": [
      {
        "sku": "4171-BL.S",
        "locations": [ {"name": "QLD DC", "stockLevel": "25"} ]
      },
      {
        "sku": "4171-BL.M",
        "locations": [ {"name": "QLD DC", "stockLevel": "30"} ]
      }
    ]
  }
}
```

---

### GET `/customer/stylePricing`

Fetch **your tiered pricing** for a **style**.

**Query params**

- `styleCode` *(string, required)*

**Example**

```bash
curl "https://api.legendlife.com.au/v1/customer/stylePricing?styleCode=4171"
```

**200 Response (shape)**

```json
{
  "data": {
    "styleCode": "4171",
    "name": "Rainier Softshell Jacket",
    "shortDescription": "...",
    "rrpPrice": 129.9,
    "pricing": {
      "qty1": 1,   "price1": 120.00,
      "qty2": 10,  "price2": 115.00,
      "qty3": 50,  "price3": 110.00,
      "qty4": 100, "price4": 105.00,
      "qty5": 250, "price5": 99.00
    }
  }
}
```

---

### GET `/customer/stylePricingAndStock`

Fetch **pricing and stock together** for a **style**.

**Query params**

- `styleCode` *(string, required)*

**Example**

```bash
curl "https://api.legendlife.com.au/v1/customer/stylePricingAndStock?styleCode=4171"
```

**200 Response (shape)**

```json
{
  "data": {
    "styleCode": "4171",
    "name": "Rainier Softshell Jacket",
    "shortDescription": "...",
    "rrpPrice": 129.9,
    "pricing": { "qty1": 1, "price1": 120.00, "qty2": 10, "price2": 115.00 },
    "stock": [ { "sku": "4171-BL.S", "locations": [ {"name":"QLD DC","stockLevel":"25"} ] } ]
  }
}
```

---

## Errors

All endpoints use common error shapes.

**401 Unauthenticated**

```json
{ "message": "Unauthenticated" }
```

**403 Authorization error**

```json
{ "message": "Unauthorized" }
```

**422 Validation error**

```json
{
  "message": "The given data was invalid.",
  "errors": { "styleCode": ["The styleCode field is required."] }
}
```

---

## FAQ / Tips

- **Passing arrays in query strings** (e.g. for decoration checks): use `skus[]` multiple times
  ```text
  /decorations/canDecorate?skus[]=8001-BL.CW&skus[]=4007-BL.BOR-S.M
  ```
- **Pagination**: `/products` returns `currentPage`, `lastPage`, `perPage`, `total`.
- **RRP vs customer pricing**: `rrpPrice` is included alongside per-customer tiered pricing in style calls.

---

## Legacy SOAP → REST mapping (for reference)

Use this if you’re migrating from the SOAP v2.1 documentation.

| SOAP (v2.1)                                         | REST (this README)                               |
| --------------------------------------------------- | ------------------------------------------------ |
| Public Product List `epicentre_products.publiclist` | `GET /products`                                  |
| Customer Specific Product Stock                     | `GET /customer/productStock` (per SKU)           |
| Customer Specific Style Pricing                     | `GET /customer/stylePricing` (per style)         |
| —                                                   | `GET /customer/styleStock` (style stock per SKU) |
| —                                                   | `GET /customer/stylePricingAndStock` (combined)  |



# Legend Life Decoration API Workflow Documentation

This guide provides a step-by-step workflow for utilizing the Legend Life Decoration API to determine decoration options, configurations, and pricing for garments. The workflow is based on the provided OpenAPI (Swagger) specification.

---

## **Step 1: Determine if a Product Can Be Decorated**

**Endpoint:** `GET /canDecorate`

**Purpose:**  
Check if your desired product(s) can be decorated.

**Required Parameter:**  
- `skus` (array of strings): The SKUs of products to check.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/canDecorate?skus=8001-BL.CW,4007-BL.BOR-S.M`

**Response:**  
Returns a map of SKU to boolean indicating decorate-ability.

---

## **Step 2: Retrieve Available Decoration Types for a SKU**

**Endpoint:** `GET /decorationTypes`

**Purpose:**  
Get a list of decoration types (e.g., embroidery, printing) available for a specific SKU.

**Required Parameter:**  
- `sku` (string): The product SKU.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/decorationTypes?sku=4007-BL.BOR-S.M`

**Response:**  
Returns an array of decoration type objects `{ id, name, decorationPositionCount }`.

---

## **Step 3: Determine Available Decoration Positions**

**Endpoint:** `GET /decorationPositions`

**Purpose:**  
Find out which areas/positions can be decorated on the garment for the selected decoration type.

**Required Parameters:**  
- `sku` (string): The product SKU.
- `decorationType` (integer): The ID of the chosen decoration type.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/decorationPositions?sku=4007-BL.BOR-S.M&decorationType=1`

**Response:**  
Returns available positions (e.g., left chest, sleeve).

---

## **Step 4: Retrieve Decoration Options**

**Endpoint:** `GET /decorationOption`

**Purpose:**  
Get miscellaneous options (e.g., thread colors, stitch count) specific to SKU, decoration type, and optionally, position.

**Required Parameters:**  
- `sku` (string): The product SKU.
- `decorationType` (integer): Decoration type ID.
- `qty` (number): Quantity of decorations.

**Optional Parameters:**  
- `decorationPosition` (integer): Position ID (to refine options).
- `decorationItem` (integer): Sub-option item ID.
- `zAxisDecorationItem` (integer): Z-axis option item ID.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/decorationOption?sku=4007-BL.BOR-S.M&decorationType=1&qty=100`

> **Tip:**  
> Request again with `decorationPosition` after selecting a position to see refined options.

---

## **Step 5: Retrieve Acceptable Logo Sizes**

**Endpoint:** `GET /logoSize`

**Purpose:**  
Determine minimum and maximum logo dimensions for the selected SKU, decoration type, position, and option.

**Required Parameters:**  
- `sku` (string): The product SKU.
- `decorationType` (integer): Decoration type ID.

**Optional Parameters:**  
- `decorationPosition` (integer): Position ID.
- `decorationItem` (integer): Decoration option item ID.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/logoSize?sku=8001-BL.CW&decorationType=1&decorationPosition=2&decorationItem=4`

**Response:**  
Returns min/max width, height, and diameter in millimeters.

---

## **Step 6: Retrieve Pricing for Decoration**

**Endpoint:** `GET /pricing`

**Purpose:**  
Get the calculated price for the selected configuration and quantity.

**Required Parameters:**  
- `sku` (string): The product SKU.
- `decorationType` (integer): Decoration type ID.
- `qty` (number): Quantity.

**Optional Parameters:**  
- `decorationItem` (integer): Decoration option item ID.
- `zAxisDecorationItem` (integer): Z-axis option item ID.

**Example Request:**  
`GET https://api.legendlife.com.au/v1/decorations/pricing?sku=8001-BL.CW&decorationType=1&decorationItem=5&qty=100`

**Response:**  
Returns price per item, setup fees, minimum charge, and pricing quantity.

---

## **Additional Endpoints**

- **Get Minimum Order Quantity (MOQ) for a Decoration Type:**  
  `GET /decorationMoq?sku=4007-BL.BOR-S.M&decorationType=1`
- **Get Information Sections:**  
  `GET /informationSections?sku=4007-BL.BOR-S.M&decorationType=1`
- **Get Decoration Types for Multiple SKUs:**  
  `GET /decorationTypesSkus?skus=8001-BL.CW,4007-BL.BOR-S.M`

---

## **Authentication**

All endpoints require authentication.  
Get your API key from the [Legend Life API Data Feed](https://www.legendlife.com.au/integrations/index/) in your account.

---

## **Typical Workflow Summary**

1. **Check if SKU(s) can be decorated** (`/canDecorate`)
2. **Fetch available decoration types** (`/decorationTypes`)
3. **Select a decoration type, get position options** (`/decorationPositions`)
4. **Get decoration options, refine choices** (`/decorationOption`)
5. **Request logo size constraints** (`/logoSize`)
6. **Request pricing** (`/pricing`)

---

## **Error Handling**

- **401 Unauthorized:**  
  Authentication failed; check your API key.
- **422 Validation Error:**  
  Provided parameters are invalid; check parameter types and values.

---

## **References**

- [Legend Life API Data Feed](https://www.legendlife.com.au/integrations/index/)
- [Legend Life Main Site](https://www.legendlife.com.au/)
