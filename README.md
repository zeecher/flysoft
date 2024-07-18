# Авиаконнектор

Авиаконнектор - это API-сервис для поиска, бронирования и выписки авиабилетов, с которым взаимодействуют остальные сервисы посредством HTTP-запросов. Сервис Авиаконнектор, в свою очередь, отправляет запросы далее на GDS авиабилетов согласно конфигурации, используя соответствующие протоколы и технологии в зависимости от типа конфигурации.
<!--Writerside adds this topic when you create a new documentation project.
You can use it as a sandbox to play with Writerside features, and remove it from the TOC when you don't need it anymore.-->

## Требования к сервису
•	Единый входной формат запроса: Входной формат запроса должен быть в формате JSON с типом содержимого application/json.

•	Единый выходной формат ответа: Выходной формат ответа должен быть в формате JSON с типом содержимого application/json.

•	Язык реализации: Сервис должен быть реализован на языке программирования Go (GoLang).

•	Маппинг ответов: Сервис должен приводить ответы от GDS авиабилетов на структуру сервиса, приведенную в этой документации.


## Задачи для реализации с GDS:

- Реализация Поиска рекомендаций
- Реализация BrandFares
- Запрос на получение условий тарифа
- Реализация Book
- Реализация CancelBook
- Реализация CheckPrice
- Реализация Ticketing
- Проверка статуса одного заказа

## Реализация Поиска рекомендаций

##### Пример запроса

```json
{
  "search_params": {
    "flight_type": "OW",
    "routes": [
      {
        "index": 0,
        "from": "DYU",
        "to": "MOW",
        "date": "18.07.2027"
      }
    ],
    "passengers": {
      "adt": "1",
      "chd": "0",
      "inf": "0",
      "ins": "0"
    },
    "language": "ru",
    "cabin": "economy"
  },
  "config": {
    "connector": "myagent",
    "config": {
      "connection_value": {
        "url": "https:myagent.cc.xly.com",
        "login": "test",
        "password": "password"
      },
      "configuration_value": {}
    }
  },
  "session_id": "628d7ca583af2935e00096d881d4f70d"
}
```

##### Параметры запроса 
| Элемент            | Формат | Обяз. | Пример                            | Описание                                                                                                                                                                                                               |
|--------------------|--------|-------|-----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| adt                | N      | Да    | 1                                 | количество взрослых                                                                                                                                                                                                    |
| chd                | N      | Да    | 0                                 | количество детей с местом от 2 до 12 лет. Для поиска перелетов только на ребенка, вы можете отправить в запросе необходимое количество CHD, при этом ADT=0                                                             |
| inf                | N      | Да    | 0                                 | количество младенцев без места от 0 до 2 лет                                                                                                                                                                           |
| ins                | N      | Да    | 0                                 | количество младенцев с местом от 0 до 2 лет                                                                                                                                                                            |
| cabin              | C      | Да    | economy                           | класс обслуживания (economy – эконом класс, business – бизнес класс, first – первый класс)                                                                                                                             |
| routes[*]from      | C      | Да    | DYU                               | сегменты путешествия: пункт отправления                                                                                                                                                                                |
| routes[*]to        | C      | Да    | IST                               | сегменты путешествия: пункт прибытия                                                                                                                                                                                   |
| routes[*]date      | D      | Да    | 2024-02-18                        | сегменты путешествия: дата отправления                                                                                                                                                                                 |
| language           | C      | Нет   | ru                                | язык интерфейса, по дефолту равен ru                                                                                                                                                                                   |
| filter_airlines[*] | A      | Нет   | S7                                | авиа компании для фильтрации. Каждая АК должна передаваться отдельным элементом filter_airlines[*] массива. Фильтрация происходит по валидирующему перевозчику. В ответе будут рекомендации с а/к указанными в фильтре |
| is_direct_only     | N      | Нет   | 1                                 | показывать только прямые рейсы                                                                                                                                                                                         |
| session_id         | C      | Да    | ce78ee8e5daf8cefb7b9c1c87adb1aac  | идентификатор сессии                                                                                                                                                                                                   |
| gds_white_list[*]  | A      | Нет   | 3                                 | Список ID GDS для фильтрации. Каждый ID должен передаваться отдельным элементом gds_white_list[*] массива                                                                                                              |
| gds_black_list[*]  | A      | Нет   | 11                                | Список ID GDS для исключения из выдачи. Каждый ID должен передаваться отдельным элементом gds_black_list[*] массива                                                                                                    |
| count              | N      | Нет   | 2                                 | лимит рекомендаций                                                                                                                                                                                                     |

