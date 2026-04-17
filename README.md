# Inventory-Management-System
inventory-management-system Stock management app with low-inventory alerts, category filtering, and automated reporting.
import json
import os
from dataclasses import dataclass
from typing import List, Dict


# =========================================
# CONFIGURATION
# =========================================

DATA_FILE = "inventory.json"


# =========================================
# MODEL LAYER
# =========================================

@dataclass
class Product:
    id: int
    name: str
    price: float
    stock: int
    low_stock_threshold: int

    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "price": self.price,
            "stock": self.stock,
            "low_stock_threshold": self.low_stock_threshold
        }

    @staticmethod
    def from_dict(data):
        return Product(
            id=data["id"],
            name=data["name"],
            price=data["price"],
            stock=data["stock"],
            low_stock_threshold=data["low_stock_threshold"]
        )


# =========================================
# REPOSITORY LAYER (Persistence)
# =========================================

def load_inventory() -> List[Product]:
    if not os.path.exists(DATA_FILE):
        return []

    with open(DATA_FILE, "r", encoding="utf-8") as f:
        data = json.load(f)
        return [Product.from_dict(p) for p in data]


def save_inventory(products: List[Product]):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump([p.to_dict() for p in products], f, indent=4)


# =========================================
# SERVICE LAYER (Business Logic)
# =========================================

def generate_id(products: List[Product]) -> int:
    return max([p.id for p in products], default=0) + 1


def add_product(products: List[Product]):
    name = input("Product name: ")
    price = float(input("Price: "))
    stock = int(input("Stock: "))
    threshold = int(input("Low stock threshold: "))

    product = Product(
        id=generate_id(products),
        name=name,
        price=price,
        stock=stock,
        low_stock_threshold=threshold
    )

    products.append(product)


def remove_product(products: List[Product]):
    pid = int(input("Product ID: "))
    products[:] = [p for p in products if p.id != pid]


def update_product(products: List[Product]):
    pid = int(input("Product ID: "))

    for p in products:
        if p.id == pid:
            new_name = input("New name (empty skip): ")
            if new_name:
                p.name = new_name

            new_price = input("New price (empty skip): ")
            if new_price:
                p.price = float(new_price)

            new_stock = input("New stock (empty skip): ")
            if new_stock:
                p.stock = int(new_stock)

            new_threshold = input("New threshold (empty skip): ")
            if new_threshold:
                p.low_stock_threshold = int(new_threshold)


def increase_stock(products: List[Product]):
    pid = int(input("Product ID: "))
    amount = int(input("Increase amount: "))

    for p in products:
        if p.id == pid:
            p.stock += amount


def decrease_stock(products: List[Product]):
    pid = int(input("Product ID: "))
    amount = int(input("Decrease amount: "))

    for p in products:
        if p.id == pid:
            p.stock = max(0, p.stock - amount)


# =========================================
# ALERT ENGINE (Core e-commerce logic)
# =========================================

def low_stock_alerts(products: List[Product]):
    alerts = []

    for p in products:
        if p.stock <= p.low_stock_threshold:
            alerts.append(p)

    return alerts


# =========================================
# REPORTING LAYER
# =========================================

def inventory_report(products: List[Product]):
    total_products = len(products)
    total_value = sum(p.price * p.stock for p in products)
    low_stock = low_stock_alerts(products)

    print("\n========== INVENTORY REPORT ==========\n")
    print(f"Total products: {total_products}")
    print(f"Total inventory value: {total_value:.2f}")
    print(f"Low stock items: {len(low_stock)}")

    if low_stock:
        print("\n--- LOW STOCK ALERTS ---")
        for p in low_stock:
            print(f"[!] {p.name} (Stock: {p.stock}, Threshold: {p.low_stock_threshold})")


def list_products(products: List[Product]):
    for p in products:
        print("-" * 40)
        print(f"ID: {p.id}")
        print(f"Name: {p.name}")
        print(f"Price: {p.price}")
        print(f"Stock: {p.stock}")
        print(f"Threshold: {p.low_stock_threshold}")


# =========================================
# CLI LAYER
# =========================================

def menu():
    print("""
1. Add product
2. Remove product
3. Update product
4. Increase stock
5. Decrease stock
6. List products
7. Inventory report
8. Save & exit
""")


def main():
    products = load_inventory()

    while True:
        menu()
        choice = input("Choice: ")

        if choice == "1":
            add_product(products)

        elif choice == "2":
            remove_product(products)

        elif choice == "3":
            update_product(products)

        elif choice == "4":
            increase_stock(products)

        elif choice == "5":
            decrease_stock(products)

        elif choice == "6":
            list_products(products)

        elif choice == "7":
            inventory_report(products)

        elif choice == "8":
            save_inventory(products)
            break


if __name__ == "__main__":
    main()
