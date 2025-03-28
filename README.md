from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes
import datetime
import asyncio

# Токен вашего бота
TOKEN = 

# Список задач
tasks = []


async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Приветственное сообщение."""
    await update.message.reply_text(
        "Привет! Я бот-напоминалка. Вот что я умею:\n\n"
        "1. Добавить задачу: /add задача, время (пример: /add Встреча с другом, 14:30)\n"
        "2. Посмотреть задачи: /tasks\n"
        "3. Очистить задачи: /clear"
    )


async def add_task(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Добавление задачи с датой и временем."""
    try:
        # Получаем текст команды
        text = " ".join(context.args)
        if not text:
            await update.message.reply_text("Пожалуйста, укажи задачу и дату/время (пример: /add Встреча, 2025-02-06 14:30).")
            return

        # Разделяем текст задачи и дату/время
        task_text, task_datetime = text.rsplit(",", 1)
        task_datetime = task_datetime.strip()

        # Преобразуем дату и время в объект datetime
        task_datetime_obj = datetime.datetime.strptime(task_datetime, "%Y-%m-%d %H:%M")

        # Добавляем задачу в список
        tasks.append({"text": task_text.strip(), "time": task_datetime_obj, "chat_id": update.message.chat_id})
        await update.message.reply_text(f"Задача '{task_text.strip()}' добавлена на {task_datetime_obj.strftime('%Y-%m-%d %H:%M')}.")

    except ValueError:
        await update.message.reply_text("Ошибка! Формат даты и времени должен быть 'YYYY-MM-DD HH:MM'.")
    except Exception as e:
        await update.message.reply_text("Произошла ошибка! Убедитесь, что вы правильно ввели команду.")


async def list_tasks(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Список задач."""
    if not tasks:
        await update.message.reply_text("Нет задач.")
        return

    task_list = "\n".join([f"{t['text']} - {t['time'].strftime('%H:%M')}" for t in tasks])
    await update.message.reply_text(f"Твои задачи:\n\n{task_list}")


async def clear_tasks(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Очистка всех задач."""
    tasks.clear()
    await update.message.reply_text("Все задачи очищены!")


async def task_reminder(application: Application):
    """Фоновая задача для отправки напоминаний."""
    while True:
        now = datetime.datetime.now()
        for task in tasks[:]:
            if now >= task["time"]:
                await application.bot.send_message(chat_id=task["chat_id"], text=f"Напоминание: {task['text']}")
                tasks.remove(task)
        await asyncio.sleep(30)  # Проверяем каждые 30 секунд


def main():
    """Запуск бота без использования asyncio.run()."""
    app = Application.builder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("add", add_task))
    app.add_handler(CommandHandler("tasks", list_tasks))
    app.add_handler(CommandHandler("clear", clear_tasks))

    loop = asyncio.get_event_loop()

    # Запускаем фоновую задачу
    loop.create_task(task_reminder(app))

    # Запускаем polling
    print("Bot zapyshen...")
    app.run_polling()

if __name__ == "__main__":
    main()
