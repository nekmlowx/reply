const Eris = require('eris');

// التوكن الخاص بالبوت
const token = 'your_token';

// إنشاء كائن البوت
const bot = new Eris(token);

// القنوات التي يرد فيها البوت
const allowedChannels = ['id_room'];

// المستخدمين الذين يرد عليهم البوت
const allowedUsers = ['id_user'];

// قائمة الكلمات التي يتجاهلها البوت
const ignoredWords = ['نقط', 'ن', 'تنقيط', 'nkt', 'nakt', 'نـޢޢـقـޢޢــطـޢޢــ','نـޢޢـ','قـޢޢــ','طـޢޢــ','ޢޢــ','نٌےـ','قَےـ','طٌےـ','نـ░ـ','قـ░ـ','طـ░ـ','نـ░ـقـ░ـطـ░ـ','قــــــ','نـــــــٌقــــــط','نـ༈ۖ҉ـ','قـ༈ۖ҉ـ','ط','نـ༈ۖ҉ـقـ༈ۖ҉ـط','نٌےـ','قَےـ','طٌےـ','نٌےـقَےـطٌےـ','نےـ','قےـ','طےـ','نُـ‘ـُ',' قُـ‘ـُ','طُـ‘ـُ','نُـ‘ـُ قُـ‘ـُطُـ‘ـُ','نـ❣ـہقـ❣ـہطـ❣ـہ','نـ❣ـہ','قـ❣ـہ','طـ❣ـہ','قـ❣ـہ','نـ❣ـہ','نۣۗہ','قۣۗہ','طۣۗہ','نۣۗہقۣۗہطۣۗہ','نِ','قَ','طٌ','نِقَطٌ','نۣۗـۙ','قۣۗـۙ','طۣۗـۙ','نہ','قہ','طہ','نہقہطہ'];
const additionalIgnoreWords = ['لو', 'انت', 'رد','لو انت ابن قحبة نقط','لو انت نقط','لو انت ابن قحبة','لو انت ابن شرموطة','ماهي رسالتك','شنو','شتقول','شدتقول','تكول','لامك','لجدك','لنسلك','لجدتك','لاختك','لابوك','لنفسك','لاخوك','صور','صورة','سكرين','سكرن','صوره'];

// الرد على الكلمات المحددة برسالة معينة
const specialReply = "كسك.";

// الرد على الصور
const imageReply = "طبونك.";

// قائمة الرسائل التي يرسلها البوت بالتتابع
const botMessages = ["طبونك", "طبتك", "ننيكك"];
let currentMessageIndex = 0;

// زمن الانتظار بين الردود
const cooldownTime = 2000;

// قائمة انتظار الرسائل
let messageQueue = [];
let isProcessing = false;

// آخر رسالة أرسلها البوت (لمنع التكرار)
let lastBotMessage = "";

// نسبة التشابه المطلوبة للتعرف على الكلمات المزخرفة
const similarityThreshold = 0.6;

// دالة تأخير
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

// دالة حساب Levenshtein Distance
function levenshteinDistance(a, b) {
    const matrix = Array.from({ length: a.length + 1 }, () => Array(b.length + 1).fill(0));

    for (let i = 0; i <= a.length; i++) {
        for (let j = 0; j <= b.length; j++) {
            if (i === 0) {
                matrix[i][j] = j;
            } else if (j === 0) {
                matrix[i][j] = i;
            } else if (a[i - 1] === b[j - 1]) {
                matrix[i][j] = matrix[i - 1][j - 1];
            } else {
                matrix[i][j] = Math.min(
                    matrix[i - 1][j] + 1, // حذف
                    matrix[i][j - 1] + 1, // إضافة
                    matrix[i - 1][j - 1] + 1 // استبدال
                );
            }
        }
    }

    return matrix[a.length][b.length];
}

// دالة حساب التشابه النسبي بين الكلمات
function getSimilarity(word1, word2) {
    const distance = levenshteinDistance(word1, word2);
    const maxLength = Math.max(word1.length, word2.length);
    return 1 - distance / maxLength;
}

// دالة التحقق من وجود كلمات مشابهة في قائمة الكلمات المحظورة
function containsSimilarWord(content, ignoreList) {
    const words = content.split(/\s+/); // تقسيم الرسالة إلى كلمات
    return words.some(word => 
        ignoreList.some(ignoredWord => getSimilarity(word.toLowerCase(), ignoredWord.toLowerCase()) >= similarityThreshold)
    );
}

