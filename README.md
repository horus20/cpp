# Crypto Push Proposals

## Push link format

Push-link part - это уникальная строка любой длины, которая состоит из символов набора url-safe. Эта строка может быть частью ссылки. Строка участвует в преобразовании из которого всегда должна получаться одинаковая seed phrase. 
Кошелек опционально может содержать пароль (строка символов произвольной длины). Пароль никогда не должен покидать устройства пользователя. 

Возможная схема:
1. На стороне клиента генерируется push-link-part в виде случайного набора символов (url-safe)
2. Ссылка передается после # (например: mypush.com/#/push-link-part)
3. Склеиваем в единую строку нашу push-link part и опциональный пароль введенный пользователем (между линком и паролем не должно быть никаких разделительных символов) 
4. От этой строки вычисляем криптостойкий хэш (sha256)
5. Полученный хэш используем в качестве entropy для Bitcoin Mnemonic (bip39) -> получаем "mnemonic seed phrase"
6. Из seed phrase получаем Minter Wallet

Весь описанный процесс можно реализовать на стороне клиента по технологии ligth wallet.


## Frontend functions

Методы, которые должны быть реализованы в клиентском интерфейсе:
1. Формирование простой транзакции перевода средств на указанный кошелек
   Метод должен формировать транзакцию, подписывать ее приватным ключом пользователя и возвращать raw transaction в виде hex строки
2. Получение баланса пользователя по адресу кошелька


## Minimal backend api

Методы, которые должны быть реализованы на backend:
1. Отправка raw transaction. Клиентский интерфейс формирует транзакцию, подписывает ее и в виде hex-строки отправляется на сервер. Сервер через своё подключение к ноде отправляет ее в сеть. 
2. Получение списка сервисов. В ответ на клиентский запрос приходит список сервисов с вариантами доступными для трат. Каждый вариант должен содержать название, описание и цену в BIP.
3. Формирование "заказа". В ответ на запрос сервер должен вернуть идентификатор заказа, стоимость в BIP, адрес для перевода средств.
4. Оплата и проверка статуса "заказа". Сервер должен по идентификатору заказа проверить поступление средств, запросить "товар" у провайдера и передать его клиенту.


# PushChain
Алгоритм консенсуса PoA (Proof-of-authority)

## Валидаторы
1. В начале количество валидаторов фиксировано 20.
2. Валидатор может перестать быть валидатором, если был неактивен (не валидировал) более 7 дней.
3. Выбор нового валидатора происходит при согласии 2/3 активных валидаторов (если имеется свободный слот).
4. Уменьшение количества валидаторов происходит при согласии 2/3 активных валидаторов (Если имеется свободный слот).
5. Увеличение количества валидаторов происходит при согласии 2/3 активных валидаторов.
6. Т.к. валидаторы выполняют работу по поддержанию сети, которая "соединяет" пользователей и мерчантов - валидаторы получают комиссию с продаж.
7. Валидаторы получают комиссию равную k/n (Где k - комиссия от продажи, n - количество валидаторов).
8. Полученные валидатором часть комиссии блокируются на 7 дней.
9. Слэш: валидатор не получает свою часть комиссии от продаж за последние 7 дней. Эта сумма распределяется между другими валидаторами.

## Мерчанты
1. Мерчанты регистрируются в PushChain с помощью отправки залога (от 10 000 BIP) на мултисиг в сети Minter.
2. Мерчанты после регистрации могут выставлять (с помощью отправки tx в сети PushChain) продукт для продажи (цена BIP/USD, количество, время на предоставление услуги).
3. Суммарная стомость товаров, выставляемых мерчантом, не может превышать 10*z, где z - залог.
4. Мерчант может снять с продажи товар с задержкой равной Rt. В момент подачи заявления на снятие товара - товар уходит из общего списка товаров (становится недоступным для покупки). Review time(Rt) = 2*t, где t - задекларированное время предоставления услуги, если 2*t < 2 часа, то Rt = 2 часа. Это время необходимо, чтобы клиент мог получить услугу и при этом оставить отзыв о получении услуги надлежащего качества.
5. Если мерчант не предоставляет услугу за задекларированное время/предоставляет товар ненадлежащего качества - его залог конфискуется. Возврат конфискованного залога происходит при согласии 2/3 валидаторов. Залог необходим для того, чтобы скамерскую деятельность сделать финансово невыгодной/опасной.

## Пользователи
1. Пользователь по api узнает какие имеются товары с указанием: характеристик товара (описание, цена, отзывы, задекларированное время предоставления услуги) и информации о мерчанте (рейтинг, залог, статистика продаж).
2. Пользователей выбирает товар.
3. Пользователь посылает средства (BIP) на мултисиг в сети Minter.
4. Мерчант предоставляет услугу.
5. Пользователь в течении Rt (review time) подтверждает получение товара надлежащего качество. Если пользователь не оставляет отзыв за этот период времени - считается, что он получил товар надлежащего качества.

