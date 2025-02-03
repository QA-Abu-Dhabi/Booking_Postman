# Тестирование API системы бронирования услуг

## Задача

**API документация:** [Restful-booker](https://restful-booker.herokuapp.com/apidoc/index.html).

1. Создать коллекцию Postman и два запроса POST и GET к API.
2. Настроить pre- и post-request и тестовые скрипты в Postman для генерации firstName и lastName.

## Решение
Коллекция запросов Postman в [JSON-файле](https://github.com/QA-Abu-Dhabi/Booking_Postman/blob/main/booking.postman_collection.json).  
Результаты тестирования коллекции в [JSON-файле](https://github.com/QA-Abu-Dhabi/Booking_Postman/blob/main/booking.postman_test_run.json).  

## 1. Создание POST запроса CreateBooking для создания нового бронирования
Отправим POST-запрос на URL: https://restful-booker.herokuapp.com/booking.  
Для генерации случайных значений **firstName** и **lastName** используем следующий pre-request script:
```javascript
pm.sendRequest('https://randomuser.me/api', function (err, res) {
    if (err) {
       console.log("Ошибка при запросе API:", err);
       return;
    }

    if (!res || !res.json() || !res.json().results || res.json().results.length == 0) {
        console.log("Некорректный ответ api");
        return;
    }

    const data = res.json();
    const firstName = data.results[0].name.first;
    const lastName = data.results[0].name.last;

    pm.collectionVariables.set("firstname", firstName);
    pm.collectionVariables.set("lastname", lastName);

    console.log("Generate firstname:", pm.collectionVariables.get("firstname"));
    console.log("Generate lastname:", pm.collectionVariables.get("lastname"));
});
```
После отправки запроса и получения ответа сохраняем значение **bookingid** в глобальную переменную и проверяем полученные данные на корректность:
```javascript
let response = pm.response.json();

if (response && response.bookingid) {
    pm.globals.set("bookingid", response.bookingid);
    console.log("Сохранённый bookingid:", response.bookingid);
} else {
    console. log("Ошибка: bookingid не найден в ответе");
}

pm.test("request и response совпадают", function () {
    pm.expect(response.booking.firstname).to.eql(pm.collectionVariables.get("firstname"));
    pm.expect(response.booking.lastname).to.eql(pm.collectionVariables.get("lastname"));
    pm.expect(response.booking.totalprice).to.eql(Number(pm.collectionVariables.get("totalprice")));
    let depositpaidBool = pm.collectionVariables.get("depositpaid");
    depositpaidBool = depositpaidBool === "true";
    pm.expect(response.booking.depositpaid).to.eql(depositpaidBool);
    pm.expect(response.booking.bookingdates.checkin).to.eql(pm.collectionVariables.get("checkin"));
    pm.expect(response.booking.bookingdates.checkout).to.eql(pm.collectionVariables.get("checkout"));
    pm.expect(response.booking.additionalneeds).to.eql(pm.collectionVariables.get("additionalneeds"));
})
```
## 2. Создание GET запроса на получение данных о бронировании по id
Отправим GET запрос для получения данных о ранее созданном бронировании по сохраненному id на URL: https://restful-booker.herokuapp.com/booking/{{bookingid}}.  
После получения ответа проверяем корректность полученных данных:
```javascript
let response = pm.response.json();

pm.test("request и response совпадают", function () {
    pm.expect(response.firstname).to.eql(pm.collectionVariables.get("firstname"));
    pm.expect(response.lastname).to.eql(pm.collectionVariables.get("lastname"));
    pm.expect(response.totalprice).to.eql(Number(pm.collectionVariables.get("totalprice")));
    let depositpaidBool = pm.collectionVariables.get("depositpaid");
    depositpaidBool = depositpaidBool === "true";
    pm.expect(response.depositpaid).to.eql(depositpaidBool);
    pm.expect(response.bookingdates.checkin).to.eql(pm.collectionVariables.get("checkin"));
    pm.expect(response.bookingdates.checkout).to.eql(pm.collectionVariables.get("checkout"));
    pm.expect(response.additionalneeds).to.eql(pm.collectionVariables.get("additionalneeds"));
})
```
## 3. Создание POST запроса для авторизации и получения токена автризациии
Отправим POST-запрос на URL: https://restful-booker.herokuapp.com/auth.  
Для проверки статус кода ответа и сохранения токена авторизации в глобальную переменную **auth_token** используем следующий post-request:
```javascript
let response = pm.response.json();

if (response && response.token) {
    pm.globals.set("auth_token", response.token);
    console.log("Сохранённый token:", response.token);
} else {
    console. log("Ошибка: token не найден в ответе");
}

pm.test("Статус 200 OK", function() {
    pm.response.to.have.status(200);
});

pm.test("Токен получен и сохранен", function () {
    pm.expect(response.token).to.eql(pm.globals.get("auth_token"));
})
```






