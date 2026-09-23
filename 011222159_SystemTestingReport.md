# Panda-Lite API — System Testing Report

Course: CSE 4495 — System Testing & Quality Assurance
System Under Test: Panda-Lite API
Team Members:
- Md. Noman Hossain, ID: 011222159 (Project Lead)
- Md. Fahad Bin Elias, ID: 011192153

---

## 1. Test Plan

We tested the Panda-Lite API against the specifications in Panda-Lite API (`documentation.md`). Testing was split between the two team members: Noman tested the all of the API end to end, covering restaurants, menu items, the full order lifecycle, cancellations, and ratings and Fahad tested test cases 01 to 15 (health check through order status update) on a separate server instance.

For every endpoint we tested:
- Happy path (valid input, correct role)
- Missing/invalid token (expect 401)
- Wrong role or wrong owner (expect 403)
- Invalid input — missing fields, negative numbers, empty/whitespace strings, out-of-range values (expect 400)
- Non-existent IDs (expect 404)
- Order lifecycle rules — allowed/disallowed status transitions, cancellation window, rating eligibility
- Sorting and filtering on list endpoints (`sort_by`, `filter_field`, `filter_value`)


### Endpoints covered
- Auth: `POST /auth/register`, `POST /auth/login`
- Restaurants: `GET /restaurants`, `GET /restaurants/:id`, `POST /restaurants`, `PATCH /restaurants/:id`
- Menu: `GET /restaurants/:id/menu`, `POST /restaurants/:id/menu`, `PATCH /menu-items/:id`
- Orders: `POST /orders`, `GET /orders`, `GET /orders/:id`, `GET /orders/:id/timeline`, `GET /orders/available`, `PATCH /orders/:id/claim`, `PATCH /orders/:id/status`, `PATCH /orders/:id/cancel`
- Ratings: `POST /orders/:id/rate`, `GET /restaurants/:id/ratings`

---

## 2. Test Cases

