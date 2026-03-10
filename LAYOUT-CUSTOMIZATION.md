# Custom Work Page Layout

Documents the changes made to create a structured, field-by-field work page layout, replacing Canopy's default single metadata block. Also serves as a guide to replicate this on another Canopy site.

---

## What the layout looks like

```
┌─────────────────────────────────────────┐
│  [Viewer]        │  Title                │
│  (image)         │  Location • Completed │
│                  │  Summary              │
├─────────────────────────────────────────┤
│  Historical Context (full width)         │
├─────────────────────────────────────────┤
│  Key People      │  Attribution          │
├─────────────────────────────────────────┤
│  Related Items (full width)              │
└─────────────────────────────────────────┘
```

---

## Files changed

### 1. `app/components/MetadataField.tsx` — new component

Canopy's built-in `<Metadata>` renders all manifest metadata fields together in one unstyled block. `MetadataField` extracts **one specific field by name** and lets you place and style it independently.

It searches `manifest.metadata` (the array of `{label, value}` pairs in a IIIF manifest) for a matching label and returns the value(s).

**Two display modes:**
- `display="inline"` — just the text, no heading (good for bylines)
- `display="block"` (default) — renders with an optional `label` heading above it

**Props:**

| Prop | Type | Description |
|---|---|---|
| `manifest` | object | The IIIF manifest passed in as `props.manifest` |
| `field` | string | The metadata label to look up (must match exactly, case-insensitive) |
| `label` | string (optional) | Heading text to display above the value |
| `prefix` | string (optional) | Text prepended inline before the value |
| `display` | `"inline"` or `"block"` | Rendering mode (default: `"block"`) |

---

### 2. `app/components/mdx.tsx` — register the component

Add one line to the `components` export so MDX files can use `<MetadataField>`:

```ts
export const components = {
  Example: './Example.tsx',
  MetadataField: './MetadataField.tsx', // add this
};
```

---

### 3. `content/works/_layout.mdx` — the page template

This file controls what every work page looks like. Key structural points:

- The two-column hero (viewer + text) uses a plain `<div className="canopy-work--hero">` **inside** `<Container>` — do NOT put the grid class on `<Container>` itself, as Canopy overrides it with its own styles.
- Field names passed to `MetadataField` must match the `label` values in your IIIF manifests exactly (case-insensitive).

```mdx
<Container variant="wide">
  <div className="canopy-work--hero">
    <div className="canopy-work--viewer">
      <Viewer iiifContent={props.manifest.id} />
    </div>
    <div className="canopy-work--header">
      <Label manifest={props.manifest} as="h1" />
      <div className="canopy-work--byline">
        <MetadataField manifest={props.manifest} field="Main Location" display="inline" />
        <MetadataField manifest={props.manifest} field="Completed" display="inline" prefix="Completed: " />
      </div>
      <Summary manifest={props.manifest} as="p" className="canopy-work--summary" />
    </div>
  </div>
</Container>

<Container className="canopy-work--context">
  <MetadataField manifest={props.manifest} field="Historical Context" label="Historical Context" />
</Container>

<Container className="canopy-work--footer-grid">
  <div>
    <MetadataField manifest={props.manifest} field="Key People" label="Key People" />
    <References />
  </div>
  <div>
    <RequiredStatement manifest={props.manifest} />
  </div>
</Container>

<Container>
  <h2>Related Items</h2>
  <RelatedItems iiifContent={props.manifest.id} top={3} />
</Container>
```

---

### 4. `app/styles/custom.css` — layout styles

Added CSS classes for each section. Paste these into the target site's `custom.css`:

```css
/* Hero: viewer left, text right */
.canopy-work--hero {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  align-items: start;
  padding-top: 1.5rem;
  padding-bottom: 0;
}

.canopy-work--viewer {
  min-width: 0;
}

.canopy-work--header {
  min-width: 0;
}

.canopy-work--header h1 {
  margin-bottom: 0.25rem;
}

/* Location • Completed row */
.canopy-work--byline {
  display: flex;
  gap: 1.5rem;
  font-size: 0.9rem;
  color: var(--color-gray-500, #6b7280);
  margin-bottom: 0.75rem;
  flex-wrap: wrap;
}

/* Summary */
p.canopy-work--summary {
  font-size: 1rem;
  line-height: 1.6;
  margin-top: 0.5rem;
}

/* Historical Context */
.canopy-work--context {
  padding-top: 2rem;
  padding-bottom: 1rem;
}

.canopy-work--context .canopy-metadata-field h3 {
  font-size: 1.2rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.canopy-work--context .canopy-metadata-field p {
  font-size: 1rem;
  line-height: 1.7;
  max-width: 80ch;
}

/* People + Attribution: two columns */
.canopy-work--footer-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 2rem;
  padding-top: 1.5rem;
  padding-bottom: 1.5rem;
}

.canopy-work--footer-grid .canopy-metadata-field h3 {
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.canopy-work--footer-grid .canopy-metadata-field ul {
  list-style: disc;
  padding-left: 1.25rem;
  font-size: 0.95rem;
  line-height: 1.7;
}

@media (max-width: 768px) {
  .canopy-work--hero {
    grid-template-columns: 1fr;
  }
  .canopy-work--footer-grid {
    grid-template-columns: 1fr;
  }
}
```

---

## How to replicate on another Canopy site

1. **Copy** `app/components/MetadataField.tsx` into the target site's `app/components/`
2. **Add one line** to their `app/components/mdx.tsx`:
   ```ts
   MetadataField: './MetadataField.tsx',
   ```
3. **Paste the CSS** above into their `app/styles/custom.css`
4. **Replace** their `content/works/_layout.mdx` with the template above
5. **Update the field names** in `_layout.mdx` to match the metadata labels in their IIIF manifests

   For example, if their manifests use `"Location"` instead of `"Main Location"`:
   ```mdx
   <MetadataField manifest={props.manifest} field="Location" display="inline" />
   ```

To find the correct field names, open one of their manifest JSON files and look at the `metadata` array — each entry has a `label` and a `value`.
