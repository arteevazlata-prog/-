Реализация приложения Weather Diary на Python с использованием tkinter для GUI и json для хранения данных.

import json
import os
from datetime import datetime
import tkinter as tk
from tkinter import ttk, messagebox

# ------------------ Файл с данными ------------------
DATA_FILE = "weather_diary.json"

# ------------------ Работа с JSON ------------------
def load_data():
    if not os.path.exists(DATA_FILE):
        return []
    with open(DATA_FILE, "r", encoding="utf-8") as f:
        return json.load(f)

def save_data(entries):
    with open(DATA_FILE, "w", encoding="utf-8") as f:
        json.dump(entries, f, indent=4, ensure_ascii=False)

# ------------------ Проверка ввода ------------------
def is_valid_date(date_str):
    try:
        datetime.strptime(date_str, "%Y-%m-%d")
        return True
    except ValueError:
        return False

def is_valid_temperature(temp_str):
    try:
        float(temp_str)
        return True
    except ValueError:
        return False

# ------------------ Основное приложение ------------------
class WeatherDiaryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Weather Diary / Дневник погоды")
        self.root.geometry("800x500")

        # Данные
        self.entries = load_data()

        # Поля ввода
        input_frame = ttk.LabelFrame(root, text="Новая запись", padding=10)
        input_frame.pack(fill="x", padx=10, pady=5)

        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="w")
        self.date_entry = ttk.Entry(input_frame, width=15)
        self.date_entry.grid(row=0, column=1, padx=5)

        ttk.Label(input_frame, text="Температура (°C):").grid(row=0, column=2, sticky="w", padx=(10,0))
        self.temp_entry = ttk.Entry(input_frame, width=8)
        self.temp_entry.grid(row=0, column=3, padx=5)

        ttk.Label(input_frame, text="Описание:").grid(row=0, column=4, sticky="w", padx=(10,0))
        self.desc_entry = ttk.Entry(input_frame, width=25)
        self.desc_entry.grid(row=0, column=5, padx=5)

        self.precip_var = tk.BooleanVar()
        self.precip_check = ttk.Checkbutton(input_frame, text="Осадки", variable=self.precip_var)
        self.precip_check.grid(row=0, column=6, padx=10)

        self.add_btn = ttk.Button(input_frame, text="Добавить запись", command=self.add_entry)
        self.add_btn.grid(row=0, column=7, padx=10)

        # Фильтры
        filter_frame = ttk.LabelFrame(root, text="Фильтрация", padding=10)
        filter_frame.pack(fill="x", padx=10, pady=5)

        ttk.Label(filter_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=0, sticky="w")
        self.filter_date_entry = ttk.Entry(filter_frame, width=15)
        self.filter_date_entry.grid(row=0, column=1, padx=5)

        ttk.Label(filter_frame, text="Температура >").grid(row=0, column=2, sticky="w", padx=(10,0))
        self.filter_temp_entry = ttk.Entry(filter_frame, width=8)
        self.filter_temp_entry.grid(row=0, column=3, padx=5)
        ttk.Label(filter_frame, text="°C").grid(row=0, column=4, sticky="w")

        self.apply_filter_btn = ttk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter)
        self.apply_filter_btn.grid(row=0, column=5, padx=10)
        self.reset_filter_btn = ttk.Button(filter_frame, text="Сбросить фильтр", command=self.reset_filter)
        self.reset_filter_btn.grid(row=0, column=6, padx=5)

        # Таблица записей
        columns = ("date", "temperature", "description", "precipitation")
        self.tree = ttk.Treeview(root, columns=columns, show="headings")
        self.tree.heading("date", text="Дата")
        self.tree.heading("temperature", text="Температура")
        self.tree.heading("description", text="Описание")
        self.tree.heading("precipitation", text="Осадки")
        self.tree.column("date", width=100)
        self.tree.column("temperature", width=80)
        self.tree.column("description", width=250)
        self.tree.column("precipitation", width=80)

        scrollbar = ttk.Scrollbar(root, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)
        scrollbar.pack(side="right", fill="y")
        self.tree.pack(fill="both", expand=True, padx=10, pady=5)

        self.refresh_table()

    # ------------------ Добавление записи ------------------
    def add_entry(self):
        date = self.date_entry.get().strip()
        temp = self.temp_entry.get().strip()
        desc = self.desc_entry.get().strip()
        precip = self.precip_var.get()

        # Валидация
        if not date or not temp or not desc:
            messagebox.showerror("Ошибка", "Заполните все поля!")
            return
        if not is_valid_date(date):
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ГГГГ-ММ-ДД")
            return
        if not is_valid_temperature(temp):
            messagebox.showerror("Ошибка", "Температура должна быть числом!")
            return

        new_entry = {
            "date": date,
            "temperature": float(temp),
            "description": desc,
            "precipitation": precip
        }

        self.entries.append(new_entry)
        save_data(self.entries)
        self.refresh_table()
        self.clear_input_fields()

    def clear_input_fields(self):
        self.date_entry.delete(0, tk.END)
        self.temp_entry.delete(0, tk.END)
        self.desc_entry.delete(0, tk.END)
        self.precip_var.set(False)

    # ------------------ Отображение данных ------------------
    def refresh_table(self, filtered_entries=None):
        for row in self.tree.get_children():
            self.tree.delete(row)

        data_to_show = filtered_entries if filtered_entries is not None else self.entries
        for entry in data_to_show:
            precip_str = "Да" if entry["precipitation"] else "Нет"
            self.tree.insert("", "end", values=(
                entry["date"],
                entry["temperature"],
                entry["description"],
                precip_str
            ))

    # ------------------ Фильтрация ------------------
    def apply_filter(self):
        filter_date = self.filter_date_entry.get().strip()
        filter_temp_str = self.filter_temp_entry.get().strip()

        filtered = self.entries[:]

        if filter_date:
            if not is_valid_date(filter_date):
                messagebox.showerror("Ошибка", "Неверный формат даты в фильтре!")
                return
            filtered = [e for e in filtered if e["date"] == filter_date]

        if filter_temp_str:
            try:
                filter_temp = float(filter_temp_str)
                filtered = [e for e in filtered if e["temperature"] > filter_temp]
            except ValueError:
                messagebox.showerror("Ошибка", "Температура для фильтра должна быть числом!")
                return

        self.refresh_table(filtered)

    def reset_filter(self):
        self.filter_date_entry.delete(0, tk.END)
        self.filter_temp_entry.delete(0, tk.END)
        self.refresh_table()