| ID | Short Title | Pre-conditions | Steps | Expected Status Code | Actual Status Code | Verdict |
|---|---|---|---|---|---|---|
| 1ffc8d | 01 - Health Check | valid auth token | GET /api/panda-262/_internal/health | 200 OK | 200 OK | PASS |
| 72f79b | 02 - Register Customer | none | POST /api/panda-262/auth/register | 201 Created | 201 Created | PASS |
| bcc267 | 02A - Register Customer - Missing Field | none | POST /auth/register?X-STQA-Key | 400 Bad Request | 400 Bad Request | PASS |
| e1e1aa | 02B - Register Customer - Invalid Role | none | POST /auth/register | 400 Bad Request | 400 Bad Request | PASS |
| 162b74 | 02C - Register Customer - Duplicate Email | none; record already exists | POST /auth/register | 409 Conflict | 409 Conflict | PASS |
| ac61a2 | 02D - Register Customer - Short Password | none | POST /auth/register | 400 Bad Request | 201 Created | FAIL |
| b7ae46 | 02E - Register Customer - Minimum Password | none | POST /auth/register | 201 Created | 201 Created | PASS |
| 3f1f9f | 02F - Register Customer - Empty Name | none | POST /auth/register | 400 Bad Request | 400 Bad Request | PASS |
| 2f776b | 02G - Register Customer - Invalid Email | none | POST /auth/register | 201 Created | 201 Created | PASS |
| 57a6e4 | 02G-2 - Register Customer - Invalid Email Recheck | none | POST /auth/register | 201 Created | 201 Created | PASS |
| 67a6e2 | 03 - Login Customer | none | POST /auth/login | 200 OK | 200 OK | PASS |
| 67e9f4 | 04 - Get Restaurants | valid auth token | GET /api/panda-262/restaurants | 200 OK | 200 OK | PASS |
| a3483a | 05 - Register Restaurant | none | POST /api/panda-262/auth/register | 201 Created | 201 Created | PASS |
| 8ebbef | 06 - Login Restaurant | none | POST /auth/login | 200 OK | 200 OK | PASS |
| 346e73 | 07 - Create Restaurant | valid auth token | POST /restaurants | 201 Created | 201 Created | PASS |
| da023b | 08A - Get Restaurant | valid auth token | GET /restaurants/5121d38f-1c70-42d4-a34b-4eb5cbc2b9fe | 200 OK | 200 OK | PASS |
| 307a0f | 08B - Get Restaurant - No Token | no auth token | GET /restaurants/5121d38f-1c70-42d4-a34b-4eb5cbc2b9fe | 401 Unauthorized | 401 Unauthorized | PASS |
| d4cb4f | 08C - Get Restaurant - Not Found | valid auth token; target ID does not exist | GET /restaurants/5121d38f-1c70-42d4-a34b-4eb5cbc2e3ct | 404 Not Found | 500 Internal Server Error | FAIL |
| 27f2cc | 09A - Get Restaurants | valid auth token | GET /restaurants | 200 OK | 200 OK | PASS |
| 49e24a | 09B - Get Restaurants - No Token | no auth token | GET /restaurants | 401 Unauthorized | 401 Unauthorized | PASS |
| 24d514 | 09C - Get Restaurants - Name Filter | valid auth token | GET /restaurants?name=Panda | 200 OK, filtered results | 200 OK | PASS |
| e749f7 | 09D - Get Restaurants - Case Insensitive Filter | valid auth token | GET /restaurants?name=pAnDa | 200 OK, filtered results | 200 OK | PASS |
| eee0d5 | 09E - Get Restaurants - Invalid Query Parameter | valid auth token | GET /restaurants?sort_by=invalid | 400 Bad Request | 400 bad request | PASS |
| a76a41 | 09F - Get Restaurants - Sort Ascending | valid auth token | GET /restaurants?sort_by=asc | 200 OK, ascending order | 200 OK | PASS |
| 9a33ed | 09G - Get Restaurants - Sort Descending | valid auth token | GET /restaurants?sort_by=desc | 200 OK, descending order | 200 OK | PASS |
| 89cd1e | 09H - Get Restaurants - Invalid Filter | valid auth token | GET /restaurants?filter_field=invalid&filter_value=test | 400 Bad Request | 400 Bad request | PASS |
| 905d6b | 10A - Create Restaurant | valid auth token | POST /restaurants | 201 Created | 201 created | PASS |
| fc43af | 10B - Create Restaurant - No Token | no auth token | POST /api/panda-262/restaurants | 401 Unauthorized | 401 Unauthorized | PASS |
| c42dbe | 10C - Create Restaurant - Customer Token | valid token, role=customer | POST /api/panda-262/restaurants | 403 Forbidden | 403 Forbidden | PASS |
| 4b2a1f | 10D - Create Restaurant - Missing Name | valid auth token | POST /api/panda-262/restaurants | 400 Bad Request | 400 Bad Request | PASS |
| 2fbe80 | 10E - Create Restaurant - Missing Address | valid auth token | POST /api/panda-262/restaurants | 400 Bad Request | 400 Bad Request | PASS |
| 66099a | 10F - Create Restaurant - Empty Values | valid auth token | POST /api/panda-262/restaurants | 400 Bad Request | 400 Bad Request | PASS |
| 70eef8 | 10G - Create Restaurant - Null Values | valid auth token | POST /api/panda-262/restaurants | 400 Bad Request | 400 Bad Request | PASS |
| bfbdbe | 10H - Create Restaurant - Whitespace Values | valid auth token | POST /api/panda-262/restaurants | 400 Bad Request | 201 Created | FAIL |
| 577143 | 10I - Create Restaurant - Extra Field | valid auth token | POST /api/panda-262/restaurants | 201 Created | 201 created | PASS |
| d50aec | 11A - Update Restaurant - Happy Path | valid auth token | PATCH /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394 | 200 OK | 200 OK | PASS |
| b1fa97 | 11B - Update Restaurant - No Token | no auth token | PATCH /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394 | 401 Unauthorized | 401 Unauthorized | PASS |
| 4b64f3 | 11C - Update Restaurant - Non-existent Restaurant | valid auth token | PATCH /restaurants/00000000-0000-4000-8000-000000000000 | 404 Not Found | 404 not found | PASS |
| 8f61aa | 11D - Get Menu - Happy Path | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 200 OK | 200 ok | PASS |
| 972b34 | 11E - Get Menu - No Token | no auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 401 Unauthorized | 401 Unauthorized | PASS |
| 186013 | 12A - Add Menu Item - Happy Path | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 201 Created | 201 Created | PASS |
| 39c38d | 12B - Add Menu Item - No Token | no auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 401 Unauthorized | 401 Unauthorized | PASS |
| 7c0ce4 | 12C - Add Menu Item - Customer Token | valid token, role=customer | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 403 Forbidden | 403 Forbidden | PASS |
| 64e9ac | 12D - Add Menu Item - Missing Name | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 Bad request | PASS |
| b19cab | 12E - Add Menu Item - Missing Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 Bad request | PASS |
| f1e68d | 12F - Add Menu Item - Negative Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 Bad request | PASS |
| 0ebbcf | 12G - Add Menu Item - Negative Stock | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 bad request | PASS |
| f91910 | 12H - Add Menu Item - Zero Stock | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 201 Created | 201 Created | PASS |
| ffd71e | 12I - Add Menu Item - Zero Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 201 Created | 201 Created | PASS |
| 769ab4 | 12J - Get Menu - After Adding Items | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 200 OK | 200 OK | PASS |
| aa2915 | 12K - Get Menu - Available Filter | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=isAvailable&filter_value=true | 200 OK | 200 OK | PASS |
| 005887 | 12L - Get Menu - Invalid Filter Field | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=isAvailable&filter_value=true | 400 Bad Request | 200 OK | FAIL |
| 94691c | 13A - Update Menu Item - Happy Path | valid auth token | PATCH /menu-items/cff8baee-0e25-49a7-87e8-6d98b2cb30c6 | 200 OK | 200 OK | PASS |
| fc5d9a | 13B - Update Menu Item - No Token | no auth token | PATCH /menu-items/cff8baee-0e25-49a7-87e8-6d98b2cb30c6 | 401 Unauthorized | 401 Unauthorized | PASS |
| 058de4 | 13C - Update Menu Item - Customer Token | valid token, role=customer | PATCH /menu-items/cff8baee-0e25-49a7-87e8-6d98b2cb30c6 | 403 Forbidden | 403 Forbidden | PASS |
| 0ff696 | 13D - Update Menu Item - Non-existent Item | valid auth token | PATCH /menu-items/00000000-0000-4000-8000-000000000000 | 404 Not Found | 404 Not Found | PASS |
| 97edeb | 13E - Update Menu Item - Negative Price | valid auth token | PATCH /menu-items/cff8baee-0e25-49a7-87e8-6d98b2cb30c6 | 400 Bad Request | 500 Internal Server Error | FAIL |
| 298f2d | 13F - Update Menu Item - Negative Stock | valid auth token | PATCH /menu-items/cff8baee-0e25-49a7-87e8-6d98b2cb30c6 | 400 Bad Request | 400 Bad Request | PASS |
| e03663 | 14A - Create Order - Happy Path | valid auth token | POST /orders | 201 Created | 201 Created | PASS |
| 6f3078 | 14B - Create Order - No Token | no auth token | POST /orders | 401 Unauthorized | 401 Unauthorized | PASS |
| 6e1fc1 | 14C - Create Order - Restaurant Token | valid token, role=restaurant | POST /orders | 403 Forbidden | 201 Created | FAIL |
| a8a476 | 14D - Create Order - Missing Restaurant ID | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| c2ad67 | 14E - Create Order - Empty Items | valid auth token | POST /orders | 400 Bad Request | 400 Bad request | PASS |
| 6d4460 | 14F - Create Order - Invalid Menu Item | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| 26fa80 | 14G - Create Order - Zero Quantity | valid auth token | POST /orders | 400 Bad Request | 500 Internal Server Error | FAIL |
| db6785 | 14H - Get My Orders - Customer | valid auth token | GET /orders | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| db22f2 | 15A - Update Order Status - Accepted | valid auth token | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 161308 | 15B - Update Order Status - Preparing | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| ae52c1 | 15C - Update Order Status - Ready For Pickup | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 5b4465 | 15D - Update Order Status - Missing Status | valid auth token | PATCH /orders//status | 400 Bad Request | 400 Bad request | PASS |
| faec38 | 15E - Update Order Status - Customer Token | valid token, role=customer | PATCH /orders//status | 403 Forbidden | 403 Forbidden | PASS |
| 8b5d35 | 16A - Update Order Status - Invalid Status | valid auth token | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 400 Bad Request | 400 bad request | PASS |
| 9c454d | 16B - Update Order Status - Invalid Order ID | valid auth token | PATCH /orders/00000000-0000-0000-0000-000000000000/status | 404 Not Found | 404 Not Found | PASS |
| 3e1fb0 | 16C - Update Order Status - No Token | no auth token | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 401 Unauthorized | 401 Unauthorized | PASS |
| 5db5c0 | 17A - Get Order - Happy Path | valid auth token | GET /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 62e19c | 17B - Get Order - No Token | no auth token | GET /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b | 401 Unauthorized | 401 Unauthorized | PASS |
| 3c780b | 17C - Get Order - Non-existent Order | valid auth token | GET /orders/00000000-0000-0000-0000-000000000000 | 404 Not Found | 404 not found | PASS |
| a30bcb | 18A - Get My Orders - No Token | no auth token | GET /orders | 401 Unauthorized | 401 Unauthorized | PASS |
| b2cfc7 | 18B - Get My Orders - Customer | valid auth token | GET /orders | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| b9d59f | 18C - Get My Orders - Restaurant Token | valid token, role=restaurant | GET /orders | 200 OK | 200 OK [passwordHash exposed] | PASS |
| 7566d4 | 18D - Get My Orders - Invalid Token | valid auth token | GET /orders | 401 Unauthorized | 401 Unauthorized | PASS |
| 829873 | 19A - Update Order Status - No Token | no auth token | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 401 Unauthorized | 401 Unauthorized | PASS |
| 213458 | 19B - Update Order Status - Customer Token | valid token, role=customer | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 403 Forbidden | 403 Forbidden | PASS |
| 10a51e | 19C - Update Order Status - Restaurant Owner | valid token, role=restaurant (owner) | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 400 Bad Request | 200 OK, duplicate timeline entry [passwordHash exposed (DEF-22)] | FAIL |
| 5d8ff0 | 19D - Update Order Status - Invalid Status | valid auth token | PATCH /orders/4f05692b-fa2c-4e04-9cc9-3f54f3d6fa9b/status | 400 Bad Request | 400 bad request | PASS |
| ce3823 | 19E - Update Order Status - Invalid Order ID | valid auth token | PATCH /orders/00000000-0000-0000-0000-000000000000/status | 404 Not Found | 404 Not Found | PASS |
| 22c185 | 20A - Get Restaurant Menu - Happy Path | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 200 OK | 200 OK | PASS |
| c3e1c2 | 20B - Get Restaurant Menu - No Token | no auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 401 Unauthorized | 401 unauthorized | PASS |
| 1f2a7b | 20C - Get Restaurant Menu - Invalid Restaurant ID | valid auth token | GET /restaurants/00000000-0000-0000-0000-000000000000/menu | 404 Not Found | 404 Not Found | PASS |
| 6c48c1 | 21A - Get Menu - Available False | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=isAvailable&filter_value=false | 200 OK | 200 OK | PASS |
| 44cd87 | 21B - Get Menu - Invalid Filter Field | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=invalidField&filter_value=test | 400 Bad Request | 400 bad rquest | PASS |
| 4dad59 | 21C - Get Menu - Sort Ascending | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?sort_by=asc | 200 OK, ascending order | 200 OK, not ascending | FAIL |
| dd7191 | 21D - Get Menu - Sort Descending | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?sort_by=desc | 200 OK, descending order | 200 OK, not descending | FAIL |
| 20443e | 21E - Get Menu - Filter by isAvailable | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=isAvailable&filter_value=true | 200 OK, filtered results | 200 OK | PASS |
| 432ec4 | 21F - Get Menu - Filter by price | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=price&filter_value=250 | 200 OK, filtered results | 200 Ok | PASS |
| 08a2a4 | 21G - Get Menu - Filter by name | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=name&filter_value=Chicken Biryani | 200 OK, filtered results | 200 OK | PASS |
| 390cb6 | 21H - Get Menu - Filter by stockQuantity | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=stockQuantity&filter_value=20 | 200 OK, filtered results | 200 OK, empty result | PASS |
| b78376 | 21I - Get Menu - Invalid filter_field | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=invalidField&filter_value=test | 400 Bad Request | 400 bad req | PASS |
| b12e8c | 21J - Get Menu - Missing filter_value | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?filter_field=price | 400 Bad Request | 400 Bad Request | PASS |
| d95ad3 | 21K - Get Menu - Invalid sort_by | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu?sort_by=invalid | 400 Bad Request | 400 Bad Request | PASS |
| 506f75 | 21L - Get Menu - Missing Token | valid auth token | GET /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 401 Unauthorized | 401 Unauthorized | PASS |
| e9cd23 | 21M - Get Menu - Restaurant Not Found | valid auth token; target ID does not exist | GET /restaurants/00000000-0000-0000-0000-000000000000/menu | 404 Not Found | 404 not found | PASS |
| 3e9e49 | 22A - Create Menu Item - Valid Data | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 201 Created | 201 created | PASS |
| 567338 | 22B - Create Menu Item - Missing Name | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 Bad req | PASS |
| b36d9c | 22C - Create Menu Item - Missing Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 bad req | PASS |
| 9642d4 | 22D - Create Menu Item - Missing Name and Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 bad req | PASS |
| e4d255 | 22E - Create Menu Item - Negative Price | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 bad req | PASS |
| 5b9d12 | 22F - Create Menu Item - Negative Stock | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 400 Bad Request | 400 bad req | PASS |
| 161bee | 22G - Create Menu Item - Non-Owner Restaurant Account | valid token, role=restaurant (owner) | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 403 Forbidden | 201 Created | FAIL |
| 90bac7 | 22H - Create Menu Item - Customer Account | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 403 Forbidden | 403 Forbidden | PASS |
| 0af2c4 | 22I - Create Menu Item - Restaurant Not Found | valid auth token; target ID does not exist | POST /restaurants/00000000-0000-0000-0000-000000000000/menu | 404 Not Found | 404 not found | PASS |
| 7cce5a | 22J - Create Menu Item - No Token | no auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 401 Unauthorized | 401 Unauthorized | PASS |
| e17e08 | 22K - Create Menu Item - Zero Stock | valid auth token | POST /restaurants/86f7de2b-6cc4-43f4-bf84-4dcab9428394/menu | 201 Created | 201 Created | PASS |
| 80b3a8 | 23A - Update Menu Item - Valid Price | valid auth token | POST /menu-items/aff444dd-8b9c-4e20-87ef-6319c201ef77 | 201 Created | 200 OK | PASS |
| 6d0695 | 23B - Update Menu Item - Valid Stock | valid auth token | PATCH /menu-items/ | 200 OK | 200 OK | PASS |
| b8ac0d | 23C - Update Menu Item - Update Availability | valid auth token | PATCH /menu-items/ | 200 OK | 200 OK | PASS |
| 3b16f4 | 23D - Update Menu Item - Update Multiple Fields | valid auth token | PATCH /menu-items/ | 200 OK | 200 OK | PASS |
| 536a4a | 23E - Update Menu Item - Negative Price | valid auth token | PATCH /menu-items/ | 400 Bad Request | 500 Internal Server Error | FAIL |
| 616931 | 23F - Update Menu Item - Negative Stock | valid auth token | PATCH /menu-items/ | 400 Bad Request | 500 Internal Server Error | FAIL |
| 2c13ca | 23G - Update Menu Item - Non-Owner | valid token, not the resource owner | PATCH /menu-items/ | 403 Forbidden | 200 OK | FAIL |
| 4955c0 | 23H - Update Menu Item - Customer Account | valid auth token | PATCH /menu-items/ | 200 OK | 403 Forbidden | PASS |
| c85bf9 | 23I - Update Menu Item - Menu Item Not Found | valid auth token; target ID does not exist | PATCH /menu-items/00000000-0000-0000-0000-000000000000 | 404 Not Found | 404 not found | PASS |
| 9ca6ec | 23J - Update Menu Item - No Token | no auth token | PATCH /menu-items/ | 401 Unauthorized | 401 unauthorized | PASS |
| 601071 | 23K - Update Menu Item - Stock Zero Sets Unavailable | valid auth token | PATCH /menu-items/ | 200 OK, isAvailable false | 200 OK, isAvailable true | FAIL |
| a4cc85 | 24A - Place Order - Valid Order | valid auth token | POST /orders | 201 Created | 201 created [passwordHash exposed (DEF-22)] | PASS |
| c45c37 | 24B - Place Order - Missing Restaurant ID | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| 791c44 | 24C - Place Order - Missing Items | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| 38d753 | 24D - Place Order - Empty Items | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| bf7400 | 24E - Place Order - Restaurant Not Found | valid auth token; target ID does not exist | POST /orders | 404 Not Found | 404 not found | PASS |
| fed0e2 | 24F - Place Order - Restaurant Closed | valid auth token; restaurant isOpen=false | POST /orders | 400 Bad Request | 201 Created [passwordHash exposed (DEF-22)] | FAIL |
| 98b74d | 24G - Place Order - Invalid Menu Item | valid auth token | POST /orders | 400 Bad Request | 400 bad request | PASS |
| 13b7bc | 24H - Place Order - Menu Item From Another Restaurant | valid auth token | POST /orders | 201 Created | 400 bad request | PASS |
| 18bc2a | 24I - Place Order - Insufficient Stock | valid auth token | POST /orders | 201 Created | 400 bad request | PASS |
| 3028fe | 24J - Place Order - Unavailable Item | valid auth token | POST /orders | 201 Created | 400 bad request | PASS |
| a9378a | 24K - Place Order - Non-Customer Account | valid auth token | POST /orders | 403 Forbidden | 400 Bad Request (wrong error) | FAIL |
| 1cadd3 | 24L - Place Order - No Token | no auth token | POST /orders | 401 Unauthorized | 401 Unauthorized | PASS |
| 013b08 | 24M - Place Order - Stock Decrement Verification | valid auth token | POST /orders | 201 Created | 201 created [passwordHash exposed (DEF-22)] | PASS |
| 519c43 | 24N - Place Order - Stock Reaches Zero | valid auth token | POST /orders | 201 Created | 201 Created | PASS |
| 99e952 | 24O - Place Order - Multiple Items | valid auth token | POST /orders | 201 Created | 201 created [passwordHash exposed (DEF-22)] | PASS |
| 66d92b | 24P - Place Order - Timeline Created | valid auth token | POST /orders | 201 Created | 201 created [passwordHash exposed (DEF-22)] | PASS |
| eca348 | 25A - Get Orders - Customer Orders | valid auth token | GET /orders | 200 OK | 200 ok [passwordHash exposed (DEF-22)] | PASS |
| 0e6b29 | 25B - Get Orders - Restaurant Owner Orders | valid token, role=restaurant (owner) | GET /orders | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 745f7b | 25C - Get Orders - Rider Orders | valid token, role=rider | GET /orders | 200 OK | 200 OK | PASS |
| c3318d | 25D - Get Orders - Sort Ascending | valid auth token | GET /orders?sort_by=asc | 200 OK, ascending, no duplicates | 200 OK, not ascending + duplicate [passwordHash exposed (DEF-22)] | FAIL |
| 0640aa | 25E - Get Orders - Sort Descending | valid auth token | GET /orders?sort_by=desc | 200 OK, descending order | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 08c012 | 25F - Get Orders - Filter by Status | valid auth token | GET /orders?filter_value=placed&filter_field=status | 200 OK, filtered results | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| aaee5f | 25G - Get Orders - Filter by Total | valid auth token | GET /orders?filter_value=275&filter_field=total | 200 OK, filtered results | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 00c6e0 | 25H - Get Orders - Filter by Restaurant ID | valid auth token | GET /orders?filter_value=&filter_field=restaurantId | 200 OK, filtered results | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 44c94b | 25I - Get Orders - Invalid filter_field | valid auth token | GET /orders?filter_value=test&filter_field=invalidField | 400 Bad Request | 400 bad request | PASS |
| de3a81 | 25J - Get Orders - Missing filter_value | valid auth token | GET /orders?filter_field=status | 400 Bad Request | 400 bad request | PASS |
| 09789f | 25K - Get Orders - Invalid sort_by | valid auth token | GET /orders?sort_by=invalid | 400 Bad Request | 400 bad request | PASS |
| 9d353d | 25L - Get Orders - No Token | no auth token | GET /orders | 401 Unauthorized | 401 unauthorized | PASS |
| 53712f | 26A - Get Order - Valid Order | valid auth token | GET /orders/ | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 998560 | 26B - Get Order - Customer Access | valid auth token | GET /orders/ | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 0ea5aa | 26C - Get Order - Restaurant Owner Access | valid token, role=restaurant (owner) | GET /orders/ | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 6f944b | 26D - Get Order - Assigned Rider Access | valid token, role=rider | GET /orders/ | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| c7fd43 | 26E - Get Order - Unauthorized Customer | valid auth token | GET /orders/ | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| 9a95ba | 26F - Get Order - Unauthorized Restaurant | valid auth token | GET /orders/ | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| fb620f | 26G - Get Order - Unauthorized Rider | valid token, role=rider | GET /orders/ | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| d0c0fc | 26H - Get Order - Order Not Found | valid auth token; target ID does not exist | GET /orders/00000000-0000-0000-0000-000000000000 | 404 Not Found | 404 not found | PASS |
| 19880b | 26I - Get Order - No Token | no auth token | GET /orders/ | 401 Unauthorized | 401 Unauthorized | PASS |
| 0e9d91 | 27A - Get Timeline - Valid Order | valid auth token | GET /orders//timeline | 200 OK, correct order | 200 OK, out of order | FAIL |
| 9874b8 | 27B - Get Timeline - Sort Ascending | valid auth token | GET /orders//timeline?sort_by=asc | 200 OK, ascending | 200 OK, not ascending | FAIL |
| dbf3f2 | 27C - Get Timeline - Sort Descending | valid auth token | GET /orders//timeline?sort_by=desc | 200 OK, descending | 200 OK, not descending | FAIL |
| c724d0 | 27D - Get Timeline - Filter by Status | valid auth token | GET /orders//timeline?filter_value=placed&filter_field=status | 200 OK, filtered results | 200 OK | PASS |
| 9bbbde | 27E - Get Timeline - Filter by Actor Role | valid auth token | GET /orders//timeline?filter_value=customer&filter_field=actorRole | 200 OK, filtered results | 200 OK | PASS |
| d02cb7 | 27F - Get Timeline - Invalid filter_field | valid auth token | GET /orders//timeline?filter_value=test&filter_field=invalidField | 400 Bad Request | 400 bad request | PASS |
| 2ddc2a | 27G - Get Timeline - Missing filter_value | valid auth token | GET /orders//timeline?filter_field=status | 400 Bad Request | 400 bad request | PASS |
| bfdbb1 | 27H - Get Timeline - Invalid sort_by | valid auth token | GET /orders//timeline?sort_by=invalid | 400 Bad Request | 400 bad request | PASS |
| d14080 | 27I - Get Timeline - Unauthorized User | valid auth token | GET /orders//timeline | 403 Forbidden | 403 Forbidden | PASS |
| 66af68 | 27J - Get Timeline - Order Not Found | valid auth token; target ID does not exist | GET /orders/00000000-0000-0000-0000-000000000000/timeline | 404 Not Found | 404 not found | PASS |
| 454fda | 27K - Get Timeline - No Token | no auth token | GET /orders//timeline | 401 Unauthorized | 401 Unauthorized | PASS |
| 44f4a0 | 28A - Get Available Orders - Valid Rider | valid token, role=rider | GET /orders/available | 200 OK | 200 OK | PASS |
| 939267 | 28B - Get Available Orders - Sort Ascending | valid auth token | GET /orders/available?sort_by=asc | 200 OK, ascending order | 200 OK | PASS |
| ce44d4 | 28C - Get Available Orders - Sort Descending | valid auth token | GET /orders/available?sort_by=desc | 200 OK, descending order | 200 OK | PASS |
| da9895 | 28D - Get Available Orders - Filter by Status | valid auth token | GET /orders/available?filter_value=ready_for_pickup&filter_field=status | 200 OK, filtered results | 200 OK | PASS |
| eab988 | 28E - Get Available Orders - Filter by Delivery Fee | valid auth token | GET /orders/available?filter_value=50&filter_field=deliveryFee | 200 OK, filtered results | 200 OK | PASS |
| 0863e7 | 28F - Get Available Orders - Invalid filter_field | valid auth token | GET /orders/available?filter_value=test&filter_field=invalidField | 400 Bad Request | 400 bad request | PASS |
| 1fc2fb | 28G - Get Available Orders - Missing filter_value | valid auth token | GET /orders/available?filter_value=&filter_field=status | 400 Bad Request | 200 OK, empty array | FAIL |
| d3dbb5 | 28H - Get Available Orders - Invalid sort_by | valid auth token | GET /orders/available?sort_by=invalid | 400 Bad Request | 400 Bad request | PASS |
| 994112 | 28I - Get Available Orders - Customer Account | valid auth token | GET /orders/available | 200 OK | 403 Forbidden | PASS |
| d5d88d | 28J - Get Available Orders - Restaurant Account | valid auth token | GET /orders/available | 200 OK | 403 Forbidden | PASS |
| a9f8f2 | 28K - Get Available Orders - No Token | no auth token | GET /orders/available | 401 Unauthorized | 401 Unauthorized | PASS |
| 743fd8 | 29A - Claim Order - Valid Rider | valid token, role=rider | PATCH /orders//claim | 200 OK | 400 bad request | PASS |
| 19038c | 29B - Claim Order - Non-Rider | valid token, role=rider | PATCH /orders//claim | 403 Forbidden | 403 Forbidden | PASS |
| 52cd40 | 29C - Claim Order - Order Not Found | valid auth token; target ID does not exist | PATCH /orders/00000000-0000-0000-0000-000000000000/claim | 404 Not Found | 404 not found | PASS |
| f2cd8c | 29D - Claim Order - Order Not Ready | valid auth token; order in later lifecycle state | PATCH /orders//claim | 200 OK | 400 bad request | PASS |
| 06ded9 | 29E - Claim Order - Already Claimed | valid auth token | PATCH /orders//claim | 409 Conflict | 400 Bad Request (order not ready_for_pickup) | INCONCLUSIVE |
| 455c67 | 29F - Claim Order - No Token | no auth token | PATCH /orders//claim | 401 Unauthorized | 401 unauthorized | PASS |
| e9c6d9 | 29G - Claim Order - Timeline Verification | valid auth token | PATCH /orders//claim | 200 OK | 400 bad request | PASS |
| adcc52 | 30A - Order Status - Placed to Accepted | valid auth token | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 5c836c | 30B - Order Status - Customer Attempts Accept | valid auth token | PATCH /orders//status | 200 OK | 403 Forbidden | PASS |
| e7f346 | 30C - Order Status - Other Restaurant Attempts Accept | valid token, not the resource owner | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| 3d872f | 30D - Order Status - Invalid Transition | valid auth token | PATCH /orders//status | 400 Bad Request | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| f1ae91 | 30E - Order Status - Missing Status | valid auth token | PATCH /orders//status | 400 Bad Request | 400 bad request | PASS |
| 99d53a | 30F - Order Status - Accepted to Preparing | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 028588 | 30G - Order Status - Customer Attempts Preparing | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 403 Forbidden | PASS |
| 403b56 | 30H - Order Status - Other Restaurant Attempts Preparing | valid token, not the resource owner; order in later lifecycle state | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| 3e728b | 30I - Order Status - Preparing to Ready | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 1bf3be | 30J - Order Status - Unauthorized Restaurant | valid auth token | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| 31ba24 | 30K - Order Status - Ready to Picked Up | valid auth token; order in later lifecycle state | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 71ad77 | 30L - Order Status - Unassigned Rider Attempts Pickup | valid token, role=rider | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed] | FAIL |
| ca03f3 | 30M - Order Status - Other Rider Attempts Pickup | valid token, role=rider | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed] | FAIL |
| c61746 | 30N - Order Status - Picked Up to Delivered | valid auth token; order already delivered | PATCH /orders//status | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 0011b7 | 30O - Order Status - Other Rider Attempts Delivery | valid token, role=rider | PATCH /orders//status | 403 Forbidden | 200 OK [passwordHash exposed (DEF-22)] | FAIL |
| c2abcb | 30P - Order Status - Invalid Status Transition | valid auth token | PATCH /orders/{order_id}/status | 400 Bad Request | 400 Bad Request | PASS |
| e5a1d4 | 30Q - Order Status - Order Not Found | valid auth token; target ID does not exist | PATCH /orders/00000000-0000-0000-0000-000000000000/status | 404 Not Found | 404 not found | PASS |
| 2ba420 | 30R - Order Status - No Token | no auth token | PATCH /orders//status | 401 Unauthorized | 401 Unauthorized | PASS |
| 7af718 | 31A - Cancel Order - Placed Free Cancellation | valid auth token | PATCH /orders//cancel | 200 OK | 200 OK | PASS |
| 383db7 | 31B - Cancel Order - Accepted Penalty Cancellation | valid auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 400 bad request | PASS |
| 4b836c | 31C - Cancel Order - Cancellation With Reason | valid auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 400 bad request | PASS |
| e8d814 | 31D - Cancel Order - Cancellation Without Reason | valid auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 400 bad request | PASS |
| 2a0193 | 31E - Cancel Order - Customer Cancels Own Order | valid auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 400 Bad Request | PASS |
| f7b7f8 | 31F - Cancel Order - Non-Owner Customer | valid token, not the resource owner | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 403 Forbidden | 400 Bad Request | FAIL |
| 2ad683 | 31G - Cancel Order - Restaurant Attempts Cancel | valid auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 403 Forbidden | PASS |
| 593d1d | 31H - Cancel Order - Rider Attempts Cancel | valid token, role=rider | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 200 OK | 403 Forbidden | PASS |
| adbfb0 | 31I - Cancel Order - Preparing Order | valid auth token; order in later lifecycle state | PATCH /orders//cancel | 400 Bad Request | 200 OK | FAIL |
| f0dfc2 | 31J - Cancel Order - Ready Order | valid auth token; order in later lifecycle state | PATCH /orders//cancel | 400 Bad Request | 200 OK | FAIL |
| e464b9 | 31K - Cancel Order - Delivered Order | valid auth token; order already delivered | PATCH /orders/c22092b1-e27a-43fa-a995-95d4171ea908/cancel | 200 OK | 400 bad request | PASS |
| 3be6e7 | 31L - Cancel Order - Order Not Found | valid auth token; target ID does not exist | PATCH /orders/00000000-0000-0000-0000-000000000000/cancel | 404 Not Found | 404 Not Found | PASS |
| aa8e10 | 31M - Cancel Order - No Token | no auth token | PATCH /orders/a30b4501-0b61-41ea-a21d-7b4156af06da/cancel | 401 Unauthorized | 401 Unauthorized | PASS |
| 513186 | 31N - Cancel Order - Timeline Verification | valid auth token | GET /orders/ | 200 OK | 200 OK [passwordHash exposed (DEF-22)] | PASS |
| 2e2414 | 32A - Rate Order - Restaurant Valid Rating | valid auth token | POST /orders//rate | 201 Created | 201 created | PASS |
| 7c1dc1 | 32B - Rate Order - Rider Valid Rating | valid token, role=rider | POST /orders//rate | 201 Created | 400 bad request | PASS |
| 94ee21 | 32C - Rate Order - Score 1 | valid auth token | POST /orders//rate | 201 Created | 201 created | PASS |
| a26020 | 32D - Rate Order - Score 5 | valid auth token | POST /orders/{order_id}/rate | 201 Created | 201 created | PASS |
| 8aaf90 | 32E - Rate Order - Score 0 | valid auth token | POST /orders//rate | 400 Bad Request | 500 Internal Server Error | FAIL |
| 88ad17 | 32F - Rate Order - Score 6 | valid auth token | POST /orders//rate | 400 Bad Request | 500 Internal Server Error | FAIL |
| 36d934 | 32G - Rate Order - Decimal Score | valid auth token | POST /orders//rate | 201 Created | 400 bad request | PASS |
| 983b56 | 32H - Rate Order - Invalid Target | valid auth token | POST /orders/{order_id}/rate | 400 Bad Request | 400 bad request | PASS |
| 68d482 | 32I - Rate Order - Missing Target | valid auth token | POST /orders//rate | 400 Bad Request | 400 Bad Request | PASS |
| 4383e7 | 32J - Rate Order - Missing Score | valid auth token | POST /orders//rate | 400 Bad Request | 400 Bad Request | PASS |
| cf0db2 | 32K - Rate Order - Undelivered Order | valid auth token | POST /orders//rate | 201 Created | 201 created | PASS |
| f223a6 | 32L - Rate Order - Non-Customer | valid auth token | POST /orders//rate | 403 Forbidden | 403 Forbidden | PASS |
| cdf35c | 32M - Rate Order - Rider Not Assigned | valid token, role=rider | POST /orders//rate | 201 Created | 500 internal server error | PASS |
| 942feb | 32N - Rate Order - Duplicate Restaurant Rating | valid auth token; record already exists | POST /orders//rate | 409 Conflict | 201 Created | FAIL |
| 38dd9f | 32O - Rate Order - Duplicate Rider Rating | valid token, role=rider; record already exists | POST /orders//rate | 400 Bad Request | 400 Bad Request | PASS |
| 8170cd | 32P - Rate Order - Order Not Found | valid auth token; target ID does not exist | POST /orders/00000000-0000-0000-0000-000000000000/rate | 404 Not Found | 404 not found | PASS |
| 12b7e7 | 32Q - Rate Order - No Token | no auth token | POST /orders//rate | 401 Unauthorized | 401 unauthorized | PASS |
| b0d0d6 | 33A - Get Ratings - Valid Restaurant | valid auth token | GET /restaurants//ratings | 200 OK | 200 OK | PASS |
| 89ea25 | 33B - Get Ratings - Sort Ascending | valid auth token | GET /restaurants//ratings?sort_by=score&sort_order=asc | 400 Bad Request | 400 Bad Request | PASS |
| f5f2dd | 33C - Get Ratings - Sort Descending | valid auth token | GET /restaurants//ratings?sort_by=score&sort_order=desc | 400 Bad Request | 400 Bad Request | PASS |
| 1b12cb | 33D - Get Ratings - Filter by Score | valid auth token | GET /restaurants//ratings?filter_value=4&filter_field=score | 200 OK, filtered results | 200 OK | PASS |
| b4cd0e | 33E - Get Ratings - Filter by Target | valid auth token | GET /restaurants//ratings?filter_value=restaurant&filter_field=target | 200 OK, filtered results | 200 OK | PASS |
| 08cef2 | 33F - Get Ratings - Invalid filter_field | valid auth token | GET /restaurants//ratings?filter_value=test&filter_field=invalid | 400 Bad Request | 400 Bad request | PASS |
| d5ba3d | 33G - Get Ratings - Missing filter_value | valid auth token | GET /restaurants//ratings?filter_field=score | 400 Bad Request | 400 bad request | PASS |
| b927d2 | 33H - Get Ratings - Invalid sort_by | valid auth token | GET /restaurants//ratings?sort_by=invalid&sort_order=asc | 400 Bad Request | 400 bad request | PASS |
| 0925a8 | 33I - Get Ratings - Restaurant Not Found | valid auth token; target ID does not exist | GET /restaurants/00000000-0000-0000-0000-000000000000/ratings | 404 Not Found | 404 not found | PASS |
| dfcd5b | 33J - Get Ratings - No Token | no auth token | GET /restaurants//ratings | 401 Unauthorized | 401 unauthorized | PASS |
---