Рекомендуется использовать одновременно только один из параметров gds_white_list или gds_black_list. При одновременном использовании обоих по приоритету должен быть использван gds_white_list, а gds_black_list - проигнорирован.

##### Пример ответа 

```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "flights": [
        {
            "id": "MY10EASYOWE1000000090MOWNYC20220121-      TUA.EK.0.2120.P7691700.P3534125.-17.FZ.2311.DXB.202201212205.VKO.202201211540.73H.KLSOSRU1.325.0.TUA.0.2PC_17.EK.205.JFK.202201221900.DXB.202201220905.388.KLSOSRU1.1135.0.TUA.0.2PC",
            "total_price": 11227,
            "fare": 10520,
            "fee": 200,
            "taxes": 507,
            "insurance": 0,
            "discount": 0,
            "currency": "RUB",
            "supplier": { // валидирующий перевозчик 
                "code": "EK",
                "title": "Emirates" 
            }
            "provider": "myagent",
            "ticketing_time_limit": 1707549258,
            "has_branded_tariffs": true, // флаг доступности семейства тарифов.
            "is_baggage": true, // возможна ли перевозка зарегистрированного багажа 
            "is_charter": false, // чартерный ли это рейс 
            "is_refund": true, // возможен ли возврат билета 
            "routes": [
                {
                    "index": 0,
                    "duration": 6600,
                    "segments": [
                        {
                            "aircraft": "320",
                            "is_baggage": true,  // признак наличия багажа
                            "baggage": {  // данные о багаже
                              "piece": 2,  // количество мест
                              "weight": 30  // вес
                            },
                            "cbaggage": {  // данные о ручной клади
                              "piece": 1,  // количество мест
                              "weight": 7,  // вес
                              "dimensions": {
                                "width": 40,
                                "length": 55,
                                "height": 23
                              },  // допустимые размеры
                               "weight_unit": "KG"
                            },
                            "is_refund": true, // возможен ли возврат
                            "is_change": true, // возможен ли обмен 
                            "free_seats": 8,
                            "arrival": {
                                "time": "18.02.2024 16:35",
                                "airport": "IST",
                                "city": "IST",
                                "country": "TR",
                                "terminal": ""
                            },
                            "departure": {
                                "time": "18.02.2024 13:45",
                                "airport": "DYU",
                                "city": "DYU",
                                "country": "TJ",
                                "terminal": ""
                            },
                            "fare_code": "MSSO",
                            "duration": 6600, // общая продолжительность перелета
                            "carrier_number": "132",
                            "index": 0,
                            "service_class": {
                                "code": "M",
                                "name": "economy"
                            },
                            "tech_stops": [],
                            "operation_supplier": "KC",
                            "code_share": null,
                            "type": "regular", // тип перелета. Могут быть регулярные рейсы "regular", чартерные "charter" и рейсы лоукост-перевозчиков "lowcost".
                            "is_charter": false,  // чартерный ли это рейс - чартеры имеют особенность по получению билета. его возможно получить не ранее, чем за сутки до вылета 
                        }
                    ],
                    "options": []
                }
            ],
            "rec_id": "132MSSODYUALA1708245900M0PCa1c0i0s0y0ins0KC10|040e9bf3f76cb35236d0b4d2a6f38166"
        }
    ]
}
```

## Запрос на получение семейств тарифов (BrandFares)
Получение семейства тарифов для определенной рекомендации.

