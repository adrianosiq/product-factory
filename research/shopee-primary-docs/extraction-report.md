# Phase 02.3 — Primary Evidence Extraction Report

date: 2026-09-03
branch: research/shopee-primary-evidence-extraction
corpus: `docs/marketplaces/shopee/open-platform/` — 35 official Shopee Open
Platform PDFs (`SPD-001`…`SPD-035`).

The pre-artifact checkpoint ("PRIMARY DOCUMENTATION REQUIRED") is **superseded**.
No secondary source was used to upgrade a claim. Text was extracted from each PDF
(stdlib PDF text decoder incl. `/ToUnicode` CMaps; no OCR needed — the pages
carry real text).

---

## Corpus & extraction coverage

| Group | Artifacts | Read in full | Notes |
|---|---|---|---|
| Auth / onboarding (A) | SPD-001…SPD-009, SPD-031, SPD-034 | SPD-001 (auth), SPD-002/003/004/005/006/007/008/009 (BR journey), SPD-031 (sensitive data), SPD-034 (references) | SPD-035 (BR SPI) read — **SPI-App-specific, not general Product API** |
| Item lifecycle (B) | SPD-010…SPD-014 | SPD-010 (`add_item`), SPD-011 (`get_item_base_info`) in full; SPD-012/013/014 cross-referenced (same common params, error family) | |
| Variations / models (C) | SPD-015…SPD-020 | SPD-015 (`init_tier_variation`), SPD-017 (`add_model`), SPD-020 (`get_model_list`) in full; SPD-016/018/019 cross-referenced | |
| Taxonomy / attributes (D) | SPD-021…SPD-024 | SPD-021 (`get_category`), SPD-022 (`category_recommend`), SPD-023 (`get_attribute_tree`) in full; SPD-024 (`get_recommend_attribute`) skimmed | |
| Brand / identifiers (E) | SPD-025, SPD-026 | SPD-025 (`get_brand_list`), SPD-026 (`register_brand`) in full; GTIN rules extracted from SPD-010/015/017/020/029/011 | |
| Commercial (F) | SPD-027…SPD-029 | SPD-027 (`update_price`), SPD-028 (`update_stock`), SPD-029 (`get_item_limit`) in full | |
| Operational / support (G) | SPD-030, SPD-032, SPD-033 | SPD-030 (webhooks), SPD-032 (FAQ), SPD-033 (ticket best-practices) skimmed — **no implementation-critical listing claims** |

~22 / 35 read in full; the rest cross-referenced. Every API-reference PDF shares
an identical "Common Request Parameters" block and a large shared error family,
so cross-reference is reliable.

---

## Primary facts locked (by domain)

Facts below are `PRIMARY_VERIFIED` unless marked. `[DYNAMIC]` = value must be
fetched per shop/category at runtime; the *sample* value in the doc is not a
constant.

### 1–2. Provenance

All 35 are official Shopee Open Platform pages (`/Title` = "Documentation" /
"Developer Guide - Shopee Open Platform", every internal link is
`open.shopee.com/*` or the doc-locator `.../documents/v2/v2.<svc>.<method>?module=89`).
All **v2**. Captured 2026-09-01 / 2026-09-03; page "Last Updated" 2026-06…2026-08.
`SPD-005…SPD-008` cleared by maintainer visual review (`evidence-registry.md`).

### 3. Brazil eligibility / onboarding

- **Available**: dedicated Brazil hosts (`openplatform.shopee.com.br/api/v2/`,
  `open.shopee.com.br/auth`), a Brazil developer journey ("BRASIL | Jornada do
  Desenvolvedor"), BR-specific `add_item` fields (NCM, CFOP, CEST, CSOSN, PIS,
  COFINS, ICMS, `export_cfop`, model-level `gtin_code` "BR local seller",
  two-decimal prices for BR).
- **Developer profile is approval-gated**: a login alone cannot create Apps;
  you apply for a developer profile and are approved by **Shopee internal
  review** (SPD-003, SPD-004).
- **BR developer types**: only **Registered Business Seller** and **Third-party
  Partner (ISV)** (SPD-004).
- **App category = permission set**, chosen at creation, **immutable**, "no
  manual or extra permissions" (SPD-005). Common: Registered Business Seller →
  `Seller In-House System`; ISV → `ERP System`. Both categories appear in the
  "APP types that can call this API" list of `add_item`, `update_item`,
  `update_stock`, `update_price`, `init_tier_variation`, `add_model`,
  `get_category`, `get_attribute_tree`, `get_brand_list`, `get_item_limit` — and
  are **not** whitelist-gated.
- **Production requires Go-Live review** by Shopee (Product Brief, Redirect URL
  domains, IP whitelist, IT-assets declaration) (SPD-009). **Sandbox** is
  independent and available first (SPD-008).
- **Whitelist-only** applies to `Swarm ERP`, `Brand Membership`, `Auto Parts
  Installation` SPI apps (SPD-035) — **not** the listing APIs.
- **`PRIMARY_NOT_FOUND`**: exact developer-profile approval criteria (link
  `open.shopee.com/developer-guide/12` not supplied); whether `register_brand` is
  supported in BR.

### 4. Authentication (SPD-001)