## 3. Defect Reports

**DEF-01 — Short password accepted at registration**
- ID: ac61a2
- Severity: Medium
- Steps to Reproduce: Send `POST /auth/register` with a valid name, email, role, and a 5-character password.
- Expected: 400 Bad Request — "Password must be at least 6 characters long"
- Actual: 201 Created — account created
- Test Case No.: 02D

**DEF-02 — GET restaurant by non-existent ID returns 500 instead of 404**
- ID: d4cb4f
- Severity: Medium
- Steps to Reproduce: `GET /restaurants/:id` with an ID that does not exist.
- Expected: 404 Not Found — "Restaurant not found"
- Actual: 500 Internal Server Error
- Test Case No.: 08C

**DEF-03 — Whitespace-only restaurant name/address accepted**
- ID: bfbdbe
- Severity: Medium
- Steps to Reproduce: `POST /restaurants` with `name` and `address` set to whitespace-only strings.
- Expected: 400 Bad Request
- Actual: 201 Created — restaurant created with whitespace-only fields
- Test Case No.: 10H

**DEF-04 — Invalid filter_field on menu list is silently accepted**
- ID: 005887
- Severity: Medium
- Steps to Reproduce: `GET /restaurants/:id/menu?filter_field=invalidField&filter_value=test`
- Expected: 400 Bad Request with allowed-field list
- Actual: 200 OK — invalid filter ignored, normal list returned
- Test Case No.: 12L

