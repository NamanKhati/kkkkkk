# kkkkkk
import mysql.connector

def create_connection():
    connection = mysql.connector.connect(
        host="localhost",
        user="root",
        password="12345",
        database="gst_invoice"
    )
    return connection

def create_invoice():
    print("=== Create Invoice ===")
    business_name = input("Enter Business Name: ")
    business_address = input("Enter Business Address: ")
    business_gstin = input("Enter Business GSTIN: ")

    customer_name = input("Enter Customer Name: ")
    customer_address = input("Enter Customer Address: ")
    customer_gstin = input("Enter Customer GSTIN: ")

    items = []
    total_amount = 0

    print("Enter items (type 'done' when finished):")
    while True:
        item_name = input("Item Name: ")
        if item_name.lower() == 'done':
            break
        quantity = float(input("Quantity: "))
        price = float(input("Price per unit: ₹"))
        gst_rate = float(input("GST Rate (%): "))
        
        subtotal = quantity * price
        gst_amount = subtotal * gst_rate / 100
        total = subtotal + gst_amount
        total_amount += total
        
        items.append(f"{item_name} x{quantity} @₹{price} GST {gst_rate}% = ₹{total:.2f}")

    items_text = "\n".join(items)

    conn = create_connection()
    cursor = conn.cursor()

    sql = """INSERT INTO invoices
             (business_name, business_address, business_gstin,
              customer_name, customer_address, customer_gstin,
              items, total_amount)
             VALUES (%s, %s, %s, %s, %s, %s, %s, %s)"""
    val = (business_name, business_address, business_gstin,
           customer_name, customer_address, customer_gstin,
           items_text, total_amount)

    cursor.execute(sql, val)
    conn.commit()

    print(f"\nInvoice saved with ID {cursor.lastrowid}!")
    cursor.close()
    conn.close()

def view_invoices():
    conn = create_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT id, business_name, customer_name, total_amount FROM invoices")
    rows = cursor.fetchall()

    print("\n=== Saved Invoices ===")
    for row in rows:
        print(f"ID: {row[0]}, Business: {row[1]}, Customer: {row[2]}, Total: ₹{row[3]:.2f}")
    
    choice = input("\nEnter invoice ID to view details (or press Enter to go back): ")
    if choice:
        try:
            invoice_id = int(choice)
            cursor.execute("SELECT * FROM invoices WHERE id = %s", (invoice_id,))
            invoice = cursor.fetchone()
            if invoice:
                print("\n=== Invoice Details ===")
                print(f"ID: {invoice[0]}")
                print(f"Business Name: {invoice[1]}")
                print(f"Business Address: {invoice[2]}")
                print(f"Business GSTIN: {invoice[3]}")
                print(f"Customer Name: {invoice[4]}")
                print(f"Customer Address: {invoice[5]}")
                print(f"Customer GSTIN: {invoice[6]}")
                print(f"Items:\n{invoice[7]}")
                print(f"Total Amount: ₹{invoice[8]:.2f}")
            else:
                print("Invoice not found!")
        except ValueError:
            print("Invalid input!")
    
    cursor.close()
    conn.close()

def main():
    print("=== GST Invoice Generator with MySQL ===")
    while True:
        print("\n1. Create Invoice")
        print("2. View Invoices")
        print("3. Exit")
        choice = input("Enter your choice: ")
        if choice == "1":
            create_invoice()
        elif choice == "2":
            view_invoices()
        elif choice == "3":
            print("Goodbye!")
            break
        else:
            print("Invalid choice!")

if __name__ == "__main__":
    main()
    







my sql 
CREATE DATABASE IF NOT EXISTS gst_invoice;
 CREATE TABLE IF NOT EXISTS invoices (
     id INT AUTO_INCREMENT PRIMARY KEY,
     business_name VARCHAR(100),
     business_address VARCHAR(255),
     business_gstin VARCHAR(20),
     customer_name VARCHAR(100),
     customer_address VARCHAR(255),
     customer_gstin VARCHAR(20),
     items TEXT,
     total_amount DECIMAL(10, 2)
    );


    