// دالة للحصول على الرسالة التالية بالتتابع
function getNextBotMessage(userMessage) {
    let nextMessage;
    do {
        nextMessage = botMessages[currentMessageIndex];
        currentMessageIndex = (currentMessageIndex + 1) % botMessages.length;
    } while (nextMessage === userMessage || nextMessage === lastBotMessage); // ضمان عدم تكرار الرسالة أو مطابقتها لرسالة المستخدم

    lastBotMessage = nextMessage; // تحديث آخر رسالة أرسلها البوت
    return nextMessage;
}

// الردود الخاصة بالكلمات المحددة
const specificReplies = {
    'سلاش': 'طبونمك/',
    'Slash': 'طبونمك/',
    'س': 'طبونمك/',
    'ل': 'طبونمك/',
    'ا': 'طبونمك/',
    'ش': 'طبونمك/',
    'ارسال': 'طبونمك/',
    'خلي': 'طبونمك/',
    'ضع': 'طبونمك/',
    'بوضع': 'طبونمك/',
    'وضع': 'طبونمك/',
    'حط': 'طبونمك/',
    '/': 'طبونمك/',
    '\\': 'طبونمك/',
    '•': 'كىىمك•',
    'نقطة كبيرة': 'كىىمك•',
    '+': 'كىىختك+',
    'زائد': 'كىىختك+',
    '➕': 'كىىختك+',
    '-': 'كىىمك-',
    'ناقص': 'كىىمك-',
    '➖': 'كىىمك-',
    '٪': 'كىىمك ٪',
    '~': 'كىىختك~',
};

// معالجة قائمة الانتظار
async function processQueue() {
    if (isProcessing || messageQueue.length === 0) return;

    isProcessing = true;

    while (messageQueue.length > 0) {
        const { channel, messageContent, referenceMessage } = messageQueue.shift();

        await delay(cooldownTime); // الانتظار قبل الرد
        const options = {
            content: messageContent,
            allowedMentions: { repliedUser: true },
        };

        if (referenceMessage) {
            options.messageReference = { messageID: referenceMessage.id };
        }

        await bot.createMessage(channel.id, options);
    }

    isProcessing = false;
}

// حدث تشغيل البوت
bot.on('ready', () => {
    console.log("Bot is ready!");
});

// حدث استقبال الرسائل
bot.on('messageCreate', async (msg) => {
    if (msg.author.bot) return; // تجاهل رسائل البوتات الأخرى
    if (!allowedChannels.includes(msg.channel.id)) return; // تجاهل الرسائل من قنوات غير مسموح بها
    if (!allowedUsers.includes(msg.author.id)) return; // تجاهل الرسائل من مستخدمين غير مسموح لهم

    // التحقق إذا كانت الرسالة تحتوي على كلمات محظورة (مزخرفة أو مشفرة)
    if (containsSimilarWord(msg.content, ignoredWords)) {
        messageQueue.push({
            channel: msg.channel,
            messageContent: specialReply,
            referenceMessage: msg,
        });
        processQueue();
        return;
    }

    if (containsSimilarWord(msg.content, additionalIgnoreWords)) {
        console.log("تم تجاهل الرسالة لاحتوائها على كلمات إضافية محظورة.");
        return;
    }

    // التحقق من الكلمات الخاصة
    for (const [triggerWord, reply] of Object.entries(specificReplies)) {
        if (msg.content.includes(triggerWord)) {
            messageQueue.push({
                channel: msg.channel,
                messageContent: reply,
                referenceMessage: msg,
            });
            processQueue();
            return;
        }
    }

    // الرد على الصور
    if (msg.attachments && msg.attachments.length > 0) {
        messageQueue.push({
            channel: msg.channel,
            messageContent: imageReply,
            referenceMessage: msg,
        });
        processQueue();
        return;
    }

    // إضافة الرسالة التالية بالتتابع إلى قائمة الانتظار، مع التحقق من عدم تطابقها مع رسالة المستخدم
    const nextMessage = getNextBotMessage(msg.content);
    messageQueue.push({
        channel: msg.channel,
        messageContent: nextMessage,
        referenceMessage: msg,
    });

    processQueue(); // بدء معالجة الرسائل
});

// تجاهل الرسائل المعدلة
bot.on('messageUpdate', (msg, oldMsg) => {
    console.log("تم تجاهل رسالة معدلة.");
});

// تسجيل الدخول
bot.connect();
