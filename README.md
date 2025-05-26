# Vibe-Coding-Hackathon
Tracking income and expenses at real time for business owners

import speech_recognition as sr
import pytesseract
import cv2
from datetime import datetime

# Optional: path to tesseract executable
# pytesseract.pytesseract.tesseract_cmd = r'/usr/bin/tesseract'  # Modify as needed

# Storage for transactions
transactions = []

# Function to process voice input
def record_voice_transaction():
    recognizer = sr.Recognizer()
    with sr.Microphone() as source:
        print("Speak now (e.g., 'Sold shoes for 50 dollars')...")
        audio = recognizer.listen(source)

    try:
        text = recognizer.recognize_google(audio)
        print("You said:", text)
        parse_transaction(text)
    except sr.UnknownValueError:
        print("Could not understand audio.")
    except sr.RequestError:
        print("API unavailable or too many requests.")

# Function to process photo input
def process_photo_transaction(image_path):
    image = cv2.imread(image_path)
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    text = pytesseract.image_to_string(gray)
    print("Extracted text:", text)
    parse_transaction(text)

# Simple parser to detect income/expense
def parse_transaction(text):
    lower = text.lower()
    amount = extract_amount(text)
    
    if "sell" in lower or "sold" in lower or "income" in lower:
        t_type = "income"
    elif "buy" in lower or "bought" in lower or "expense" in lower or "paid" in lower:
        t_type = "expense"
    else:
        t_type = "unknown"

    if amount is not None and t_type != "unknown":
        transactions.append({
            "type": t_type,
            "amount": amount,
            "description": text,
            "date": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        })
        print(f"{t_type.capitalize()} of ${amount} recorded.")
    else:
        print("Could not determine transaction type or amount.")

# Extracts numeric amount from text
def extract_amount(text):
    import re
    match = re.search(r'\d+(?:\.\d{1,2})?', text)
    return float(match.group()) if match else None

# Summary report
def show_summary():
    income = sum(t['amount'] for t in transactions if t['type'] == "income")
    expense = sum(t['amount'] for t in transactions if t['type'] == "expense")
    print("\n==== Financial Summary ====")
    print(f"Total Income: ${income:.2f}")
    print(f"Total Expenses: ${expense:.2f}")
    print(f"Profit/Loss: ${income - expense:.2f}")
    print("===========================\n")

# CLI Menu
def main():
    while True:
        print("1. Add transaction via voice")
        print("2. Add transaction via photo")
        print("3. Show summary")
        print("4. Exit")
        choice = input("Choose an option: ")

        if choice == "1":
            record_voice_transaction()
        elif choice == "2":
            image_path = input("Enter path to receipt/photo image: ")
            process_photo_transaction(image_path)
        elif choice == "3":
            show_summary()
        elif choice == "4":
            break
        else:
            print("Invalid choice.")

if __name__ == "__main__":
    main()