**DEF-05 — Negative price/stock on menu item update crashes to 500**
- ID: 97edeb, 536a4a, 616931
- Severity: High
- Steps to Reproduce: `PATCH /menu-items/:id` with a negative `price` or negative `stockQuantity`.
- Expected: 400 Bad Request — "price and stockQuantity must be non-negative"
- Actual: 500 Internal Server Error. Reproduced for negative price (13E, 23E) and negative stock (23F).
- Test Case No.: 13E, 23E, 23F

**DEF-06 — Restaurant-role token can place an order as a customer**
- ID: 6e1fc1
- Severity: High
- Steps to Reproduce: Authenticate with a restaurant account's token, then `POST /orders` with a valid payload.
- Expected: 403 Forbidden — "Only customer accounts can place orders"
- Actual: 201 Created — order created with the restaurant account recorded as the customer
- Test Case No.: 14C

**DEF-07 — Zero-quantity order item crashes to 500**
- ID: 26fa80
- Severity: High
- Steps to Reproduce: `POST /orders` with an item `quantity` of 0.
- Expected: 400 Bad Request
- Actual: 500 Internal Server Error
- Test Case No.: 14G

**DEF-08 — Re-applying an already-applied status transition adds a duplicate timeline event**
- ID: 10a51e
- Severity: Medium
- Steps to Reproduce: Move an order to `accepted`, then send `PATCH /orders/:id/status` with `accepted` again on the same order. Check `GET /orders/:id/timeline`.
- Expected: 400 Bad Request, or at least no second timeline entry
- Actual: 200 OK — transition succeeds again, duplicate `accepted` timeline event recorded
- Test Case No.: 19C

