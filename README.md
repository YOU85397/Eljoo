import telebot
import json
import os
import threading

# توكن البوت الخاص بك
BOT_TOKEN = 'YOUR_BOT_TOKEN'  # ضع توكن البوت هنا
bot = telebot.TeleBot(BOT_TOKEN)

# إعدادات الأمان
المستخدمين_المسموح_لهم = {"The_engineer09"}  # إضافة المستخدم الجديد
المستخدمين_مفعلين = set()

# لتخزين آخر رسالة تم نشرها
آخر_رسالة = {
    "chat_id": None,
    "message_id": None
}

# -------------------------------------------------------
# تحميل أو إنشاء ملف تخزين القنوات
file_path = "channels.json"
if os.path.exists(file_path):
    with open(file_path, "r") as f:
        القنوات_المحفوظة = json.load(f)
else:
    القنوات_المحفوظة = {}

def حفظ_معرف_قناة(المعرف):
    if المعرف not in القنوات_المحفوظة.values():
        الرقم = str(len(القنوات_المحفوظة) + 1)
        القنوات_المحفوظة[الرقم] = المعرف
        with open(file_path, "w") as f:
            json.dump(القنوات_المحفوظة, f, ensure_ascii=False, indent=2)

# -------------------------------------------------------
# /start و التحقق من الاسم
@bot.message_handler(commands=['start'])
def البداية(message):
    username = message.from_user.username
    if username not in المستخدمين_المسموح_لهم:
        bot.send_message(message.chat.id, "عذرًا، ليس لديك صلاحية استخدام هذا البوت.")
        return

    if message.from_user.id in المستخدمين_مفعلين:
        متابعة_البداية(message)
    else:
        msg = bot.send_message(message.chat.id, "أدخل كلمة السر:")
        bot.register_next_step_handler(msg, تحقق_كلمة_السر)

def تحقق_كلمة_السر(message):
    if message.text.strip() == "853974":  # كلمة السر
        المستخدمين_مفعلين.add(message.from_user.id)
        bot.send_message(message.chat.id, "كلمة السر صحيحة! يمكنك الآن استخدام الأوامر.")
        متابعة_البداية(message)
    else:
        bot.send_message(message.chat.id, "كلمة السر غير صحيحة، حاول مرة أخرى.")

def متابعة_البداية(message):
    msg = bot.send_message(message.chat.id, "اكتب معرف القناة (مثال: @mychannel):")
    bot.register_next_step_handler(msg, استلام_المعرف)

# -------------------------------------------------------
# إضافة مستخدم جديد
@bot.message_handler(commands=['add_user'])
def إضافة_مستخدم(message):
    # تحقق من أن المستخدم هو صاحب البوت
    if message.from_user.username == "The_engineer09":  # يمكنك تغيير هذا للتأكد من أن المستخدم صاحب البوت
        msg = bot.send_message(message.chat.id, "أدخل اسم المستخدم لإضافته:")
        bot.register_next_step_handler(msg, استلام_مستخدم_جديد)
    else:
        bot.send_message(message.chat.id, "عذراً، ليس لديك صلاحية لإضافة مستخدمين.")

def استلام_مستخدم_جديد(message):
    user = message.text.strip()
    if user not in المستخدمين_المسموح_لهم:
        المستخدمين_المسموح_لهم.add(user)
        bot.send_message(message.chat.id, f"تم إضافة {user} إلى قائمة المستخدمين المسموح لهم.")
    else:
        bot.send_message(message.chat.id, f"{user} هو بالفعل في قائمة المستخدمين المسموح لهم.")

# -------------------------------------------------------
# استلام معلومات النشر
@bot.message_handler(func=lambda m: False)
def استلام_المعرف(message):
    معرف_القناة = message.text.strip()
    msg = bot.send_message(message.chat.id, "اكتب النص الذي تريد نشره:")
    bot.register_next_step_handler(msg, استلام_النص, معرف_القناة)

def استلام_النص(message, المعرف):
    النص = message.text.strip()
    msg = bot.send_message(message.chat.id, "اكتب نص الزر:")
    bot.register_next_step_handler(msg, استلام_نص_الزر, المعرف, النص)

def استلام_نص_الزر(message, المعرف, النص):
    نص_الزر = message.text.strip()
    msg = bot.send_message(message.chat.id, "أرسل الرابط الذي سيفتح عند الضغط على الزر:")
    bot.register_next_step_handler(msg, إرسال, المعرف, النص, نص_الزر)

def إرسال(message, المعرف, النص, نص_الزر):
    الرابط = message.text.strip()
    markup = telebot.types.InlineKeyboardMarkup()
    button = telebot.types.InlineKeyboardButton(text=نص_الزر, url=الرابط)
    markup.add(button)

    sent = bot.send_message(chat_id=المعرف, text=النص, reply_markup=markup)
    آخر_رسالة["chat_id"] = المعرف
    آخر_رسالة["message_id"] = sent.message_id

    حفظ_معرف_قناة(المعرف)
    bot.send_message(message.chat.id, "تم نشر الرسالة في القناة!")

# -------------------------------------------------------
# تشغيل البوت باستخدام polling بدلاً من webhook
bot.polling(none_stop=True)
