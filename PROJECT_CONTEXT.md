# Project Context — Файна Перукарня + Studio Dashboard

Документ для відновлення контексту у нових сесіях. Оновлюй після кожної великої зміни.

**Last updated:** 2026-04-20

---

## 1. Що це за проєкт

Андрій Красік (фрилансер-таргетолог) веде маркетинговий проєкт для салону «Файна Перукарня» (Поділ, Київ). Клієнтка — Людмила Чумак. Запит прийшов через Freelancehunt 20 квіт 2026.

У межах цієї роботи побудовано систему з трьох взаємопов'язаних частин:
1. **Публічний аудит** бізнесу клієнтки (нейтральна аналітика, без продажу)
2. **Публічний прогрес-трекер** проєкту (з адмін-режимом, оплатами, запитами до клієнта)
3. **Studio Dashboard** — кабінет Андрія з усіма його клієнтськими проєктами

Плюс:
- Telegram-бот `@faina_perukarnya_bot` у групі «Файна Перукарня звіт» шле сповіщення про ключові події
- Клод-скіл `client-audit` для повторення flow з новими клієнтами

---

## 2. Архітектура (папки, репо, URL)

| Що | Локальна папка | Git repo | Публічна URL |
|---|---|---|---|
| Аудит бізнесу | `/Users/andreykrasik/Таргет для салона красоты./index.html` | `krasprod/fajna-audit` | https://krasprod.github.io/fajna-audit/ |
| Прогрес-трекер | те саме, файл `progress.html` | той самий repo | https://krasprod.github.io/fajna-audit/progress.html |
| Дані проєкту | те саме, файл `data.json` | той самий repo | — (читається прогрес-сторінкою) |
| Studio Dashboard | `/Users/andreykrasik/krasprod-studio/index.html` | `krasprod/studio` | https://krasprod.github.io/studio/ |
| Список проєктів | те саме, файл `projects.json` | той самий repo | — |
| Скіл аудиту | `/Users/andreykrasik/.claude/skills/client-audit/` | — | — |

⚠ Назва папки Fajna містить пробіли + крапку в кінці: `Таргет для салона красоты.` Завжди обгортай у лапки у bash.

---

## 3. Admin-режим і URL-ключі

Адмін-режим вмикається параметром `?admin=<key>`. Ключі:

| Проєкт | URL-ключ |
|---|---|
| Studio Dashboard | `krasik2026` |
| Fajna progress.html | `fajna2026` |

Повні адмін-посилання:
- Studio: https://krasprod.github.io/studio/?admin=krasik2026
- Fajna progress: https://krasprod.github.io/fajna-audit/progress.html?admin=fajna2026

---

## 4. GitHub PAT (як зберігати зміни)

Редагування інлайн і «Зберегти» працює через GitHub Contents API. Потрібен Fine-grained Personal Access Token з правом **Contents: Read and write** на відповідний репо.

Токен вводиться один раз при першому сейві → зберігається у `localStorage`:
- `gh_pat` — для `fajna-audit` (прогрес-трекер)
- `gh_pat_studio` — для `studio`

Створити: https://github.com/settings/tokens?type=beta

---

## 5. Структура `data.json` (прогрес-трекер)

```jsonc
{
  "project": { "name", "client", "started", "last_updated", "currency" },
  "current_status": "текст, який видно в hero",
  "contact": { "telegram", "telegram_label", "email", "formsubmit" },
  "notifications": {
    "telegram_bot_token": "8264822499:...",
    "telegram_chat_id": "-5141489549"
  },
  "phases": [
    {
      "id": 1,
      "title": "...",
      "priority": "critical" | "high" | "medium",
      "description": "...",
      "price": 3500,
      "price_note": "що входить в суму",
      "paid": false,
      "paid_date": "",
      "paid_amount": 0,
      "client_requests": [
        {
          "id": "1.a",
          "title": "...",
          "description": "...",   // supports \n for line breaks (white-space: pre-wrap)
          "status": "pending" | "received" | "done",
          "response": "",          // заповнюється адміном після отримання
          "response_date": ""
        }
      ],
      "tasks": [
        { "id": "1.1", "title": "...", "done": false }
      ]
    }
  ],
  "metrics": [
    { "key", "label", "start", "current", "goal" }
  ],
  "events": [
    { "date", "time", "text", "type": "milestone" | "progress" }
  ]
}
```

---

## 6. Структура `projects.json` (studio)

```jsonc
{
  "studio": { "owner", "role", "city", "last_updated" },
  "projects": [
    {
      "slug": "fajna-audit",
      "name": "Файна Перукарня",
      "client": "Людмила Чумак",
      "niche": "Салон краси · Поділ, Київ",
      "status": "active" | "paused" | "archived",
      "started": "2026-04-20",
      "repo": "krasprod/fajna-audit",
      "admin_key": "fajna2026",
      "data_path": "data.json",
      "links": { "audit": "...", "progress": "..." },
      "notes": ""
    }
  ]
}
```

Studio підтягує `data.json` кожного проєкту через GitHub API, рахує % виконаних задач і кількість запитів, що чекають на клієнта.

---

## 7. Telegram-бот

