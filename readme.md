# 🕵️‍♂️ Auntie Som's Noodle Shop - API Audit Report 6733289421

## 📝 Mission Summary
During the automated API audit using Bruno, several critical business logic vulnerabilities and security flaws were discovered in Nephew Lek's system.

## 🐛 Discovered Bugs

### 1. 💸 Incorrect Price Calculation
**Endpoint:** `POST /orders`
**Description:** The API calculates the `totalPrice` incorrectly, seemingly swapping the prices of the items.
* **Evidence:** * Ordering Item ID `1` (Actual price: 50) calculates the total using 45. (Bruno output: `Bug Found! ID:1 Price Should 50 But Get 45`)
  * Ordering Item ID `2` (Actual price: 45) calculates the total using 50. (Bruno output: `Bug Found! ID:2 Price Should 45 But Get 50`)

### 2. 📦 Flawed Stock Limit Validation
**Endpoint:** `POST /orders`
**Description:** The system allows users to order a quantity that exceeds the available stock, but this bug only triggers if the initial stock is greater than 0. If the stock is already less than 0, the API correctly returns an "out of stock" response. However, if stock > 0, the system fails to prevent ordering more than what is available.

### 3. ➖ Negative Quantity Allowed
**Endpoint:** `POST /orders`
**Description:** The API does not validate if the `quantity` is a positive number. Users can successfully place orders with negative quantities.

### 4. 🔓 Insecure Direct Object Reference (IDOR)
**Endpoint:** `GET /orders/<orderId>`
**Description:** The system fails to verify ownership of an order. An authenticated user can pass any valid `orderId` that does not belong to them and successfully view the details of that order.

---

## 🛠️ How the Tests Detect These Bugs
To catch these silent bugs, the Bruno collection utilizes Javascript assertions in the **Tests** tab rather than just checking for a `200 OK` status:
* **Dynamic Price Checking:** The script dynamically stores the correct price from `GET /menu`, calculates `Expected Price = Price * Quantity`, and strictly asserts `expect(data.totalPrice).to.equal(expectedPrice)`.
* **Edge Case Testing:** Tests were specifically written to expect failures (`expect(res.getStatus()).to.not.equal(200)`) when ordering quantities greater than available stock or using negative numbers.
* **Authorization Checks:** The test suite verifies that requesting someone else's order ID should not return a `200 OK`, catching the IDOR vulnerability.