**DEF-09 — sort_by=asc/desc does not sort correctly on several endpoints**
- ID: 4dad59, dd7191, c3318d, 9874b8, dbf3f2
- Severity: High
- Steps to Reproduce: Request the list with `sort_by=asc` and `sort_by=desc` on the menu list, order list, and order timeline, and compare the order returned.
- Expected: Correctly sorted ascending/descending by the endpoint's default field
- Actual: Menu list not sorted correctly either direction (21C, 21D); order list not sorted ascending (25D); timeline not sorted either direction (27B, 27C)
- Test Case No.: 21C, 21D, 25D, 27B, 27C

**DEF-10 — Non-owner restaurant account can create/update another restaurant's menu items**
- ID: 161bee, 2c13ca
- Severity: Critical
- Steps to Reproduce: Authenticate as Restaurant B, then `POST /restaurants/:A_id/menu` or `PATCH /menu-items/:id` on an item owned by Restaurant A.
- Expected: 403 Forbidden — "Only the restaurant owner can manage the menu"
- Actual: Create (22G) — 201 Created. Update (23G) — 200 OK, item modified by the non-owner.
- Test Case No.: 22G, 23G

**DEF-11 — stockQuantity = 0 via update does not force isAvailable = false**
- ID: 601071
- Severity: Medium
- Steps to Reproduce: `PATCH /menu-items/:id` with `stockQuantity: 0, isAvailable: true`.
- Expected: isAvailable becomes false automatically
- Actual: isAvailable stays true
- Test Case No.: 23K

