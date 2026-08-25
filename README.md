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
            "nameAndIds": [
              {
                "id": 1,
                "name": "Embroidery"
              }
            ],
            "decorationPositions": [
              {
                "names": ["Left Chest"],
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

### Umbrella Products: Pre-loaded Decoration Options

For **umbrella products** (`class: "UMBRELLAS"`), the API includes additional decoration option data directly in the `/products` response. This eliminates the need to make separate calls to `/decorations/decorationOption` for these products.

Each decoration type for an umbrella product may include:

- `umbrellaDecorationOption` - The primary decoration option (e.g., "Number of Panels")
- `umbrellaZAxisDecorationOption` - The secondary/z-axis decoration option (e.g., "Transfer Size")

**Example umbrella product decoration type:**

```json
{
  "decorationTypes": [
    {
      "names": ["Supacolour"],
      "nameAndIds": [{ "id": 6, "name": "Supacolour" }],
      "decorationPositions": [],
      "umbrellaDecorationOption": {
        "id": 19,
        "name": "Number of Panels (Supacolour)",
        "items": [
          { "id": 70, "name": "1 Panel" },
          { "id": 72, "name": "2 Panels" },
          { "id": 73, "name": "4 Panels" },
          { "id": 74, "name": "8 Panels" }
        ]
      },
      "umbrellaZAxisDecorationOption": {
        "id": 15,
        "name": "Size (Umbrellas)",
        "items": [
          { "id": 55, "name": "150mm x 100mm" },
          { "id": 56, "name": "210mm x 150mm" },
          { "id": 57, "name": "300mm x 210mm" }
        ]
      }
    }
  ]
}
```

**Using umbrella decoration options for pricing:**

The item IDs from `umbrellaDecorationOption` and `umbrellaZAxisDecorationOption` can be passed directly to the `/decorations/pricing` endpoint:

```bash
curl "https://api.legendlife.com.au/v1/decorations/pricing?sku=2005-BL&decorationType=6&decorationItem=70&zAxisDecorationItem=55&qty=100"
```

In this example:
- `decorationItem=70` corresponds to "1 Panel" from `umbrellaDecorationOption`
- `zAxisDecorationItem=55` corresponds to "150mm x 100mm" from `umbrellaZAxisDecorationOption`

> **Note:** These properties only appear for umbrella products. Non-umbrella products will not include `umbrellaDecorationOption` or `umbrellaZAxisDecorationOption` in the response.

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

## Bulk Decoration Pricing

### GET `/decorations/pricingGrid`

Fetch the **entire online decoration price grid in one response**: every decoration grouping
available for online ordering — with its decoration type, its selectable decoration items, its
z-axis items where it has them, and its complete quantity-break price rows — plus a map of every
style code to the groupings that style may be decorated with. Together they are everything needed
to quote decoration offline.

This replaces a per-`(sku, decorationType, decorationItem, zAxisDecorationItem)` fan-out over
`/decorations/pricingBreakpoints`: pull the grid once, cache it, and price any decoration on any
style at any quantity out of your own database. Only groupings that can actually be ordered online
appear — precisely the set the per-SKU decoration endpoints serve — so the grid never offers a
decoration method Legend Life will not take online. SYSPRO charge and stock codes are withheld.

**Query params**

None.

**Example**

```bash
curl -H "Authorization: Bearer $API_KEY" \
  "https://api.legendlife.com.au/v1/decorations/pricingGrid"
```

**200 Response (shape)**

```json
{
  "hash": "9f2c1b7e…",
  "generatedAt": "2026-08-25T02:14:07+00:00",
  "groupings": [
    {
      "id": 12,
      "name": "Embroidery - Left Chest",
      "decorationType": { "id": 1, "name": "Embroidery" },
      "items": [ { "id": 4, "name": "Up to 5,000 stitches" } ],
      "zAxisItems": [],
      "prices": [
        {
          "decorationItem": 4,
          "zAxisDecorationItem": null,
          "minQty": 1,
          "maxQty": 24,
          "each": 6.50,
          "setupFee": 25.00,
          "repeatSetupFee": 0.00,
          "minimumCharge": 65.00,
          "pricingQty": 1
        }
      ]
    }
  ],
  "styles": {
    "4171": [ { "grouping": 12, "moq": 10 } ]
  }
}
```

`groupings[].decorationType` is `null` on a grouping whose decoration type has since been retired.

---

#### Pricing a decoration from the grid

Look the style code up in `styles`, take one of the groupings it lists, and read that grouping's
`prices` rows for the `decorationItem` you are quoting — and for `zAxisDecorationItem`, which is
`null` on rows for groupings that have no z-axis. Select the row whose quantity break contains your
quantity:

- `minQty` and `maxQty` are both **inclusive**, and `null` means unbounded on that side.
- A row with **neither** bound set covers no quantity at all, which is how our own reads treat it.
- Do not quote below the `moq` that `styles` gives for that style and grouping.

**Within one grouping, where two rows for the same decoration item and z-axis item both cover your
quantity, the lowest `each` applies.** The configuration permits overlapping quantity breaks and
Legend Life's own per-SKU pricing read resolves an overlap that way.

That rule is per grouping and **does not extend across them**. Where a style lists several groupings
that could carry the same decoration, our per-SKU read prices against one of them rather than taking
the cheapest of all, so do not assume the lowest `each` across groupings is the price we will
charge. Ask `/decorations/pricing` for an exact SKU and quantity when a style's groupings overlap
and the difference matters.

**Price row fields**

| Field | Meaning |
| --- | --- |
| `decorationItem` | Id of the decoration item this row prices; matches an entry in the grouping's `items`. |
| `zAxisDecorationItem` | Id of the z-axis item, or `null` on groupings with no z-axis. |
| `minQty` / `maxQty` | Inclusive quantity break bounds; `null` is unbounded on that side. |
| `each` | Price of **one pricing unit** at this break. |
| `pricingQty` | How many pricing units the row covers — `1` on an ordinary row (see below). |
| `setupFee` | Setup charge configured on this price row, or `null` where none is set. Charged on top of the per-unit price. |
| `repeatSetupFee` | Repeat setup charge configured on this price row, or `null` where none is set. |
| `minimumCharge` | The least the decoration is charged at, however small the run; `0` where there is no minimum. |

`each` is the price of one pricing unit and `pricingQty` is how many of those units the row covers —
`1` on an ordinary row. **Above `1` it is the multiplier carried by a decoration item that decorates
several places at once**, an umbrella's panels being the usual case: `each` is then the price per
panel, and `each * pricingQty` is the price of that decoration item at that break, which is what
decorating one product with it costs. The two are split so `each` stays comparable per unit.

`minimumCharge` is the least the decoration will be charged at however small the run, and `0` where
there is no minimum; quoting without it under-charges small orders.

---

#### The style map

`styles` is a JSON **object keyed by style code** — `{}` when empty, never an array — whose values
list `{ "grouping": <id>, "moq": <int> }` pairs. Every grouping id it names appears in `groupings`.

The map is complete by construction, so **a style code absent from `styles` means there is no online
decoration pricing for that style**: an answer, not missing data.

It answers **per style, not per SKU**. Where one style's SKUs sit in several product classes, its
list is the union across them and its `moq` the lowest that applies to any of them, so a colour or
size of that style can in principle be narrower than the style's own entry. Quote from the grid;
`/decorations/decorationTypes` and `/decorations/decorationMoq` answer for an exact SKU when you
need one.

---

#### Keeping a cached copy in step

`hash` is a content hash over the whole grid, **excluding `generatedAt`**, and is also returned as
the `ETag` header. Send it back on the next pull as `If-None-Match` — quoted as the header gave it,
or bare as the body published it — and an unchanged grid answers `304 Not Modified` with no body, so
a nightly sync transfers nothing on a quiet day. A changed grid answers `200` with the new payload
and a new `hash`.

```bash
# First pull: keep the hash
curl -H "Authorization: Bearer $API_KEY" \
  "https://api.legendlife.com.au/v1/decorations/pricingGrid"

# Subsequent pulls: 304 Not Modified with no body when nothing changed
curl -H "Authorization: Bearer $API_KEY" \
     -H 'If-None-Match: "9f2c1b7e…"' \
  "https://api.legendlife.com.au/v1/decorations/pricingGrid"
```

`generatedAt` is when the grid was **composed**, not when it was served: the payload is cached for
up to an hour, so a `200` can carry a `generatedAt` older than the request. Saving a price or
configuration change queues a flush of that cache, so a change normally reaches the next pull rather
than waiting the hour out.

One exception: the **product catalogue** itself is synced on its own schedule and does not queue that
flush. A style newly added, withdrawn or reclassified can therefore take up to the full hour to
appear in, or leave, `styles` — and until it does the `ETag` is unchanged, so a conditional pull
answers `304`. Treat `styles` as accurate to within one hour of the product catalogue, and
`groupings` as immediate.

**Suggested integration**

1. Pull the grid, store `hash`, and load `groupings` + `styles` into your own tables.
2. Price locally from those tables.
3. Re-pull nightly with `If-None-Match: "<hash>"`; on `304` do nothing, on `200` replace and store
   the new `hash`.

**Errors**

- `401 Unauthenticated` — see [Errors](#errors). The endpoint is authenticated like every other
  decoration endpoint and takes no parameters, so there is no `422` to handle.

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
`GET https://api.legendlife.com.au/v1/decorations/canDecorate?skus[]=8001-BL.CW&skus[]=4007-BL.BOR-S.M`

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
  `GET /decorations/decorationMoq?sku=4007-BL.BOR-S.M&decorationType=1`
- **Get Information Sections:**  
  `GET /decorations/informationSections?sku=4007-BL.BOR-S.M&decorationType=1`
- **Get Decoration Types for Multiple SKUs:**  
  `GET /decorations/decorationTypesSkus?skus[]=8001-BL.CW&skus[]=4007-BL.BOR-S.M`
- **Get All Available Decoration Types**  
  `GET /decorations/allDecorationTypes`
- **Get Decoration Pricing Breakpoints**
  `GET /decorations/pricingBreakpoints`
- **Get the whole online decoration price grid in one call:**  
  `GET /decorations/pricingGrid` — see [Bulk Decoration Pricing](#bulk-decoration-pricing). Pull it
  once, cache it, and price any decoration on any style at any quantity locally instead of walking
  Steps 1-6 per SKU.

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

### Umbrella Products: Simplified Workflow

For **umbrella products**, the `/products` endpoint already includes `umbrellaDecorationOption` and `umbrellaZAxisDecorationOption` data, allowing you to skip steps 3-4:

1. **Fetch product data** (`/products?sku=2005`) — decoration options are included
2. **Request pricing** (`/pricing`) — use the item IDs from the product response directly

### Bulk Integrators: Skip the Walk

If you are importing pricing rather than quoting one configuration at a time, skip the whole
workflow and pull `GET /decorations/pricingGrid` once — see
[Bulk Decoration Pricing](#bulk-decoration-pricing).

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
