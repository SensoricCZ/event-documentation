# Specifikace předávání událostí
Dokument popisuje způsob předávání událostí ze systému SENSORIC do aplikace partnera.
Způsob komunikace je mezi SENSORIC a partnerem dohodnut a na straně SENSORIC nastaven. V budoucnu bude umožněno partnerovi aby si sám nastavení měnil ve webovém rozhraní nebo přes API.
## 1 Způsoby předávání dat
Data jsou předávána komunikačním protokolem ve formátu JSON a popisují události, které vznikají v systému SENSORIC. Položky datetime jsou v UTC podle ISO 8601. Pořadí parametrů není zaručeno a může se měnit.
## 2.1 Předávání událostí přes HTTP callback
Partner může specifikovat URL endpointů na které jsou události zasílány formou HTTP(S) POST requestů. Requesty mají kódování UTF-8 a Content-Type “application/json”.

Partner může specifikovat URL pro dvě skupiny událostí:
1) systémové události - souvisí se zařízením
2) datové události - souvisí s dekódovanými daty ze zařízení

### 2.1.1 URL endpointu pro systémové události
Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

Parametry pro sestavení URL:
| Parametr          | Popis                         |
| ------------------|-------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceSerial      | sériové číslo senzoru         |
| EventType         | typ události                  |

Příklad URL (doporučené nastavení):

`https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceSerial}/{EventType}/`

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/battery-low/

https://nejakaadresa.cz/event/v1/abc123/payload/

### 2.1.2 URL endpointu pro datové události
Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

Parametry pro sestavení URL:
| Parametr          | Popis                         |
| ------------------|-------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceSerial      | sériové číslo senzoru         |
| DeviceType        | typ zařízení                  |
| EventType         | typ události                  |

Příklad URL (doporučené nastavení):

`https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceSerial}/{DeviceType}/{EventType}/`

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/panic-button/pressed/

https://nejakaadresa.cz/event/v1/abc123/thermometer/measured/

### 2.1.3 Hlavičky HTTP requestu
Partner může specifikovat další konfiguraci přidáním HTTP hlavičy (klíč a hodnota). Tím lze např. vyřešit autorizaci.

### 2.1.4 Odpověď na HTTP request

Systém očekává v odpovědi HTTP status 200-299, kterým partner potvrdí přijetí události. Jiná odpověď je vyhodnocena jako nedoručení.

### 2.1.5 Chování v případě nedoručení události
 V budoucnu bude řešena funkcionalita pro opakované odesílání nedoručených událostí. V současné verzi je nedoručená událost zahozena.

## 2.1 Předávání událostí přes MQTT
Bude upřesněno ...

## 3 Komunikační protokol
Data jsou odesílána vždy jako samostatné události. Události mají společnou část parametrů.

Společné parametry:
| Parametr          | Typ     | Popis                         |
| ------------------|---------|-------------------------------|
| ProtocolVersion   | integer | verze komunikačního protokolu |
| DeviceSerial      | string  | sériové číslo senzoru         |
| EventId           | string  | identifikátor události        |
| EventTime         | string  | čas události                  |
| EventType         | string  | typ události                  |

Datové události mají navíc parametr `DeviceType`:
| Parametr          | Typ     | Popis                         |
| ------------------|---------|-------------------------------|
| DeviceType        | string  | typ zařízení                  |

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

## 3.1 Systémové události
Systémové události souvisí se zařízením, jsou společné pro všechna zařízení a vznikají nezávisle na dekódování dat přicházejících ze zařízení.

### 3.1.1 Událost payload
Jedná se o speciální typ události která vzniká pouze u zařízení nastavených do režimu "raw komunikace", u kterých neprobíhá dekódování payloadu příchozích zpráv, ale data se partnerovi předávají v původní nezpracované podobě. Tato zařízení neposílají datové události (neprobíhá dekódování payloadu).

Parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| Payload           | string  | ano     | obsah payloadu v hexadecimálním tvaru

Ukázka zaslané události:
```json
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "payload",
    "Payload": "00aa11bb"
}
```

### 3.1.1 Událost activation
Informuje o úspěšné aktivaci senzoru v systému.
Bude upřesněno ...

### 3.1.1 Událost disable
Informuje o zastavení komunikace ze zařízení.
Bude upřesněno ...

### 3.1.2 Událost battery-warning
Nastává při vyhodnocení nízkého stavu baterie.
Bude upřesněno ...

### 3.1.3 Událost data-warning
Upozornění na vysoký objem přenesených dat.
Bude upřesněno ...

### 3.1.3 Událost communication-warning
Upozornění na nestandardní chování zařízení.
Bude upřesněno ...

## 3.1 Datové události pro teploměr (DeviceType = thermo)
Události pro teploměr.

## 3.1.1 Událost measured
Nastává při odeslání naměřené hodnoty.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| Temperature       | float   | ano     | naměřená teplota

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

## 3.1.1 Událost measured-lost
Při příchodu zpráv z teploměru probíhá kontrola, zda před aktuální datovou zprávou nedošlo k výpadku. Pokud je detekován výpadek, systém vygeneruje událost *measured-lost* (jednu nebo více) a následně odešle i aktuální zprávu událostí *measured*.

Zpráva *measured-lost* obsahuje odhadovaný čas kdy k vypadení zprávy došlo a hodnotu s upřesněním typu detekce.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| ------------------|---------|---------|------
| LostValueTime     | string  | ano     | odhadovaný čas výpadku
| LostValueType     | string  | ano     | upřesnění typu
| Temperature       | float   | ano     | hodnota naměřené teploty

LostValueType informuje o tom zda byla zpráva rekonstruována (LostValueType = recovered) a hodnota je tedy přesná, nebo se hodnotu nepodařilo rekonstruovat (LostValueType = unknown) a v tom případě parametr Temperature obsahuje poslední známou hodnotu. 

Ukázka zaslané události:
```json
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "thermo",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "measured-lost",
    "LostValueTime": "2021-05-03T14:20:31.8437511Z",
    "LostValueType": "recovered",
    "Temperature": 25.5
}
```