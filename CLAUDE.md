# AGENTS.md

🚨 MANDATORY: YOU MUST CALL "learn_shopify_api" ONCE WHEN WORKING WITH LIQUID THEMES.

## Theme Architecture

**Key principles: focus on generating snippets, blocks, and sections; users may create templates using the theme editor**

### Directory structure

```
.
├── assets          # Stores static assets (CSS, JS, images, fonts, etc.)
├── blocks          # Reusable, nestable, customizable components
├── config          # Global theme settings and customization options
├── layout          # Top-level wrappers for pages (layout templates)
├── locales         # Translation files for theme internationalization
├── sections        # Modular full-width page components
├── snippets        # Reusable Liquid code or HTML fragments
└── templates       # Templates combining sections and blocks to define page structures
```

#### `sections`

- Sections are `.liquid` files for reusable modules customizable by merchants
- Sections can include blocks which allow merchants to add, remove, and reorder content
- Sections require `{% schema %}` tag — validate using `schemas/section.json`
- Examples: hero banners, product grids, testimonials, featured collections

#### `blocks`

- Blocks are `.liquid` files for reusable small components editable in the theme editor
- Blocks can include nested blocks
- Blocks require `{% schema %}` tag — validate using `schemas/theme_block.json`
- Blocks must have `{% doc %}` as the header if statically rendered via `{% content_for 'block', id: '42', type: 'block_name' %}`
- Examples: individual testimonials, slides in a carousel, feature items

#### `snippets`

- Snippets are reusable code fragments rendered via the `render` tag
- Ideal for logic that needs reuse but not direct theme editor editing
- Snippets accept parameters for dynamic behavior
- Snippets must have the `{% doc %}` tag as the header
- Examples: buttons, meta-tags, css-variables, form elements

#### `layout`

- Defines the overall HTML structure including `<head>` and `<body>`
- Must include `{{ content_for_header }}` in `<head>` and `{{ content_for_layout }}` for page content

#### `config`

- `config/settings_schema.json` — global theme settings schema. Validate using `schemas/theme_settings.json`
- `config/settings_data.json` — holds the data for those settings

#### `assets`

- Keep only `critical.css` and static files needed on every page
- Prefer `{% stylesheet %}` and `{% javascript %}` tags for component-level styles/scripts

#### `locales`

- Translation files by language code (e.g., `en.default.json`, `fr.json`)
- Validate using `schemas/translations.json`

#### `templates`

- JSON files defining which sections and blocks appear on each page type

### CSS & JavaScript

- Write CSS and JavaScript per component using `{% stylesheet %}` and `{% javascript %}` tags
- `{% stylesheet %}` and `{% javascript %}` are only supported in `snippets/`, `blocks/`, and `sections/`

### LiquidDoc

Snippets and statically-rendered blocks must include a LiquidDoc header:

```liquid
{% doc %}
  Renders a responsive image that might be wrapped in a link.

  @param {image} image - The image to be rendered
  @param {string} [url] - An optional destination URL for the image

  @example
  {% render 'image', image: product.featured_image %}
{% enddoc %}

<a href="{{ url | default: '#' }}">{{ image | image_url: width: 200, height: 200 | image_tag }}</a>
```

## The `{% schema %}` tag on blocks and sections

### Good practices

**Single property settings** → use CSS variables:
```liquid
<div class="collection" style="--gap: {{ block.settings.gap }}px">
  Example
</div>

{% stylesheet %}
  .collection { gap: var(--gap); }
{% endstylesheet %}

{% schema %}
{
  "settings": [{
    "type": "range",
    "label": "gap",
    "id": "gap",
    "min": 0,
    "max": 100,
    "unit": "px",
    "default": 0
  }]
}
{% endschema %}
```

**Multiple property settings** → use CSS classes:
```liquid
<div class="collection {{ block.settings.layout }}">
  Example
</div>

{% schema %}
{
  "settings": [{
    "type": "select",
    "id": "layout",
    "label": "layout",
    "values": [
      { "value": "collection--full-width", "label": "t:options.full" },
      { "value": "collection--narrow", "label": "t:options.narrow" }
    ]
  }]
}
{% endschema %}
```

#### Mobile layouts

For merchant-selectable column counts on mobile, use a select input:
```liquid
{% schema %}
{
  "type": "select",
  "id": "columns_mobile",
  "label": "Columns on mobile",
  "options": [
    { "value": 1, "label": "1" },
    { "value": "2", "label": "2" }
  ]
}
{% endschema %}
```

## Liquid

### Liquid delimiters

- `{{ ... }}` — output a value
- `{{- ... -}}` — output, trim surrounding whitespace
- `{% ... %}` — logic tag, no output
- `{%- ... -%}` — logic tag, trim surrounding whitespace

### Liquid conditions — key rules

- **ALWAYS use nested `if`** when logic requires more than one logical operator
- **No parentheses** in Liquid conditions
- **No ternary operator** — always use `{% if cond %}...{% endif %}`
- Operators: `==`, `!=`, `>`, `<`, `>=`, `<=`, `or`, `and`, `contains`
- `contains` checks substrings or array membership (strings only, not objects in arrays)

```liquid
{% if product.available %}
  {% if product.price > 1000 %}
    Premium in-stock item
  {% endif %}
{% endif %}
```

### Variables

```liquid
{% assign my_variable = 'value' %}
{% capture my_variable %}Contents{% endcapture %}
{% increment counter %}
{% decrement counter %}
```

### Shopify-specific Liquid tags

#### `content_for`
Requires a type: `'blocks'` renders all child blocks; `'block'` renders a single static block.
```liquid
{% content_for 'blocks' %}
{% content_for 'block', type: "slide", id: "slide-1" %}
```