**DEF-12 — Closed restaurant still accepts new orders**
- ID: fed0e2
- Severity: High
- Steps to Reproduce: Set a restaurant's `isOpen` to false, then place an order against it as a customer.
- Expected: 400 Bad Request — "Restaurant is currently closed"
- Actual: 201 Created — order placed
- Test Case No.: 24F

**DEF-13 — Non-customer order placement returns the wrong error**
- ID: a9378a
- Severity: Medium
- Steps to Reproduce: Authenticate with a non-customer role, then `POST /orders` with a valid body.
- Expected: 403 Forbidden — "Only customer accounts can place orders"
- Actual: 400 Bad Request — "Insufficient stock for one or more items" (role check not enforced before stock check)
- Test Case No.: 24K

**DEF-14 — Duplicate order returned in GET /orders**
- ID: c3318d
- Severity: Medium
- Steps to Reproduce: `GET /orders?sort_by=asc` as the customer; check for repeated order IDs in the array.
- Expected: Each order appears once
- Actual: One order appears twice in the response
- Test Case No.: 25D

**DEF-15 — Unauthorized users can view orders that don't belong to them**
- ID: c7fd43, 9a95ba, fb620f
- Severity: Critical
- Steps to Reproduce: As a customer/restaurant/rider not related to the order, call `GET /orders/:id`.
- Expected: 403 Forbidden — "You do not have permission to view this order"
- Actual: 200 OK — full order returned to an unrelated customer (26E), restaurant (26F), and rider (26G)
- Test Case No.: 26E, 26F, 26G

