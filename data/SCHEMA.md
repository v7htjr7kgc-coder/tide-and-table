# Tide & Table data schema (`deals.json`, schema_version 1)

Single source of truth: `/workspace/deals/data/deals.json`.
`src/build.py` validates it, builds the xlsx/html/pdf, and copies it to `site/data/deals.json` (that copy is what the website loads).

```jsonc
{
  "meta": {
    "schema_version": 1,
    "title": "South OC restaurant deals",
    "updated": "YYYY-MM-DD",              // shown in the site footer; bump when data changes
    "home": { "label": "San Clemente, CA",
              "lat": 33.4270, "lng": -117.6120, "coord_source": "..." }, // site default origin; xlsx/pdf distances are measured from here
    "coverage_radius_mi": 15,             // xlsx/pdf include only restaurants within this radius of home
    "schema_doc": "SCHEMA.md",
    "deal_type_vocabulary": [...],        // allowed values for deals[].deal_types
    "food_type_vocabulary": [...],        // current food_type values (the site builds its chips from the data)
    "notes": "..."
  },
  "restaurants": [
    {
      "id": "the-cellar-san-clemente",    // REQUIRED, unique, lowercase slug of name + city
      "name": "The Cellar",               // REQUIRED
      "city": "San Clemente",             // REQUIRED (display city/neighborhood)
      "address": "156 Avenida Del Mar, San Clemente, CA 92672",   // REQUIRED, full street address
      "lat": 33.4265206, "lng": -117.614671,   // REQUIRED (decimal degrees); run src/geocode_missing.py to fill
      "coord_source": "OpenStreetMap Nominatim geocode",          // where the coordinates came from
      "website": "https://thecellarsite.com/",                     // official site or "" if unknown
      "menu_url": "https://thecellarsite.com/menu",                // the restaurant's own menu page/PDF; "" if none found. Never guess.
      "food_type": "Wine Bar",            // REQUIRED, broad category used for filters (reuse an existing value when possible)
      "cuisine": "Wine bar / small plates",  // free-text detail
      "deals": [                          // REQUIRED, at least one
        {
          "id": "the-cellar-san-clemente-1",  // unique; restaurant id + "-n"
          "days": ["Mon","Tue","Wed","Thu","Fri"],   // REQUIRED, subset of Mon Tue Wed Thu Fri Sat Sun
          "time": "4-6 PM",               // free text, "" if all day/unknown
          "description": "Happy hour: ...",      // REQUIRED, the deal as sourced (no invented prices)
          "headline": "$3 Baja fish tacos",   // OPTIONAL: the short big-type line on the site. If omitted, the site
                                          // takes the text after "Label:" up to the first comma/semicolon
          "price": "$5 · $8",             // short price summary shown as a chip (may be "")
          "deal_types": ["Happy Hour"],   // from meta.deal_type_vocabulary
          "source_url": "https://...",    // REQUIRED, where the deal was found
          "source_note": "Restaurant site", // provenance/confidence note
          "verify": false                 // true = third-party/older source or conflicting info -> "Verify" badge
        }
      ]
    }
  ]
}
```

Rules: only add deals you found at `source_url`; set `verify: true` when the source is third-party, old, or conflicts; leave `menu_url` blank rather than guessing; one restaurant record per physical location (chains get one record per location, ids must differ).