- **Auth flow** (3 steps): build authorization link → seller authorizes shop(s)
  → exchange `code` for tokens, then refresh.
- **Authorization link** = fixed URL + params. Fixed URLs: Production Global
  `https://open.shopee.com/auth`, Mainland China `https://open.shopee.cn/auth`,
  **Brazil `https://open.shopee.com.br/auth`**; Sandbox variants under
  `open.sandbox.test-stable.shopee.*`. Params: `partner_id` (req), `auth_type`
  (req; enum **`seller`** / **`supplier`** / **`user`**), `redirect_uri` (req;
  domain must match the Console-configured Redirect URL Domain — Test + Live),
  `response_type` (req; fixed `"code"`), `state` (opt; CSRF).
- **Account types**: Shop account (1 shop), Main account (many merchants/shops),
  Sub-account (cannot authorize).
- Redirect returns `code` + `shop_id` (shop account) or `code` +
  `main_account_id` (main account).
- **Lifetimes**: `code` — single-use, **10 min**; `timestamp` (for sign) —
  **5 min**; `access_token` — **4 h**, multi-use (`expire_in` seconds);
  `refresh_token` — **30 d**, single-use per `shop_id`/`merchant_id`; after
  refresh the old `access_token` stays valid **+5 min**. **Authorization
  validity ≤ 360 days** (presets 7/30/90/180/360, seller-customisable) —
  `PRIMARY_CONFLICT`: SPD-006 says "no máximo 365 dias".
- **Token exchange**: `POST /api/v2/auth/token/get` (prod
  `partner.shopeemobile.com`; sandbox `openplatform.sandbox.test-stable.shopee.sg`).
  Query: `sign`, `partner_id`, `timestamp`. Body: `code`, `partner_id`,
  `shop_id` **or** `main_account_id`. Returns `access_token`, `refresh_token`,
  `expire_in`, `merchant_id_list`, `shop_id_list`, `supplier_id_list`,
  `user_id_list`.
- **Refresh**: `POST /api/v2/auth/access_token/get`. Body: `refresh_token`,
  `partner_id`, `shop_id`/`merchant_id`. Returns new `refresh_token`,
  `access_token`, `expire_in`.
- **Sign base string** (path without host, concatenated in order):
  - Shop APIs: `partner_id + api_path + timestamp + access_token + shop_id`
  - Merchant APIs: `partner_id + api_path + timestamp + access_token + merchant_id`
  - Public APIs (`auth_partner`, `token/get`, `access_token/get`):
    `partner_id + api_path + timestamp`
  - HMAC-SHA256, key = **partner key**, output hex lowercase. `partner_id` +
    `timestamp` + `sign` in the **query**; other params in the JSON body.
- **CB SIP**: a CB SIP primary shop's authorization propagates to all linked SIP
  shops (with limited API permissions).
- **Cancel authorization**: `.../cancel_auth` URLs, or Seller Center → "Platform
  Partner" (local) / "Open Platform Management" (CNSC/KRSC).

### 5. Item contract — `add_item` (SPD-010)

- `POST /api/v2/product/add_item` → returns `item_id` (int64).
- **Hosts**: Global `partner.shopeemobile.com`, CN `openplatform.shopee.cn`,
  **BR `openplatform.shopee.com.br`**, Sandbox Global
  `openplatform.sandbox.test-stable.shopee.sg`, Sandbox CN.
- **Common params** (all product APIs): `partner_id` (int), `timestamp`
  (5 min), `access_token` (string, 4 h), `shop_id` (int), `sign` (HMAC-SHA256).
- **Required request fields** (`Required = True`): `original_price` (float),
  `description` (string; when `description_type = normal`), `weight` (float, KG),
  `item_name` (string), `category_id` (int), `image.image_id_list` (string[]),
  `logistic_info[].{enabled, logistic_id}`, `brand.{brand_id, original_brand_name}`
  ("No Brand if not brand"), `pre_order.is_pre_order` (bool). `attribute_list`
  is `False` at field level but "Must contain all mandatory attribute".
  `dimension` object is `False`; its `package_height/length/width` (cm) are
  `True` when `dimension` is supplied.
- **Optional**: `item_status` (`UNLIST` | `NORMAL`), `image_ratio` (`"1:1"`
  default | `"3:4"` **whitelisted sellers only**), `days_to_ship` (int; range
  from `get_item_limit.days_to_ship_limit` — prose also names `get_dts_limit`),
  `item_sku`, `condition` (`NEW` | `USED`), `wholesale[]` (`min_count`,
  `max_count`, `unit_price` — all model prices must be equal when wholesale set),
  `video_upload_id` (string[], one), `gtin_code` (see §11), `seller_stock[]`
  (`location_id` opt, `stock`), `ds_cat_rcmd_id` (links `category_recommend`),
  `promotion_images` (one; ratio **must be 3:4**), `compatibility_info.vehicle_info_list`
  (auto-parts: brand/model/year/version id), `scheduled_publish_time` (UNLIST
  items only; now+1h … now+90d, minute precision), `authorised_brand_id`,
  `size_chart_info` (`size_chart` image — CB + local; `size_chart_id` template —
  local only), `certification_info` (PH), `description_info.extended_description`
  (`description_type = extended` — **whitelist sellers only**),
  `complaint_policy` (local PL), `item_dangerous` (ID/MY local),
  `group_item_info` (kit/pack), `tax_info` + `export_cfop` (BR — see §17).