**DEF-16 — Order timeline events are out of order**
- ID: 0e9d91
- Severity: Medium
- Steps to Reproduce: `GET /orders/:id/timeline` for an order that has gone through several status changes.
- Expected: Events listed in the order they actually happened
- Actual: A "preparing" event appears before "placed"; "preparing" and "ready_for_pickup" appear before "accepted"
- Test Case No.: 27A

**DEF-17 — filter_field without filter_value is silently accepted**
- ID: 1fc2fb
- Severity: Medium
- Steps to Reproduce: `GET /orders/available?filter_field=status` (no filter_value) as a rider.
- Expected: 400 Bad Request — "filter_value is required when filter_field is provided"
- Actual: 200 OK, empty result returned instead of an error
- Test Case No.: 28G

**DEF-18 — Restaurant/rider ownership is not enforced on order status transitions**
- ID: e7f346, 403b56, 1bf3be, 71ad77, ca03f3
- Severity: Critical
- Steps to Reproduce: Authenticate as a restaurant/rider account not tied to the order, then `PATCH /orders/:id/status` to advance it (accept / preparing / ready / picked up).
- Expected: 403 Forbidden
- Actual: 200 OK in every case — an unrelated restaurant accepted an order (30C) and marked it preparing (30H) and ready (30J); an unassigned rider (30L) and an unrelated rider (30M) both marked an order picked up
- Test Case No.: 30C, 30H, 30J, 30L, 30M

