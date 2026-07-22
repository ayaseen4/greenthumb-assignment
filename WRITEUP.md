# CYSE 411 · Assignment 3 Write-up — GreenThumb Secure Build

### Defect 1: SQL Injection (Authentication Bypass)
* **Vulnerable Lines & Flaw:** In `server.js` inside the `POST /login` handler, credentials were glued into the database string via string concatenation. This allowed input values to escape their data context and manipulate the SQL interpreter directly.
* **Exact Payload Used:** `curator' --` (entered into the username field). This terminated the string literal early and commented out the entire password validation check.
* **Fix & Why it Works:** Replaced string concatenation with safe parameterized bindings (`?`). The database engine now compiles the query structure first and safely maps inputs strictly as primitive text values.
* **Limits of Fix:** Parameterization does not protect against logical authentication bypass bugs, such as faulty state logic or weak passwords vulnerable to brute-forcing.

### Defect 2: SQL Injection (UNION Data Extraction & Unsafe ORDER BY)
* **Vulnerable Lines & Flaw:** In `server.js` inside the `GET /search` handler, user input was injected directly into a `LIKE` filter and an `ORDER BY` statement, enabling attackers to extend the query block.
* **Exact Payload Used:** `%' UNION SELECT username, password, 'blank3', 'blank4' FROM users --`
* **Fix & Why it Works:** Handled the text search query parameter via array positional database bindings, and constructed a rigid structural map object layer (`validSorts`) acting as an explicit white-list filter for column sorts.
* **Limits of Fix:** The allow-list only protects the pre-mapped structural configurations; if a developer accidentally includes an internal administrative column in the dictionary array map, users can still sort and map it.

### Defect 3: Reflected Cross-Site Scripting (XSS)
* **Vulnerable Lines & Flaw:** In `server.js` inside `GET /search`, the raw search text query string parameter (`q`) was reflected immediately back to the browser interface template layout without text character transformations.
* **Exact Payload Used:** `<img src=x onerror=alert(1)>`
* **Fix & Why it Works:** Added an `escapeHtml()` text filter converter utility function replacing active structure characters (such as `<`, `>`, `&`, `"`, `'`) with safe alphanumeric HTML entities.
* **Limits of Fix:** HTML entity escaping only works perfectly inside standard structural text bodies; it does not protect the site if input is reflected directly inside JavaScript execution contexts or raw attribute URIs.

### Defect 4: Stored XSS & DOM-based XSS
* **Vulnerable Lines & Flaw:** The comment engine saved raw text properties directly to the storage database and displayed them raw to visitors. Additionally, `public/app.js` processed fragment URL properties directly using high-risk `.innerHTML` parsing assignments.
* **Exact Payload Used:** `<img src=x onerror="xss_marker">` inside text comments, and `#note=<img src=x onerror=alert(1)>` inside URL fragment locations.
* **Fix & Why it Works:** Implemented the `escapeHtml()` utility sanitizer block to clean comment variables server-side, and updated the client script script processing code to print text inside a dedicated inner span element using secure text parameters (`.textContent`).
* **Limits of Fix:** Strict plain text rendering methods (`.textContent`) completely strip out all structural elements, meaning you cannot use this approach if you legitimately want to allow harmless rich-text styling like bolding or hyperlinks.

### Defect 5: Cookie Hardening & Content-Security-Policy (CSP)
* **Vulnerable Lines & Flaw:** Session cookies (`sid`) lacked structural protection constraints against client script inspection engines, and the host application did not pass resource management safety directives to the web browser.
* **Fix & Why it Works:** Appended explicit `HttpOnly` and `SameSite=Strict` header flags onto the cookie constructor to prevent script-based cookie theft, and deployed a global Content-Security-Policy middleware tracking valid file execution domains.
* **Limits of Fix:** Content-Security-Policies mitigate code execution risk but cannot prevent severe cross-site interactions or malicious actions if an attacker discovers a way to alter or inject bad files hosted directly on your own domain.
