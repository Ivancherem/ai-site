[bot.py.txt](https://github.com/user-attachments/files/24546284/bot.py.txt)
import telebot
from telebot.types import ReplyKeyboardMarkup, KeyboardButton
import json
import time

# Конфигурация
BOT_TOKEN = 'ВАШ_ТОКЕН_OT_BOTFATHER'
ADMIN_CHAT_ID = 'ВАШ_CHAT_ID'  # Узнать можно через @userinfobot

# Инициализация бота
bot = telebot.TeleBot(BOT_TOKEN)

# Файл для хранения заявок (простая база)
LEADS_FILE = 'leads.json'

# Загружаем заявки из файла
def load_leads():
    try:
        with open(LEADS_FILE, 'r', encoding='utf-8') as f:
            return json.load(f)
    except FileNotFoundError:
        return []

# Сохраняем заявку
def save_lead(lead):
    leads = load_leads()
    leads.append(lead)
    with open(LEADS_FILE, 'w', encoding='utf-8') as f:
        json.dump(leads, f, ensure_ascii=False, indent=2)

# Команда /start
@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_name = message.from_user.first_name
    welcome_text = f"""
🤖 Привет, {user_name}!

Я бот NeuraAI. Чем могу помочь?

• 🚀 *Демо-доступ* к платформе
• 📋 *Консультация* по внедрению AI
• 💰 *Спецпредложение* для новых клиентов

Выберите действие ниже или напишите вопрос:
    """
    
    # Создаём клавиатуру
    markup = ReplyKeyboardMarkup(resize_keyboard=True, row_width=2)
    btn1 = KeyboardButton('🚀 Получить демо-доступ')
    btn2 = KeyboardButton('📋 Оставить заявку')
    btn3 = KeyboardButton('💰 Узнать цены')
    btn4 = KeyboardButton('👨‍💻 Поддержка')
    markup.add(btn1, btn2, btn3, btn4)
    
    bot.send_message(message.chat.id, welcome_text, 
                     reply_markup=markup, parse_mode='Markdown')

# Обработка кнопок и текста
@bot.message_handler(func=lambda message: True)
def handle_messages(message):
    user_input = message.text
    chat_id = message.chat.id
    
    if user_input == '🚀 Получить демо-доступ':
        bot.send_message(chat_id, "Отлично! Заполните короткую форму на нашем сайте: [тык сюда](http://ваш-сайт.ru)", parse_mode='Markdown')
    
    elif user_input == '📋 Оставить заявку':
        msg = bot.send_message(chat_id, "Напишите ваше имя и номер телефона, и мы перезвоним в течение 15 минут!")
        bot.register_next_step_handler(msg, process_lead)
    
    elif user_input == '💰 Узнать цены':
        price_text = """
*Наши тарифы:*

🎯 *Старт* — 2990₽/мес
• До 1000 диалогов в месяц
• Базовая аналитика
• Email-поддержка

🚀 *Бизнес* — 8990₽/мес
• До 10000 диалогов
• Расширенная аналитика
• Приоритетная поддержка 24/7

🏢 *Корпоративный* — от 19990₽/мес
• Индивидуальный лимит
• Кастомные интеграции
• Персональный менеджер
        """
        bot.send_message(chat_id, price_text, parse_mode='Markdown')
    
    elif user_input == '👨‍💻 Поддержка':
        bot.send_message(chat_id, "Напишите ваш вопрос, и специалист ответит в течение 5 минут!")
    
    else:
        # Если просто текст, пересылаем админу
        forward_to_admin(message, is_lead=False)

# Обработка заявки
def process_lead(message):
    user_info = {
        'id': message.from_user.id,
        'username': message.from_user.username,
        'full_name': f"{message.from_user.first_name} {message.from_user.last_name or ''}",
        'text': message.text,
        'timestamp': time.strftime("%Y-%m-%d %H:%M:%S")
    }
    
    # Сохраняем
    save_lead(user_info)
    
    # Отправляем пользователю
    bot.send_message(message.chat.id, "✅ Спасибо! Ваша заявка принята. Менеджер свяжется с вами в течение 15 минут.")
    
    # Отправляем админу
    lead_text = f"""
📥 *НОВАЯ ЗАЯВКА ИЗ БОТА*
——————————————
👤 *Имя:* {user_info['full_name']}
📞 *Контакты:* {user_info['text']}
🕐 *Время:* {user_info['timestamp']}
ID: {user_info['id']} (@{user_info['username']})
——————————————
    """
    bot.send_message(ADMIN_CHAT_ID, lead_text, parse_mode='Markdown')

# Пересылка сообщений админу
def forward_to_admin(message, is_lead=True):
    if is_lead:
        text = f"📥 Заявка с сайта:\n{message}"
        bot.send_message(ADMIN_CHAT_ID, text)
    else:
        # Просто пересылаем сообщение
        bot.forward_message(ADMIN_CHAT_ID, message.chat.id, message.message_id)

# Функция для отправки заявки ИЗ САЙТА в бот (будет вызываться из скрипта сайта)
def send_lead_from_site(name, email, phone, message):
    lead_text = f"""
🌐 *НОВАЯ ЗАЯВКА С САЙТА*
——————————————
👤 *Имя:* {name}
📧 *Email:* {email}
📱 *Телефон:* {phone}
💬 *Сообщение:* {message}
🕐 *Время:* {time.strftime("%Y-%m-%d %H:%M:%S")}
——————————————
    """
    bot.send_message(ADMIN_CHAT_ID, lead_text, parse_mode='Markdown')
    return True

# Запуск бота
print("🤖 Бот запущен...")
bot.polling(none_stop=True)