- **Bot:** `@faina_perukarnya_bot` (створений через @BotFather)
- **Token:** зберігається в **localStorage** як `tg_bot_token_fajna`. **НЕ в `data.json`** — оригінальний токен витік (лежав у публічному JSON, його знайшов GitHub-сканер і шле крапки в групу), revoked 13 трав 2026.
- **Group:** «Файна Перукарня звіт», chat_id: `-5141489549` (це в `data.json`, ок — chat_id без токена нічого не дає)
- **Учасники групи:** Андрій, бот, (заплановано) Людмила
- **Як налаштувати в новому браузері:** progress.html?admin=fajna2026 → адмін-панель → «🤖 Telegram-токен» → вводиш токен один раз. Зберігається лише локально.
- **notifyTelegram() тригерить ТІЛЬКИ якщо `state.admin === true`** — клієнт ніколи нічого не шле. Замість Telegram у клієнта — Formsubmit email.

Сповіщення у групу шле прогрес-сторінка (JS → Telegram Bot API). Тригери:
1. 📍 Оновлення статусу проєкту — коли адмін змінює поле `current_status`
2. 📩 Нова відповідь від клієнта — коли клієнт надіслав відповідь (наразі ТІЛЬКИ якщо адмін теж відкрив сторінку — інакше email через Formsubmit)
3. 💰 Етап оплачено — коли адмін позначив етап оплаченим
4. ✅ Етап завершено — коли всі задачі етапу стали `done`

Клієнт-форма шле відповідь через **Formsubmit.co** (email на krasprod.nk@gmail.com). Telegram-сповіщення про відповідь спрацює лише якщо адмін відкрив сторінку, але email прийде завжди.

⚠ Якщо хтось знову буде слати спам у групу — `/revoke` у @BotFather, отримати новий токен, оновити в адмін-панелі через кнопку «🤖 Telegram-токен». Токен НІКОЛИ не пушити у `data.json`.

Як витягнути chat_id (коли група новостворена):
```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getUpdates"
```
Шукати `"chat":{"id":...}` у групових повідомленнях.

---

## 8. Етапи проєкту Fajna (6 штук, 29 задач, 24 000 грн)

1. **Оптимізація Google Бізнес-профілю** · critical · 3 500 грн · 7 задач
2. **Запуск таргетованої реклами у Meta** · critical · 8 000 грн · 6 задач
3. **Перезапуск Instagram** · high · 4 500 грн · 6 задач
4. **Робота з репутацією** · high · 2 500 грн · 3 задачі
5. **Google Ads на локальні запити** · medium · 4 000 грн · 4 задачі
6. **Facebook — технічне підтягування** · medium · 1 500 грн · 3 задачі

Усі ціни — одноразова оплата за setup. Подальше ведення — за окремим тарифом (окрім етапу 4 і 6, які truly one-time). Рекламний бюджет для Meta (2000-3000 грн) і Google Ads (200-300 грн/день) клієнтка сплачує напряму платформам.

---

## 9. Client-audit skill

Скіл `client-audit` лежить у `~/.claude/skills/client-audit/`. Використання:
- Коли користувач каже «зроби аудит для клієнта X» — скіл генерує нейтральний маркетинг-аудит (без пакетів, без CTA) як HTML і деплоїть на GitHub Pages за патерном `krasprod/{slug}-audit`.
- Шаблон: `assets/template.html` (копія Fajna аудиту).
- Ключові правила: не додавати пакети послуг, не писати «я зроблю», ставити чесні оцінки 0-10.

---

## 10. Ключові команди (копі-пейст)

**Commit + push до fajna-audit:**
```bash
cd "/Users/andreykrasik/Таргет для салона красоты." && \
git add data.json progress.html && \
git -c user.email=krasprod.nk@gmail.com -c user.name="Andrii Krasik" \
  commit -m "..." && git push
```

**Commit + push до studio:**
```bash
cd /Users/andreykrasik/krasprod-studio && \
git add . && \
git -c user.email=krasprod.nk@gmail.com -c user.name="Andrii Krasik" \
  commit -m "..." && git push
```

**Створити новий клієнтський проєкт (аудит):**
```bash
cd "<client-folder>"
git init -b main
git add index.html
git -c user.email=krasprod.nk@gmail.com -c user.name="Andrii Krasik" \
  commit -m "Marketing audit for <Name>"
gh repo create <slug>-audit --public --source=. --push
gh api -X POST repos/krasprod/<slug>-audit/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

---

## 11. Що ще можна додати (ідеї на майбутнє)

- [ ] Окремий бот або single-bot для всіх проєктів (зараз один бот = один проєкт)
- [ ] Вкладка «Документи» на прогрес-сторінці (договір, акти, реквізити)
- [ ] Автоматичне створення нового client-проєкту з studio dashboard (зараз треба вручну створювати repo + data.json через Claude Code)
- [ ] Сторінка «Ведення» — після завершення setup-етапів перемикання на щомісячний retainer-режим з іншою структурою даних
- [ ] Історія всіх оплат з інвойсами (зараз лише статус paid/unpaid)
- [ ] Кнопка «Оплатити» (Monobank jar / LiqPay) прямо на етапі
- [ ] Переклад на англ/рос для клієнтів з інших ринків
- [ ] Автоматичне оновлення метрик (`google_reviews`, `ig_followers`) — скрейпер + cron
- [ ] Діалог з клієнтом прямо на сторінці (чат через Telegram Bot API, не тільки запити)

---

## 12. Як відновити контекст у новій сесії

Скажи новому Клоду щось типу:
> Перечитай `/Users/andreykrasik/Таргет для салона красоты./PROJECT_CONTEXT.md`.
> Продовжуємо роботу над проєктом Fajna + Studio Dashboard.

Цього достатньо, щоб новий Клод зрозумів архітектуру, паролі, структури даних і куди пушити.
