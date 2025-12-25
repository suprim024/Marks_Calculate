# Marks_Calculate
from datetime import datetime
from tabulate import tabulate

def calculate_overall_score(grades, weights):
    if len(grades) != len(weights):
        raise ValueError("Grades and weights must have the same length")
    overall = 0
    for i in range(len(grades)):
        overall += grades[i] * weights[i]
    return overall

def determine_mark(raw_score):
    marks = [100, 92, 85, 82, 78, 75, 72, 68, 65, 62, 58, 55, 52, 48, 45, 42, 38, 35, 32, 25, 15, 5, 0]
    return min(marks, key=lambda x: abs(x - raw_score))

def determine_category(score):
    nearest_mark = determine_mark(score)
    if nearest_mark == 100:
        return "Gold Standard"
    if nearest_mark in [92, 85, 82]:
        return "Upper First"
    if nearest_mark in [78, 75, 72]:
        return "First"
    if nearest_mark in [68, 65, 62]:
        return "Upper Second"
    if nearest_mark in [58, 55, 52]:
        return "Lower Second"
    if nearest_mark in [48, 45, 42]:
        return "Third"
    if nearest_mark in [38, 35, 32]:
        return "Condonable Fail"
    if nearest_mark in [25, 15, 5]:
        return "Fail"
    return "No Submission"

def round_to_category(score):
    marks = [100, 92, 85, 82, 78, 75, 72, 68, 65, 62, 58, 55, 52, 48, 45, 42, 38, 35, 32, 25, 15, 5, 0]
    closest = min(marks, key=lambda x: abs(x - score))
    ties = [m for m in marks if abs(m - score) == abs(closest - score)]
    nearest = max(ties)
    return nearest, determine_category(nearest)

def calculate_age(dob):
    birth = datetime.strptime(dob, "%Y-%m-%d")
    today = datetime.today()
    return today.year - birth.year - ((today.month, today.day) < (birth.month, birth.day))

def main():
    headers = ["UID", "Name", "D.o.B", "Age", "Raw Score", "Rounded Score", "Category"]
    table_data = []

    choice2 = input("Press n for fixed module OR y for custom module: ").lower()

    if choice2 == "n":
        choice1 = input("Press 1 to type manually OR 2 to import data from students.txt: ")

        if choice1 == "2":
            try:
                with open("students.txt", "r") as file:
                    for line in file:
                        uid, name, dob, c1, c2, c3, fe = line.strip().split(",")
                        age = calculate_age(dob)
                        grades = [float(c1), float(c2), float(c3), float(fe)]
                        weights = [0.10, 0.20, 0.30, 0.40]
                        raw = calculate_overall_score(grades, weights)
                        mark, cat = round_to_category(raw)
                        table_data.append([uid, name, dob, age, round(raw, 4), mark, cat])
            except FileNotFoundError:
                print("students.txt not found")
                return

        elif choice1 == "1":
            student_count = 0
            while student_count < 3:
                student_id = input("Please enter the student_id (or 'end' to stop): ")
                student_int = int(student_id)
                if student_id.lower() == "end":
                    break
                if student_int<1 or student_int>99:
                    print("Invalid student ID. Please enter a number between 1 and 99.")
                    continue

                name = input("Please enter the name of the student: ")
                dob = input("Enter date of birth (YYYY-MM-DD): ")
                age = calculate_age(dob)

                coursework_1 = float(input("Please enter the score of coursework-1 (0-100): "))
                coursework_2 = float(input("Please enter the score of coursework-2 (0-100): "))
                coursework_3 = float(input("Please enter the score of coursework-3 (0-100): "))
                final_exam = float(input("Please enter the score of final_exam (0-100): "))
                if not all(0 <= score <= 100 for score in [coursework_1, coursework_2, coursework_3, final_exam]):
                    print("Invalid score entered. Please enter scores between 0 and 100.")
                    continue
                grades = [coursework_1, coursework_2, coursework_3, final_exam]
                weights = [0.10, 0.20, 0.30, 0.40]

                raw_score = calculate_overall_score(grades, weights)
                nearest_mark, category = round_to_category(raw_score)

                table_data.append([
                    student_id,
                    name,
                    dob,
                    age,
                    round(raw_score, 4),
                    nearest_mark,
                    category
                ])

                student_count += 1

        if table_data:
            print(tabulate(table_data, headers=headers))

    elif choice2 == "y":
        module_name = input("Enter module name: ")
        component_count = int(input("How many assessment components does this module have? "))

        components = []
        weights = []

        for i in range(component_count):
            cname = input(f"Enter component {i+1} name: ")
            weight = float(input(f"Enter component {i+1} weight (%): "))
            components.append(cname)
            weights.append(weight / 100)
        print("1")
        while True:
            student_id = input("Please enter the student_id (or 'end' to stop): ")
            if student_id.lower() == "end":
                break

            name = input("Please enter the name of the student: ")
            dob = input("Enter date of birth (YYYY-MM-DD): ")
            age = calculate_age(dob)

            grades = []
            for comp in components:
                score = float(input(f"Enter score for {comp} (0-100): "))
                grades.append(score)

            category = determine_category(score)
            table_data.append([
                student_id,
                name,
                dob,
                age,
                round(raw_score, 4),
                nearest_mark,
                category
            ])

        if table_data:
            print(tabulate(table_data, headers=headers))

if __name__ == "__main__":
    main()
