# Mini Project: Inventory Tracker

## Persistent storage

```text
inventory.json
```

## Features

```text
1. Add Product
2. View Products
3. Update Product
4. Delete Product
5. Add Stock
6. Remove Stock
7. Inventory Summary
8. Exit
```

### Product Information

Each product should contain:

* ID
* Name
* Category
* Price
* Quantity

### Add Product

Add a new product to the inventory.

Example:

```text
Name: Keyboard
Category: Electronics
Price: 1200
Quantity: 10
```

### View Products

Display all products.

```text
ID    Name       Category       Price    Quantity
1     Keyboard   Electronics    ₹1200    10
2     Mouse      Electronics    ₹600     20
3     Notebook   Stationery     ₹80      50
```

### Update Product

Update product details using the product ID.

The user should be able to update:

* Name
* Category
* Price
* Quantity

### Delete Product

Delete a product using its ID.

### Add Stock

Increase the quantity of an existing product.

```text
Enter Product ID: 1
Enter quantity to add: 5

Stock updated.
New quantity: 15
```

### Remove Stock

Decrease the quantity of an existing product.

Do not allow the quantity to become negative.

```text
Enter Product ID: 1
Enter quantity to remove: 20

Not enough stock.
```

### Inventory Summary

Display:

```text
Total Products: 3
Total Items: 80
Total Inventory Value: ₹24,000
```

Inventory value:

```text
price × quantity
```

## Requirements

Students must use:

* Functions
* Modules
* Lists / Dictionaries
* Loops
* Conditions
* Exception handling
* File handling
* JSON

## Persistence

The program should:

* Load products from `inventory.json` when it starts.
* Save changes after adding, updating, deleting, or changing stock.
* Preserve the inventory when the program is restarted.

## Suggested Structure

```text
inventory-tracker/
│
├── main.py
├── inventory.py
├── file_manager.py
└── inventory.json
```

## Focus

The main focus is to practice **CRUD operations, functions, modules, JSON, file handling, validation, and calculations** while managing changing quantities.


## Evaluation Focus

Evaluate the student's ability to:

* Structure and manipulate data using dictionaries/lists
* Break the program into clean, reusable functions
* Use function arguments appropriately
* Apply loops and conditions correctly
* Use comprehensions where appropriate
* Handle missing contacts and invalid operations
* Keep code readable and organized

## Restrictions

* No AI models
* No ChatGPT/Copilot
* No coding agents
* No hard-coded operation results

## Submission

Create a GitHub repository named:

`fullstack-gwt-week2`

Push the completed project to the `main` branch.

## Expected Outcome

The application should allow a user to add, delete, search, update, and list contacts through the CLI.

All the best! 🚀
