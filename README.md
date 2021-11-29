# Specifikace předávání událostí
Dokument popisuje způsob předávání událostí ze systému SENSORIC do aplikace partnera.
Způsob komunikace je mezi SENSORIC a partnerem dohodnut a na straně SENSORIC nastaven. V budoucnu bude umožněno partnerovi aby si sám nastavení měnil ve webovém rozhraní nebo přes API.
## 1 Způsoby předávání dat
Data jsou předávána komunikačním protokolem ve formátu JSON a popisují události, které vznikají v systému SENSORIC. Položky datetime jsou v UTC podle ISO 8601. Pořadí parametrů není zaručeno a může se měnit.
## 1.1 Předávání událostí přes HTTP callback
Partner může specifikovat URL endpointů na které jsou události zasílány formou HTTP(S) POST requestů. Requesty mají kódování UTF-8 a Content-Type “application/json”.

Partner může specifikovat URL pro dvě skupiny událostí:
1) systémové události - souvisí se zařízením
2) datové události - souvisí s dekódovanými daty ze zařízení

### 1.1.1 URL endpointu pro systémové události
Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

Parametry pro sestavení URL:
| Parametr          | Popis                         |
| :-----------------|:------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceSerial      | sériové číslo senzoru         |
| EventType         | typ události                  |

Příklad URL (doporučené nastavení):

`https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceSerial}/{EventType}/`

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/reset/

https://nejakaadresa.cz/event/v1/abc123/battery-warning/

### 1.1.2 URL endpointu pro datové události
Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

Parametry pro sestavení URL:
| Parametr          | Popis                         |
| :-----------------|:------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceSerial      | sériové číslo senzoru         |
| DeviceType        | typ zařízení                  |
| EventType         | typ události                  |

Příklad URL (doporučené nastavení):

`https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceSerial}/{DeviceType}/{EventType}/`

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/panic-button/pressed/

https://nejakaadresa.cz/event/v1/abc123/thermometer/measured/

### 1.1.3 Hlavičky HTTP requestu
Partner může specifikovat další konfiguraci přidáním HTTP hlavičy (klíč a hodnota). Tím lze např. vyřešit autorizaci.

### 1.1.4 Odpověď na HTTP request

Systém očekává v odpovědi HTTP status 200-299, kterým partner potvrdí přijetí události. Jiná odpověď je vyhodnocena jako nedoručení.

### 1.1.5 Chování v případě nedoručení události
 V budoucnu bude řešena funkcionalita pro opakované odesílání nedoručených událostí. V současné verzi je nedoručená událost zahozena.

## 1.2 Předávání událostí přes MQTT
Podpora plánována v budoucnu, bude upřesněno.

## 2 Komunikační protokol
Data jsou odesílána vždy jako samostatné události. Události mají společnou část parametrů.

Společné parametry:
| Parametr          | Typ     | Popis                         |
| :-----------------|:--------|:------------------------------|
| ProtocolVersion   | integer | verze komunikačního protokolu |
| DeviceSerial      | string  | sériové číslo senzoru         |
| EventId           | string  | identifikátor události        |
| EventTime         | string  | čas události                  |
| EventType         | string  | typ události                  |

Datové události mají navíc parametr `DeviceType`:
| Parametr          | Typ     | Popis                         |
| :-----------------|:--------|:------------------------------|
| DeviceType        | string  | typ zařízení                  |

Další parametry jsou závislé na typu události.

## 2.1 Systémové události
Systémové události souvisí se zařízením, jsou společné pro všechna zařízení a vznikají nezávisle na dekódování dat přicházejících ze zařízení.

### 2.1.1 Událost 'payload'
Jedná se o speciální typ události která vzniká pouze u zařízení nastavených do režimu "raw komunikace", u kterých neprobíhá dekódování payloadu příchozích zpráv, ale data se partnerovi předávají v původní nezpracované podobě. Tato zařízení neposílají datové události (neprobíhá dekódování payloadu).

Parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| Payload           | string  | ano     | obsah payloadu v hexadecimálním tvaru

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "payload",
    "Payload": "00aa11bb"
}
```

### 2.1.2 Událost 'activation'
Informuje o úspěšné aktivaci senzoru v systému.
Bude upřesněno ...

## 2.2 Společné datové události
Události vzniklé na základě příchozích zpráv společné pro více typů zařízení. U každého typu je upřesněno které ze společných událostí u něj mohou nastat.

## 2.2.1 Událost 'reset'
Nastává při resetu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "water",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "reset"
}
```

## 2.2.2 Událost 'test'
Nastává při příjmu testovací zprávy ze senzoru. Testovací zpráva souvisí s procesem aktivace a standardně není partnerům předávána.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "water",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "test"
}
```

## 2.2.3 Událost 'alive'
Nastává při příjmu ohlašovací zprávy ze senzoru. Ohlašovací zpráva informuje o tom, že zařízení žije a odesílá se v pravidelném intervalu od odeslání poslední zprávy.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "water",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "alive"
}
```

