# 🍸 Automated API Testing Suite – TheCocktailDB (Postman + Newman)

This project is a fully automated API testing suite for **TheCocktailDB**, built using **Postman**, executed with **Newman**, and integrated with **GitHub Actions CI**.

It serves both as a **professional portfolio project** and a real-world **regression test suite** for applications that consume CocktailDB (including my *Cocktail Finder UI* project).

---

## ✨ What This Project Demonstrates

This project showcases strong API testing and automation skills, including:

### 🔍 **Test Design & Coverage**
- Full coverage of **10 endpoints**, including:
  - Search  
  - Lookups  
  - Filters  
  - Random cocktail  
  - Invalid & edge-case scenarios
- **Data-driven testing** using iteration CSV files
- **Positive & negative test flows**
- **Edge case handling**, including empty search terms, invalid IDs, URL-encoded names, uppercase variations

### 🧪 **Automated Postman Test Logic**
- Functional validations  
- Negative-case validations  
- **Schema-style JSON validation** for multiple endpoints  
- Case-insensitive name matching  
- Null/empty array checks  
- Special handling for **API quirks** (e.g., empty search returning a full list)

### ⚡ **Performance Testing**
- Response time assertions (< **800ms**)

### 🔁 **Continuous Integration**
- Execution using **Newman**
- Automatic HTML reports with `newman-reporter-htmlextra`
- Fully automated CI pipeline via **GitHub Actions**

---

## 🧱 Tech Stack

- **Postman** – Request definitions & JS tests  
- **Newman** – CLI test runner  
- **newman-reporter-htmlextra** – HTML reporting  
- **JavaScript (ChaiJS)** – Assertion library within Postman  
- **Node.js + npm** – Dependency management  
- **GitHub Actions** – CI integration  

---

## 📂 Project Structure

```
api-testing-postman-newman/
├─ postman/
│  ├─ collections/
│  │  └─ cocktail-api-testing-collection.json
│  └─ environments/
│     └─ thecocktaildb-live.postman_environment.json
├─ data/
│  └─ search-data.csv
├─ reports/
│  └─ cocktail-api-report.html
├─ .github/
│  └─ workflows/
│     └─ api-tests.yml
├─ package.json
├─ package-lock.json
└─ README.md
```

---

## 🌐 API Under Test

All requests target:

```
https://www.thecocktaildb.com/api/json/v1/1
```

This is stored in the Postman environment as:

```
baseUrl = https://www.thecocktaildb.com/api/json/v1/1
```

---

# 🧪 Endpoints & Test Scenarios

This suite covers **10 endpoints**, ranging from simple searches to complex validations.

---

## 1️⃣ Search Cocktails by Name (Data-Driven)

```
GET /search.php?s={{name}}
```

Driven by: `data/search-data.csv`

Example CSV:

```
name,expectResults
margarita,true
Bloody Mary,true
sdjasdjhasdjk,false
,false
MARGARITA,true
```

### Tests include:

- Status code = **200**
- Response time < **800ms**
- JSON body exists
- If `expectResults = true`:
  - `drinks` is a **non-empty array**
  - At least one drink name contains the search term (case-insensitive)
  - **Schema validation**:
    - `idDrink` (string, required)
    - `strDrink` (string, required)
    - `strDrinkThumb`
    - `strInstructions`
    - `strGlass`
- If `expectResults = false`:
  - `drinks` is `null` OR an empty array  
  - **Exception:**  
    - When `name` is empty (`""`), TheCocktailDB returns a **full list of cocktails**, not “no results”
    - This suite *documents this real behavior* and intentionally **skips strict validation** for this case

---

## 2️⃣ Get Cocktail by ID

```
GET /lookup.php?i=11007
```

### Tests:

- Status code = **200**
- JSON response
- `drinks` array exists and has one element
- The drink has ID `"11007"`
- **Schema validation** for:
  - `idDrink`
  - `strDrink`
  - `strInstructions`
  - `strDrinkThumb`
  - `strGlass`

