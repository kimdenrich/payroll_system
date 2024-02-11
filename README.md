import os
import datetime

event = {}
employees = {'Denrich': {'type': 'Regular'}, 'Rhoart': {'type': 'Contract'}, 'Lawrence': {'type': 'Regular'}, 'Kurt': {'type': 'Contract'}}
emp_gross = {'Denrich': '', 'Rhoart': '', 'Lawrence': '', 'Kurt': ''} 
emp_total =  {'Denrich': '', 'Rhoart': '', 'Lawrence': '', 'Kurt': ''} 
emp_net_pay =  {'Denrich': '', 'Rhoart': '', 'Lawrence': '','Kurt': ''} 
emp_total_deduct = {'Denrich': '', 'Rhoart': '', 'Lawrence': '', 'Kurt':''} 
payroll_data = {}
day = {}

admin_username = "admin"
admin_password = "admin123"

def clear_console():
    os.system('cls' if os.name == 'nt' else 'clear')

def login():
    clear_console() 
    print("========== WELCOME TO THE PAYROLL SYSTEM ==========")
    while True:
        print("###########################################")
        username = input("Enter admin username: ")
        password = input("Enter admin password: ")
        print("###########################################")
        if username == admin_username and password == admin_password:
            print("Admin login successful.")
            admin_menu()
            break
        else:
            print("Invalid admin credentials. Please try again.")
            



def main():
    if login():
        admin_menu()

def admin_menu():
    clear_console()
    while True:
        print("#############################################")
        print("Command List:                               #")
        print("1. Payroll                                  #")
        print("2. Add Employee                             #")
        print("3. Remove Employee                          #")
        print("4. Employee Information                     #") 
        print("5. Logout                                   #")
        print("#############################################")
        choice = input("Enter your choice (1/2/3/4/5/6): ")

        if choice == "1":
            print("Payroll")
            print("Available Employees:")
            for employee, info in employees.items():
                print(f"{employee} ({info['type']})")
            employee_name = input("Enter employee name for payroll: ")
            if employee_name in employees:
                employee_type = employees[employee_name]['type']
                generate_payroll(employee_name, employee_type)
            else:
                print("Employee not found.")
        elif choice == "2":
            print("Add Employee")
            register_employee()
        elif choice == "3":
            print("Remove Employee")
            remove_employee()
        elif choice == "4":
            print("Employee Information")
            view_employee_info()
        elif choice == "5":
            print("Logout")
            print("Thank you")
            break
        else:
            print("Invalid Number.")


def generate_payroll(employee_name, employee_type):
    clear_console()
    current_date = datetime.date.today()
    start_date = current_date - datetime.timedelta(days=current_date.weekday())  # Start of the current week
    end_date = start_date + datetime.timedelta(days=4)  # End of the current week (Friday)

    print(f"Generating payroll for {employee_name}:")
    total_hours_worked = 0
    days_worked = []  # List to track the days worked by the employee
    for i in range(5):  # Iterate over Monday to Friday
        day = start_date + datetime.timedelta(days=i)
        hours_worked = get_hours_worked(day.strftime("%A"))  # Convert day to string format (e.g., Monday)
        if hours_worked is not None:
            total_hours_worked += hours_worked
            days_worked.append(day)
    
    if days_worked:  # Check if the employee worked any day this week
        calculate_payroll(employee_name, employee_type, days_worked, total_hours_worked)
        print("Payroll generated successfully.")
    else:
        print("Employee was absent from Monday to Friday this week.")


def calculate_payroll(employee_name, employee_type, days_worked, total_hours_worked):
    clear_console()
    # Convert total_hours_worked to float
    total_hours_worked = float(total_hours_worked)
    
    # Calculate gross salary
    hourly_rate = 80  # Example rate per hour
    gross_salary = total_hours_worked * hourly_rate

    # Calculate deductions
    sss_rate = 0.1
    philhealth_rate = 0.0275
    sss_max = 1000
    sss_contribution = min(gross_salary * sss_rate, sss_max)
    philhealth_contribution = gross_salary * philhealth_rate

    # Calculate net pay
    net_pay = gross_salary - (sss_contribution + philhealth_contribution)

    # Update dictionaries with payroll information
    emp_gross[employee_name] = gross_salary
    emp_total_deduct[employee_name] = {'SSS': sss_contribution, 'PhilHealth': philhealth_contribution}  # Updated to dictionary
    emp_net_pay[employee_name] = net_pay

    # Print salary details
    print("Payroll Details:")
    print("Employee Name:", employee_name)
    print("Employee Type:", employee_type)
    print("Days Worked:", ", ".join(day.strftime("%A") for day in days_worked))
    print("Total Hours Worked:", total_hours_worked)
    print("Gross Salary:", gross_salary)
    print("SSS Deduction:", sss_contribution)
    print("PhilHealth Deduction:", philhealth_contribution)
    print("Net Pay:", net_pay)

 

