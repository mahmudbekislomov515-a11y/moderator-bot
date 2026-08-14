const { Telegraf } = require('telegraf');

// Bot tokeningizni shu yerga yozing
const BOT_TOKEN = process.env.BOT_TOKEN || 'TOKENINGIZNI_SHU_YERGA_YOZING';
const bot = new Telegraf(BOT_TOKEN);

bot.start((ctx) => {
    ctx.reply('Assalomu alaykum! Moderator bot muvaffaqiyatli ishga tushdi va guruhni qo\'riqlashga tayyor.');
});

// Guruhda taqiqlangan so'zlar ro'yxati (o'zingiz xohlagancha qo'shishingiz mumkin)
const badWords = ['soz1', 'soz2', 'axlat', 'tentak'];

bot.on('text', async (ctx) => {
    // Faqat guruh va superguruhlarda ishlashi uchun
    if (ctx.chat.type === 'supergroup' || ctx.chat.type === 'group') {
        const text = ctx.message.text.toLowerCase();
        
        // Xatoda taqiqlangan so'z bor-yo'qligini tekshirish
        const isBad = badWords.some(word => text.includes(word));
        
        if (isBad) {
            try {
                // Taqiqlangan xabarni o'chirish
                await ctx.deleteMessage();
                
                // Ogohlantirish yuborish
                const name = ctx.from.first_name || 'Foydalanuvchi';
                const warning = await ctx.reply(`Hurmatli ${name}, guruhda bunday so'zlarni yozish taqiqlanadi! Xabaringiz o'chirildi.`);
                
                // 5 sekunddan keyin ogohlantirishni ham o'chirib yuborish
                setTimeout(async () => {
                    try {
                        await ctx.telegram.deleteMessage(ctx.chat.id, warning.message_id);
                    } catch (e) {}
                }, 5000);
            } catch (e) {
                console.log("Xabarni o'chirishda xatolik (botda admin huquqi yo'q bo'lishi mumkin):", e);
            }
        }
    }
});

bot.launch();
console.log('Moderator bot ishga tushdi...');

// Dastur to'xtaganda botni to'g'ri yopish
process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));
