# 🎙️ Local Video/Audio Transcriber with Speaker Diarization

Локальный транскрибатор видео и аудио в текст с автоматическим разделением голосов спикеров. Работает полностью офлайн на GPU - никакие данные не отправляются в облако.

Лично использую его для выполнения конспеков курсов, подкастов и тд

Можно скачать подкаст (сами знаете откуда), загрузить сюда и получить текст. Затем отредакторовать и загрузить в LLM, чтобы получить суммаризацию и тему разговора.

**В процессе**:
- подключение LLM-агента
- возможно, подключу Obsidian для построения карты знаний

---

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)
![CUDA](https://img.shields.io/badge/CUDA-12.4+-76B900?logo=nvidia&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)

---

## ✨ Возможности

- **Транскрипция** - модель Whisper large-v3-turbo через [whisper](https://huggingface.co/openai/whisper-large-v3-turbo) 
- **Диаризация** - автоматическое определение и разделение спикеров через [pyannote.audio](https://github.com/pyannote/pyannote-audio)
- **GPU-ускорение** - все вычисления на видеокарте (если мощная, то все это выполняется быстро, у меня 5070 и 40 мин. подкаст переводит 1.5 мин.)
- **Любые форматы** - видео (mp4, mkv, avi, mov) и аудио (mp3, wav, flac, ogg)
- **Экспорт** - текстовый файл с таймкодами + SRT-субтитры
- **100% локально** - без API-ключей, лимитов и отправки данных. Можно было бы подключить любой агент, но в дальшейшем развитии проекта. 

---


## 🚀 Быстрый старт

### 1. Скопировать репозиторий

```bash
git clone https://github.com/YOUR_USERNAME/Transcriber.git
cd Transcriber
```

### 2. Создать виртуальное окружение

```bash
python -m venv venv
venv\Scripts\activate
```

### 3. Установить библиотеки

```bash
pip install torch
pip install torchaudio --index-url https://download.pytorch.org/whl/cu124
pip install ffmpeg
pip install faster-whisper 
pip install pyannote.audio
```

### 4. Получить токен HuggingFace (это бесплатно)

1. Зарегистрироваться на [huggingface.co](https://huggingface.co/join)
2. Принять условия использования моделей, написать любой ВУЗ и его сайт:
   - [pyannote/speaker-diarization-3.1](https://huggingface.co/pyannote/speaker-diarization-3.1)
   - [pyannote/segmentation-3.0](https://huggingface.co/pyannote/segmentation-3.0)

3. Создать токен: [Settings → Tokens](https://huggingface.co/settings/tokens)
   - обязательно нужно добовать модели сверху в токен и уже после этого создать токег

### 5. Запустить ноутбук (можно на любом IDE)

```bash
jupyter notebook transcriber_diarization.ipynb
```

Обязательно напиши свой HF-токен и путь к файлу в ячейке **«Настройки»**, затем выполнить все ячейки по порядку. Да, можно было сразу написать скрипт, но... это моя прога)

---

## 📂 Структура проекта

```
transcriber-diarization/
├── transcriber_diarization.ipynb   # Основной ноутбук
├── README.md
├── LICENSE
└── examples/
    └── sample_output.txt           # Пример результата
```

---

## 📄 Пример результата

```
[00:00:02 → 00:00:08] SPEAKER_00:
Добрый день, коллеги. Давайте начнём совещание.

[00:00:09 → 00:00:15] SPEAKER_01:
Да, у меня есть обновления по проекту. На прошлой неделе мы завершили тестирование.

[00:00:16 → 00:00:22] SPEAKER_00:
Отлично. Какие результаты?
```

---

## ⚙️ Параметры

| Параметр | По умолчанию | Описание |
|----------|-------------|----------|
| `INPUT_FILE` | `"video.mp4"` | Путь к видео или аудиофайлу |
| `HF_TOKEN` | — | Токен HuggingFace (обязателен) |
| `LANGUAGE` | `None` | Язык (`None` = авто, `"ru"`, `"en"`, `"de"` …) |
| `NUM_SPEAKERS` | `None` | Количество спикеров (`None` = авто) |
| `WHISPER_MODEL` | `"large-v3-turbo"` | Модель Whisper |
| `OUTPUT_FILE` | `"transcript.txt"` | Файл для сохранения |

---

## 🧠 Как это работает

```
Видео/Аудио
    │
    ▼
┌──────────┐
│  FFmpeg   │──→ WAV 16kHz mono, иначе будут проблемы с чанками
└──────────┘
    │
    ├──────────────────────┐
    ▼                      ▼
┌──────────┐       ┌──────────────┐
│ pyannote │       │ faster-whisper│
│ (GPU)    │       │ (GPU)        │
└──────────┘       └──────────────┘
    │                      │
    │  кто когда           │  что сказано
    │  говорит             │  (+ таймкоды)
    ▼                      ▼
┌──────────────────────────────┐
│     Склейка результатов      │
│  спикер + таймкод + текст    │
└──────────────────────────────┘
    │
    ├──→ transcript.txt
    └──→ transcript.srt
```

---

## ❓ FAQ

<details>
<summary><b>PyTorch не видит GPU</b></summary>

Убедись, что установлена версия с CUDA. Проверь:

```python
import torch
print(torch.cuda.is_available())
```

Если `False` — переустанови PyTorch с правильным индексом:

```bash
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cu124
```

Для новейших карт (RTX 50xx) может потребоваться nightly-версия.
</details>

<details>
<summary><b>Ошибка 401 или 403 при загрузке pyannote</b></summary>

Токен HuggingFace невалиден или ты не принял условия использования моделей. Перейди по ссылкам из раздела «Быстрый старт» и нажми «Agree» на странице каждой модели. Также добавь в токен данные модели.
</details>

<details>
<summary><b>FFmpeg не найден</b></summary>

Установи FFmpeg (без этого ничего не будет работать и не забудь добавить в PATH):

- **Windows:** `winget install ffmpeg` или скачай с [ffmpeg.org](https://ffmpeg.org/download.html)
- **Linux:** `sudo apt install ffmpeg`
- **macOS:** `brew install ffmpeg`
</details>

<details>
<summary><b>Не хватает VRAM</b></summary>

Попробуй модель поменьше:

```python
WHISPER_MODEL = "medium"  # ~5 GB VRAM вместо ~6 GB
```

Или используй `compute_type="int8"` вместо `"float16"`.
</details>

---