##### Пример запроса
POST /brand-fares
```json 
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
  "rec_id": "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC",
  "flight_type": "RT",
  "language": "ru",
  "config": {
    "connector": "myagent",
    "config": {
      "connection_value": {
        "url": "https:myagent.cc.xly.com",
        "login": "test",
        "password": "password"
      },
      "configuration_value": {}
    }
  }
}
```
##### Параметры запроса

* rec_id - идентификатор рекомендации(полученный при поиске)
* session_id - идентификатор сессии которую надо вернуть обратно в теле ответа
* flight_type - тип полета 
* настройки подключения к GDS

##### Пример ответа 

```json
{
    "session": "628d7ca583af2935e00096d881d4f70d",
    "flights": [
        {
            "rec_id": "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC",
            "total_price": 11227,
            "fare": 10520,
            "fee": 200,
            "taxes": 507,
            "insurance": 0,
            "discount": 0,
            "currency": "RUB",
            "validating_supplier": "TK",
            "provider": "myagent",
            "ticketing_time_limit": 1721015586,
            "has_branded_tariffs": true,
            "routes": [
                {
                    "index": 0,
                    "duration": 49800,
                    "segments": [
                        {
                            "aircraft": "333",
                            "baggage": "40 KG",
                            "hand_luggage": "8 KG",
                            "hand_luggage_weight": "8 KG",
                            "is_refund": false,
                            "is_change": true,
                            "free_seats": 4,
                            "arrival": {
                                "time": "01.08.2024 06:30",
                                "airport": "IST",
                                "city": "IST",
                                "country": "TR",
                                "terminal": ""
                            },
                            "departure": {
                                "time": "01.08.2024 03:10",
                                "airport": "DYU",
                                "city": "DYU",
                                "country": "TJ",
                                "terminal": ""
                            },
                            "fare_code": "KF1BX",
                            "duration": 19200,
                            "carrier_number": "255",
                            "index": 0,
                            "service_class": {
                                "code": "K",
                                "name": "business"
                            },
                            "tech_stops": [],
                            "operation_supplier": "TK",
                            "code_share": null,
                            "is_charter": false
                        },
                        {
                            "aircraft": "32B",
                            "baggage": "40 KG",
                            "hand_luggage": "8 KG",
                            "hand_luggage_weight": "8 KG",
                            "is_refund": false,
                            "is_change": true,
                            "free_seats": 4,
                            "arrival": {
                                "time": "01.08.2024 16:00",
                                "airport": "GYD",
                                "city": "BAK",
                                "country": "AZ",
                                "terminal": "1"
                            },
                            "departure": {
                                "time": "01.08.2024 12:10",
                                "airport": "IST",
                                "city": "IST",
                                "country": "TR",
                                "terminal": ""
                            },
                            "fare_code": "KF1BX",
                            "duration": 10200,
                            "carrier_number": "340",
                            "index": 1,
                            "service_class": {
                                "code": "K",
                                "name": "business"
                            },
                            "tech_stops": [],
                            "operation_supplier": "TK",
                            "code_share": null,
                            "is_charter": false
                        }
                    ],
                    "options": []
                },
                {
                    "index": 1,
                    "duration": 49800,
                    "segments": [
                        {
                            "aircraft": "32B",
                            "baggage": "40 KG",
                            "hand_luggage": "8 KG",
                            "hand_luggage_weight": "8 KG",
                            "is_refund": false,
                            "is_change": true,
                            "free_seats": 4,
                            "arrival": {
                                "time": "02.08.2024 00:05",
                                "airport": "IST",
                                "city": "IST",
                                "country": "TR",
                                "terminal": ""
                            },
                            "departure": {
                                "time": "01.08.2024 21:55",
                                "airport": "GYD",
                                "city": "BAK",
                                "country": "AZ",
                                "terminal": "1"
                            },
                            "fare_code": "KF1BX",
                            "duration": 11400,
                            "carrier_number": "335",
                            "index": 0,
                            "service_class": {
                                "code": "K",
                                "name": "business"
                            },
                            "tech_stops": [],
                            "operation_supplier": "TK",
                            "code_share": null,
                            "is_charter": false
                        },
                        {
                            "aircraft": "333",
                            "baggage": "40 KG",
                            "hand_luggage": "8 KG",
                            "hand_luggage_weight": "8 KG",
                            "is_refund": false,
                            "is_change": true,
                            "free_seats": 4,
                            "arrival": {
                                "time": "03.08.2024 01:10",
                                "airport": "DYU",
                                "city": "DYU",
                                "country": "TJ",
                                "terminal": ""
                            },
                            "departure": {
                                "time": "02.08.2024 18:35",
                                "airport": "IST",
                                "city": "IST",
                                "country": "TR",
                                "terminal": ""
                            },
                            "fare_code": "KF1BX",
                            "duration": 16500,
                            "carrier_number": "254",
                            "index": 1,
                            "service_class": {
                                "code": "K",
                                "name": "business"
                            },
                            "tech_stops": [],
                            "operation_supplier": "TK",
                            "code_share": null,
                            "is_charter": false
                        }
                    ],
                    "options": []
                }
            ]
        }
    ]
}
```

