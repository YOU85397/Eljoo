import yfinance as yf
import talib
import pandas as pd
from telegram import Bot
from telegram.ext import Application, CommandHandler, CallbackContext
from telegram import Update
import asyncio

# توكن البوت
bot = Bot(token="8052080147:AAF5YvAz0onMcFxV4A_aKQpzw4l_QIefttk")

# معرّف القناة أو المحادثة
chat_id = "@jsissgo"

# لائحة أزواج العملات
currency_pairs = ['EURUSD=X', 'GBPUSD=X', 'USDJPY=X', 'AUDUSD=X', 'USDCHF=X', 'NZDUSD=X']

# حساب المؤشرات الفنية لكل زوج من العملات
def calculate_indicators(pair):
    df = yf.download(pair, period="1d", interval="1m")
    
    # حساب المؤشرات الفنية باستخدام TA-Lib
    df['RSI'] = talib.RSI(df['Close'], timeperiod=14)
    df['MACD'], df['MACD_signal'], df['MACD_hist'] = talib.MACD(df['Close'], fastperiod=12, slowperiod=26, signalperiod=9)
    df['EMA'] = talib.EMA(df['Close'], timeperiod=9)
    
    return df

# حساب إشارات الدخول بناءً على المؤشرات
def generate_signal(pair, df):
    # الحصول على آخر القيم للمؤشرات
    latest_rsi = df['RSI'].iloc[-1]
    latest_macd = df['MACD'].iloc[-1]
    latest_ema = df['EMA'].iloc[-1]
    
    signal = f"إشارة الدخول لـ {pair}:\n"
    
    # إشارات RSI
    if latest_rsi > 70:
        signal += f"RSI مرتفع: {latest_rsi} - يمكن أن يكون هذا وقت البيع.\n"
    elif latest_rsi < 30:
        signal += f"RSI منخفض: {latest_rsi} - يمكن أن يكون هذا وقت الشراء.\n"
    else:
        signal += f"RSI: {latest_rsi} - السوق في حالة محايدة.\n"
    
    # إشارات MACD
    if latest_macd > 0:
        signal += f"MACD إيجابي: {latest_macd} - يمكن أن يكون هذا وقت الشراء.\n"
    else:
        signal += f"MACD سلبي: {latest_macd} - يمكن أن يكون هذا وقت البيع.\n"
    
    # إشارات EMA
    if latest_ema > df['Close'].iloc[-2]:  # مقارنة مع القيمة السابقة
        signal += f"EMA صاعد: {latest_ema} - السوق في اتجاه صاعد.\n"
    else:
        signal += f"EMA هابط: {latest_ema} - السوق في اتجاه هابط.\n"
    
    return signal

# إعدادات الأوامر الخاصة بالبوت
async def start(update: Update, context: CallbackContext):
    await update.message.reply_text("مرحبًا! أنا بوت التداول. استخدم /signals للحصول على إشارات الدخول لأزواج العملات.")

async def signals(update: Update, context: CallbackContext):
    for pair in currency_pairs:
        df = calculate_indicators(pair)
        signal = generate_signal(pair, df)
        
        # إرسال الإشارة إلى القناة
        await bot.send_message(chat_id=chat_id, text=signal)
        await update.message.reply_text(signal)

# لتشغيل البوت داخل Google Colab باستخدام asyncio
def run_bot():
    application = Application.builder().token("8052080147:AAF5YvAz0onMcFxV4A_aKQpzw4l_QIefttk").build()
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("signals", signals))
    application.run_polling()

# استخدام asyncio.run لتشغيل البوت في Google Colab
asyncio.run(run_bot())
