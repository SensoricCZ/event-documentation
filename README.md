# Events documentation
Dokument popis způsob předávání událostí ze systému SENSORIC do aplikace partnera.
Způsob komunikace je mezi SENSORIC a partnerem dohodnut a na straně SENSORIC nastaven. V budoucnu bude umožněno partnerovi aby si sám nastavení měnil ve webovém rozhraní nebo přes API.
## 1 Způsoby předávání dat
Data jsou předávána komunikačním protokolem ve formátu JSON a popisují události, které vznikají v systému SENSORIC. Položky datetime jsou v UTC podle ISO 8601. Pořadí parametrů není zaručeno a může se měnit.
## 2.1 Předávání událostí přes HTTP callback
Partner může specifikovat jednu nebo vice URL na které jsou události zasílány formou HTTP(S) POST requestů. Requesty mají kódování UTF-8 a Content-Type “application/json”.

Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

Parametry pro sestavení URL:
| Parametr          | Popis                         |
| ------------------|-------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceSerial      | sériové číslo senzoru         |
| DeviceType        | typ zařízení *                |
| EventType         | typ události                  |

*Parametr `DeviceType` má hodnotu `unknown` v případě že typ zařízení není znám (týká se případu kdy je zařízení nastaveno tak že předává pouze raw data).

Příklad URL (doporučené nastavení):
https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceSerial}/DeviceType/{EventType}/

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/panic-button/pressed/

https://nejakaadresa.cz/event/v1/abc123/thermometer/measured/

https://nejakaadresa.cz/event/v1/abc123/movement/battery-low/

https://nejakaadresa.cz/event/v1/abc123/unknown/payload/

Partner může specifikovat další konfiguraci přidáním HTTP hlaviček. Tím lze např. vyřešit autorizaci.

Systém očekává v odpovědi HTTP status 200-299, kterým partner potvrdí přijetí události. Jiná odpověď je vyhodnocena jako nedoručení. V budoucnu bude řešena funkcionalita pro opakované odesílání nedoručených událostí.

## 2.1 Předávání událostí přes MQTT
TODO ...

## 3 Komunikační protokol
Data jsou odesílána vždy jako samostatné události. Události mají společnou část parametrů.

Společné parametry:
| Parametr          | Typ     | Popis                         |
| ------------------|---------|-------------------------------|
| ProtocolVersion   | integer | verze komunikačního protokolu |
| DeviceSerial      | string  | sériové číslo senzoru         |
| DeviceType        | string  | typ zařízení *                |
| EventId           | string  | identifikátor události        |
| EventTime         | string  | čas události                  |
| EventType         | string  | typ události                  |

*Parametr `DeviceType` má hodnotu `unknown` v případě že typ zařízení není znám (týká se případu kdy je zařízení nastaveno tak že předává pouze raw data).

Další parametry jsou závislé na typu události.

Možné hodnoty pro DeviceType:
| DeviceType        | Zařízení                | Podporované typy zpráv        |
| ------------------|-------------------------|-------------------------------|
| unknown           | neznámé zařízené        | payload, data-volume-warning  |
| thermo            | teploměr                | reset, test, measurement      |
| humidity          | vlhkoměr                | reset, test, measurement      |
| sos-button        | sos tlačítko            | reset, test, press            |
| move              | pohybový senzor         | reset, test, ...              |
| magnetic          | magnetický senzor       | reset, test, ...              |
| pir               | pir senzor              | reset, test, ...              |


## 3.x Událost measured
Nastává při odeslání naměřené hodnoty teploměrem.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| Temperature       | float   | ano     | Namšřená teplota

Ukázka zaslané události:
```json
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "thermo",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "measured",
    "Temperature": 25.5
}
```

## 3.x Událost battery-level-warning
Nastává při vyhodnocení nízkého stavu baterie.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| Level             | integer | ano     | procentuální stav baterie

Ukázka zaslané události:
```json
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "thermo",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "battery-level-warning",
    "Level": 30
}
```

## 3.x Událost payload
Nastává při příchodu zprávy u zařízení která jsou nastavena na předávání nezpracovaného payloadu (u těchto zařízení neprobíhá dekódování obsahu payloadu).

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| Payload           | string  | ano     | obsah payloadu v hexadecimálním tvaru

Ukázka zaslané události:
```json
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "unknown",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "payload",
    "Payload": "00aa11bb"
}
```