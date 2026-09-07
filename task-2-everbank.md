# Threat Model — EverBank

##  Границы доверия

| № | Границы |
|---|------------|
| 1 | Пользователь/Внешняя платёжная система/Оператор банка/Мерчант → Nginx (API Gateway) |
| 2 | Nginx (API Gateway) → Services (payment-gw/notification/auth-service/kyc/stories/reporting-service/profile/forum/admin-panel/transaction/account-service/card-service) |
| 3 | Services (payment-gw/notification/auth-service/kyc/stories/reporting-service/profile/forum/admin-panel/transaction/account-service/card-service/processing-service/antifraud-service) → Kafka |
| 4 | Services (notification/auth-service/kyc) → Redis |
| 5 | Services (stories/kyc) → minio |
| 6 | Services (auth-service/kyc/stories/reporting-service/profile/forum/admin-panel/account-service/card-service/processing-service) → Postgres |




---

# Модель угроз

| # | STRIDE | Элемент(ы) диаграммы | Описание элемента | Угроза | Сценарий реализации | Критичность | Защита | Требование (задача) |
|---|---|---|---|---|---|---|---|---|
| 1 | Spoofing | `Operator` → `admin-panel` | Оператор входит в бэк-офис по отдельному логину и паролю | Компрометация учётной записи оператора позволяет получить доступ к бэк-офису | Атакующий перебирает пароль через `/login` или использует утёкшие credentials | Высокая | Rate limiting, MFA, защита от credential stuffing | Реализовать rate limiting и MFA для `/login` |
| 2 | Spoofing | `Внешняя платёжная система` → `processing-service` | Внешняя ИС отправляет webhook сообщения о платеже | Возможно подделать webhook и выдать себя за иную платёжную систему | Атакующий отправляет webhook, структуры и формата платёжной ИС, но с своими значениями, например с поддельным идентификатором и статусом успешной оплаты. Если источник и подпись не проверяются, система принимает событие как валидное | Высокая - возможна некорректная смена статуса платежа или запуск дальнейшей обработки | Добавить криптографию для проверки webhook через подпись, mTLS | Реализовать обязательную проверку подписи для всех входящих webhook + mTLS |
