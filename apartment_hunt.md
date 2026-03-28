# Apartment Hunt - Summer 2026

## Project Overview

Static HTML site for tracking 3BR apartment listings across NYC, deployed via Netlify from GitHub.

**Repo:** https://github.com/canarybrad/apartment-hunt
**Files:** `index.html` (single-page site), `apartment_hunt.md` (this file)

---

## Search Parameters

| Parameter | Value |
|---|---|
| Bedrooms | 3 |
| Bathrooms | 2 preferred |
| Max rent | $7,000/month |
| Pet-friendly | Preferred |
| Move-in | Before July 2026 |
| School 1 | Stuyvesant HS — 345 Chambers St, Manhattan (Tribeca) |
| School 2 | Frank Sinatra School of the Arts — 35-12 35th Ave, Astoria, Queens |
| Commute threshold | Both schools under 60 minutes transit |

---

## How to Run a Listing Refresh

When the user says "refresh the apartment search", "update the listings", or similar:

### Step 1: Search for new listings

Use Playwright (NOT plain web fetch — rental sites block bots) to browse these sources. For each, capture: address, unit, price, net effective, BR/BA, sqft, key features, and the **specific listing URL** (NOT a generic search page).

**Primary sources (browse via Playwright):**
- StreetEasy: `https://streeteasy.com/3-bedroom-apartments-for-rent/{neighborhood}` for each neighborhood below
- Nooklyn: `https://nooklyn.com/rentals?bedrooms=3&maxPrice=7000&sortBy=recently_created`

**Secondary sources (use search agents):**
- RentHop: `https://www.renthop.com/apartments-for-rent/{neighborhood}-new-york-ny/3-bedroom`
- Craigslist Brooklyn: `https://newyork.craigslist.org/search/brk/apa?max_price=7000&min_bedrooms=3&pets_cat=1&pets_dog=1`
- Craigslist Queens: `https://newyork.craigslist.org/search/que/apa?max_price=7000&min_bedrooms=3`
- Zumper: `https://www.zumper.com/apartments-for-rent/new-york-ny/{neighborhood}/3-beds`

**Direct building sites (check for 3BR availability):**
- Fabric Astoria: https://fabricastoria.com/ — (646) 300-7245
- AtlanticBK: https://atlanticbk.com/ — (718) 400-1057
- The Arcadian: https://www.thearcadianbk.com/ — (929) 925-0975
- The Botanica: https://www.botanicabk.com/
- 207 Madison: https://207madison.com/
- Metropolitan Property Group (no-fee Astoria): 212-353-2003

### Step 2: Target neighborhoods

Search these neighborhoods, listed by commute quality (best first):

| Neighborhood | → Stuyvesant | → Sinatra | Worst | StreetEasy slug |
|---|---|---|---|---|
| Chinatown / Two Bridges | 13 min | 39-44 min | 44 min | `chinatown` |
| Williamsburg / E. Williamsburg | 27 min | 42-45 min | 45 min | `williamsburg` |
| LES | 27-30 min | 44-46 min | 46 min | `les` |
| East Village | ~25 min | ~48 min | 48 min | `east-village` |
| LIC | ~40 min | ~20 min | 40 min | `long-island-city` |
| Downtown Brooklyn | 25 min | ~53 min | 53 min | `downtown-brooklyn` |
| Astoria | 49-50 min | 5-10 min | 50 min | `astoria` |
| Sunnyside / Woodside | 49-53 min | 15-20 min | 53 min | `sunnyside-queens` / `woodside` |
| Fort Greene | 27-33 min | 54-55 min | 55 min | `fort-greene` |
| Greenpoint | 45 min | 37-39 min | 45 min | `greenpoint` |
| Prospect Heights | ~35 min | ~55 min | 55 min | `prospect-heights` |
| Clinton Hill | ~32 min | ~56 min | 56 min | `clinton-hill` |
| Park Slope / Gowanus | ~30 min | ~55 min | 55 min | `park-slope` / `gowanus` |
| Cobble Hill / Carroll Gardens | ~25 min | ~55 min | 55 min | `cobble-hill` / `carroll-gardens` |
| Bushwick | ~38 min | ~58 min | 58 min | `bushwick` |
| Bed-Stuy | 38 min | 1h 7min | 67 min | `bedford-stuyvesant` |
| Crown Heights | ~42 min | ~1h 5min | 65 min | `crown-heights` |

### Step 3: Verify listing links

**CRITICAL:** Every listing must have a specific URL that goes to the actual listing page. Acceptable URL patterns:
- `streeteasy.com/building/{building-slug}/{unit}`
- `nooklyn.com/listings/{listing-slug}`
- `renthop.com/listings/{address}/{unit}/{id}`
- `newyork.craigslist.org/brk/apa/d/{title}/{id}.html`
- Direct building site URLs (fabricastoria.com, atlanticbk.com, etc.)

**NOT acceptable:** Generic search pages like `streeteasy.com/3-bedroom-apartments-for-rent/williamsburg`. If you can't find a specific listing URL, don't include the listing.

### Step 4: Update index.html

