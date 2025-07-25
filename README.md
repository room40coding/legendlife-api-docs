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
`GET https://api.legendlife.com.au/decorations/v1/canDecorate?skus=8001-BL.CW,4007-BL.BOR-S.M`

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
`GET https://api.legendlife.com.au/decorations/v1/decorationTypes?sku=4007-BL.BOR-S.M`

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
`GET https://api.legendlife.com.au/decorations/v1/decorationPositions?sku=4007-BL.BOR-S.M&decorationType=1`

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
`GET https://api.legendlife.com.au/decorations/v1/decorationOption?sku=4007-BL.BOR-S.M&decorationType=1&qty=100`

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
`GET https://api.legendlife.com.au/decorations/v1/logoSize?sku=8001-BL.CW&decorationType=1&decorationPosition=2&decorationItem=4`

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
`GET https://api.legendlife.com.au/decorations/v1/pricing?sku=8001-BL.CW&decorationType=1&decorationItem=5&qty=100`

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
