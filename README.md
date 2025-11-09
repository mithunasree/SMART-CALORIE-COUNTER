# SMART-CALORIE-COUNTER
The Calorie Counter application helps users track daily calorie intake. Users create accounts, log in, add food items, record meals with quantities, and calculate daily calorie needs using BMR and activity level. It shows food history and remaining calories. All data is saved in JSON for easy access and storage.





import json
from datetime import datetime

class CalorieCounter:
    def _init_(self, food_data_file="food_data.json", user_data_file="user_data.json"):
        self.food_data_file = food_data_file
        self.user_data_file = user_data_file
        self.food_data = self.load_data(self.food_data_file)
        self.user_data = self.load_data(self.user_data_file)
        self.current_user = None

    def load_data(self, file_path):
        try:
            with open(file_path, 'r') as file:
                return json.load(file)
        except FileNotFoundError:
            return {}

    def save_data(self, file_path, data):
        with open(file_path, 'w') as file:
            json.dump(data, file, indent=4)

    def create_user(self, username, age, weight, height, activity_level):
        if username in self.user_data:
            print("Username already exists.")
            return False
        self.user_data[username] = {
            "age": age,
            "weight": weight,
            "height": height,
            "activity_level": activity_level,
            "daily_calories": 0,
            "food_log": []
        }
        self.save_data(self.user_data_file, self.user_data)
        print(f"User {username} created successfully.")
        return True

    def login(self, username):
        if username not in self.user_data:
            print("Invalid username.")
            return False
        self.current_user = username
        print(f"Logged in as {username}.")
        return True

    def calculate_bmr(self):
        user = self.user_data[self.current_user]
        age, weight, height = user["age"], user["weight"], user["height"]
        bmr = (10 * weight) + (6.25 * height) - (5 * age) + 5  # Mifflin-St Jeor equation
        return bmr

    def calculate_daily_calories(self):
        user = self.user_data[self.current_user]
        activity_level = user["activity_level"]
        bmr = self.calculate_bmr()
        activity_factors = {
            "sedentary": 1.2,
            "light": 1.375,
            "moderate": 1.55,
            "active": 1.725,
            "very active": 1.9
        }
        daily_calories = bmr * activity_factors.get(activity_level, 1.2)
        self.user_data[self.current_user]["daily_calories"] = daily_calories
        self.save_data(self.user_data_file, self.user_data)
        return daily_calories

    def add_food_item(self, food_name, calories):
        self.food_data[food_name] = calories
        self.save_data(self.food_data_file, self.food_data)
        print(f"{food_name} added to food database.")

    def log_food(self, food_name, quantity=1):
        if self.current_user is None:
            print("Please log in first.")
            return False
        if food_name not in self.food_data:
            print(f"{food_name} not found in database. Please add it first.")
            return False
        calories = self.food_data[food_name] * quantity
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        self.user_data[self.current_user]["food_log"].append({
            "food_name": food_name,
            "calories": calories,
            "quantity": quantity,
            "timestamp": timestamp
        })
        self.save_data(self.user_data_file, self.user_data)
        print(f"{quantity} {food_name}(s) logged successfully.")
        return True

    def view_food_log(self):
        if self.current_user is None:
            print("Please log in first.")
            return
        food_log = self.user_data[self.current_user]["food_log"]
        if not food_log:
            print("No food logged yet.")
            return
        print("----- Food Log -----")
        total_calories = 0
        for entry in food_log:
            print(f"{entry['timestamp']}: {entry['quantity']} {entry['food_name']} - {entry['calories']} calories")
            total_calories += entry['calories']
        print(f"----- Total Calories: {total_calories} -----")

    def get_remaining_calories(self):
        if self.current_user is None:
            print("Please log in first.")
            return 0
        daily_calories = self.user_data[self.current_user]["daily_calories"]
        consumed_calories = sum(entry["calories"] for entry in self.user_data[self.current_user]["food_log"])
        return daily_calories - consumed_calories

    def clear_food_log(self):
        if self.current_user is None:
            print("Please log in first.")
            return
        self.user_data[self.current_user]["food_log"] = []
        self.save_data(self.user_data_file, self.user_data)
        print("Food log cleared.")


if _name_ == "_main_":
    calorie_counter = CalorieCounter()

    while True:
        print("\n--- Calorie Counter Menu ---")
        print("1. Create User")
        print("2. Login")
        print("3. Calculate Daily Calories")
        print("4. Add Food Item to Database")
        print("5. Log Food")
        print("6. View Food Log")
        print("7. Get Remaining Calories")
        print("8. Clear Food Log")
        print("9. Exit")

        choice = input("Enter your choice: ")

        if choice == "1":
            username = input("Enter username: ")
            age = int(input("Enter age: "))
            weight = float(input("Enter weight (kg): "))
            height = float(input("Enter height (cm): "))
            activity_level = input("Enter activity level (sedentary, light, moderate, active, very active): ").lower()
            calorie_counter.create_user(username, age, weight, height, activity_level)

        elif choice == "2":
            username = input("Enter username: ")
            calorie_counter.login(username)

        elif choice == "3":
            if calorie_counter.current_user:
                daily_calories = calorie_counter.calculate_daily_calories()
                print(f"Your recommended daily calorie intake is: {daily_calories:.2f} calories")
            else:
                print("Please log in first.")

        elif choice == "4":
            food_name = input("Enter food name: ")
            calories = float(input("Enter calories per serving: "))
            calorie_counter.add_food_item(food_name, calories)

        elif choice == "5":
            if calorie_counter.current_user:
                food_name = input("Enter food name to log: ")
                quantity = int(input("Enter quantity: "))
                calorie_counter.log_food(food_name, quantity)
            else:
                print("Please log in first.")

        elif choice == "6":
            calorie_counter.view_food_log()

        elif choice == "7":
            if calorie_counter.current_user:
                remaining_calories = calorie_counter.get_remaining_calories()
                print(f"Remaining calories for today: {remaining_calories:.2f}")
            else:
                print("Please log in first.")

        elif choice == "8":
            calorie_counter.clear_food_log()

        elif choice == "9":
            print("Exiting Calorie Counter.")
            break

        else:
            print("Invalid choice. Please try again.")