The HTML has this structure:
- **Commute Summary table** — neighborhood-level commute times (don't change unless schools change)
- **Rankings section** — "All Listings Ranked by Commute Balance" table sorted by worst-case commute, and "Best Value" table sorted by price
- **Neighborhood sections** — collapsible `<div class="neighborhood">` blocks, each containing a `<table>` of listings
- **Search Links** — links to search pages on each platform
- **JavaScript** at bottom that:
  - Toggles neighborhood collapse/expand
  - Auto-adds Google Maps transit direction pills (S/F) to every listing row
  - Linkifies commute times in the ranking table to Google Maps

**For each listing row in a neighborhood table:**
```html
<tr class="clickable-row" onclick="window.open('LISTING_URL','_blank')" data-price="PRICE_INT" data-pets="yes|no">
  <td><a href="LISTING_URL" target="_blank" onclick="event.stopPropagation()">ADDRESS</a></td>
  <td>UNIT</td>
  <td class="price">$X,XXX</td>
  <td class="net">$X,XXX or &mdash;</td>
  <td>BR/BA</td>
  <td>SQFT or &mdash;</td>
  <td>NOTES with optional <span class="tag tag-new">NEW DEV</span> <span class="tag tag-nofee">NO FEE</span> <span class="tag tag-stabilized">RENT STABILIZED</span> <span class="tag tag-open">OPEN DATE</span></td>
</tr>
```

**For ranking table rows (commute balance):**
```html
<tr data-addr="ADDRESS+ENCODED+WITH+BOROUGH"><td>RANK</td><td><a href="LISTING_URL" target="_blank">ADDRESS</a></td><td>NEIGHBORHOOD</td><td class="price">PRICE</td><td>XX min</td><td>XX min</td><td>XX min</td></tr>
```
The JS auto-linkifies cells[4] and cells[5] to Google Maps.

### Step 5: Commit and push

```bash
cd /Users/bradandrews/Documents/selfprocess/apartment_hunt
git add -A
git commit -m "Refresh listings - $(date +%Y-%m-%d)"
git push origin main
```

Netlify auto-deploys from the repo.

---

## Tag reference

| Class | Color | Use for |
|---|---|---|
| `tag-new` | Green | New development / construction |
| `tag-nofee` | Blue | No broker fee |
| `tag-stabilized` | Orange | Rent stabilized |
| `tag-open` | Custom | Open house date/time |
| `tag-penthouse` | Pink | Penthouse unit |
| `tag-luxury` | Orange | Luxury amenities |

Pet status in data attribute: `data-pets="yes"` or `data-pets="no"`
Pet display: `<span class="pet-yes">&#9989; Dogs+cats</span>` or `<span class="pet-yes">ALL PETS</span>`

---

## Google Maps link format

For transit direction links:
```
https://www.google.com/maps/dir/ORIGIN_ADDRESS/DESTINATION_ADDRESS/data=!4m2!4m1!3e3
```
The `!3e3` forces transit mode. Addresses use `+` for spaces and `,` for city/state separation.

Stuyvesant destination: `345+Chambers+Street,+New+York,+NY`
Sinatra destination: `35-12+35th+Avenue,+Astoria,+NY`

---

## Borough mapping for JS auto-links

The JS at the bottom uses a `boroughs` map to determine the city suffix for Google Maps links. If adding a new neighborhood in Manhattan or Queens, add its `id` to the map:

```javascript
const boroughs = {
  'n-astoria': 'Queens,+NY',
  'n-chinatown': 'New+York,+NY',
  'n-les': 'New+York,+NY',
  'n-sunnyside': 'Queens,+NY',
  'n-eastvillage': 'New+York,+NY',
};
// Default: 'Brooklyn,+NY'
```

---

## Current listing counts by neighborhood

| Neighborhood | Count | Price range |
|---|---|---|
| Astoria / LIC | 16 | $3,100-$6,700 |
| Williamsburg / E. Williamsburg | 18 | $3,995-$5,775 |
| Fort Greene | 9 | $3,495-$5,250 |
| Bed-Stuy / Prospect Hts / Crown Hts | 22 | $2,450-$6,790 |
| Bushwick | 9 | $3,495-$5,450 |
| Greenpoint | 6 | $4,599-$6,800 |
| Sunnyside / Woodside | 7 | $3,595-$5,500 |
| East Village | 3 | $6,200-$6,500 |
| Park Slope / Gowanus / Carroll Gardens | 5 | $5,500-$6,692 |
| Prospect Heights | 3 | $5,595-$7,000 |
| Chinatown | 1 | $6,695 |
| LES | 4 | $5,799-$6,995 |
| Clinton Hill | 2 | $4,750-$4,775 |
| Downtown Brooklyn | 1 | $6,250 |
| Other | 2 | $3,350-$3,450 |

**Total: ~115+ listings across 19 neighborhoods**

---

## Key contacts

| Contact | Phone | Notes |
|---|---|---|
| Fabric Astoria | (646) 300-7245 | fabricastoria.com |
| AtlanticBK | (718) 400-1057 | atlanticbk.com |
| The Arcadian | (929) 925-0975 | thearcadianbk.com |
| Metropolitan Property Group | 212-353-2003 | No-fee Astoria |
| 1614 Prospect Pl (Craigslist) | 917-817-9529 | Ask for Adam |
