# Проект-план разработки

Документ описывает примерный план разработки системы
с учётом актуальной доменной модели
(Products, Inventory, Recipes, Users, Procurement, Moderation).

Цель — получить устойчивый core-продукт,
который можно развивать в сторону enterprise,
маркетплейсов и creator-economy без переписывания архитектуры.

---

## 0. Принципы планирования

### Что считается MVP

MVP — это замкнутый операционный цикл:

Продукты → Инвентаризация → Заказы → Базовая аналитика

В MVP не входят:
- маркетплейс
- creator economy
- NFT / лицензирование
- биллинг и платежи
- сложные enterprise-иерархии

---

### Ключевые допущения

- Небольшая команда
- Backend + iPad / Web
- Offline-first (iPad) — обязательное требование
- Историческая целостность данных важнее UX-деталей
- Доменные контракты важнее скорости UI-разработки

---

## 1. Фаза 0 — Архитектура и доменная фиксация

### Цель
Зафиксировать доменную модель так,
чтобы в дальнейшем не ломать базу данных и API.

### Задачи
- Products Domain (RU / EN)
- Inventory Domain (RU / EN)
- Recipes Domain (RU / EN)
- Users / Teams / Organizations Domain (RU / EN)
- Procurement Domain (RU / EN)
- Бизнес-процессы AS-IS → TO-BE
- DDD_OVERVIEW.md

### Результат
- Архитектурная конституция
- Чёткие правила ownership, snapshot, lifecycle

### Оценка времени
1–2 недели

---

## 2. Фаза 1 — Core Backend и модель данных

### 2.1 Products + Moderation

#### Задачи
- Product Label / Product Variant
- Category и Category Attributes
- Категория Other
- Очередь модерации
- approve / assign / ignore
- Глобальные vs локальные продукты
- Snapshot-безопасные ссылки

#### Время
6–8 недели

---

### 2.2 Users / Teams / Organizations

#### Задачи
- User
- Organization (плоская)
- Team
- Membership & Roles
- Ownership resolution

#### Время
~2 недели

---

### 2.3 Inventory Core (без рецептов)

#### Задачи
- Inventory Session lifecycle
- Ownership + Location
- Inventory Items
- Inventory Snapshots
- Previous / diff / usage
- Save & Send с повторной отправкой
- Read-only после завершения

#### Время
4–5 недель

---

### Итог Фазы 1
- Рабочая инвентаризация
- Историческая целостность
- Экспорт отчётов

---

## 3. Фаза 2 — Recipes и breakdown

### 3.1 Recipes Core
- Recipe
- Recipe Version (immutable)
- Ingredients (Product Variant only)
- Ownership / visibility

Время: ~2 недели

---

### 3.2 Inventory → Recipe Breakdown
- Учёт рецептов в инвентаре
- Snapshot ингредиентов
- Breakdown в Inventory Items

Время: 2–3 недели

---

## 4. Фаза 3 — Procurement (Заказы)

### 4.1 Поставщики
- Supplier
- Pairing lifecycle
- Email-first коммуникация

Время: 1–2 недели

---

### 4.2 Заказы
- Procurement Order
- Order Items
- Создание из инвентаря
- Email-отправка
- Статусы

Время: 2–3 недели

---

### Итог Фазы 3
Инвентаризация → Заказ → Поставщик → Инвентаризация

---

## 5. Фаза 4 — Базовая аналитика

### Задачи
- Consumption
- Waste
- Разницы
- CSV / XLSX
- Агрегация по Team / Organization

Время: 2–3 недели

---

## 6. Фаза 5 — Интеграции и стабилизация

### Задачи
- POS (read-only)
- Barcode → создание продуктов
- Проверки целостности
- Performance
- Audit-инструменты

Время: 3–4 недели

---

## Общая оценка сроков

- Фаза 0: 1–2 недели
- Фаза 1: 8–10 недель
- Фаза 2: 4–5 недель
- Фаза 3: 3–5 недель
- Фаза 4: 2–3 недели
- Фаза 5: 3–4 недели

Итого: ~5–6 месяцев

---

## Минимальный состав команды

- Product / Domain Architect
- Senior Backend Engineer
- Frontend / iPad Developer
- QA (part-time)
- Модератор (part-time)

---

## Критические риски

- Упрощение домена ради скорости
- Нарушение snapshot / immutable правил
- Ранний marketplace или billing
- POS как dependency, а не integration

---

## Контрольные точки

- После Фазы 1 — можно ли жить год без миграций БД
- После Фазы 2 — корректен ли учёт реальных баров
- После Фазы 3 — ежедневное использование заказов
- После Фазы 4 — готовность клиентов платить