## Запрос на получение условий тарифа

##### Пример запроса 

```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "rec_id": "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC",
    "language": "ru"
}
```

##### Пример ответа 

```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "error_code": "",
    "error_desc": "",
    "rules": {
            "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC": [
                {
                    "departure": {
                        "name": "Dushanbe",
                        "iata": "DYU",
                        "country": {
                            "name": "Таджикистан",
                            "iata": "TJ" 
                        }
                    },
                    "arrival": {
                        "name": "Истанбул",
                        "iata": "IST",
                        "country": {
                            "name": "Турция",
                            "iata": "TR" 
                        }
                    },
                    "fare_code": "PROMO",
                    "text": "Норма бесплатного провоза багажа: не предоставляется\nСкидки для детей до 2-х лет без предоставления отдельного места: 90%, при внутренних перевозках по РФ – бесплатно\nСкидки для детей до 2-х лет и детей от 2 до 12 лет с предоставлением отдельного места: без сопровождения взрослым скидка не предоставляется, с сопровождением взрослым в том же классе обслуживания – 25%\nВозврат провозной платы при уведомлении об отказе от перевозки до окончания установленного времени регистрации пассажиров на указанный в билете рейс: не разрешено\nБонусные мили – 75%\nИзменения не позднее 30 минут после времени отправки рейса, указанного в оформленном билете (при условии наличия мест по оплаченному тарифу): разрешено с доплатой\nБолее подробные правила на сайте перевозчика: https://www.aeroflot.ru//ru-ru/information/purchase/rate/fare_rules" 
                }
            ]
        }
}
```

## Реализация Book

##### Пример запроса
```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "rec_id": "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC",
    "language": "ru",
    "currency": "RUB",
    "passengers": [
        {
            "index": 0,
            "name": "DALER",
            "surname": "PULATOV",
            "citizenship": "TJ",
            "gender": "M",
            "type": "adt",
            "email": "suhrob.sobirov1993@mail.ru",
            "phone": "+992917039843",
            "date_of_birth": "22-02-1994",
            "document": {
                "type": "NP",
                "num": "2014454343",
                "original_number": "2014454343",
                "expire": "2044-05-28 00:00:00"
            }
        }
    ],
    "payer": {
        "name": "zohir",
        "email": "zohir88@gmail.com",
        "phone": "+9999844444366"
    },
    "config": {
        "connector": "myagent",
        "config": {
            "connection_value": {
                "url": "https:myagent.cc.xly.com",
                "login": "test",
                "password": "password"
            },
            "configuration_value": {}
        }
    }
}
```

##### Пример ответа 

