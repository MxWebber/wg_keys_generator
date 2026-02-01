# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

WireGuard Key Generator — одностраничное веб-приложение для локальной генерации пар ключей WireGuard. Приложение использует нативную JavaScript реализацию Curve25519 без внешних зависимостей.

## Architecture

### Single-File Application
Проект реализован как единый HTML-файл (`index.html`), содержащий:
- HTML структуру
- Встроенные CSS стили с CSS Custom Properties для тем
- JavaScript код (криптография + UI логика)

### Core Components

#### 1. Cryptography Module (Curve25519)
Расположение: `<script>` секция, функции `mod`, `modpow`, `x25519`, `generateKeypair`

**Ключевые функции:**
- `clamp(bytes)` — применяет clamping к приватному ключу согласно спецификации
- `x25519(k, u)` — выполняет скалярное умножение на кривой Curve25519
- `generateKeypair()` — основная функция генерации пары ключей

**Важно:** Криптографическая логика использует BigInt для точных вычислений с большими числами. Модификации этой части требуют понимания RFC 7748.

#### 2. UI Logic
Расположение: `<script>` секция, переменные `genCount`, `currentKeys` и обработчики событий

**Основные функции:**
- `renderKeys()` — создаёт DOM-элементы для отображения ключей
- `generate()` — инициирует генерацию новой пары ключей
- `copyToClipboard()` — копирует ключ с fallback для старых браузеров
- `saveToFile()` — создаёт текстовый файл для скачивания

#### 3. Theme System
Темы реализованы через CSS Custom Properties (`--bg`, `--text-primary`, и т.д.) с классами `body.light` и `body.dark`.

**Переключение темы:** JavaScript toggle добавляет/убирает классы `light`/`dark` на элементе `<body>`.

## Development Workflow

### Testing Changes
Просто откройте `index.html` в браузере:
```bash
open index.html
```

Для тестирования с HTTP-сервером:
```bash
python -m http.server 8000
# открыть http://localhost:8000
```

### Browser DevTools
Используйте Console для отладки:
```javascript
// Генерация пары ключей вручную
const keys = generateKeypair();
console.log(keys);

// Проверка криптографических функций
const testKey = new Uint8Array(32).fill(1);
console.log(x25519(toBI(clamp(testKey)), 9n));
```

## Common Modifications

### Adding New Features

**Добавление нового UI элемента:**
1. Добавьте HTML в секцию `.content`
2. Добавьте стили в оба раздела (`body.light` и `body.dark`)
3. Добавьте обработчики событий в JavaScript секции

**Изменение криптографии:**
⚠️ Не модифицируйте криптографические функции без глубокого понимания Curve25519 и RFC 7748. Неправильная реализация приведёт к небезопасным ключам.

### Styling Changes

Все стили находятся в `<style>` секции. Для изменения цветов:
1. Найдите CSS Custom Property в `body.light` или `body.dark`
2. Измените значение
3. Обновите страницу в браузере

Пример:
```css
body.light {
  --accent-pub: #0891b2;  /* Цвет акцента для публичного ключа */
}
```

## Code Style Guidelines

### JavaScript
- Используйте `const` для неизменяемых переменных
- Функции криптографии используют BigInt (суффикс `n`)
- Callback-based асинхронность для UI операций
- Event listeners добавляются после определения функций

### CSS
- CSS Custom Properties для всех цветов и размеров
- Transitions на 0.3-0.4s для плавности
- Используйте `rem` и `px` последовательно
- Группируйте связанные стили вместе

### HTML
- Семантические теги где возможно
- SVG иконки встроены inline
- Data attributes не используются (состояние через CSS классы)

## Security Considerations

1. **Не добавляйте внешние зависимости** — это увеличивает поверхность атаки
2. **Не модифицируйте `crypto.getRandomValues()`** — это единственный источник энтропии
3. **Не передавайте ключи по сети** — вся генерация локальная
4. **Проверяйте clamping** — функция `clamp()` критична для безопасности

## File Structure

```
wg_keys_generator/
├── index.html          # Единственный исходный файл
├── README.md          # Пользовательская документация (на русском)
└── CLAUDE.md          # Этот файл (руководство для разработчиков)
```

## Dependencies

**Внешние зависимости:** Нет
**Шрифты:** Google Fonts (Orbitron, Share Tech Mono) — загружаются через CDN
**APIs:** Web Crypto API (встроенный в браузер)

## Browser Compatibility

Требуется поддержка:
- BigInt (Chrome 67+, Firefox 68+, Safari 14+)
- Web Crypto API (все современные браузеры)
- CSS Custom Properties (все современные браузеры)
- ES6+ (const, let, arrow functions, template literals)

## Deployment

Приложение статичное и не требует сборки. Для деплоя:

**GitHub Pages:**
```bash
git add index.html
git commit -m "Deploy to GitHub Pages"
git push origin main
```
Затем включите GitHub Pages в настройках репозитория.

**Netlify/Vercel:**
Просто подключите репозиторий — сборка не требуется.

## Testing Checklist

При внесении изменений проверьте:

- [ ] Генерация ключей работает
- [ ] Кнопки Copy и Save функционируют
- [ ] Переключение темы работает плавно
- [ ] UI responsive на мобильных устройствах
- [ ] Нет ошибок в Browser Console
- [ ] Ключи корректно закодированы в Base64
- [ ] Счётчик генераций обновляется
- [ ] Анимации работают без задержек

## Known Limitations

1. **Single Page Application** — нет роутинга или множественных страниц
2. **No Persistence** — ключи не сохраняются между сессиями (это фича безопасности)
3. **No Key Validation** — нет проверки корректности сгенерированных ключей
4. **Desktop-First** — UI оптимизирован для десктопа, но работает на мобильных

## Performance Notes

- Генерация ключей занимает ~1-5ms на современном CPU
- Искусственная задержка 340ms добавлена для UI feedback
- Операции DOM минимизированы в `renderKeys()`
- CSS анимации используют GPU-ускорение (transform, opacity)