**DEF-19 — Invalid/skipped status transitions are allowed**
- ID: 3d872f
- Severity: High
- Steps to Reproduce: `PATCH /orders/:id/status` requesting a status that isn't the next valid step (e.g. jumping straight to `delivered`).
- Expected: 400 Bad Request — "Cannot transition from X to Y"
- Actual: 200 OK — status updated despite skipping steps
- Test Case No.: 30D

**DEF-20 — Orders can be cancelled after preparation has started**
- ID: adbfb0, f0dfc2
- Severity: High
- Steps to Reproduce: Advance an order to `preparing` (31I) or `ready_for_pickup` (31J), then `PATCH /orders/:id/cancel` as the customer.
- Expected: 400 Bad Request — "Orders can only be cancelled before preparation begins"
- Actual: 200 OK, order cancelled, in both states
- Test Case No.: 31I, 31J

**DEF-21 — Out-of-range rating score crashes to 500**
- ID: 8aaf90, 88ad17
- Severity: Medium
- Steps to Reproduce: `POST /orders/:id/rate` with `score: 0` (32E) or `score: 6` (32F) on a delivered order.
- Expected: 400 Bad Request — "score must be an integer between 1 and 5"
- Actual: 500 Internal Server Error, both cases
- Test Case No.: 32E, 32F

**DEF-22 — passwordHash exposed in API responses**
- ID: e03663
- Severity: Critical
- Steps to Reproduce: Call any endpoint that returns an order with a nested customer (e.g. `POST /orders`, `GET /orders`, `GET /orders/:id`) and check the `customer` object.
- Expected: `passwordHash` must never appear in any response
- Actual: `passwordHash` is present in the customer object on most order-related endpoints — confirmed across 42 test cases: 14A, 14C, 14H, 15A–15C, 17A, 18B, 18C, 19C, 24A, 24F, 24M, 24O, 24P, 25A, 25B, 25D–25H, 26A–26G, 30A, 30C, 30D, 30F, 30H–30N, 31N
- Test Case No.: 14A (+ 41 more)

**DEF-23 — Order cancellation ownership check not enforced before state check**
- ID: f7b7f8
- Severity: Medium
- Steps to Reproduce: As a customer who does not own the order, and where the order is already past `accepted`, call `PATCH /orders/:id/cancel`.
- Expected: 403 Forbidden — "Only the customer who placed this order can cancel it"
- Actual: 400 Bad Request — "Orders can only be cancelled before preparation begins" (ownership never checked)
- Test Case No.: 31F

**DEF-24 — Duplicate restaurant rating is accepted**
- ID: 942feb
- Severity: Medium
- Steps to Reproduce: Rate the restaurant on a delivered order once, then submit `POST /orders/:id/rate` with `target: "restaurant"` again for the same order.
- Expected: 409 Conflict — "You have already rated this order's restaurant"
- Actual: 201 Created — a second rating is created
- Test Case No.: 32N

---

## 4. Individual Reflections

**Md. Noman Hossain (011222159):** I tested the whole project from start to finish — restaurants, menu items, the full order lifecycle, cancellations, and ratings. I also put together this report from both of our results.

**Md. Fahad Bin Elias (011192153):** Fahad tested test cases 01 to 15 (health check up to order status update) on a separate server that we set up on our own. We worked separately so we could cross-check each other's results.