```json
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
    "total_price": 11227,
    "fare": 10520,
    "fee": 200,
    "taxes": 507,
    "insurance": 0,
    "discount": 0,
    "currency": "RUB",
    "validating_supplier": "KC",
    "provider": "myagent",
    "ticketing_time_limit": 1707554301,
    "has_branded_tariffs": true,
    "config_id": 18,
    "config_name": "KC_Airline",
    "routes": [
        {
            "index": 0,
            "duration": 6600,
            "segments": [
                {
                    "aircraft": "320",
                    "baggage": "0 PC",
                    "hand_luggage": "N/A",
                    "hand_luggage_weight": "N/A",
                    "is_refund": null,
                    "is_change": null,
                    "free_seats": 9,
                    "arrival": {
                        "time": "18.02.2024 16:35",
                        "airport": "ALA",
                        "city": "ALA",
                        "terminal": ""
                    },
                    "departure": {
                        "time": "18.02.2024 13:45",
                        "airport": "DYU",
                        "city": "DYU",
                        "terminal": ""
                    },
                    "fare_code": "PSSO",
                    "duration": 6600,
                    "carrier_number": "132",
                    "index": 0,
                    "service_class": {
                        "code": "P",
                        "name": "economy"
                    },
                    "tech_stops": [],
                    "operation_supplier": "KC",
                    "code_share": null,
                    "is_charter": false
                }
            ],
            "options": []
        }
    ],
  "rec_id": "MY01EASYOWA1000010090MOWLED20231106-TUA.SU.0.95.F322717S374200.OENCTkRDX1NV..-31.SU.6.SVO.202311060655.LED.202311060830.32A.RNOR.95.0.TUA.0.0PC",
    "fare_rules": [
        {
            "fare_code": "PSSO",
            "penalties": "RU.RULE APPLICATION\nKC BASIC FARE BRAND\n APPLICATION\n CLASS OF SERVICE\n THE
        }
    ],
    "booking_number": "125354686465",
    "order_id": "A-17072964339167",
    "airline_booking_number": "KC/TQZED9",
    "timelimit": "10.02.2024 08:40",
    "passengers": [
        {
            "index": 0,
            "name": "DALER",
            "surname": "PULATOV",
            "citizenship": "TJ",
            "gender": "M",
            "type": "adt",
            "email": "suhrob.sobirov1993@mail.ru",
            "phone": "+992917039843",
            "date_of_birth": "22-02-1994",
            "document": {
                "type": "NP",
                "num": "2014454343",
                "original_number": "2014454343",
                "expire": "2044-05-28 00:00:00"
            },
            "amounts": {
                "currency": "RUB",
                "total_price": 1587.3,
                "fare": 1587.3,
                "taxes": 0,
                "fees": 0
            }
        }
    ]
}
```

## Реализация CancelBook

##### Пример запроса 
GET /cancel-book

```json
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
  "booking_number": "125354686465",
  "language": "ru",
  "config": {
    "connector": "myagent",
    "config": {
      "connection_value": {
        "url": "https:myagent.cc.xly.com",
        "login": "test",
        "password": "password"
      },
      "configuration_value": {}
    }
  }
}
```

##### Пример ответа

```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "booking_number": "125354686465",
    "booking_status": "bookingCancel",
    "error_code": "",
    "error_desc": ""
}
```
Если есть ошибка при обработке error_code должен быть и error_message содержать текст ошибки
и booking_status должен быть равен bookingCancelError


## Оплата заказа (Ticketing)

##### Пример запроса

POST /ticketing

```json
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
  "booking_number": "125354686465",
  "total_price": 11227,
  "currency": "RUB",
  "language": "ru",
  "configs": {
    "connector": "myagent",
    "config": {
      "connection_value": {
        "url": "https:myagent.cc.xly.com",
        "login": "test",
        "password": "password"
      },
      "configuration_value": {}
    }
  }
}
```
##### Пример ответа

```json 
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "billing_number": "125354686465",
    "billing_status": "paySuccess",
    "error_code": "",
    "error_desc": "",
    "tickets": []
}
```
 статус заказа после успешной оплаты может принимает значение paySuccess или payFail если оплата не пуспешна


## Проверка статуса одного заказа

##### Пример запроса 

POST /ticketing-status

