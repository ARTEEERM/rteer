import tkinter as tk
from tkinter import ttk, messagebox
import json
import os

class BookTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Book Tracker")
        self.root.geometry("800x600")

        # Файл для хранения данных
        self.data_file = "books_data.json"
        self.books = []
        self.load_books()

        self.setup_ui()

    def setup_ui(self):
        # Форма добавления книги
        form_frame = ttk.LabelFrame(self.root, text="Добавить книгу")
        form_frame.pack(pady=10, padx=10, fill="x")

        # Название книги
        ttk.Label(form_frame, text="Название:").grid(row=0, column=0, sticky="w", padx=5, pady=5)
        self.title_entry = ttk.Entry(form_frame, width=30)
        self.title_entry.grid(row=0, column=1, padx=5, pady=5)

        # Автор
        ttk.Label(form_frame, text="Автор:").grid(row=1, column=0, sticky="w", padx=5, pady=5)
        self.author_entry = ttk.Entry(form_frame, width=30)
        self.author_entry.grid(row=1, column=1, padx=5, pady=5)

        # Жанр
        ttk.Label(form_frame, text="Жанр:").grid(row=2, column=0, sticky="w", padx=5, pady=5)
        genres = ["Роман", "Фантастика", "Детектив", "Биография", "Поэзия", "Научная литература"]
        self.genre_var = tk.StringVar()
        self.genre_combo = ttk.Combobox(form_frame, textvariable=self.genre_var, values=genres, state="readonly")
        self.genre_combo.grid(row=2, column=1, padx=5, pady=5)

        # Количество страниц
        ttk.Label(form_frame, text="Страниц:").grid(row=3, column=0, sticky="w", padx=5, pady=5)
        self.pages_entry = ttk.Entry(form_frame, width=30)
        self.pages_entry.grid(row=3, column=1, padx=5, pady=5)

        # Кнопка добавления
        add_btn = ttk.Button(form_frame, text="Добавить книгу", command=self.add_book)
        add_btn.grid(row=4, column=0, columnspan=2, pady=10)

        # Фильтры
        filter_frame = ttk.LabelFrame(self.root, text="Фильтры")
        filter_frame.pack(pady=5, padx=10, fill="x")

        # Фильтр по жанру
        ttk.Label(filter_frame, text="Жанр:").pack(side="left", padx=5)
        self.filter_genre_var = tk.StringVar(value="все")
        genre_filter = ttk.Combobox(
            filter_frame,
            textvariable=self.filter_genre_var,
            values=["все"] + genres,
            state="readonly"
        )
        genre_filter.pack(side="left", padx=5)

        # Фильтр по страницам
        ttk.Label(filter_frame, text="Страницы >:").pack(side="left", padx=5)
        page_filters = ["0", "200", "300", "500"]
        self.filter_pages_var = tk.StringVar(value="0")
        pages_filter = ttk.Combobox(
            filter_frame,
            textvariable=self.filter_pages_var,
            values=page_filters,
            state="readonly"
        )
        pages_filter.pack(side="left", padx=5)

        # Кнопки фильтров
        apply_filter_btn = ttk.Button(filter_frame, text="Применить фильтр", command=self.apply_filters)
        apply_filter_btn.pack(side="left", padx=5)

        reset_filter_btn = ttk.Button(filter_frame, text="Сбросить", command=self.reset_filters)
        reset_filter_btn.pack(side="left", padx=5)

        # Таблица книг
        table_frame = ttk.LabelFrame(self.root, text="Список книг")
        table_frame.pack(pady=10, padx=10, fill="both", expand=True)

        columns = ("title", "author", "genre", "pages")
        self.tree = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)

        self.tree.heading("title", text="Название")
        self.tree.heading("author", text="Автор")
        self.tree.heading("genre", text="Жанр")
        self.tree.heading("pages", text="Страниц")

        self.tree.column("title", width=200)
        self.tree.column("author", width=150)
        self.tree.column("genre", width=100)
        self.tree.column("pages", width=80)

        scrollbar = ttk.Scrollbar(table_frame, orient="vertical", command=self.tree.yview)
        self.tree.configure(yscrollcommand=scrollbar.set)

        self.tree.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")


        # Кнопки сохранения/загрузки
        action_frame = ttk.Frame(self.root)
        action_frame.pack(pady=10)

        save_btn = ttk.Button(action_frame, text="Сохранить в JSON", command=self.save_books)
        save_btn.pack(side="left", padx=5)

        load_btn = ttk.Button(action_frame, text="Загрузить из JSON", command=self.load_books)
        load_btn.pack(side="left", padx=5)

        delete_btn = ttk.Button(action_frame, text="Удалить выбранную", command=self.delete_book)
        delete_btn.pack(side="left", padx=5)


        clear_btn = ttk.Button(action_frame, text="Очистить всё", command=self.clear_all)
        clear_btn.pack(side="left", padx=5)


        self.refresh_table()

    def validate_input(self):
        """Проверка корректности ввода"""
        title = self.title_entry.get().strip()
        author = self.author_entry.get().strip()
        genre = self.genre_var.get()
        pages = self.pages_entry.get().strip()

        if not title:
            messagebox.showerror("Ошибка", "Название книги не может быть пустым!")
            return False
        if not author:
            messagebox.showerror("Ошибка", "Автор не может быть пустым!")
            return False
        if not genre:
            messagebox.showerror("Ошибка", "Выберите жанр!")
            return False
        if not pages:
            messagebox.showerror("Ошибка", "Количество страниц не может быть пустым!")
            return False

        try:
            pages_num = int(pages)
            if pages_num <= 0:
                messagebox.showerror("Ошибка", "Количество страниц должно быть положительным числом!")
                return False
        except ValueError:
            messagebox.showerror("Ошибка", "Количество страниц должно быть числом!")
            return False

        return True, title, author, genre, pages_num

    def add_book(self):
