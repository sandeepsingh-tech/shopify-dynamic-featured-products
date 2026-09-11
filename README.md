# Shopify OS 2.0 Dynamic Featured Products Implementation Guide

A scalable, data-driven workflow to dynamically display unique featured products on specific collection pages without creating static, redundant templates.

---

## 1. Why Implement Dynamic Featured Products? (Benefits for Fresh/Standard Themes)

Most fresh or standard Shopify OS 2.0 themes (such as Dawn, Tinker, Craft, etc.) come with static design limitations out of the box:

* **Eliminates Template Bloat:** Without this approach, showing different featured items on different categories forces you to create dozens of duplicate collection templates (e.g., `collection.baking.json`, `collection.cookware.json`). Using a single dynamic metafield keeps your entire store running on just one clean `Default collection` template.
* **Higher Conversion on Long Collections:** Large collections can overwhelm buyers. Pinning high-margin, top-rated, or seasonal items directly at the top helps guide purchase decisions immediately.
* **Zero Theme Code Modifications:** Standard sections only accept entire collection handles, lacking native support for an ad-hoc product list. This implementation injects full product-list functionality through a safe, self-contained Custom Liquid block without modifying core theme assets or breaking theme update paths.
* **API & Headless/Mobile Ready:** By enabling Storefront API access on the metafield definition, the exact same curated product lists can be queried by external mobile apps (FlutterFlow, React Native) or headless storefronts.

---

## 2. Metafields vs. Metaobjects: Key Differences

* **Metafield:** Custom data attached directly to an existing Shopify resource (such as a Collection, Product, or Order). Use a Collection Metafield when you need to add specific attributes—like a targeted list of featured products—directly to individual collection entities.
* **Metaobject:** A standalone, custom multi-field data structure defined independently in Shopify (similar to a custom database table or CMS model, such as "Brand Profiles", "Size Charts", or "Lookbooks").

| Feature | Metafield | Metaobject |
| --- | --- | --- |
| **Data Scope** | Attached directly to an existing resource (Collection, Product) | Independent custom data model |
| **Best Used For** | Unique attributes per resource (e.g., custom product list for a category) | Reusable modular components across the store |
| **Location** | Bottom of individual Collection/Product admin pages | Managed under **Content > Metaobjects** |

---

## 3. Step 1: Create the Collection Metafield Definition

1. Navigate to **Shopify Admin > Settings > Custom data**.
2. Select **Collections** under the *Metafields* section.
3. Click **Add definition**.
4. Configure the fields:
* **Name:** `Featured Products`
* **Namespace and key:** `custom.featured_products`
* **Type:** Select **List** $\rightarrow$ **Product**.
* **Access:** Check/Toggle **Storefront API access** (allows external clients, mobile apps, or headless frontends to query this field).


5. Click **Save**.

---

## 4. Step 2: Assign Featured Products to Collections

1. Navigate to **Shopify Admin > Products > Collections**.
2. Open the desired collection (e.g., *Baking* or *Kitchen*).
3. Scroll down to the bottom of the page to the **Metafields** card.
4. Click into the **Featured Products** field.
5. Search and select the specific products you want highlighted for this collection.
6. Click **Save**.
7. Repeat this step for each collection where featured items are desired. Collections left blank will automatically hide the section on the storefront.

---

## 5. Step 3: Implement the Custom Liquid Section in the Theme Editor

Shopify’s native theme sections (e.g., *Featured Collection*) expect a single Collection reference rather than a `List of Products` metafield. To render this list dynamically across standard templates without theme code modifications, use a **Custom Liquid** section.

1. Navigate to **Online Store > Themes**.
2. Click **Customize** on your active theme.
3. In the top page selector dropdown, select **Collections > Default collection**.
4. In the left panel, click **Add section** and select **Custom Liquid**.
5. Position the section where you want the featured grid to display (e.g., above the main product grid).
6. Paste the following responsive code into the **Custom Liquid** code editor:

```liquid
{% if collection.metafields.custom.featured_products.value %}
  <div class="custom-featured-wrapper">
    <h2 class="custom-featured-heading">Featured in {{ collection.title }}</h2>
    <div class="custom-featured-grid">
      {% for product in collection.metafields.custom.featured_products.value %}
        <div class="custom-featured-card">
          <a href="{{ product.url }}" class="custom-featured-link">
            <div class="custom-featured-image-box">
              {% if product.featured_image %}
                <img 
                  src="{{ product.featured_image | image_url: width: 600 }}" 
                  alt="{{ product.title | escape }}" 
                  loading="lazy" 
                  class="custom-featured-img"
                >
              {% else %}
                {{ 'product-1' | placeholder_svg_tag: 'custom-featured-img' }}
              {% endif %}
            </div>
            <div class="custom-featured-info">
              <h3 class="custom-featured-title">{{ product.title }}</h3>
              <p class="custom-featured-price">{{ product.price | money }}</p>
            </div>
          </a>
        </div>
      {% endfor %}
    </div>
  </div>

  <style>
    .custom-featured-wrapper {
      max-width: 1200px;
      margin: 40px auto;
      padding: 0 20px;
      box-sizing: border-box;
      width: 100%;
    }

    .custom-featured-heading {
      font-size: 26px;
      font-weight: 600;
      margin-bottom: 24px;
      color: inherit;
      text-align: left;
    }

    .custom-featured-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 20px;
      width: 100%;
    }

    .custom-featured-card {
      background: transparent;
      border-radius: 12px;
      overflow: hidden;
      display: flex;
      flex-direction: column;
      transition: transform 0.2s ease;
    }

    .custom-featured-card:hover {
      transform: translateY(-4px);
    }

    .custom-featured-link {
      text-decoration: none;
      color: inherit;
      display: flex;
      flex-direction: column;
      height: 100%;
    }

    .custom-featured-image-box {
      width: 100%;
      aspect-ratio: 1 / 1;
      overflow: hidden;
      border-radius: 12px;
      background-color: #f7f7f7;
    }

    .custom-featured-img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
    }

    .custom-featured-info {
      padding: 12px 4px;
      display: flex;
      flex-direction: column;
      gap: 4px;
    }

    .custom-featured-title {
      font-size: 14px;
      line-height: 1.4;
      margin: 0;
      font-weight: 400;
      color: inherit;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
      overflow: hidden;
      word-break: break-word;
    }

    .custom-featured-price {
      font-size: 15px;
      font-weight: 600;
      margin: 0;
      color: inherit;
    }

    @media (max-width: 900px) {
      .custom-featured-grid {
        grid-template-columns: repeat(3, 1fr);
        gap: 16px;
      }
    }

    @media (max-width: 600px) {
      .custom-featured-grid {
        grid-template-columns: repeat(2, 1fr);
        gap: 12px;
      }

      .custom-featured-heading {
        font-size: 20px;
        margin-bottom: 16px;
      }

      .custom-featured-title {
        font-size: 13px;
      }

      .custom-featured-price {
        font-size: 14px;
      }
    }
  </style>
{% endif %}

```

7. Click **Save** in the top right corner.




### Verification Checklist & Troubleshooting

**Pre-Flight Verification**

* **Namespace & Key Accuracy:** Ensure the definition uses `custom.featured_products` exactly. If you change the namespace or key in Shopify Admin, update the Liquid variable path `collection.metafields.custom.featured_products.value` accordingly.
* **Storefront API Enabled:** Verify the checkbox/toggle for Storefront API access is checked if you plan to fetch these items in a headless setup, FlutterFlow app, or external API.
* **Dynamic Hiding:** If a collection does not have any products assigned in the metafield, the outer `{% if collection.metafields.custom.featured_products.value %}` wrapper guarantees zero markup or whitespace is rendered.

**Troubleshooting Common Issues**

* **Products do not show up on the collection page:**
* Confirm that products are actively published to the **Online Store** sales channel.
* Verify that the collection you are viewing in preview actually has products selected under the bottom **Metafields** card.


* **Images appear distorted or stretched:**
* The CSS uses `object-fit: cover` with an `aspect-ratio: 1 / 1` container. If your catalog uses wide landscape or tall portrait photography, you can change `object-fit: cover` to `object-fit: contain` inside `.custom-featured-img` to avoid cropping.


* **Currency formatting doesn't show properly:**
* `{{ product.price | money }}` relies on your store's default currency formatting settings found in **Settings > General > Store currency**.