```json
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
  "billing_number": "125354686465",
  "configs": {
    "connector": "myagent",
    "config": {
      "connection_value": {
        "url": "https:myagent.cc.xly.com",
        "login": "test",
        "password": "password"
      },
      "configuration_value": {}
    }
  },
  "language": "ru"
}
```


##### Пример ответа 

```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "error_code": "",
    "error_desc": "",
    "order_id": 21845082, // Номер заказа GDS если присутсвует
    "billing_number": 125354686465,
    "expire": "30.05.2024 16:52:00",
    "expire_remain": 85492,
    "created": "28.05.2024 17:02:06",
    "status": {
        "sign": "Ticketed",
        "title": "Закрыт"
    },
    "price": {
        "RUB": {
            "fare": 10520,
            "fee": 200,
            "taxes": 507,
            "insurance": 0,
            "discount": 0,
            "extra_baggage": 0,
            "total_price": 11227
        }
    },
    "tickets": [
        {
            "locator": "CBKZVT", // локатор авиакомпании, используется для регистрации на рейс
            "booking_provider": "TUA", // название провайдера
            "currency": "RUB", // код валюты заказа по классификации ISO 4217  https://www.iso.org/ru/iso-4217-currency-codes.html
            "carrier": {
                "id": 636, //id оперирующего перевозчика
                "code": "TK", //IATA код оперирующего перевозчика
                "title": "Turkish Arilines" //название оперирующего перевозчика
            },
            "duration": {
                "flight": {
                    "common": 90,
                    "hour": 1,
                    "minute": 30
                }
            },
            "passengers": [
                {
                  "name": "DALER",
                  "surname": "PULATOV",
                  "citizenship": "TJ",
                  "gender": "M",
                  "type": "adt",
                  "email": "suhrob.sobirov1993@mail.ru",
                  "phone": "+992917039843",
                  "date_of_birth": "22-02-1994",
                  "document": {
                        "type": "NP",
                        "num": "2014454343",
                        "original_number": "2014454343",
                        "expire": "2044-05-28 00:00:00"
                    },
                    "ticketData": {
                        "number": "5552327744382",
                        "refunded": false
                    },
                    "key": "PULATOV_DALER_2014454343_14-03-1989",
                    "accompanying_adults": []
                }
            ]
        }
    ],
    "flight": {
        "rec_id": "255KF1BXBXDYUIST1722463800K40KG340KF1BXBXISTGYD1722503400K40KG335KF1BXBXGYDIST1722534900K40KG254KF1BXBXISTDYU1722612900K40KGa2c0i0s0y0ins0TK12|625533abc0b07dc388f4e6c2184aae79",
        "total_price": 11227,
        "fare": 10520,
        "fee": 200,
        "taxes": 507,
        "insurance": 0,
        "discount": 0,
        "currency": "RUB",
        "validating_supplier": "TK",
        "provider": "myagent",
        "ticketing_time_limit": 1721015586,
        "has_branded_tariffs": true,
        "routes": [
            {
                "index": 0,
                "duration": 49800,
                "segments": [
                    {
                        "aircraft": "333",
                        "baggage": "40 KG",
                        "hand_luggage": "8 KG",
                        "hand_luggage_weight": "8 KG",
                        "is_refund": false,
                        "is_change": true,
                        "free_seats": 4,
                        "arrival": {
                            "time": "01.08.2024 06:30",
                            "airport": "IST",
                            "city": "IST",
                            "country": "TR",
                            "terminal": ""
                        },
                        "departure": {
                            "time": "01.08.2024 03:10",
                            "airport": "DYU",
                            "city": "DYU",
                            "country": "TJ",
                            "terminal": ""
                        },
                        "fare_code": "KF1BX",
                        "duration": 19200,
                        "carrier_number": "255",
                        "index": 0,
                        "service_class": {
                            "code": "K",
                            "name": "business"
                        },
                        "tech_stops": [],
                        "operation_supplier": "TK",
                        "code_share": null,
                        "is_charter": false
                    },
                    {
                        "aircraft": "32B",
                        "baggage": "40 KG",
                        "hand_luggage": "8 KG",
                        "hand_luggage_weight": "8 KG",
                        "is_refund": false,
                        "is_change": true,
                        "free_seats": 4,
                        "arrival": {
                            "time": "01.08.2024 16:00",
                            "airport": "GYD",
                            "city": "BAK",
                            "country": "AZ",
                            "terminal": "1"
                        },
                        "departure": {
                            "time": "01.08.2024 12:10",
                            "airport": "IST",
                            "city": "IST",
                            "country": "TR",
                            "terminal": ""
                        },
                        "fare_code": "KF1BX",
                        "duration": 10200,
                        "carrier_number": "340",
                        "index": 1,
                        "service_class": {
                            "code": "K",
                            "name": "business"
                        },
                        "tech_stops": [],
                        "operation_supplier": "TK",
                        "code_share": null,
                        "is_charter": false
                    }
                ],
                "options": []
            },
            {
                "index": 1,
                "duration": 49800,
                "segments": [
                    {
                        "aircraft": "32B",
                        "baggage": "40 KG",
                        "hand_luggage": "8 KG",
                        "hand_luggage_weight": "8 KG",
                        "is_refund": false,
                        "is_change": true,
                        "free_seats": 4,
                        "arrival": {
                            "time": "02.08.2024 00:05",
                            "airport": "IST",
                            "city": "IST",
                            "country": "TR",
                            "terminal": ""
                        },
                        "departure": {
                            "time": "01.08.2024 21:55",
                            "airport": "GYD",
                            "city": "BAK",
                            "country": "AZ",
                            "terminal": "1"
                        },
                        "fare_code": "KF1BX",
                        "duration": 11400,
                        "carrier_number": "335",
                        "index": 0,
                        "service_class": {
                            "code": "K",
                            "name": "business"
                        },
                        "tech_stops": [],
                        "operation_supplier": "TK",
                        "code_share": null,
                        "is_charter": false
                    },
                    {
                        "aircraft": "333",
                        "baggage": "40 KG",
                        "hand_luggage": "8 KG",
                        "hand_luggage_weight": "8 KG",
                        "is_refund": false,
                        "is_change": true,
                        "free_seats": 4,
                        "arrival": {
                            "time": "03.08.2024 01:10",
                            "airport": "DYU",
                            "city": "DYU",
                            "country": "TJ",
                            "terminal": ""
                        },
                        "departure": {
                            "time": "02.08.2024 18:35",
                            "airport": "IST",
                            "city": "IST",
                            "country": "TR",
                            "terminal": ""
                        },
                        "fare_code": "KF1BX",
                        "duration": 16500,
                        "carrier_number": "254",
                        "index": 1,
                        "service_class": {
                            "code": "K",
                            "name": "business"
                        },
                        "tech_stops": [],
                        "operation_supplier": "TK",
                        "code_share": null,
                        "is_charter": false
                    }
                ],
                "options": []
            }
        ]
    },
    "passengers_price_details": [
        {
            "key": "PULATOV_DALER_2014454343_14-03-1989",
            "affiliate_fee": 0,
            "agent_affiliate_fee": 0,
            "partner_affiliate_fee": 0,
            "comsa": 1,
            "ticket_price": 11226,
            "acquiring": 0,
            "insurance_price": 0,
            "vat": 956.36,
            "taxes_amount": 507,
            "tariff": 10520,
            "fee": 200,
            "commissions": {
                "other_commission": 1
            },
            "taxes": [
                {
                    "code": "RI",
                    "amount": 507,
                    "currency": "RUB"
                }
            ]
        }
    ]
}
```

## Реализация CheckPrice

##### Пример запроса
POST /check-price

```json
{
  "session_id": "628d7ca583af2935e00096d881d4f70d",
  "billing_number": 125355187996

}
```

##### Пример ответа 
```json
{
    "session_id": "628d7ca583af2935e00096d881d4f70d",
    "billing_number": "125354686465",
     "new_price": 5000,
    "is_price_changed": true,
    "error_code": "",
    "error_desc": ""  
}
```
