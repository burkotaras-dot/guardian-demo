# Вівторок 01.09.2026 — звірка за планом "Вівторок" (Einstein) і "Середа" (Felix & Jupiter)

Окремий документ від `2026-09-01.md` (той — про полірування Roles & Permissions, план п'ятниці).
Цей — прицільна звірка: user попросив пройтись саме по пунктах, написаних у плані на **вівторок**
і **середу**, і зробити те, що не зроблено, нічого не пропускаючи.

## Вівторок — Einstein: define role, use cases, interaction states, action hierarchy

1. **Define Einstein's role** — зафіксовано в пам'яті й підтверджено клієнтом (Stephen Earl, 31.08):
   Einstein → Changes/Drift. Точка входу — Detected Drift / Change Detail, кнопка "Explain with AI"
   (`cdAiExplain`, `index.html`).
2. **Select use cases** — один use case, свідомо: пояснити конкретну зміну (чому вона сталась,
   чому важлива), не більше. Це вже узгоджено 28-31.08, повторної роботи не було.
3. **Implement interaction states** — Trigger → Loading → Result/Error → Retry → Dismiss, через
   спільний `_aiRunPanel`. Реалізовано 28.08, перевірено в наскрізному ревʼю 31.08 (Close/Escape/Retry
   по всіх 5 точках). Без змін сьогодні.
4. **Verify action hierarchy** — правило: AI-панель ніколи не забирає primary CTA сторінки; рішення
   (Approve/Flag/Skip, `#cd-exec-row`) лишається окремим, головним об'єктом на сторінці, AI —
   допоміжна панель під ним.

   **Тут і знайшовся пропуск.** Порівнявши текст `cdAiExplain` з тим самим правилом, застосованим у
   `pdAiExplain` (Jupiter) — той завжди закінчується `<strong>Recommendation:</strong>` реченням, що
   веде користувача до конкретної дії — виявилось, що `cdAiExplain` пояснював причину й вплив зміни,
   але **не давав рекомендації**, після якої зрозуміло, що робити далі. Це порушення того самого
   правила "AI-текст без зрозумілого наступного кроку" з плану середи ("Користувач не повинен
   отримувати AI-текст, після якого незрозуміло, що робити") — те саме правило застосовне і до
   вівторка (action hierarchy має бути *ясною*, а не лише не забирати CTA).

   **Виправлено сьогодні.** Додано речення, що прив'язує AI-висновок до реальних кнопок рядка
   `#cd-exec-row` (Approve / Flag / Skip):

   > "…similar nodes still have the setting enabled, which suggests this is a local change rather than
   > a policy-driven one. **Recommendation:** since it doesn't match an approved policy change, flag it
   > for investigation rather than approving it outright — only skip it if you already know the context
   > behind this node."

   Перевірено в прев'ю: `showView('change-detail')` → `cdAiExplain()` → текст панелі містить нове
   речення; темна тема — колір `<strong>` `rgb(203,213,225)`, узгоджений зі стилем інших AI-панелей;
   `preview_console_logs` — чисто. Стан скинуто (`cdAiDismiss()`, світла тема, dashboard).

## Середа — Felix & Jupiter: differentiate strands, place in product, integrate Jupiter, verify transitions

1. **Differentiate strands via Purpose → Entry point → Output → Available actions** — такої таблиці
   раніше не існувало як окремого документа (перевірено `grep` по репозиторію — знайдено лише текст
   самого скріншота плану, не структурований опис). Складено зараз:

   | | **Einstein** | **Bertha** | **Jupiter** |
   |---|---|---|---|
   | **Purpose** | Пояснити конкретну зміну — чому вона сталась і чому важлива | Допомогти діагностувати, чому скан провалився, і що зробити далі | Пояснити, чому конкретна перевірка політики провалена, і який вплив на compliance score |
   | **Entry point** | Detected Drift → Change Detail, кнопка "Explain with AI" (`cdAiExplain`) | Failed Scan, кнопка "Help me troubleshoot" (`sfAiTroubleshoot`) | Policy Detail, кнопка "Explain with AI" — **лише на FAILED-рядках** (`pdAiExplain`) |
   | **Output** | Причина зміни + чому вона поза базовою лінією групи + рекомендація (Approve/Flag/Skip) | Тип помилки скану (auth/connect/timeout/permission) + причина + рекомендований перший крок | Причина провалу перевірки + вплив на policy score + рекомендація виправлення |
   | **Available actions** | AI не виконує дію сам — рішення (Approve/Flag/Skip) лишається окремим, головним CTA сторінки | AI не виконує дію сам — рекомендує, куди дивитись першим (credential/firewall/account) | AI не виконує дію сам — рекомендує виправлення, Retry scan лишається окремою дією користувача |

   Спільне для всіх трьох: одна й та сама механіка (`_aiRunPanel`, Trigger→Loading→Result/Error→
   Retry→Dismiss), генерик-текст кнопок у UI (жодна назва strand-у ніде не показується користувачу),
   AI ніколи не виконує дію сам — завжди лише рекомендує, дію робить людина.

2. **Place Felix (Bertha) and Jupiter in the product, not as standalone screens** — обидва
   реалізовані як панель усередині наявної сторінки (Failed Scan card / Policy Detail row), не окремий
   екран і не модалка. Зроблено 28.08, без змін сьогодні.

3. **Integrate Jupiter into policy view, not standalone** — `pdAiExplain` живе всередині
   `v-policy-detail`, кнопка з'являється точково на FAILED-рядках перевірок, а не на окремій
   AI-сторінці. Зроблено 28-31.08.

4. **Verify Problem → AI insight → recommendation → user action transitions** — пройдено по всіх трьох:
   - **Bertha** (`sfAiTroubleshoot` + `SCAN_FAIL_TYPES[*].aiHelp`, 4 типи помилок) — кожен `aiHelp`-текст
     вже закінчувався конкретною порадою куди дивитись першим (наприклад "start with the credential
     itself before touching firewall or account settings") — відповідає вимозі без змін.
   - **Jupiter** (`pdAiExplain`) — вже мав `<strong>Recommendation:</strong>` речення з чіткою дією
     (poправити налаштування або відкрити related change, потім пересканувати) — відповідає без змін.
   - **Einstein** (`cdAiExplain`) — **не відповідав** (див. вівторок п.4 вище) — виправлено сьогодні,
     тепер усі три однаково завершуються дією.

## Підсумок

Єдина реальна прогалина за обома днями — відсутність рекомендації в Einstein — знайдена і закрита.
Решта пунктів вівторка й середи вже були виконані раніше (28-31.08) і сьогодні лише звірені проти
коду, без розбіжностей. Таблиця Purpose/Entry point/Output/Available actions, якої не існувало
окремим документом, складена в цьому файлі.

## Не закомічено

Правка `cdAiExplain` (додавання Recommendation) і фікс шапки `.perm-row.perm-head` — код змінено й
перевірено в прев'ю, але **не закомічено** (комітили раніше сьогодні окремим `feat(roles)` до цих
правок). Комітити/деплоїти — лише за окремим проханням.