## 2.2.4 Událost 'alert-start'
Nastává při poplachovém stavu na senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "move",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "alert-start"
}
```

## 2.2.5 Událost 'alert-continue'
Nastává při reportu pokračování poplachovém ze senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "move",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "alert-continue"
}
```

## 2.2.6 Událost 'alert-end'
Nastává při ukončení poplachového stavu na senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "move",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "alert-end"
}
```

## 2.2.7 Událost 'tamper-open'
Nastává při rozepnutí tamperu - otevření krytu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "move",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "tamper-open"
}
```

## 2.2.8 Událost 'tamper-closed'
Nastává při sepnutí tamperu - zavření krytu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "move",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "tamper-closed"
}
```

## 2.2.9 Událost 'free-fall'
Nastává při detekci volného pádu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "moto",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "free-fall"
}
```

## 2.3 Datové události pro zařízení 'water'
Události pro vodní/záplavový senzor.

### Společné datové události
- reset
- alive
- alert-start   *... začátek zaplavení*
- alert-continue   *... zaplavení pokračuje*
- alert-end   *... konec zaplavení*

## 2.4 Datové události pro zařízení 'move'
Události pro pohybový senzor.

### Společné datové události
- reset
- alive
- alert-start   *... začátek pohybu*
- alert-continue   *... pohyb pokračuje*
- alert-end   *... konec pohybu*
- tamper-open
- tamper-closed

## 2.5 Datové události pro zařízení 'pir'
Události pro PIR senzor.

### Společné datové události
- reset
- alive
- alert-start   *... začátek pohybu*
- alert-continue   *... pohyb pokračuje*
- alert-end   *... konec pohybu*
- tamper-open
- tamper-closed

## 2.6 Datové události pro zařízení 'magnet'
Události pro magnetický senzor.

### Společné datové události
- reset
- alive
- alert-start   *... význam zavisí na režimu senzoru*
- alert-continue   *... význam a to zda událost může nastat zavisí na režimu senzoru*
- alert-end   *... význam zavisí na režimu senzoru*
- tamper-open
- tamper-closed

## 2.7 Datové události pro zařízení 'thermo'
Události pro teploměr.

### Společné datové události
Kromě datových zpráv specifických pro tento senzor mohou nastat tyto společné události:
- reset
- alive

## 2.7.1 Událost 'measured'
Nastává při odeslání naměřené hodnoty.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| Temperature       | float   | ano     | naměřená teplota

Ukázka zaslané události:
```yaml
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

## 2.7.2 Událost measured-lost *
**\* Zpráva v současnosti není implementována.**

Při příchodu zpráv z teploměru probíhá kontrola, zda před aktuální datovou zprávou nedošlo k výpadku. Pokud je detekován výpadek, systém vygeneruje událost *measured-lost* (jednu nebo více) a následně odešle i aktuální událost *measured*.

Zpráva *measured-lost* obsahuje odhadovaný čas kdy k vypadení zprávy došlo, upřesnění typu detekce a hodnotu.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| LostValueTime     | string  | ano     | odhadovaný čas výpadku
| LostValueType     | string  | ano     | upřesnění typu
| Temperature       | float   | ano     | hodnota teploty

*LostValueType* informuje o tom, zda byla zpráva rekonstruována (*LostValueType: recovered*) a hodnota *Temperature* je tedy přesná, nebo byla dopočítána (*LostValueType: calculated*). 

Ukázka zaslané události:
```yaml
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

## 2.8 Datové události pro zařízení 'humidity'
Události pro vlhkoměr.

### Společné datové události
Kromě datových zpráv specifických pro tento senzor mohou nastat tyto společné události:
- reset
- alive

## 2.8.1 Událost 'measured'
Nastává při odeslání naměřené hodnoty.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| Temperature       | float   | ano     | naměřená teplota
| Humidity          | float   | ano     | naměřená vlhkost

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "humidity",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "measured",
    "Temperature": 25.5,
    "Humidity": 27.5
}
```

## 2.8.2 Událost measured-lost *
**\* Zpráva v současnosti není implementována.**

Při příchodu zpráv z vlhkoměru probíhá kontrola, zda před aktuální datovou zprávou nedošlo k výpadku. Pokud je detekován výpadek, systém vygeneruje událost *measured-lost* (jednu nebo více) a následně odešle i aktuální událost *measured*.

Zpráva *measured-lost* obsahuje odhadovaný čas kdy k vypadení zprávy došlo, upřesnění typu detekce a hodnoty.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| LostValueTime     | string  | ano     | odhadovaný čas výpadku
| LostValueType     | string  | ano     | upřesnění typu
| Temperature       | float   | ano     | hodnota teploty
| Humidity          | float   | ano     | hodnota vlhkosti

*LostValueType* informuje o tom, zda byla zpráva rekonstruována (*LostValueType: recovered*) a hodnota *Temperature* je tedy přesná, nebo byla dopočítána (*LostValueType: calculated*). 

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceSerial": "abc123",
    "DeviceType": "humidity",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "measured-lost",
    "LostValueTime": "2021-05-03T14:20:31.8437511Z",
    "LostValueType": "recovered",
    "Temperature": 25.5,
    "Humidity": 27.5
}
```