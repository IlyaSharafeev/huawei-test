<template>
    <div class="quote-app">
        <div class="quote-app__content">
            <p v-if="error" class="quote-app__error">{{ error }}</p>
            <p v-else class="quote-app__text">{{ quote.content }}</p>
            <p v-if="quote.author" class="quote-app__author">— {{ quote.author }}</p>
            <button @click="fetchQuote" class="quote-app__button">Отримати нову цитату</button>
            <button @click="copyToClipboard" v-if="quote.content" class="quote-app__button">Копіювати цитату</button>
        </div>

        <div class="quote-app__settings">
            <h3 class="quote-app__settings-title">Налаштування генерації</h3>
            <label class="quote-app__label">
                Довжина:
                <select v-model="quoteLength" @change="updateLengthParams" class="quote-app__select">
                    <option value="">Будь-яка</option>
                    <option value="short">Коротка</option>
                    <option value="medium">Середня</option>
                    <option value="long">Довга</option>
                </select>
            </label>
            <label class="quote-app__label">
                Тема:
                <input v-model="quoteTag" placeholder="Введіть тему (напр. 'inspire')" class="quote-app__input" />
            </label>
        </div>

        <div v-if="quoteHistory.length" class="quote-app__history">
            <h3 class="quote-app__history-title">Історія цитат</h3>
            <ul class="quote-app__history-list">
                <li v-for="(item, index) in quoteHistory" :key="index" class="quote-app__history-item">
                    <span class="quote-app__history-quote">"{{ item.content }}" — {{ item.author }}</span>
                    <button @click="removeQuote(index)" class="quote-app__history-remove">×</button>
                </li>
            </ul>
            <button @click="clearHistory" class="quote-app__clear-history">Очистити історію</button>
        </div>

        <div class="quote-app__share">
            <h3 class="quote-app__share-title">Поділитися</h3>
            <button @click="shareToTelegram" class="quote-app__share-button">Telegram</button>
            <button @click="shareToTwitter" class="quote-app__share-button">X</button>
            <button @click="shareToLinkedIn" class="quote-app__share-button">Linkedin</button>
        </div>
    </div>
</template>

<script lang="ts" setup>
import axios from 'axios';
import { onMounted, ref, watch } from 'vue';

const quote = ref({});
const error = ref(null);
const quoteHistory = ref<any[]>(JSON.parse(localStorage.getItem('quoteHistory') || '[]'));
const quoteLength = ref('short');
const lengthParams = ref({ minLength: 0, maxLength: 50 });
const quoteTag = ref('')

// Слежение за историей цитат и сохранение в localStorage
watch(quoteHistory, (newHistory) => {
    localStorage.setItem('quoteHistory', JSON.stringify(newHistory));
});

// Обновление параметров длины цитаты в зависимости от выбора
const updateLengthParams = () => {
    if (quoteLength.value === 'short') {
        lengthParams.value = { minLength: 0, maxLength: 50 };
    } else if (quoteLength.value === 'medium') {
        lengthParams.value = { minLength: 100, maxLength: 140 };
    } else if (quoteLength.value === 'long') {
        lengthParams.value = { minLength: 140, maxLength: 200 };
    }
};

const fetchQuote = async () => {
    error.value = null;

    const params = {
        tags: quoteTag.value,
        minLength: lengthParams.value.minLength,
        maxLength: lengthParams.value.maxLength,
    };

    try {
        const { data } = await axios.get('https://api.quotable.io/quotes/random', { params });

        if (!data.length) {
            throw new Error("Помилка при отриманні запиту");
        }

        const newQuote = data[0];
        quote.value = newQuote;
        quoteHistory.value.unshift(newQuote);
    } catch (e) {
        error.value = "Помилка при завантаженні цитати. Спробуйте ще раз.";
    }
};

const copyToClipboard = () => {
    navigator.clipboard.writeText(`"${quote.value.content}" — ${quote.value.author}`);
    alert('Цитата скопійована в буфер обміну!');
};

const shareToTelegram = () => {
    const telegramUrl = `https://t.me/share/url?url=${encodeURIComponent(
            quote.value.content
    )} — ${encodeURIComponent(quote.value.author)}`;
    window.open(telegramUrl, '_blank');
};

const shareToTwitter = () => {
    const quoteText = encodeURIComponent(`"${quote.value.content}" — ${quote.value.author}`);
    const url = `https://twitter.com/intent/tweet?text=${quoteText}`;
    window.open(url, '_blank');
};

const shareToLinkedIn = () => {
    const quoteText = encodeURIComponent(`"${quote.value.content}" — ${quote.value.author}`);
    const url = `https://www.linkedin.com/sharing/share-offsite/?title=${quoteText}`;
    window.open(url, '_blank');
};

const removeQuote = (index: number) => {
    quoteHistory.value.splice(index, 1);
};

const clearHistory = () => {
    quoteHistory.value = [];
    localStorage.removeItem('quoteHistory');
};

onMounted(() => {
    fetchQuote();
});
</script>

<style scoped>
.quote-app {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    font-family: Arial, sans-serif;
    max-width: 500px;
    margin: auto;
    background-color: #f9f9f9;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}
.quote-app__content {
    text-align: center;
    margin-bottom: 20px;
}
.quote-app__error {
    color: #ff4d4d;
    font-weight: bold;
}
.quote-app__text {
    font-size: 1.2em;
    color: #333;
}
.quote-app__author {
    font-style: italic;
    color: #777;
    margin-top: 5px;
}
.quote-app__button {
    margin: 10px 5px;
    padding: 8px 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
.quote-app__button:hover {
    background-color: #0056b3;
}
.quote-app__settings,
.quote-app__share {
    margin: 20px 0;
    width: 100%;
}
.quote-app__settings-title,
.quote-app__share-title {
    font-size: 1.1em;
    margin-bottom: 10px;
    color: #333;
}
.quote-app__label {
    display: block;
    margin-bottom: 10px;
    color: #555;
}
.quote-app__select,
.quote-app__input {
    width: 100%;
    padding: 6px;
    margin-top: 5px;
    border: 1px solid #ccc;
    border-radius: 4px;
}
.quote-app__share-button {
    margin: 5px;
    padding: 8px 16px;
    background-color: #3b5998;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
.quote-app__share-button:hover {
    background-color: #2a406e;
}
.quote-app__history {
    margin-top: 20px;
    width: 100%;
}
.quote-app__history-title {
    font-size: 1.1em;
    color: #333;
    margin-bottom: 10px;
}
.quote-app__history-list {
    list-style: none;
    padding: 0;
}
.quote-app__history-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #fff;
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    margin-bottom: 5px;
}
.quote-app__history-quote {
    color: #333;
}
.quote-app__history-remove {
    background: none;
    border: none;
    color: #ff4d4d;
    font-size: 30px;
    cursor: pointer;
    padding: 0 15px;
}
.quote-app__clear-history {
    display: block;
    margin: 20px auto;
    padding: 8px 16px;
    background-color: #dc3545;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}
.quote-app__clear-history:hover {
    background-color: #c82333;
}
</style>