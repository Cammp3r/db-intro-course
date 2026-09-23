# Звіт: Реляційна схема бази даних для системи управління студентами та гуртожитками

## 1. users
Зберігає облікові дані користувачів системи (базова таблиця для ролей STUDENT та ADMIN_OF_THE_DORM).
- `user_id` (PK) — унікальний ідентифікатор
- `first_name`, `last_name` — ім'я та прізвище
- `email` (UNIQUE, NOT NULL) — унікальна електронна пошта
- `date_of_birth` — дата народження
- `registration_date` — дата реєстрації (за замовчуванням поточна дата)
- `status` — статус користувача

## 2. department
Довідник факультетів/кафедр.
- `department_id` (PK)
- `department_name` (NOT NULL) — назва кафедри

## 3. dorm
Довідник гуртожитків.
- `dorm_id` (PK)
- `parking_slot_id` (FK → parking_slots.parking_slot_id) — паркомісце, пов'язане з гуртожитком

## 4. student
Основна таблиця студентів.
- `student_id` (PK)
- `first_name`, `last_name`, `email` (UNIQUE, NOT NULL)
- `enrollment_year` — рік вступу
- `dorm_id` (FK → dorm.dorm_id) — гуртожиток, у якому проживає студент
- `status` — статус студента
- `user_id` (FK → users.user_id) — пов'язаний обліковий запис

## 5. admin_of_the_dorm
Адміністратори гуртожитків.
- `admin_id` (PK)
- `first_name`, `last_name`, `email` (UNIQUE, NOT NULL)
- `user_id` (FK → users.user_id) — пов'язаний обліковий запис
- `dorm_id` (FK → dorm.dorm_id) — гуртожиток, яким керує адміністратор

## 6. rooms
Кімнати в гуртожитках.
- `room_id` (PK)
- `student_id` (FK → student.student_id) — студент, якому призначена кімната
- `dorm_id` (FK → dorm.dorm_id) — гуртожиток, до якого належить кімната

## 7. parking_slots
Паркувальні місця.
- `parking_slot_id` (PK)
- `student_id` (FK → student.student_id) — студент, який використовує місце
- `dorm_id` (FK → dorm.dorm_id) — гуртожиток, до якого належить місце
- `admin_id` (FK → admin_of_the_dorm.admin_id) — адміністратор, що керує місцем

## 8. course
Курси, що читаються на кафедрах.
- `course_id` (PK)
- `department_id` (FK → department.department_id) — кафедра, яка веде курс
- `student_id` (FK → student.student_id) — студент, записаний на курс

## 9. vacancy
Вакансії, на які подаються студенти.
- `vacancy_id` (PK)
- `student_id` (FK → student.student_id) — студент, який подав заявку
- `company_name` — назва компанії
- `job_name` — назва посади

## Важливі припущення та обмеження

- Таблиця `users` є базовою для `student` та `admin_of_the_dorm` — обидві сутності пов'язані з обліковим записом через `user_id`.
- Зв'язки `USER → STUDENT` та `USER → ADMIN_OF_THE_DORM` у вихідній ERD позначені як "один-до-багатьох" (`||--o{`), тому `UNIQUE` на `user_id` у цих таблицях **не додається**: один користувач формально може бути пов'язаний з кількома записами студента/адміністратора.
- Вихідна ERD містила циклічну залежність: `DORM` посилається на `PARKING_SLOTS` (через `ParkingSlotID`), а `PARKING_SLOTS` посилається назад на `DORM` (через `DormID`). Вирішено так: таблиця `dorm` створюється з полем `parking_slot_id` без обмеження зовнішнього ключа, а саме обмеження (`FOREIGN KEY`) додається пізніше через `ALTER TABLE`, коли таблиця `parking_slots` вже існує. Дані для `dorm.parking_slot_id` заповнюються через `UPDATE` після вставки рядків у `parking_slots`.
- Сутність `USER` перейменована в `users`, оскільки `USER` є зарезервованим словом у PostgreSQL.
- Усі поля email мають обмеження `UNIQUE NOT NULL` для запобігання дублюванню користувачів/студентів/адміністраторів.
- Кожна таблиця заповнена 3–7 тестовими рядками для перевірки коректності зв'язків.