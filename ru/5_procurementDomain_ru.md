# Procurement Domain — Domain Design Draft

## Назначение (Purpose)

Домен Procurement описывает:
- как товары заказываются,
- как управляются поставщики,
- как процессы закупок интегрируются с инвентаризацией.

Цели домена:
- быстрые заказы на основе данных инвентаризации
- работа с поставщиками без принуждения их к использованию системы
- email-first и supplier-friendly подход
- интеграция с Products и Inventory
- база для будущей автоматизации и аналитики

Procurement — это **операционный workflow**, а не финансовая система.  
Цены, биллинг, платежи и бухгалтерия **вне зоны ответственности**.

---

## Обзор ключевых концепций

1. Supplier (Поставщик)
2. Supplier Relationship / Pairing
3. Procurement Order (Заказ)
4. Order Item (Позиция заказа)
5. Коммуникация и статусы поставщиков
6. Жизненный цикл закупки

---

## 1. Supplier (Поставщик)

### Определение

Supplier — внешняя сущность, поставляющая товары
Команде или Организации.

Поставщики **не являются пользователями системы**.

### Примеры
- алкогольный дистрибьютор
- оптовый поставщик напитков
- табачный поставщик
- локальный food-вендор

### Характеристики
- создаётся пользователем
- может быть inactive / pending / active
- не требует логина
- взаимодействие — через email

### Ключевые поля (концептуально)
- id
- name
- contact_email
- contact_name (optional)
- phone (optional)
- legal_name (optional)
- notes (optional)
- created_by_team_id
- created_at
- status

---

## 2. Supplier Relationship (Pairing)

### Определение

Операционная связь между Командой / Организацией
и Поставщиком.

### Pairing Flow

1. Supplier создаётся (inactive)
2. Отправляется Pairing Request
3. Поставщик получает email
4. Ответ:
   - accepted
   - rejected
   - no response

### Состояния

- **inactive** — заказы запрещены
- **pending** — ожидается ответ, заказы запрещены
- **accepted** — заказы разрешены
- **rejected** — заказы запрещены

### Ключевые поля
- supplier_id
- owner_type (Team | Organization)
- owner_id
- status
- pairing_requested_at
- pairing_resolved_at

---

## 3. Procurement Order (Заказ)

### Определение

Запрос поставщику на поставку товаров.

### Характеристики
- принадлежит одному ownership scope
- отправляется одному поставщику
- создаётся вручную или из инвентаря
- неизменяем после отправки

### Ключевые поля
- id
- supplier_id
- owner_type
- owner_id
- created_by_user_id
- created_at
- submitted_at
- status
- notes (optional)

---

## 4. Order Item (Позиция заказа)

### Определение

Один товар в заказе.

### Характеристики
- ссылка на Product Variant
- явное количество и единицы
- цена опциональна (информационная)

### Поля
- procurement_order_id
- product_variant_id
- quantity
- unit
- price (optional)

---

## 5. Email-first коммуникация

### Принцип

Поставщики работают через:
- email
- read-only web-страницы
- простые формы комментариев

Поставщик **никогда не обязан**:
- регистрироваться
- устанавливать ПО

### Надёжность
- проверка no-reply email
- предупреждения пользователю
- повторная отправка

---

## 6. Жизненный цикл закупки

### Статусы

1. Draft
2. Submitted
3. Accepted (optional)
4. Completed (manual)
5. Cancelled

### Правила
- Submitted нельзя редактировать
- Procurement не меняет инвентарь автоматически

---

## 7. Интеграция с Inventory

### Поток

1. Завершена инвентаризация
2. Обнаружен дефицит
3. Создаётся заказ
4. Заказ ссылается на Product Variant
5. Поставка вне системы
6. Инвентарь обновляется вручную

### Принцип

> Procurement **не мутирует инвентарь автоматически**

---

## 8. Ownership и агрегация

Поддерживаемые scope:
- Team
- Organization

Ownership определяет:
- видимость
- агрегацию
- аналитику

---

## 9. Связь с другими доменами

- Products → Product Variant
- Inventory → источник потребностей
- Users / Teams → права и ownership
- Analytics → агрегация
- Notifications → email

---

## Вне зоны ответственности

- платежи
- счета
- бухгалтерия
- enforced pricing
- supplier auth

---

## Ключевые принципы

> Procurement — операционный  
> Поставщики — внешние  
> Email — фича  
> Инвентарь первичен  
> Ownership всегда явен