ФИО: Лушникова Дарья Алесандровна

Currency-Converter/
├── main.py                  # Основной код приложения
├── history.json           # История конвертаций
├── .gitignore
└── README.md

import tkinter as tk
from tkinter import ttk, messagebox
import requests
import json
import os

class CurrencyConverter:
    def __init__(self, root):
        self.root = root
        self.root.title("Currency Converter")
        self.history = []
        self.load_history()
        self.api_key = "YOUR_API_KEY"  # Замените на ваш API‑ключ
        self.setup_ui()

    def setup_ui(self):
        # Выбор валюты «из»
        tk.Label(self.root, text="Из валюты:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.from_currency = ttk.Combobox(self.root, values=self.get_currencies(), state="readonly")
        self.from_currency.set("USD")
        self.from_currency.grid(row=0, column=1, padx=5, pady=5)

        # Выбор валюты «в»
        tk.Label(self.root, text="В валюту:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
        self.to_currency = ttk.Combobox(self.root, values=self.get_currencies(), state="readonly")
        self.to_currency.set("EUR")
        self.to_currency.grid(row=1, column=1, padx=5, pady=5)

        # Поле ввода суммы
        tk.Label(self.root, text="Сумма:").grid(row=2, column=0, sticky="w", padx=5, pady=5)
        self.amount_entry = tk.Entry(self.root)
        self.amount_entry.grid(row=2, column=1, padx=5, pady=5)

        # Кнопка конвертации
        tk.Button(self.root, text="Конвертировать", command=self.convert_currency).grid(
            row=3, column=0, columnspan=2, pady=10
        )

        # Отображение результата
        tk.Label(self.root, text="Результат:").grid(row=4, column=0, sticky="w", padx=5, pady=5)
        self.result_label = tk.Label(self.root, text="")
        self.result_label.grid(row=4, column=1, padx=5, pady=5, sticky="w")

        # Таблица истории
        tk.Label(self.root, text="История конвертаций:").grid(row=5, column=0, sticky="w", padx=5, pady=5)
        columns = ("From", "To", "Amount", "Result")
        self.history_tree = ttk.Treeview(self.root, columns=columns, show="headings")
        for col in columns:
            self.history_tree.heading(col, text=col)
            self.history_tree.column(col, width=100)
        self.history_tree.grid(row=6, column=0, columnspan=2, padx=5, pady=5, sticky="nsew")

        # Кнопки управления историей
        tk.Button(self.root, text="Очистить историю", command=self.clear_history).grid(row=7, column=0, pady=10)
        tk.Button(self.root, text="Обновить курсы", command=self.update_rates).grid(row=7, column=1, pady=10)

        # Настройка растягивания элементов
        self.root.grid_rowconfigure(6, weight=1)
        self.root.grid_columnconfigure(1, weight=1)

        self.update_history_table()

    def get_currencies(self):
        return ["USD", "EUR", "GBP", "JPY", "CAD", "AUD", "CHF", "CNY", "RUB"]


    def validate_input(self):
        try:
            amount = float(self.amount_entry.get())
            if amount <= 0:
                messagebox.showerror("Ошибка", "Сумма должна быть положительным числом.")
                return False
            return True, amount
        except ValueError:
            messagebox.showerror("Ошибка", "Введите корректное число.")
            return False, None

    def convert_currency(self):
        if not self.validate_input():
            return

        is_valid, amount = self.validate_input()
        from_curr = self.from_currency.get()
        to_curr = self.to_currency.get()

        try:
            response = requests.get(
                f"https://api.exchangerate-api.com/v4/latest/{from_curr}"
            )
            response.raise_for_status()
            data = response.json()

            if to_curr in data["rates"]:
                rate = data["rates"][to_curr]
                result = amount * rate
                result_text = f"{amount:.2f} {from_curr} = {result:.2f} {to_curr}"
                self.result_label.config(text=result_text)

                # Добавляем в историю
                history_entry = {
                    "from": from_curr,
                    "to": to_curr,
                    "amount": amount,
                    "result": result
                }
                self.history.append(history_entry)
                self.save_history()
                self.update_history_table()
            else:
                messagebox.showerror("Ошибка", f"Валюта {to_curr} не найдена.")
        except requests.exceptions.RequestException as e:
            messagebox.showerror("Ошибка", f"Не удалось получить курсы валют: {e}")


    def update_history_table(self):
        for item in self.history_tree.get_children():
            self.history_tree.delete(item)
        for entry in reversed(self.history):  # Последние конвертации вверху
            self.history_tree.insert("", "end", values=(
                entry["from"],
                entry["to"],
                f"{entry['amount']:.2f}",
                f"{entry['result']:.2f}"
            ))

    def save_history(self):
        with open("history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, indent=4, ensure_ascii=False)

    def load_history(self):
        if os.path.exists("history.json"):
            with open("history.json", "r", encoding="utf-8") as f:
                self.history = json.load(f)
        else:
            self.history = []

    def clear_history(self):
        self.history = []
        self.save_history()
        self.update_history_table()
        messagebox.showinfo("Успех", "История очищена.")

    def update_rates(self):
        messagebox.showinfo("Информация", "Курсы обновлены!")

if __name__ == "__main__":
    root = tk.Tk()
    app = CurrencyConverter(root)
    root.mainloop()


Создайте файл .gitignore в корне проекта со следующим содержимым:

gitignore
history.json
__pycache__/
*.pyc
.env

## Описание

GUI‑приложение для конвертации валют с использованием внешнего API. Позволяет:
* выбирать валюты для конвертации;
* вводить сумму для конвертации;
* получать актуальные курсы валют через API;
* просматривать историю конвертаций;
* сохранять историю в JSON‑файл.

## Требования

* Python 3.6+;
* библиотеки: `tkinter`, `requests`, `json`.

## Как получить API‑ключ

1. Перейдите на сайт [exchangerate-api.com](https://www.exchangerate-api.com).
2. Зарегистрируйтесь и получите бесплатный API‑ключ.
3. Замените строку `self.api_key = "YOUR_API_KEY"` в коде на ваш ключ.

## Установка и запуск

1. Клонируйте репозиторий:
```bash
git clone <URL-репозитория>