import mysql.connector as db
from tabulate import tabulate

con = db.connect(user='root', password='sam@1982', host='localhost', database='Django14')
cur = con.cursor()

def owner_access():
    username = input("Enter username: ")
    password = input("Enter password: ")

    if username == 'sam' and password == 'sam@123':
        return True
    else:
        print("Invalid username or password.")
        return False

def get_price(veg_name, quantity):
    cur.execute(f"select price from vegetables where veg_name = '{veg_name}'")
    price_per_kg = cur.fetchone()[0]
    total_cost = price_per_kg * quantity
    return total_cost

print('-' * 75)
print('                     Welcome to the Vegetable Store')
print('-' * 75)
print('                             Available Vegetables')
print('                             *****************')
cur.execute('select veg_id, veg_name, quantity_kg, price from vegetables')
vegetables = cur.fetchall()
print(tabulate(vegetables, headers=['Veg_id', 'Vegetable name', 'Quantity', 'Price']))

total_bill = 0
bill_items = []

while True:
    print('1. Buy Vegetables')
    print('2. Bill Generation')
    print('3. Owner Access')
    print('4. Exit')
    ch = input('Enter your choice: ')

    if ch == '1':
        veg_name = input('Enter vegetable name: ')
        quantity = float(input('Enter quantity (kg): '))
        cur.execute(f"select veg_name, price, quantity_kg from vegetables where veg_name = '{veg_name}'")
        result = cur.fetchone()
        if result:
            veg_name, price_per_kg, available_quantity = result
            if quantity <= available_quantity:
                total_cost = get_price(veg_name, quantity)
                total_bill += total_cost
                remaining_quantity = available_quantity - quantity
                cur.execute(f"update vegetables set quantity_kg = {remaining_quantity} where veg_name = '{veg_name}'")
                con.commit()
                bill_items.append((veg_name, quantity, total_cost))  # Add item to bill items
            else:
                print('Sorry, quantity is not available.')
        else:
            print('Invalid vegetable name. Please enter a valid one.')

    elif ch == '2':
        print('-' * 75)
        print('            Bill Generation')
        print('-' * 75)
        print(tabulate(bill_items, headers=['Vegetable name', 'Quantity', 'Cost']))
        print('-' * 75)
        print(f'Total Bill: Rs.{total_bill}')
        print('-' * 75)
        print('                        THANK YOU\N{smiling face with halo}\N{smiling face with halo}')
        break

    elif ch == '3':
        if owner_access():
            print("Owner Access Granted.")
            # Here is the update query for owner access
            veg_name = input("Enter vegetable name to update/insert: ")
            # Check if the vegetable name is valid
            if any(veg_name in row for row in vegetables):
                new_quantity = float(input("Enter new quantity: "))
                cur.execute(f"update vegetables set quantity_kg = quantity_kg+{new_quantity} where veg_name = '{veg_name}'")
                cur.execute('select veg_id, veg_name, quantity_kg, price from vegetables')
                vegetables = cur.fetchall()
                print(tabulate(vegetables, headers=['Veg_id', 'Vegetable name', 'Quantity', 'Price']))
                con.commit()
                print(f"{veg_name} updated successfully!")
            else:
                insert_choice = input("Vegetable not found in available vegetables. Do you want to add it? (yes/no): ").lower()
                if insert_choice == 'yes':
                    price = float(input("Enter price per kg: "))
                    quantity = float(input("Enter quantity (kg): "))
                    if not vegetables:
                        veg_id = 1
                    else:
                        veg_id = max(row[0] for row in vegetables) + 1  # Increment veg_id
                    cur.execute(f"insert into vegetables (veg_id, veg_name, quantity_kg, price) values ({veg_id}, '{veg_name}', {quantity}, {price})")
                    cur.execute('select veg_id, veg_name, quantity_kg, price from vegetables')
                    vegetables = cur.fetchall()
                    print(tabulate(vegetables, headers=['Veg_id', 'Vegetable name', 'Quantity', 'Price']))
                    con.commit()
                    print(f"{veg_name} added successfully!")
                else:
                    print("Operation canceled.")
        else:
            print("Owner access denied.")

    elif ch == '4':
        print('You are exit from vegetable store')
        print('      ....thank you....\N{smiling face with halo}\N{smiling face with halo}')
        break
    else:
        print('Invalid choice. Please try again.')

cur.close()
con.close()

    
                

