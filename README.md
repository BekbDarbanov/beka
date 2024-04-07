from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.scrollview import ScrollView
from kivy.uix.gridlayout import GridLayout
from kivy.uix.button import Button
from kivy.uix.popup import Popup
from kivy.uix.label import Label
from kivy.core.window import Window
import pyttsx3

# Глобальные данные (примеры, замените на ваши данные)
russian_books_data = {
    "Война и мир": "path/to/15.jpg",
    "Преступление и наказание": "path/to/15.jpg"
    # Добавьте остальные книги сюда
}

# Допустим, у вас есть вопросы и ответы для "Война и мир"
russian_questions_and_answers = {
    "Война и мир": {
        "Каким годом и месяцем датируется начало событий романа «Война и мир»?": ["1805", "1812", "1890"]
    },
    "Преступление и наказание": {
        "Кто является автором романа «Преступление и наказание»?": ["Лев Толстой","Иван Тургенев","Федор Достоевский","Александр Пушкин"]
    }
}

class BookApp(App):
    def build(self):
        return MainScreen()

class MainScreen(BoxLayout):
    engine = pyttsx3.init()  # Создаем экземпляр движка pyttsx3

    def _init_(self, **kwargs):
        super()._init_(orientation='vertical', **kwargs)
        self.add_widget(Button(text='Русские книги', on_press=lambda x: self.show_books(russian_books_data)))

    def show_books(self, books_data):
        layout = GridLayout(cols=2, spacing=10, size_hint_y=None)
        layout.bind(minimum_height=layout.setter('height'))
        scroll = ScrollView(size_hint=(1, None), size=(Window.width, Window.height))
        scroll.add_widget(layout)

        for title, img in books_data.items():
            btn = Button(text=title, size_hint_y=None, height=200)
            btn.bind(on_press=lambda x, book=title: self.show_question(book, russian_questions_and_answers[book]))
            layout.add_widget(btn)

        popup = Popup(title='Выберите книгу', content=scroll, size_hint=(0.9, 0.9))
        popup.open()

    def show_question(self, book, questions_and_answers):
        layout = BoxLayout(orientation='vertical')
        for question, answers in questions_and_answers.items():
            self.speak_text(question)  # Озвучиваем текст вопроса
            layout.add_widget(Label(text=question))
            for answer in answers:
                btn = Button(text=answer)
                btn.bind(on_press=lambda x, book=book, question=question, answer=answer: self.check_answer(book, question, answer))
                layout.add_widget(btn)
        
        popup = Popup(title=book, content=layout, size_hint=(0.8, 0.8))
        popup.open()

    def check_answer(self, book, question, answer):
        # Здесь должна быть логика для проверки правильности ответа
        # Для примера предположим, что правильный ответ всегда "1805" для "Война и мир" и "Федор Достоевский" для "Преступление и наказание"
        if book == "Война и мир":
            correct_answer = "1805"
        elif book == "Преступление и наказание":
            correct_answer = "Федор Достоевский"
        else:
            # Если книга неизвестна, просто отмечаем, что ответ неверный
            correct_answer = "Unknown"

        if answer == correct_answer:
            result = "Правильно!"
        else:
            result = "Неправильно. Правильный ответ: " + correct_answer
        self.speak_text(result)  # Озвучиваем результат
        popup = Popup(title="Результат", content=Label(text=result), size_hint=(0.6, 0.4))
        popup.open()

    def speak_text(self, text):
        self.engine.say(text)
        self.engine.runAndWait()

if _name_ == '_main_':
    BookApp().run()
