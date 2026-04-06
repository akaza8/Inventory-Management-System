# StockFlow: B2B Inventory Management System
### Backend Engineering Intern Case Study Solution

This repository contains my solution to the "StockFlow" Backend Engineering Intern case study. It demonstrates my proficiency in **Java, Spring Boot, and Relational Database Design** while addressing critical issues found in legacy Python code.

---

## 1. Part 1 – Code Review and Debugging
The initial API for product creation contained logic flaws that would cause data corruption.

### Identified Issues
* **Lack of Transaction Management:** Due to two separate commits, there was inconsistency in the state (e.g., product created but inventory failed, leading to a system break)
* **No Enforcement of SKU Uniqueness:** SKU must be unique for duplicate product handling
* **No Validation:** Validations were missing, including the risks of null values, negative quantities, and invalid prices
* **Minor and Major** price handling and product-warehouse relationship
### The Fix
I refactored the logic to use a single transaction block with `db.session.flush()` to securely obtain the product ID before a final `commit()`, ensuring **atomicity**. I also added validations, solved product-warehouse conflicts, and handled price precision.


### Code 
````
@app.route('/api/products', methods=['POST'])
def create_product():
    data = request.json
    required_fields = ['name', 'sku', 'price', 'warehouse_id', 'initial_quantity']
    for field in required_fields:
        if field not in data:
            return {"error": f"{field} is required"}, 400

    try:
        existing = Product.query.filter_by(sku=data['sku']).first()
        if existing:
            return {"error": "SKU already exists"}, 409

        product = Product(
            name=data['name'],
            sku=data['sku'],
            price=Decimal(str(data['price']))
        )

        db.session.add(product)
        db.session.flush()  # get product.id without commit

        inventory = Inventory(
            product_id=product.id,
            warehouse_id=data['warehouse_id'],
            quantity=data['initial_quantity']
        )

        db.session.add(inventory)
        db.session.commit()

        return {"message": "Product created", "product_id": product.id}, 201

    except Exception as e:
        db.session.rollback()
        return {"error": str(e)}, 500
 ````

---

## 2. Part 2 – Database Design
Designed to support multi-warehouse tracking, product bundling, and supplier relationships.



### Questions for the Product Team
* [cite_start]Is bundle stock calculated dynamically from components or tracked physically[cite: 46]?
* [cite_start]Are SKUs unique globally or per company[cite: 37]?
* [cite_start]Can low-stock thresholds be customized per specific warehouse[cite: 54]?

### Design Decisions
* [cite_start]**Composite Primary Key:** Used `(warehouse_id, product_id)` in the `inventory` table to ensure data integrity[cite: 43].
* [cite_start]**Audit Logging:** Included an `inventory_transactions` table to track every change in stock levels[cite: 44].

---

## 3. Part 3 – API Implementation
Implemented using **Java 17 and Spring Boot** to demonstrate production-grade architecture.

### Endpoint Specification
* [cite_start]**Path:** `GET /api/companies/{company_id}/alerts/low-stock` [cite: 57]
* [cite_start]**Logic:** Filters for products below their threshold with recent sales activity [cite: 54-55].

### Handled Edge Cases
* **Zero Recent Activity:** Prevents alert fatigue by ignoring "dead" stock.
* [cite_start]**Multi-Warehouse:** Stock is tracked and alerted per specific warehouse[cite: 55].
* [cite_start]**Supplier Integration:** Includes reordering contact details [cite: 71-75].

---

## Setup & Running
1. **Clone:** `git clone https://github.com/akaza8/stockflow-backend-case-study.git`
2. **Open:** Import into **IntelliJ IDEA** as a Maven project.
3. **Run:** Execute `StockFlowApplication.java`.