# ------------------ Запуск ------------------
if __name__ == "__main__":
    root = tk.Tk()
    app = WeatherDiaryApp(root)
    root.mainloop()
    
Инструкция по Git и GitHub (для отчёта)
1. Создание локального репозитория
bash
git init
git add weather_diary.py
git commit -m "Initial commit: Weather Diary app"
2. Создать .gitignore
text
__pycache__/
*.pyc
weather_diary.json
.DS_Store
venv/
3. Выложить на GitHub
bash
git remote add origin https://github.com/your-username/weather-diary.git
git branch -M main
git push -u origin main

Пример README.md (можно сохранить как .md или .txt)
markdown

# Weather Diary / Дневник погоды

**Автор:** Артеева Злата

## Описание
Приложение для ведения дневника погоды: добавление записей (дата, температура, описание, осадки), фильтрация по дате и температуре, сохранение/загрузка в JSON.

## Запуск
1. Установите Python 3.6+.
2. Запустите файл:
   ```bash
   python weather_diary.py
   
Пример использования
1. Введите 2025-04-01, 15, Солнечно, отметьте Осадки → Нажмите Добавить запись.

2. В фильтре укажите Температура > 10 → записи с температурой выше 10°C.

3. Закройте и откройте приложение заново → данные загрузятся из weather_diary.json.#
