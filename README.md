# Multi-Agent RAG Banking Chatbot

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?logo=pytorch&logoColor=white) ![License](https://img.shields.io/badge/License-MIT-green)

پروژه‌ی درس NLP پیشرفته — یک دستیار بانکی چندنوبتی (multi-turn) به زبان فارسی که تشخیص نیت (Intent Classification)، بازیابی اطلاعات (RAG) و پرکردن اسلات (Slot Filling) را در یک پایپ‌لاین واحد ترکیب می‌کند.

## معرفی پروژه

این پروژه یک چت‌بات بانکی end-to-end است که می‌تواند درخواست‌های کاربر مانند انتقال کارت‌به‌کارت، پرداخت قبض، مسدودسازی کارت و پاسخ به سوالات متداول (FAQ) را در یک گفتگوی چندنوبتی مدیریت کند. پایپ‌لاین شامل شش فاز است:

| فاز | توضیح |
|---|---|
| ۱ | NLU — فاین‌تیون ParsBERT برای طبقه‌بندی نیت + fallback با LLM |
| ۲ | RAG — بازیابی برداری با Qdrant (in-memory) و embedding چندزبانه |
| ۳ | Hybrid Search — ترکیب BM25 (sparse) و بازیابی متراکم (dense) |
| ۴ | Pipeline — چت‌بات کامل چندنوبتی با اعتبارسنجی Luhn/IBAN، مدیریت نشست، مرحله‌ی تایید تراکنش |
| ۵ | ارزیابی — RAGAS (Faithfulness, Context Precision, Answer Relevancy) و LLM-as-Judge |
| ۶ | رابط کاربری Gradio |

پایپ‌لاین اصلی (فاز ۴) پس از تحلیل ۵۰ سناریوی دستی شکار باگ (`manual_bug_hunt_results.jsonl` و نسخه‌ی extra) اصلاح شده تا مشکلاتی مانند گم‌شدن اسلات‌ها هنگام تغییر موضوع، تشخیص شماره کارت پاره‌شده بین چند پیام و ابهام در تایید تراکنش را برطرف کند.

## ساختار ریپازیتوری

```
.
├── README.md
├── requirements.txt
├── .gitignore
├── .env.example
├── LICENSE
├── notebooks/
│   └── Banking_Chatbot.ipynb     # نوت‌بوک اصلی پروژه (شامل سلول‌های Setup برای Colab)
├── data/                          # دیتاست الزامی درس (۶ فایل)
├── results/                       # خروجی سناریوهای تست و متریک‌های ارزیابی
└── docs/
    └── demo_script.md             # اسکریپت کامل دموی ۵ دقیقه‌ای
```

## داده‌های مورد نیاز

فایل‌های دیتاست الزامی درس در همین ریپازیتوری، در مسیر `data/` قرار دارند:

- `intent_train.json`, `intent_val.json`, `test_intent.json`
- `test_rag.json`, `banking_kb.csv`, `slots.xlsx`

نوت‌بوک به‌طور خودکار در Google Colab این مسیر را تشخیص می‌دهد (`DATA_DIR = COLAB_DATA_DIR if IN_COLAB else ...`). برای اجرا روی Colab کافی است در سلول Setup 2/3، فایل `datasets.zip` (شامل همین ۶ فایل) را آپلود کنید.

## نصب و اجرا

```bash
git clone https://github.com/USERNAME/REPO-NAME.git
cd REPO-NAME
pip install -r requirements.txt
```

سپس یک فایل `.env` بسازید (نمونه در `.env.example`) و کلید API را در آن قرار دهید:

```
GOOGLE_API_KEY=your_key_here
```

نوت‌بوک `notebooks/Banking_Chatbot.ipynb` را در Google Colab باز و اجرا کنید (توصیه می‌شود). سلول‌های Setup در ابتدای نوت‌بوک به‌طور خودکار پکیج‌ها را نصب می‌کنند و داده را از پوشه‌ی `data/` می‌خوانند. برای اجرا از GPU (مثلاً T4) استفاده کنید.

## نمونه‌ی ورودی و خروجی

**ورودی:**
```
می‌خواهم ۵۰۰۰۰۰ تومان از کارت 6037997522224444 به کارت 5022291000111222 انتقال دهم
```

**رفتار مورد انتظار:** تشخیص نیت `transfer_card_to_card`، استخراج اسلات‌های مبلغ/کارت مبدا/کارت مقصد، اعتبارسنجی با الگوریتم Luhn، و درخواست تایید نهایی از کاربر قبل از ثبت تراکنش.

## دموی زنده

رابط کاربری تعاملی با Gradio در فاز ۶ نوت‌بوک ساخته می‌شود و یک لینک عمومی موقت (`https://xxxx.gradio.live`) تولید می‌کند. اسکریپت کامل دموی ۵ دقیقه‌ای (شامل پیام‌های نمونه و توضیحات) در [`docs/demo_script.md`](docs/demo_script.md) موجود است.

## نتایج و ارزیابی

خروجی‌های ارزیابی در پوشه‌ی `results/` ذخیره می‌شوند:
- `test_predictions.jsonl` — پیش‌بینی‌های طبقه‌بند نیت روی داده‌ی تست
- `retrieval_metrics.json` — Precision@K / Recall@K / F1@K برای RAG
- `manual_bug_hunt_results.jsonl` و `manual_bug_hunt_results_extra.jsonl` — نتایج ۵۰ سناریوی چالشی چندنوبتی برای شکار باگ

خلاصه‌ی متریک‌های اصلی (Accuracy، F1 طبقه‌بندی نیت، متریک‌های RAGAS) در بخش «Summary» نوت‌بوک چاپ می‌شود.

## نکته‌ی امنیتی

هیچ کلید API یا اطلاعات حساسی نباید در کد commit شود. تمام کلیدها باید از طریق فایل `.env` (که در `.gitignore` قرار دارد) یا Secrets پلتفرم اجرا (Kaggle/Colab) خوانده شوند.



