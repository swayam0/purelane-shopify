# Build notes — Purelane homepage

## What I'd flag about the original file
- Every product image was an inline SVG/base64 asset baked directly into the HTML — none of this survives contact with real Shopify data, since product imagery has to come from `product.featured_image`.
- Prices, compare-at prices, and discount percentages were hardcoded strings (₹499 ~~₹897~~ 33% off) rather than computed values — a merchant changing a price in Shopify admin would have no effect on the page.
- The review marquee duplicated its own 4 reviews to fake an infinite loop, and the CSS `animation: marq` had no `prefers-reduced-motion` guard — a real accessibility gap, since the JS-level reduced-motion check in the file wasn't wired to that particular animation.
- Combos and bundles had no underlying data model at all — a merchant adding a 5th combo would need a developer to hand-write another chunk of near-identical HTML.
- Global CSS variables and scroll-scene JS lived at the page level rather than being scoped to sections, which risks collisions if a section is duplicated or reordered in the theme editor.

## What I changed and why

### Hero
Rebuilt as a schema-driven section with editable heading, subheading, two CTA buttons, a real image picker, and repeatable "badge" blocks — a merchant can now change all copy and swap the product photo without touching code.

### Shop grid
Built a reusable `card-purelane-product` snippet driven entirely by a real Shopify collection. Sold-out state, sale badge, and discount percentage are all computed from `product.available` / `product.compare_at_price`, not hardcoded. This snippet is reused (with modification) by combos and bundles per the "several sections render similar cards" requirement.

### Combos
Since a "combo" isn't a native Shopify object, I created a **Combo metaobject** (title, flag, description, linked products, bundle price, compare price) so merchants can add/edit combos entirely from the admin, with real product data and computed savings.

### Bundles
Same approach with a **Bundle tier metaobject**. The harder problem here was checkout — Shopify has no native "add several distinct products as one bundle" flow, so I built a small custom element (`<purelane-bundle-add>`) that calls the cart AJAX endpoint (`/cart/add.js`) with all the bundle's variant IDs at once. Confirmed working end-to-end (adds real products, redirects to cart).

### Reviews rail
Rebuilt as merchant-editable review blocks (name, star rating, text) rather than fabricated review-app data. Fixed the missing `prefers-reduced-motion` handling — the marquee now pauses on hover and falls back to a static wrapped list for reduced-motion users.

## What I'd do with more time
1. **Real Reviews App Integration:** Wire the review-rating display on product cards to a real reviews app's metafield (`product.metafields.reviews.rating`) rather than omitting it — I left this out entirely since no reviews app is installed on the dev store, and I didn't want to fabricate ratings.
2. **Pixel-Perfect Adjustments:** Tighten pixel-accuracy against the original file's exact spacing/type scale — I matched structure and behavior closely but didn't do a full side-by-side measurement pass against the source CSS.
3. **Native Bundle Support:** Add proper Shopify "bundle" product support (native SellingPlan/bundle app) for a more robust checkout path than the custom multi-add-to-cart element, which works but sits outside Shopify's standard purchase flow.
4. **Scope Scroll-Scene CSS:** Scope the global scroll-scene CSS variables (from the original file, not yet ported) into section-level custom properties before building the hero's parallax/scene background, which I skipped for this pass in favor of getting all 5 required sections functional first.