---

## 3️⃣ Search with Invalid Cocktail Name (Negative)

```
GET /search.php?s=sdjasdjhasdjk
```

### Tests:

- Status code = **200**
- `drinks` is `null` or empty

---

## 4️⃣ Filter Cocktails by Category

```
GET /filter.php?c=Cocktail
```

### Tests:

- Status code = **200**
- Response time < **800ms**
- Response is JSON
- `drinks` array exists
- **Schema validation** for:
  - `idDrink`
  - `strDrink`
  - `strDrinkThumb`

---

## 5️⃣ Filter Cocktails by Ingredient (Vodka)

```
GET /filter.php?i=Vodka
```

### Tests mirror the Category filter:

- 200 OK  
- JSON  
- Performance  
- Array exists  
- **Schema validation** on each drink object  

---

## 6️⃣ Get Random Cocktail

```
GET /random.php
```

### Tests:

- Status code = **200**
- JSON response
- `drinks` array exists
- **Schema validation** for full drink objects

---

## 7️⃣ Get Cocktail by Invalid ID (Negative)

```
GET /lookup.php?i=9999999
```

### Tests:

- Status code = **200**
- `drinks` is `null` or empty

---

## 8️⃣ Search with Empty Name (API Quirk)

```
GET /search.php?s=
```

### Actual API Behavior:
> TheCocktailDB returns a **full list of drinks**, not “no results.”

### Tests:

- Status code = **200**
- Tests explicitly **document this behavior**
- Negative-case strict assertions are skipped  
  (to avoid false failures and accurately reflect real API behavior)

---

## 9️⃣ Search with Uppercase Name

```
GET /search.php?s=MARGARITA
```

### Tests:

- 200 OK  
- Case-insensitive name matching  
- Schema-style validation  

---

## 🔟 Search with Spaced Name (URL Encoding)

```
GET /search.php?s=Bloody%20Mary
```

### Tests:

- 200 OK  
- Array not empty  
- One result contains “Bloody Mary”  
- Schema-style validation  

---

## 📊 Suite Summary

- **393 assertions** executed  
- **100% passing**  
- Comprehensive schema-style checks  
- Full support for positive, negative, and edge-case testing  
- Validated real-world quirks of the API  
- Suitable for CI, regression testing, and portfolio demonstration  

---

# ▶️ Running the Suite with Newman (Locally)

### 1. Install dependencies

```
npm install
```

### 2. Run all tests

```
npm run test:api
```

### 3. Under-the-hood Newman command

```
newman run postman/collections/cocktail-api-testing-collection.json \
  -e postman/environments/thecocktaildb-live.postman_environment.json \
  --iteration-data data/search-data.csv \
  -r cli,htmlextra \
  --reporter-htmlextra-export reports/cocktail-api-report.html
```

---

# 🤖 GitHub Actions CI

GitHub Actions workflow:  
```
.github/workflows/api-tests.yml
```

### The pipeline:

1. Install Node.js  
2. Install Newman + HTML reporter  
3. Run the Postman collection  
4. Use the CSV file for data-driven tests  
5. Attach the HTML report as an artifact  
6. Fail the pipeline if any assertion fails

This simulates a **professional CI-ready API regression suite**.

---

## 📊 HTML Report

Newman generates:

```
reports/cocktail-api-report.html
```

Includes:

- Per-endpoint breakdown  
- Performance charts  
- Iteration-by-iteration results  
- Schema validation breakdown  
- Screenshots-ready summary  

---

## 🔗 Related Project

### **Cocktail Finder UI**  
A front-end application that consumes this API.  
This suite can be used as its automated regression safety net.

https://heathergauthier2018.github.io/cocktail-finder2.0/

---

# 👩‍💻 Author

**Heather Gauthier**  
API Testing • QA Automation • Software Engineering Student  

GitHub: [@heathergauthier2018](https://github.com/heathergauthier2018)
