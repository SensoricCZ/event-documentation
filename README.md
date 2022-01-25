# Specifikace předávání událostí
Dokument popisuje způsob předávání událostí ze systému SENSORIC do aplikace partnera.

**Způsob komunikace je mezi SENSORIC a partnerem dohodnut a na straně SENSORIC nastaven.** V budoucnu bude umožněno partnerovi aby si sám nastavení měnil ve webovém rozhraní nebo přes API.

## Obsah
- [Specifikace předávání událostí](#specifikace-předávání-událostí)
  - [Obsah](#obsah)
- [Způsoby předávání dat](#způsoby-předávání-dat)
  - [Předávání událostí přes HTTP callback](#předávání-událostí-přes-http-callback)
    - [URL HTTP requestu](#url-http-requestu)
    - [Hlavičky HTTP requestu](#hlavičky-http-requestu)
    - [Odpověď na HTTP request](#odpověď-na-http-request)
    - [Chování v případě nedoručení události](#chování-v-případě-nedoručení-události)
- [Komunikační protokol](#komunikační-protokol)
- [Zařízení a podporované události](#zařízení-a-podporované-události)
  - [Zařízení WaterDetection](#zařízení-waterdetection)
    - [Režim Simple](#režim-simple)
  - [Zařízení MovementDetection](#zařízení-movementdetection)
    - [Režim Continuous](#režim-continuous)
  - [Zařízení Magnetic](#zařízení-magnetic)
    - [Režim Simple](#režim-simple-1)
    - [Režim Continuous](#režim-continuous-1)
  - [Zařízení Pir](#zařízení-pir)
    - [Režim Simple](#režim-simple-2)
    - [Režim Continuous](#režim-continuous-2)
  - [Zařízení AlertButton](#zařízení-alertbutton)
    - [Režim Simple](#režim-simple-3)
  - [Zařízení Thermometer](#zařízení-thermometer)
    - [Režim Momentary](#režim-momentary)
    - [Režim Average](#režim-average)
    - [EventType MeasuredTemperature](#eventtype-measuredtemperature)
  - [Zařízení HumidityMeter](#zařízení-humiditymeter)
    - [Režim Momentary](#režim-momentary-1)
    - [Režim Average](#režim-average-1)
    - [EventType MeasuredHumidityTemperature](#eventtype-measuredhumiditytemperature)
- [Společné události](#společné-události)
  - [EventType Restart](#eventtype-restart)
  - [EventType Alive](#eventtype-alive)
  - [EventType Transport](#eventtype-transport)
  - [EventType TamperOpen](#eventtype-tamperopen)
  - [EventType TamperClosed](#eventtype-tamperclosed)
  - [EventType AlertStart](#eventtype-alertstart)
  - [EventType AlertContinue](#eventtype-alertcontinue)
  - [EventType AlertEnd](#eventtype-alertend)

# Způsoby předávání dat
Data jsou předávána komunikačním protokolem ve formátu JSON a popisují události, které vznikají v systému SENSORIC. Položky datetime jsou v UTC podle ISO 8601. Pořadí parametrů není zaručeno a může se měnit.

## Předávání událostí přes HTTP callback
Partner může specifikovat URL endpointů na které jsou události zasílány formou HTTP(S) POST requestů. Requesty mají kódování UTF-8 a Content-Type “application/json”. Systém se snaží předat události v režimu at-least-once, může tedy nastat situace že je událost doručena víckrát. Tuto situaci lze ošetři s využitím položky EventId, která obsahuje identifikátor události.

### URL HTTP requestu
Do URL je možné vložit zástupné parametry, které budou nahrazeny odpovídající hodnotou. 

| Parametr          | Popis                         |
| :-----------------|:------------------------------|
| ProtocolVersion   | verze komunikačního protokolu |
| DeviceId          | identifikátor senzoru         |
| MessageType       | typ zprávy                    |
| EventType         | typ události                  |

Příklad URL (doporučené nastavení):

`https://nejakaadresa.cz/event/v{ProtocolVersion}/{DeviceId}/{MessageType}/{EventType}/`

Pro uvedenou URL budou volány např. tyto requesty:

https://nejakaadresa.cz/event/v1/abc123/MagneticSimple/AlertStart/

https://nejakaadresa.cz/event/v1/abc123/ThermometerAverage/Measured/

### Hlavičky HTTP requestu
Partner může specifikovat další konfiguraci přidáním HTTP hlavičy (klíč a hodnota). Tím lze např. vyřešit autorizaci.

### Odpověď na HTTP request

Systém očekává v odpovědi HTTP status 200-299, kterým partner potvrdí přijetí události. Jiná odpověď je vyhodnocena jako nedoručení.

### Chování v případě nedoručení události
 V případě, že se nepodaří událost předat, systém pokus 10x opakuje s 5s prodlevami. Následně je událost zahozena.

# Komunikační protokol
Data jsou odesílána vždy jako samostatné události. Události mají společnou část parametrů.

Společné parametry:
| Parametr          | Typ     | Popis                         |
| :-----------------|:--------|:------------------------------|
| ProtocolVersion   | integer | verze komunikačního protokolu |
| DeviceId          | string  | identifikátor senzoru         |
| EventId           | string  | identifikátor události        |
| EventTime         | string  | čas události                  |
| MessageType       | string  | typ zprávy                    |
| EventType         | string  | typ události                  |

Další parametry jsou závislé na typu zprávy a události.

# Zařízení a podporované události

## Zařízení WaterDetection
Detekuje přítomnost vody v ohraničeném prostoru. 

![WaterDetection](images/devices/waterdetection.png)

### Režim Simple
Pokud v klidovém stavu dojde k zaplavení, je vyvolána událost `AlertStart`. Následně kontroluje každou minutu zda zaplavení trvá a pokud trvá, tak po 10 minutách vyvolá událost `AlertContinue`. Pokud i nadále zaplavení pokračuje pošle po dalších 10 minutách událost `AlertContinue`, že zaplavení pokračuje, další již neposílá. Po skončení zaplavení zařízení posílá `AlertEnd`.

> MessageType: WaterDetectionSimple

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [AlertStart](#eventtype-alertstart)         | Detekce vzniku zaplavení. |
| [AlertContinue](#eventtype-alertcontinue)   | Zaplavení pokračuje. |
| [AlertEnd](#eventtype-alertend)             | Konec zaplavení. |

## Zařízení MovementDetection
Detekuje pohyb předmětu, na kterém je čidlo připevněno nebo položeno. 

![MovementDetection](images/devices/movementdetection.png)

### Režim Continuous
Pro případy, kdy chceme být informováni o tom, že se nějaký předmět pohnul. Například dveře, okno, kancelářský šuplík, taška, auto, motocykl, kolo, kočárek, batoh, kufr …

Pokud v klidovém stavu dojde pohybu, je vyvolána událost `AlertStart`. Pokud v následujících 10 minutách znovu dojde k pohybu tak počítá opakování pohybu a po 10 minutách pošle `AlertContinue`. `AlertContinue` se opakuje dokud dochází k pohybu. Pokud je zařízení 10 minut od začátku nebo pokračování pohybu v klidu, posílá `AlertEnd`.

> MessageType: MovementDetectionContinuous

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [TamperOpen](#eventtype-tamperopen)         | Otevření krytu zařízení, rozepnutí bezpečnostního spínače. |
| [TamperClosed](#eventtype-tamperclosed) *   | Uzavření krytu zařízení, sepnutí bezpečnostního spínače. |
| [AlertStart](#eventtype-alertstart)         | Detekce začátku pohybu. |
| [AlertContinue](#eventtype-alertcontinue)   | Bohyb pokračuje. |
| [AlertEnd](#eventtype-alertend)             | Během 10 minut nedošlo k pohybu. |

\* TamperClosed aktuální verze senzorů nepodporuje

## Zařízení Magnetic
Rozpozná oddálení/přiblížení čidla od magnetu.

![Magnetic](images/devices/magnetic.png)

### Režim Simple
Pro identifikaci, že došlo k otevření/zavření skříní, oken, dveří nebo pro identifikaci vzdálení se nějakého předmětu od jiného.

Vždy při oddálení magnetu je vyvolána událost `AlertStart`. Při přiblížení magnetu zpět vyvolá `AlertEnd`.

> MessageType: MagneticDetectionSimple

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [TamperOpen](#eventtype-tamperopen)         | Otevření krytu zařízení, rozepnutí bezpečnostního spínače. |
| [TamperClosed](#eventtype-tamperclosed) *   | Uzavření krytu zařízení, sepnutí bezpečnostního spínače. |
| [AlertStart](#eventtype-alertstart)         | Magnet oddálen, začátek poplachu. |
| [AlertEnd](#eventtype-alertend)             | Magnet přiblížen zpět, konec poplachu. |

\* TamperClosed aktuální verze senzorů nepodporuje

### Režim Continuous
Pro sledování četnosti otevření/zavření dveří, krytů, průchodu pohyblivých částí.

Pokud v klidovém stavu dojde k oddálení magnetu, je vyvolána událost `AlertStart`. Na přiblížení magnetu nijak nereaguje, ale pokud dojde do 10 minut k opětovnému oddálení magnetu tak počítá kolikrát se oddálil a po 10 minutách pošle `AlertContinue`. `AlertContinue` se opakuje dokud se něco děje. Pokud se během 10 minut nic nestane (nedojde k oddálení magnetu), zařízení posílá `AlertEnd`.

> MessageType: MagneticDetectionContinuous

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [TamperOpen](#eventtype-tamperopen)         | Otevření krytu zařízení, rozepnutí bezpečnostního spínače. |
| [TamperClosed](#eventtype-tamperclosed) *   | Uzavření krytu zařízení, sepnutí bezpečnostního spínače. |
| [AlertStart](#eventtype-alertstart)         | Magnet oddálen, začátek poplachu. |
| [AlertContinue](#eventtype-alertcontinue)   | Dění na magnetu se opakuje, poplach pokračuje. |
| [AlertEnd](#eventtype-alertend)             | Během 10 minut nedošlo k oddálení magnetu, konec poplachu. |

\* TamperClosed aktuální verze senzorů nepodporuje

## Zařízení Pir
Detekuje pohyb nebo přítomnost člověka ve vymezeném prostoru do vzdálenosti 10m. 

![Pir](images/devices/pir.png)

### Režim Simple
Pro detekci, že v místnosti nebo vymezeném prostoru došlo k pohybu. 

Jakmile senzor detekuje pohyb pošle zprávu s událostí `AlertStart`. Pokud i nadále detekuje pohyb, pošle po 10 minutách zprávu s událostí `AlertContinue`, že pohyb pokračuje, kolik pohybů zaznamenal a kdy nastal poslední. Pokud i nadále detekuje pohyb pošle po dalších 10-ti minutách zprávu `AlertContinue`, že pohyb pokračuje a potom již zprávy o pokračování neposílá. Senzor pošle zprávu s událostí `AlertEnd`, že pohyb skončil pokud 10 minut nenastane žádný pohyb.

> MessageType: PirSimple

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [TamperOpen](#eventtype-tamperopen)         | Otevření krytu zařízení, rozepnutí bezpečnostního spínače. |
| [TamperClosed](#eventtype-tamperclosed) *   | Uzavření krytu zařízení, sepnutí bezpečnostního spínače. |
| [AlertStart](#eventtype-alertstart)         | Detekce začátku pohybu. |
| [AlertContinue](#eventtype-alertcontinue)   | Bohyb pokračuje. |
| [AlertEnd](#eventtype-alertend)             | Během 10 minut nedošlo k pohybu. |

\* TamperClosed aktuální verze senzorů nepodporuje

### Režim Continuous
Pro identifikaci, že se v místnosti nebo ohraničeném prostoru pohybuje člověk, kdy a jak často. 

Jakmile senzor detekuje pohyb pošle zprávu s událostí `AlertStart`. Pokud i nadále detekuje pohyb, posílá v 10 minutových intervalech zprávy s událostí `AlertContinue`, že pohyb pokračuje, kolik pohybů zaznamenal a kdy nastal poslední. Senzor pošle zprávu s událostí `AlertEnd`, že pohyb skončil pokud 10 minut nenastane žádný pohyb.

> MessageType: PirContinuous

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [TamperOpen](#eventtype-tamperopen)         | Otevření krytu zařízení, rozepnutí bezpečnostního spínače. |
| [TamperClosed](#eventtype-tamperclosed) *   | Uzavření krytu zařízení, sepnutí bezpečnostního spínače. |
| [AlertStart](#eventtype-alertstart)         | Detekce začátku pohybu. |
| [AlertContinue](#eventtype-alertcontinue)   | Bohyb pokračuje. |
| [AlertEnd](#eventtype-alertend)             | Během 10 minut nedošlo k pohybu. |

\* TamperClosed aktuální verze senzorů nepodporuje

## Zařízení AlertButton
Zařízení s tlačítkem pro přivolání pomoci nebo spuštění poplachu.

![AlertButton](images/devices/panic.png)
![AlertButton](images/devices/sos.png)

### Režim Simple
Zařízení po stitknutí tlačítka pošle zprávu s událostí `AlertStart`.

> MessageType: AlertButtonSimple

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [AlertStart](#eventtype-alertstart)         | Stisknuto, začátek poplachu. |

## Zařízení Thermometer
Měří teplotu okolního prostředí.

![Thermometer](images/devices/humiditymeter.png)

### Režim Momentary
Jednou za X minut provede měření teploty a odešle událost `MeasuredTemperature`.

> MessageType: ThermometerMomentary

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [MeasuredTemperature](#eventtype-measuredtemperature) | Naměřené veličiny. |

### Režim Average
Každou minutu měří teplotu. Po X měření provede výpočet průměrné hodnoty a odešle událost `MeasuredTemperature`.

> MessageType: ThermometerAverage

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [MeasuredTemperature](#eventtype-measuredtemperature) | Naměřené veličiny. |

### EventType MeasuredTemperature
Nastává při odeslání naměřené hodnoty.

Dodatečné předávané parametry:
| Parametr          | Typ     | Povinný | Popis
| :-----------------|:--------|:--------|:-----
| Temperature       | float   | ano     | naměřená teplota

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "ThermometerMomentary",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "MeasuredTemperature",
    "Temperature": 25.5
}
```

## Zařízení HumidityMeter
Měří teplotu a vlhkost okolního prostředí.

![HumidityMeter](images/devices/humiditymeter.png)
![HumidityMeter](images/devices/movementdetection.png)

### Režim Momentary
Jednou za X minut provede měření teploty a vlhkosti a odešle událost `MeasuredHumidityTemperature`.

> MessageType: HumidityMeterMomentary

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [MeasuredHumidityTemperature](#eventtype-measuredhumiditytemperature) | Naměřené veličiny. |

### Režim Average
Každou minutu měří teplotu a vlhkost. Po X měření provede výpočet průměrné hodnoty a odešle událost `MeasuredHumidityTemperature`.

> MessageType: HumidityMeterAverage

| EventType                                   | Popis |
|:--------------------------------------------|:------|
| [Restart](#eventtype-restart)               | Restart zařízení. |
| [Alive](#eventtype-alive)                   | Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení. |
| [Transport](#eventtype-transport)           | Přechod do transportního režimu - neaktivní stav s minimální spotřebou. |
| [MeasuredHumidityTemperature](#eventtype-measuredhumiditytemperature) | Naměřené veličiny. |

### EventType MeasuredHumidityTemperature
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
    "DeviceId": "abc123",
    "MessageType": "HumidityMeterMomentary",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "MeasuredHumidityTemperature",
    "Temperature": 25.5,
    "Humidity": 27.5
}
```

# Společné události
Události na ktré je odkázáno z konkrétních typů zpráv.

## EventType Restart
Nastává při restartu zařízení. K restartu může dojít stiskem restartovacího tlačítka umístěného na plošném spoji čidla nebo restart může vyvolat firmware čidla při hardwarové chybě nebo u některých změn konfigurace čidla pomocí příkazu zaslaného prostřednictvím downlink API.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "Restart"
}
```

## EventType Alive
Nastává v pravidelném intervalu, potvrzuje funkčnost zařízení.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "Alive"
}
```

## EventType Transport
Nastává při přechodu senzoru do transportního režimu, ke kterému dojde po vložení nové baterie nebo pomocí příkazu zaslaného prostřednictvím downlink API.

Čidlo v transportním režimu má velmi nízkou spotřebu a neposílá žádné zprávy. Pro probuzení čidla z transportního režimu je třeba stisknout RESET tlačítko umístěné na plošném spoji čidla.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "Transport"
}
```

## EventType TamperOpen
Nastává při rozepnutí tamperu - otevření krytu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "TamperOpen"
}
```

## EventType TamperClosed
Nastává při sepnutí tamperu - zavření krytu senzoru.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "TamperClosed"
}
```

## EventType AlertStart
Nastává při prvním vzniku události jako např. zaplavení kontaktů, oddálení magnetu, zaregistrování pohybu.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "AlertStart"
}
```

## EventType AlertContinue
Nastává pokud událost pokračuje.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "AlertContinue"
}
```

## EventType AlertEnd
Situace kdy událost nastane je popsána u každého zařízení které tuto událost zasílá.

Ukázka zaslané události:
```yaml
{
    "ProtocolVersion": 1,
    "DeviceId": "abc123",
    "MessageType": "MagneticDetectionSimple",
    "EventId": "c4056fc4-d433-4d2c-bb7f-23a691fd3dac",
    "EventTime": "2021-05-03T14:25:31.8437511Z",
    "EventType": "AlertEnd"
}
```