#### `form`
Requires a form type. Common types: `cart`, `contact`, `create_customer`, `customer_login`, `new_comment`, `product`, `recover_customer_password`. See Shopify docs for the full list.
```liquid
{% form 'form_type' %}
  content
{% endform %}
```

#### `javascript` / `stylesheet`
One per section/block/snippet. Liquid is NOT rendered inside these tags.
```liquid
{% javascript %}
  javascript_code
{% endjavascript %}

{% stylesheet %}
  css_styles
{% endstylesheet %}
```

#### `paginate`
Required when iterating over more than 50 items. Max pagination to 25,000th item.
```liquid
{% paginate collection.products by 12 %}
  {% for product in collection.products %}
    {{ product.title }}
  {% endfor %}
  {{ paginate | default_pagination }}
{% endpaginate %}
```

#### `section` / `sections`
```liquid
{% section 'name' %}
{% sections 'name' %}
```

#### `style`
Color settings inside `{% style %}` tags update live in the theme editor without page refresh.

## Translation standards

- **Every user-facing text** must use translation filters: `{{ 'key' | t }}`
- **Update `locales/en.default.json`** with all new keys
- **Use descriptive, hierarchical keys** (snake_case, max 3 levels deep)
- **Only add English text**; translators handle other languages
- **Use sentence case** for all user-facing text (first word + proper nouns only)
- Use interpolation for dynamic content: `{{ 'key' | t: var: value }}`
- Escape variables unless they output HTML: `{{ variable | escape }}`

```liquid
<h2>{{ 'sections.featured_collection.title' | t }}</h2>
<p>{{ 'products.price_range' | t: min: product.price_min | money, max: product.price_max | money }}</p>
```

Locale file example:
```json
{
  "general": { "cart": "Cart", "checkout": "Checkout" },
  "products": {
    "add_to_cart": "Add to cart",
    "price_range": "From {{ min }} to {{ max }}"
  }
}
```

Schema locale files use `.schema.json` extension and IETF language tags (e.g., `en.schema.json`).

## Localization file structure

```
locales/
├── en.default.json          # English (required)
├── en.default.schema.json   # English schema (required)
├── es.json / es.schema.json
├── fr.json / fr.schema.json
└── pt-BR.json / pt-BR.schema.json
```

## Examples

### `snippet`

```liquid
{% doc %}
  Renders a responsive image optionally wrapped in a link.

  @param {image} image - The image to be rendered
  @param {string} [url] - An optional destination URL
  @param {string} [css_class] - Optional class for the wrapper
  @param {number} [width] - Max width of the rendered image
  @param {number} [height] - Max height of the rendered image
  @param {string} [crop] - Crop position

  @example
  {% render 'image', image: product.featured_image %}
  {% render 'image', image: product.featured_image, url: product.url %}
{% enddoc %}

{% liquid
  unless height
    assign width = width | default: image.width
  endunless
  if url
    assign wrapper = 'a'
  else
    assign wrapper = 'div'
  endif
%}

<{{ wrapper }}
  class="image {{ css_class }}"
  {% if url %}href="{{ url }}"{% endif %}
>
  {{ image | image_url: width: width, height: height, crop: crop | image_tag }}
</{{ wrapper }}>

{% stylesheet %}
  .image { display: block; width: 100%; height: auto; overflow: hidden; }
  .image > img { width: 100%; height: auto; }
{% endstylesheet %}
```

### `block`

```liquid
{% doc %}
  Renders a text block.

  @example
  {% content_for 'block', type: 'text', id: 'text' %}
{% enddoc %}

<div
  class="text {{ block.settings.text_style }}"
  style="--text-align: {{ block.settings.alignment }}"
  {{ block.shopify_attributes }}
>
  {{ block.settings.text }}
</div>

{% stylesheet %}
  .text { text-align: var(--text-align); }
  .text--title { font-size: 2rem; font-weight: 700; }
  .text--subtitle { font-size: 1.5rem; }
{% endstylesheet %}

{% schema %}
{
  "name": "t:general.text",
  "settings": [
    { "type": "text", "id": "text", "label": "t:labels.text", "default": "Text" },
    {
      "type": "select",
      "id": "text_style",
      "label": "t:labels.text_style",
      "options": [
        { "value": "text--title", "label": "t:options.text_style.title" },
        { "value": "text--subtitle", "label": "t:options.text_style.subtitle" },
        { "value": "text--normal", "label": "t:options.text_style.normal" }
      ],
      "default": "text--title"
    },
    { "type": "text_alignment", "id": "alignment", "label": "t:labels.alignment", "default": "left" }
  ],
  "presets": [{ "name": "t:general.text" }]
}
{% endschema %}
```

### `section`

```liquid
<div class="example-section full-width">
  {% if section.settings.background_image %}
    <div class="example-section__background">
      {{ section.settings.background_image | image_url: width: 2000 | image_tag }}
    </div>
  {% endif %}
  <div class="example-section__content">
    {% content_for 'blocks' %}
  </div>
</div>

{% stylesheet %}
  .example-section { position: relative; overflow: hidden; width: 100%; }
  .example-section__background {
    position: absolute; width: 100%; height: 100%; z-index: -1; overflow: hidden;
  }
  .example-section__background img {
    position: absolute; width: 100%; height: auto;
    top: 50%; left: 50%; transform: translate(-50%, -50%);
  }
  .example-section__content { display: grid; grid-template-columns: var(--content-grid); }
  .example-section__content > * { grid-column: 2; }
{% endstylesheet %}

{% schema %}
{
  "name": "t:general.custom_section",
  "blocks": [{ "type": "@theme" }],
  "settings": [
    { "type": "image_picker", "id": "background_image", "label": "t:labels.background" }
  ],
  "presets": [{ "name": "t:general.custom_section" }]
}
{% endschema %}
```