- **Response**: `item_id`, plus echo (`image_url_list`, `price_info`
  {current/original}, `video_info` {video_url, thumbnail_url, duration},
  `warning`, `request_id`).
- **Error family** (`PRIMARY_VERIFIED`, publication-relevant subset):
  `error_title_exceeds_max_length` / `error_item_name_is_too_short` /
  `error_title_character_forbidden` / `error_name_length_limit` /
  `error_item_name_empty`; `error_desc_length_min_limit` /
  `error_desc_hash_tag_over_limit` ("hash tags > 18"); `error_image_num_min`;
  `error_price_exceed_min_limit` / `error_price_exceed_max_limit` /
  `error_price_out_of_range`; `error_param_dts_exceeds_max_limit` /
  `error_category_dts` / `error_invalid_days_to_ship` /
  `error_param_category_not_support_pre_order`; `error_forbidden_category` /
  `error_category_is_block` ("Category is restricted") / `error_invalid_category`
  ("L1 and L2 do not match"); `error_brand_forbidden` / `error_invalid_brand`
  ("Brand ID value should be '0'" / "Brand name required" / "Brand ID required")
  / `error_less_required_brand`; `error_less_required_attribute` /
  `error_invalid_category_attribute` / `error_invalid_attribute_value`;
  `error_value_name_required` / `error_value_id_must_equal_zero`;
  `error_param_validate` ("This is not a valid GTIN") / `product.error_busi`
  ("The GTIN code is mandatory"); `error_reach_shop_item_limit` ("Item published
  item count reaches limit"); `error_repeated_mtsku` ("A similar product has
  already been uploaded"); `error_unlist_item_fail` (some flows require UNLIST
  first); multi-warehouse errors (see §18); `error_auth` ("Your shop can not use
  model level dts"); `error_busi_cannot_edit_vsku`.
- **API permissions**: ERP System, Seller In House System, Product Management,
  Swam ERP.

### 6. Item read — `get_item_base_info` (SPD-011)

- `GET /api/v2/product/get_item_base_info`. Request: `item_id_list` int64[]
  (**limit [0, 50]**), `need_tax_info` (bool), `need_complaint_policy` (bool).
- **`item_status` enum** (definitive): `NORMAL`, `BANNED`, `UNLIST`,
  `SELLER_DELETE`, `SHOPEE_DELETE`, `REVIEWING`.
- Flags: `has_model` (bool), `deboost` (bool — "search ranking is lowered"),
  `has_promotion` (bool), `is_fulfillment_by_shopee` (bool — true when the item
  "only has a **default model**" and is FBS), `tag.kit` (bool).
- `price_info` "**not returned if the item has models** — get per-model price via
  `get_model_list`". Fields: `currency` (3-letter), `original_price`,
  `current_price` (= promo price when `has_promotion`), `inflated_price_*` (CB),
  `sip_item_price`, `local_price`, `local_promotion_price`.
- `stock_info_v2`: `summary_info` {`total_reserved_stock`, `total_available_stock`};
  `seller_stock[]` {`location_id`, `stock`, `if_saleable`}; `shopee_stock[]`
  {`location_id`, `stock`}; `advance_stock` {`sellable_advance_stock`,
  `in_transit_advance_stock`} (PH/VN/ID/MY only).
- `ssp_id` = "Shopee's unique identifier for **Shopee Standard Product**" — a
  catalogue-like node (relates to gap G6).
- `gtin_code` = "gtin code for BR region, returned only when item has a default
  model; `"00"` = Item without GTIN".
- **API permissions**: broad (13 app types incl. Order Management, Marketing,
  Ads, Livestream, Shopee Video).

### 7. `update_item` / `unlist_item` / `delete_item` (SPD-012/013/014, cross-ref)

- `unlist_item` = list/unlist toggle (batch); `delete_item` by `item_id`.
- `update_item` shares `add_item`'s fields + error family; edits gated by
  `item_status` (`error_item_uneditable` "item status can not support editing"),
  promotion locks, holiday mode, penalty (`error_seller_under_penalty`).
- **`PRIMARY_NOT_FOUND`**: full `update_item` field-by-field diff vs `add_item`
  (not read in full — same page family, extract in Phase 02.4 if needed).

### 8. Variation / model contract (SPD-015 / SPD-017 / SPD-020)

- `init_tier_variation` (`POST`): `item_id` + `standardise_tier_variation[]`
  (the **`tier_variation` structure was deprecated 2025-09-12** — use
  `standardise_tier_variation`) + `model[]` (**"model number at most 50"**).
  Per model: `tier_index` int32[], `original_price`, `model_sku` (**≤ 100 chars**),
  `seller_stock[]` {`location_id` opt (from `v2.shop.get_warehouse_detail`),
  `stock`}, `gtin_code`, model-level `weight` / `dimension` / `pre_order`
  (default to item values). "Wait ≥ 5 s after item creation before creating
  variants."
- **Max tiers = 2** ("color + size … maximum supported"; `error_param` "The
  level of tier-variation over 2").
- Model-count rules (`PRIMARY_PARTIAL`/`PRIMARY_CONFLICT`): model list per call
  ≤ 50; `error_model_count_over_limit` "**under 20 (50 for TW)**"; combinations
  "**under 50**"; `error_tier_opt_too_many` "option > 20".
- `add_model` (`POST`): `item_id` + `model_list[]` (same per-model shape). Error
  "Item without tier_variation. Please use `init_tier_variation` api to upgrade
  the structure."
- `get_model_list` (`GET`): `item_id` → `tier_variation[]` + `model[]`
  {`model_id` int64, `tier_index`, `model_sku`, **`model_status`** ∈
  `MODEL_NORMAL` | `MODEL_UNAVAILABLE` ("only whitelisted users can use"),
  `price_info`, `pre_order`, `stock_info_v2`, `gtin_code` ("only TW seller and BR
  local seller"), `weight`, `dimension`, `is_fulfillment_by_shopee`,
  `standardise_tier_variation[]`}.
- **No-variation / default model**: addressed as **`model_id = 0`**
  (`update_stock` / `update_price`: "0 for no model item"). Shopee keeps an
  internal "default model" for such items.
- Promotion locks: `error_cannt_init_tier_in_promotion`,
  `error_cannt_be_no_variation_in_promotion`,
  `error_cannt_change_tier_variation_in_promotion`,
  `error_cannt_delete_option_in_promotion`, `error_cannt_edit_price_in_promotion`.
- `error_cnsc_shop_block_update_tier_variation`, `error_auth_product_is_pff`
  (FBS item/model not editable).

### 9. Category contract (SPD-021 / SPD-022)

- `get_category` (`GET`): request `language` (BR: `pt-br` / `en`), **no
  category_id** — returns the whole tree. Response `category_list[]`
  {`category_id`, `parent_category_id`, `original_category_name`,
  `display_category_name`, **`has_children` bool**}.
- **Leaf = `has_children: false`**. No `listing_allowed`-style flag. Restricted
  categories surface at `add_item` (`error_category_is_block` "Category is
  restricted", `error_forbidden_category`). `error_invalid_category` "Category
  IDs for L1 and L2 do not match" (implies leaf listing).
- `category_recommend` (`GET`): `item_name` (req) + `product_cover_image` (opt,
  image id) → `response.category_id` int[] (ranked). Also yields `ds_cat_rcmd_id`
  for `add_item`.
- `PRIMARY_PARTIAL`: an explicit "you must list under a leaf" sentence is not on
  the `get_category` page; it is strongly implied by `has_children` + the L1/L2
  error + per-leaf brand/attribute lookups.

### 10. Attribute contract (SPD-023 / SPD-024)

- `get_attribute_tree` (`GET`, new 2023-07-24): `category_id_list` (**max 20**) +
  `language` (BR: `pt-BR`/`en`). Response `list[]` (one per category):
  `attribute_tree[]` {`attribute_id`, **`mandatory` bool**, `name`,
  `attribute_value_list[]` {`value_id`, `name`, `value_unit`,
  `child_attribute_list[]` (recursive)}, `multi_lang[]`}, `attribute_info`
  {`input_type` int, `input_validation_type` int, `format_type` int,
  `date_format_type` int, `attribute_unit_list[]`, `max_value_count`, `is_oem`,
  `support_search_value` bool}, `category_id`.
- **`input_type`**: `SINGLE_DROP_DOWN=1`, `SINGLE_COMBO_BOX=2`,
  `FREE_TEXT_FILED=3`, `MULTI_DROP_DOWN=4`, `MULTI_COMBO_BOX=5`.
  **`input_validation_type`**: `NO_VALIDATE=0`, `INT=1`, `STRING=2`, `FLOAT=3`,
  `DATE=4`. **`format_type`**: `NORMAL=1`, `QUANTITATIVE_WITH_UNIT=2`.
  **`date_format_type`**: `DD/MM/YYYY=0`, `MM/YYYY=1`.
- `mandatory` is a **static per-category boolean** — no conditional-requiredness
  mechanism in the corpus.
- `support_search_value = true` → call `v2.product.search_attribute_value_list`
  for default values (endpoint **not** in corpus).
- `add_item.attribute_value_list`: `value_id = 0` + `original_value_name`
  mandatory when `input_type` is `TEXT_FILED` / `COMBO_BOX` /
  `MULTIPLE_SELECT_COMBO_BOX` (custom value); `original_value_name` may carry a
  timestamp string for `DATE_TYPE` / `TIMESTAMP_TYPE`.
- `get_recommend_attribute` — separate endpoint (skimmed): recommended, not
  mandatory attributes → `QUALITY`.
- API permissions: ERP System, Seller In House System, Product Management,
  Customized APP, Swam ERP.

### 11. GTIN / identifiers (SPD-010/015/017/020/029/011)

- Field `gtin_code` (string) on `add_item` / `add_model` /
  `init_tier_variation`; **model-level** ("only TW seller and BR local seller"
  per `get_model_list`); item-level value shown for default-model items.
- **"Item without GTIN" ⇒ `gtin_code = "00"`.** No `EMPTY_GTIN_REASON` analogue —
  the mechanism is the literal `"00"`.
- Validation governed by `get_item_limit.gtin_limit.gtin_validation_rule` ∈
  **`Mandatory`** / **`Flexible`** (valid GTIN or `"00"`) / **`Optional`**.
- GS1 note in doc: GTIN 8–14 digits (UPC/EAN/JAN/ISBN); "aids Search and
  Recommendation in Shopee". Errors: `error_param_validate` ("not a valid
  GTIN"), `product.error_busi` ("GTIN code is mandatory").
- BR-only: NCM must be 8 digits or `"00"`; CEST 7 digits or `"00"`; `group_gtin_sscc`
  / `group_grai_gtin_sscc` for pack/box.

### 12. Brand contract (SPD-025 / SPD-026)

- `get_brand_list` (`GET`): "brand data of a **leaf category**". Request:
  `offset` (paginate via `next_offset`), `page_size` (**max 100**),
  `category_id`, `status` (**1 normal / 2 pending**), `language` (BR: `en`/`pt-br`).
  Response: `brand_list[]` {`brand_id`, `original_brand_name`,
  `display_brand_name`}, `has_next_page`, `next_offset`; plus `is_mandatory`
  (bool) + `input_type` (`"DROP_DOWN"`) for the brand attribute.
- `add_item` **always** needs `brand.brand_id` + `brand.original_brand_name`
  (True/True). **"No Brand" ⇒ `brand_id = 0`** (`error_invalid_brand` "Brand ID
  value should be '0'").
- `register_brand` (`POST`, new 2021-09-06): `original_brand_name` (≤ 254),
  `category_list` int[] (L1/L2, max 50), `product_image.image_id_list` (max 10),
  `brand_region` (req), `app_logo_image_id` / `pc_logo_image_id`,
  `brand_website` / `brand_description`, `licenses[]` + `brand_registration_website`
  (mandatory when name hits blacklist). Response: `brand_id`,
  `original_brand_name`. Errors: `unsupport_region_for_register_brand` (market
  doesn't support brand registration — **BR support `PRIMARY_NOT_FOUND`**),
  `error_busi_pending_qc` ("in the inspection process" — brand registration is
  **QC-reviewed**), `error_busi_duplicated`, `error_busi_blacklist`,
  `error_busi_blacklist_need_license`.

### 13. Numeric limits — the `get_item_limit` contract (SPD-029)

`GET /api/v2/product/get_item_limit`. Request: `category_id` (int, **optional**).
"Get item **upload control**." Response `response` object (all `[DYNAMIC]`, the
numbers below are the doc's **sample** values, not constants):

| Field | Sample | Scope |
|---|---|---|
| `item_name_length_limit` {min_limit, max_limit} | 5 / 100 | shop + category |
| `item_description_length_limit` {min, max} | 10 / 2000 | shop + category |
| `extended_description_limit` {text_length_min/max, image_num_min/max, image_width_min, image_height_min, image_aspect_ratio_min/max} | — | shop + category |
| `item_image_count_limit` {min, max} | 1 / 9 | shop + category |
| `price_limit` {min_limit, max_limit} (float) | 5.5 / 10 000 000.0 | shop + category |
| `stock_limit` {min, max} (int) | 5 / 10 000 000 | shop + category |
| `wholesale_price_threshold_percentage` {min, max} | 30 / 100 | shop + category |
| `tier_variation_name_length_limit` {min, max} | 0 / 14 | shop |
| `tier_variation_option_length_limit` {min, max} | 0 / 20 | shop |
| `days_to_ship_limit` {min, max, non_pre_order_days_to_ship} | — | category |
| `weight_limit` {`weight_mandatory` bool} | — | category |
| `dimension_limit` {`dimension_mandatory` bool} | — | category |
| `size_chart_limit` {`size_chart_mandatory`, `support_image_size_chart`, `support_template_size_chart`} | — | category |
| `gtin_limit` {`gtin_validation_rule` string} | Mandatory / Flexible / Optional | category |
| `item_count_limit` {max_limit} | 5000 | **shop** — the total-listing quota |

Update log: 2024-06-27 added `category_id` + `weight_limit`/`dimension_limit`/
`dts_limit`; 2025-01-08 added `gtin_validation_rule`; 2024-10-25 added
`size_chart_limit`; 2022-02-21 added `extended_description_limit`. Error
`error_auth` "Your shop can not use model level dts".

### 14. Image contract

- Upload via `v2.media_space.upload_image` (referenced by `add_item` /
  `category_recommend`; **the media_space page is not in the corpus**).
- `add_item.image.image_id_list` (required); count bounded by
  `get_item_limit.item_image_count_limit`.
- `image_ratio`: `"1:1"` default; `"3:4"` **whitelisted sellers only**.
- `promotion_images`: one image; ratio **must be 3:4**; allowed only when the
  product images' ratio is 3:4.
- `PRIMARY_NOT_FOUND` in this corpus: pixel min/max, file type, byte cap, cover
  rules — those live on the `media_space` / seller-education pages.

### 15. Listing video contract

- `add_item.video_upload_id` (string[]) — "Only accept **one** video_upload_id".
- Returned in reads as `video_info` {`video_url`, `thumbnail_url`, `duration`}.
- Upload endpoint (`v2.media_space.upload_video`) **not in corpus**; duration/
  aspect/size constraints `PRIMARY_NOT_FOUND`.

### 16. Price contract (SPD-027)

- `update_price` (`POST`): `item_id` + `price_list[]` (**length 1–50**)
  {`model_id` (0 = no model item), `original_price` (float)}.
- **BR (+ SG/MY/MX/PL/ES/AR): two decimal places allowed**; other regions
  integer only.
- Response: `failure_list[]` {model_id, failed_reason}, `success_list[]`
  {model_id, original_price}.
- Errors: `error_price_exceed_min/max_limit`, `error_price_out_of_range`,
  `error_busi_price_lower_then_wholesale_price`,
  `error_price_should_be_same_for_wholesales`,
  `error_edit_item_price_for_item_has_model` ("Can't edit item price directly
  while item has models"), `error_cannt_edit_price_in_promotion`,
  `error_seller_under_penalty`, `error_item_uneditable`.
- API permissions: ERP System, Seller In House System, Product Management.

### 17. Stock / warehouse contract (SPD-028) + BR tax fields

- `update_stock` (`POST`): one `item_id` per call; `stock_list[]` (**length
  1–50**) {`model_id` (0 = no model item), `seller_stock[]` {`location_id` opt,
  `stock`}}. **Updates only `seller_stock`** (`normal_stock` offlined
  2022-10-31). Value is the new stock (**absolute**). `location_id` from
  `v2.shop.get_warehouse_detail`; omit when the shop has no warehouse.
- Multi-warehouse: cannot mix stock structures (all with `location_id` or all
  without). `error_busi` "The merchant/shop has multi warehouse, please input
  location id". `error_wms_shop_block_upate_stock` "Warehouse shop can't update
  stock". FBS/B2C: "normal stock must be equal to 0".
- Response: `failure_list[]` / `success_list[]` per model_id.
- Errors: `error_edit_item_stock_for_item_has_model`, promotion locks
  (`error_cannt_edit_stock_in_promotion`), `error_seller_under_penalty`,
  reserved-stock guards ("Total stock must be more than reserved stock").
- **BR `add_item.tax_info`** (`PRIMARY_VERIFIED`, BR-scoped): `ncm` (8 digits or
  `"00"`), `same_state_cfop`, `diff_state_cfop`, `csosn`, `origin` (0–8 codes),
  `cest` (7 digits or `"00"`), `measure_unit` (uppercase, fixed list: AMPOLA…
  VIDRO), `pis`, `cofins`, `icms_cst`, `pis_cofins_cst`, `federal_state_taxes`,
  `operation_type` (1 Retailer / 2 Manufacturer), `ex_tipi`, `fci_num`,
  `recopi_num`, `additional_info`; `export_cfop` (7101 self-produced / 7102
  resale); `group_item_info` (kit/pack). Error `error_param` "all BR tax field
  should be empty or be filled at same time".

### 18. Item lifecycle & operational

- `item_status` transitions: `add_item` creates with `NORMAL` or `UNLIST`;
  `unlist_item` toggles NORMAL⇄UNLIST; `scheduled_publish_time` publishes an
  UNLIST item later; `SELLER_DELETE` / `SHOPEE_DELETE` are removal states;
  `BANNED` = policy removal; `REVIEWING` = under moderation. Full transition
  graph and "does an edit re-trigger review" are `PRIMARY_NOT_FOUND` (not stated
  explicitly).
- App-category permission model (SPD-005, SPD-007): each endpoint lists "APP
  types that can call this API"; calling outside it → **Permission Denied**.
- Webhooks (`SPD-030`, skimmed): `v2.push` mechanism exists; not
  listing-contract critical.
- **Validator**: `NO_DEDICATED_VALIDATOR_FOUND_IN_PRIMARY_CORPUS`. No `validate`
  / `precheck` / `dry-run` / content-diagnosis / violation endpoint in the 35
  PDFs. The `add_item` / `update_item` response + the enumerated business error
  codes are the gate.

### 19. Numeric-values summary

| Value | Primary evidence | Scope | Static/Dynamic |
|---|---|---|---|
| `item_id_list` read max | `get_item_base_info` "[0,50]" | GLOBAL_API | STATIC = 50 |
| `price_list` / `stock_list` write max | "length 1–50" | GLOBAL_API | STATIC = 50 |
| model list per `init_tier_variation`/`add_model` call | "at most 50" | GLOBAL_API | STATIC = 50 |
| total models per item | `error_model_count_over_limit` "< 20 (50 for TW)" | MARKET | ~STATIC (conflict w/ "50") |
| options per tier | `error_tier_opt_too_many` "> 20" | GLOBAL_API | ~STATIC = 20 |
| 2-level combinations | "< 50" | GLOBAL_API | ~STATIC = 50 |
| max variation tiers | "two … maximum supported" / `over 2` error | GLOBAL_API | STATIC = 2 |
| `model_sku` length | "no more than 100 characters" | GLOBAL_API | STATIC = 100 |
| title length | `get_item_limit.item_name_length_limit` (sample 5–100) | SHOP+CATEGORY | DYNAMIC |
| description length | `get_item_limit.item_description_length_limit` (sample 10–2000) | SHOP+CATEGORY | DYNAMIC |
| image count | `get_item_limit.item_image_count_limit` (sample 1–9) | SHOP+CATEGORY | DYNAMIC |
| price bounds | `get_item_limit.price_limit` (sample 5.5–1e7) | SHOP+CATEGORY | DYNAMIC |
| stock bounds | `get_item_limit.stock_limit` (sample 5–1e7) | SHOP+CATEGORY | DYNAMIC |
| DTS bounds | `get_item_limit.days_to_ship_limit` | CATEGORY | DYNAMIC |
| tier name / option name length | `get_item_limit` (sample 14 / 20) | SHOP | DYNAMIC |
| shop total-listing quota | `get_item_limit.item_count_limit` (sample 5000) | SHOP | DYNAMIC |
| hashtags in description | `error_desc_hash_tag_over_limit` "> 18" | GLOBAL_API | ~STATIC = 18 |
| BR price decimals | `update_price` "two decimal place" for BR | BRAZIL | STATIC = 2dp |
| access_token TTL | auth doc "4 hours" | GLOBAL_API | STATIC = 4h |
| refresh_token TTL | auth doc "30 days" | GLOBAL_API | STATIC = 30d |
| auth `code` TTL | "expires after 10 minutes" | GLOBAL_API | STATIC = 10min |
| sign `timestamp` TTL | "valid for 5 minutes" | GLOBAL_API | STATIC = 5min |
| authorization validity | "≤ 360 days" (SPD-006: "365") | GLOBAL_API | STATIC (conflict) |
| `category_id_list` for `get_attribute_tree` | "max count is 20" | GLOBAL_API | STATIC = 20 |
| `get_brand_list` page_size | "Max=100" | GLOBAL_API | STATIC = 100 |
| `register_brand` category_list | "Max input num … is 50" | GLOBAL_API | STATIC = 50 |
| `scheduled_publish_time` window | "current +1h to +90days" | GLOBAL_API | STATIC |

No value obtained solely from triangulation. Every row cites a primary artifact.

---

## Corrections to prior research

See `reconciliation-report.md` for the full table. Material corrections:

1. **`get_item_limit` IS the dynamic source of field limits** (Phase 02.2 said it
   was "probably a quota, not the limit source"). It returns title / description /
   image / price / stock / tier / DTS limits + weight/dimension/size-chart/GTIN
   requiredness. The quota is one field (`item_count_limit`).
2. **`item_status`** enum corrected: `NORMAL, BANNED, UNLIST, SELLER_DELETE,
   SHOPEE_DELETE, REVIEWING` (no plain `DELETED`).
3. **Max variation tiers = 2** — primary-confirmed.
4. **`model_id = 0` = "no model item"** and Shopee keeps an internal **default
   model** for no-variation items — primary-confirmed (resolves the SCL-033 open
   question).
5. **GTIN**: `gtin_code = "00"` is the "no GTIN" mechanism; validation via
   `gtin_validation_rule` (Mandatory/Flexible/Optional); model-level; BR+TW.
6. **Attribute field is `mandatory`** (not `is_mandatory`) in `get_attribute_tree`.
7. **Brazil host** `https://openplatform.shopee.com.br/api/v2/` is the primary,
   explicitly documented BR Product API base.
8. **Brazil eligibility** upgraded from `UNRESOLVED` to `PRIMARY_VERIFIED`:
   available to Registered Business Sellers / ISVs, gated by developer-profile
   approval + Go-Live review; not whitelist-gated for listing APIs.
9. **Sign base string** — primary-verified per API type (Shop/Merchant/Public).
10. **Token lifetimes** — primary-verified (4h / 30d / 10min / 5min / ≤360d).
11. **`product_id`** is not a primary parameter — drop the "cross-border alias"
    worry. New concept surfaced: **`ssp_id` / Shopee Standard Product**.
12. **Logistics gap** narrowed: DTS/weight/dimension/size-chart requiredness +
    item-payload logistics fields are covered by `get_item_limit` + `add_item`;
    the logistics **service** endpoints are still missing.

---

## Remaining unresolved (`PRIMARY_NOT_FOUND` in corpus)

- Logistics service: `logistics/get_channel_list`, `logistics/get_address(_list)`,
  `v2.shop.get_warehouse_detail`, dedicated `get_dts_limit` page.
- Media service: `v2.media_space.upload_image` / `upload_video` (pixel/byte/type/
  cover-photo rules, video duration/aspect).
- `v2.product.search_attribute_value_list` (searchable attribute values).
- `get_recommend_attribute` full schema (skimmed only).
- `update_item` full field-by-field contract (page family known; not read in full).
- Developer-profile approval **criteria** (`developer-guide/12`).
- Whether `register_brand` is supported in BR.
- Full `item_status` transition graph; whether an edit re-triggers `REVIEWING`.
- Draft-via-API (beyond `UNLIST` + `scheduled_publish_time`).
- Shopee Standard Product (`ssp_id`) — catalogue model / how items attach.
- `v2.product.get_item_content_diagnosis_result` / `get_item_violation_info`
  (Phase 02.2 SEARCH_INDEXED names) — not in corpus; unverified.

---

## P0 coverage (brief §13)

| # | Domain | Status | Basis |
|---|---|---|---|
| 1 | Brazil eligibility / onboarding | **COVERED** | SPD-003/004/005/008/009 (approval + Go-Live model; BR developer types; app-category permissions; not whitelisted for listing APIs). Approval *criteria* not supplied → minor gap. |
| 2 | Authentication | **COVERED** | SPD-001 (flow, hosts, lifetimes, sign base string per API type, token endpoints). |
| 3 | Add Item | **COVERED** | SPD-010 (required/optional fields, response, full error family, permissions). `PRIMARY_PARTIAL` on some optional-field enums. |
| 4 | Get Item | **COVERED** | SPD-011 (`item_id_list ≤ 50`, `item_status` enum, `stock_info_v2`, `price_info` model rule, `ssp_id`, flags). |
| 5 | Variations / Models | **COVERED** | SPD-015/017/020 (max 2 tiers, `standardise_tier_variation`, `model_id = 0`, model_status, default model). Model-count 20-vs-50 `PRIMARY_CONFLICT`. |
| 6 | Categories | **COVERED** | SPD-021/022 (`get_category` tree + `has_children`; `category_recommend`). Explicit leaf-only sentence `PRIMARY_PARTIAL`. |
| 7 | Attributes | **COVERED** | SPD-023 (`get_attribute_tree`, `mandatory` bool, `input_type`/`format_type` enums, `max_value_count`). |
| 8 | Brands | **COVERED** | SPD-025/026 (`get_brand_list` per leaf, `status`, per-category `is_mandatory`; `add_item` brand object required; `register_brand` + QC). BR `register_brand` support `PRIMARY_NOT_FOUND`. |
| 9 | Identifiers / GTIN | **COVERED** | SPD-010/015/017/020/029/011 (`gtin_code`, `"00"`, `gtin_validation_rule` Mandatory/Flexible/Optional; model-level; BR+TW). |
| 10 | Price | **COVERED** | SPD-027 (`update_price`, batch 1–50, `model_id 0`, BR 2dp, error family). |
| 11 | Stock | **COVERED** | SPD-028 (`update_stock`, `seller_stock` absolute, `location_id` conditional, WMS/FBS rules, `stock_info_v2`). Warehouse *detail* endpoint missing. |
| 12 | Logistics | **PARTIAL** | limits + `add_item.logistic_info` + `pre_order` + `condition` + DTS/weight/dimension requiredness (via `get_item_limit`) covered. **`get_channel_list` / `get_address` / `get_warehouse_detail` / `get_dts_limit` pages MISSING.** |

**`LOGISTICS PRIMARY DOCUMENTATION MISSING`** — retained. 11 / 12 P0 areas
COVERED (a few with minor `PRIMARY_PARTIAL` / criteria gaps); 1 PARTIAL.

---

## Final decision

```
PRIMARY EVIDENCE EXTRACTION COMPLETE — ADDITIONAL PRIMARY ARTIFACTS REQUIRED
```

The listing/publication contract is now overwhelmingly primary-verified, but the
**logistics service** (channels, address, warehouse detail) is required before
Phase 02.4 can lock the full publication + execution rule set. See the
acquisition list below.

## Follow-up acquisition list (exact, minimal)

Only what the reconciliation proves is still needed for the **listing /
publication** contract:

1. **`v2.logistics.get_channel_list`** — logistics channels enabled for a shop
   (`logistic_id`, `fee_type` incl. `SIZE_SELECTION` / `CUSTOM_PRICE`, `enabled`).
   Needed for `add_item.logistic_info`.
2. **`v2.logistics.get_address_list`** (a.k.a. `get_address`) — pickup / return /
   default `address_id`. Referenced by `add_item.complaint_policy` and warranty.
3. **`v2.shop.get_warehouse_detail`** — `location_id` list for multi-warehouse
   `seller_stock`. Referenced by `update_stock` / `init_tier_variation` /
   `add_model`.
4. **`v2.product.get_dts_limit`** — days-to-ship min/max per category. Referenced
   by `add_item` / `add_model` / `init_tier_variation` prose (though
   `get_item_limit.days_to_ship_limit` may already cover it — confirm).
5. **`v2.media_space.upload_image`** + **`upload_video`** — image pixel/byte/type
   rules, cover-photo rules, video duration/aspect. Needed for `IMAGES` and
   `video.md`.
6. *(lower priority)* `v2.product.search_attribute_value_list`;
   `v2.product.get_recommend_attribute` (full); `v2.product.update_item` (full);
   the developer-profile approval criteria page (`developer-guide/12`); a page on
   Shopee Standard Product (`ssp_id`).

Not requested: Orders, Returns, Ads, Affiliate, Chat, Finance.

## Phase 02.4

Do **not** start. Recommended next phase after the acquisition list is filled:
**`Phase 02.4 — Rule Locking & Dynamic Requirement Resolution`** (turn the
`PRIMARY_VERIFIED` facts into static rules vs `get_item_limit`-resolved dynamic
checks vs publication blockers vs quality recommendations).