def get_hours_worked(day_of_week):
    clear_console()
    start_time = input(f"Enter start time for {day_of_week} (HH:MM AM/PM) or leave blank if absent: ")
    end_time = input(f"Enter end time for {day_of_week} (HH:MM AM/PM) or leave blank if absent: ")

    if not start_time.strip() or not end_time.strip():
        return None  # Return None if absent

    took_break = input("Did the employee take a break during work? (yes/no): ").lower()  # Prompt for break
    
    # If the employee took a break, ask for the break start and end times
    if took_break == "yes":
        break_start = input(f"Enter break start time for {day_of_week} (HH:MM AM/PM): ")
        break_end = input(f"Enter break end time for {day_of_week} (HH:MM AM/PM): ")

        # Calculate the duration of the break in hours
        break_duration = calculate_break_duration(break_start, break_end)

        # Adjust the total hours worked by subtracting the break duration
        total_hours_worked = calculate_total_hours(start_time, end_time, break_duration)
    else:
        # If no break was taken, calculate the total hours worked without adjustments
        total_hours_worked = calculate_total_hours(start_time, end_time)

    return max(total_hours_worked, 0)


def calculate_break_duration(break_start, break_end):
    # Parse break start and end times into datetime objects
    break_start_time = datetime.datetime.strptime(break_start, "%I:%M %p")
    break_end_time = datetime.datetime.strptime(break_end, "%I:%M %p")

    # Calculate the break duration by subtracting break start time from break end time
    break_duration = break_end_time - break_start_time

    # Convert break duration to hours (float)
    break_hours = break_duration.total_seconds() / 3600

    return max(break_hours, 0) 
    
def calculate_total_hours(start_time, end_time, break_duration=0):
    # Parse start and end times into datetime objects
    start_time_obj = datetime.datetime.strptime(start_time, "%I:%M %p")
    end_time_obj = datetime.datetime.strptime(end_time, "%I:%M %p")

    # Calculate the total work duration by subtracting start time from end time
    total_duration = end_time_obj - start_time_obj

    # Convert total duration to hours (float)
    total_hours = total_duration.total_seconds() / 3600

    # Subtract break duration from total hours worked
    total_hours -= break_duration

    return max(total_hours, 0) 


# Define other functions (register_employee, remove_employee, print_pay, view_employee_info) here...
def register_employee():
    clear_console()
    """
    Registers a new employee by collecting employees name and type.
    Initializes corresponding dictionaries for the new employee.
    """
    employee_name = input("Enter employee name to add: ")
    employee_type = input("Enter employee type (Regular/Contract): ")
    if employee_name in employees:
        print(f"Employee {employee_name} already exists")
    else:
        # Initialize employee dictionaries with default values
        employees[employee_name] = {'type': employee_type}  # Set employee type
        emp_gross[employee_name] = None
        emp_total_deduct[employee_name] = None
        emp_net_pay[employee_name] = None
        print(f"Employee {employee_name} registered successfully.")


def remove_employee():
    clear_console()
    """
    Removes an employee from the system. Prompts for employee names and deletes corresponding entries.
    """
    emp_name = input("Enter employee name to remove: ")
    if emp_name in employees:
        # Remove entries related to the employee
        del employees[emp_name]
        del emp_gross[emp_name]
        del emp_total_deduct[emp_name]
        del emp_net_pay[emp_name]
        print(f"Employee {emp_name} removed successfully.")
    else:
        print("Employee not found.")


def view_employee_info():
    clear_console()
    """
         Displays employees' information, including their name, type, gross pay, deductions, and net pay.
    """
    print("============================")
    print("Employee Information")
    print("============================")
    for employee, info in employees.items():
        print(f"Employee Name: {employee}")
        print(f"Employee Type: {info['type']}")
        print(f"Gross Pay: {emp_gross[employee]} pesos")
        deductions = emp_total_deduct.get(employee)
        if deductions:
            sss_deduction = deductions.get('SSS', 'Not available')
            philhealth_deduction = deductions.get('PhilHealth', 'Not available')
            print(f"SSS Deduction: {sss_deduction} pesos")
            print(f"PhilHealth Deduction: {philhealth_deduction} pesos")
            print(f"Net Pay: {emp_net_pay[employee]} pesos")
        else:
            print("Deductions not available")
            print(f"Net Pay: {emp_net_pay[employee]} pesos")
        print("=" * 30)


if __name__ == "__main__":
    main()

