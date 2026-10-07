import asyncio
import logging
from aiogram import Bot, Dispatcher, F
from aiogram.filters import CommandStart
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from aiogram.types import Message, ReplyKeyboardMarkup, KeyboardButton, ReplyKeyboardRemove

8628036278:AAF9CrHBeyJ6Yrp-tRRB2RTcyrpc4OTjxqo
BOT_TOKEN = "YOUR_TELEGRAM_BOT_TOKEN_HERE"

class ResumeForm(StatesGroup):
    full_name = State()
    phone = State()
    profession = State()
    experience = State()
    skills = State()
    education = State()

bot = Bot(token=BOT_TOKEN)
dp = Dispatcher()

@dp.message(CommandStart())
async def start_cmd(message: Message, state: FSMContext):
    await state.clear()
    await message.answer(
        "👋 **Salom! Rezume yaratuvchi botga xush kelibsiz.**\n\n"
        "Boshlash uchun **Ism va Familiyangizni** kiriting:"
    )
    await state.set_state(ResumeForm.full_name)

@dp.message(ResumeForm.full_name)
async def get_name(message: Message, state: FSMContext):
    await state.update_data(full_name=message.text)
    
    phone_btn = KeyboardButton(text="📱 Telefon raqamni yuborish", request_contact=True)
    kb = ReplyKeyboardMarkup(keyboard=[[phone_btn]], resize_keyboard=True, one_time_keyboard=True)
    
    await message.answer("📱 **Telefon raqamingizni** yuboring yoki pastdagi tugmani bosing:", reply_markup=kb)
    await state.set_state(ResumeForm.phone)

@dp.message(ResumeForm.phone)
async def get_phone(message: Message, state: FSMContext):
    phone = message.contact.phone_number if message.contact else message.text
    await state.update_data(phone=phone)
    
    await message.answer(
        "💼 **Qaysi kasb yoki lavozimda ishlamoqchisiz?**\n*(Masalan: Python dasturchi, Buxgalter, Sotuvchi)*",
        reply_markup=ReplyKeyboardRemove()
    )
    await state.set_state(ResumeForm.profession)

@dp.message(ResumeForm.profession)
async def get_profession(message: Message, state: FSMContext):
    await state.update_data(profession=message.text)
    await message.answer("🛠 **Ko'nikmalaringiz va biladigan dasturlaringizni** yozing:\n*(Masalan: Excel, Python, Git, Muloqotchanlik)*")
    await state.set_state(ResumeForm.skills)

@dp.message(ResumeForm.skills)
async def get_skills(message: Message, state: FSMContext):
    await state.update_data(skills=message.text)
    await message.answer("💼 **Ish tajribangiz haqida** ma'lumot bering:\n*(Masalan: 2 yil ABC kompaniyasida, yoki Tajribam yo'q)*")
    await state.set_state(ResumeForm.experience)

@dp.message(ResumeForm.experience)
async def get_experience(message: Message, state: FSMContext):
    await state.update_data(experience=message.text)
    await message.answer("🎓 **Ma'lumotingiz va ta'lim maskaningiz** haqida yozing:\n*(Masalan: TATU, Bakalavr, 2020-2024)*")
    await state.set_state(ResumeForm.education)

@dp.message(ResumeForm.education)
async def get_education(message: Message, state: FSMContext):
    await state.update_data(education=message.text)
    data = await state.get_data()
    
    resume = (
        "📄 **TAYYOR REZUME**\n"
        "━━━━━━━━━━━━━━━━━━━━━\n"
        f"👤 **Ism-Familiya:** {data['full_name']}\n"
        f"📞 **Telefon:** {data['phone']}\n"
        f"💼 **Kutilayotgan lavozim:** {data['profession']}\n\n"
        f"🛠 **Ko'nikmalar:**\n{data['skills']}\n\n"
        f"💼 **Ish tajribasi:**\n{data['experience']}\n\n"
        f"🎓 **Ta'lim:**\n{data['education']}\n"
        "━━━━━━━━━━━━━━━━━━━━━\n"
        "✨ *Bot orqali shakllantirildi.*"
    )
    
    await message.answer(resume, parse_mode="Markdown")
    await message.answer("Yangi rezume tuzish uchun qayta /start tugmasini bosing.")
    await state.clear()

async def main():
    logging.basicConfig(level=logging.INFO)
    await dp.start_polling(bot)

if __name__ == "__main__":
    asyncio.run(main())
