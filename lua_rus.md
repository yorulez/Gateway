# Поддержка lua скриптов

В шлюзе SLS имеется  поддержка автоматизаций на основе скриптового языка программирования [lua](https://ru.wikipedia.org/wiki/Lua). Редактор скриптов находится в меню Actions -> Scripts.  

Для написания  скрипта необходимо  создать новый файл, например  с именем script.lua и в него ввести  код на языке lua. 


Скриптовый stdout по команде print выводит информацию на экран страницы Scripts и в системный лог шлюза. Запустить скипт для тестов можно нажатием кнопки RUN.

При запуске шлюза выполняется файл /init.lua
![](/img/lua.png)


## Варианты запуска скриптов
1) Запуск скрипта при изменении состояния устройства. На вкладке Zigbee необходимо зайти в параметры устройства и в окне SimpleBind указать имя файла скрипта (префикс / необязателен).
2) Запуск скрипта по времени. Ожидается.
3) Запуск скрипта по подписке mqtt. Ожидается.
4) Запуск скрипта при вызове http api. /api/scripts?action=evalFile&path=/test.lua.


## Список доступных функций и структур
1) [http.request()](lua_rus.md#httprequest)
2) [zigbee.value()](lua_rus.md#zigbeevalue)
3) [zigbee.get()](lua_rus.md#zigbeeget)
4) [zigbee.set()](lua_rus.md#zigbeeset)
5) [Event](lua_rus.md#event)
6) [os.time()](lua_rus.md#ostime) 
7) [obj.get()/obj.set()](lua_rus.md#objget--objset) 
8) [mqtt.pub()](lua_rus.md#mqttpub) 


### http.request 
Вызов URL запроса http.request(url[:port], [method, headers, body])

В данный момент поддерживается только 'http://' протокол.


Пример переключение gpio 12 для прошивки wifi-iot
```
http.request("http://192.168.1.34/gpio?st=2&pin=12")
```

Пример отправки POST запроса:
```
http.request("http://postman-echo.com:80/post?foo1=bar1", "POST", "Content-Type: text/text; charset=utf-8\r\n", "body") 
```

Пример переключения реле sw1 в прошивке espHome:

```
  http.request("http://192.168.1.71/switch/sw1/toggle", "POST") 
```

Пример переключение gpio для MegaD при однократном нажатии btn_2 пульта Jager
```
  if Event.State.Value == "btn_2_single"  then
    http.request("http://192.168.2.200/objects/?object=MegaD1-12&op=m&m=switch")
  end
```

Запрос инфомации со стороннего ресурса
```
local Response = http.request("http://wtfismyip.com/text")
print("My IP: " .. Response)
```


### zigbee.value()
Получение значения состояния устройства из кэша zigbee.value("ieeard", "temperature")

```
-- Получаем значение температуры и округляем до целых  
temp = zigbee.value("0x00158D0001A2D2FE", "temperature")
temp = math.floor(temp)
print("Текущая температура: " .. temp .. " C°")
```

Вместо адреса устройства можно испрользовать FriendlyName (в том числе кириллицу), либо текущий адрес устройства в сети (0x9EC8).
```
-- Получаем значение температуры и округляем до целых  
temp = zigbee.value("датчик в комнате", "temperature")
temp = math.floor(temp)
print("Текущая температура: " .. temp .. " C°")
```

### zigbee.get()
Вызывает команду GET в ковертере. Используется для ручного чтения состояний из устройств.

Пример: 
```
  zigbee.get("lamp1", "brightness")
```


### zigbee.join()
Синтаксис: zigbee.join(duration, [target])

Открывает сеть для подключения новых устройств на duration секунд (макс. 255), для устройства target или для всей сети. 

```
  zigbee.join(255, "plug1")
```


### zigbee.set()
Установка значения  устройства zigbee.set(Ident, StateName, StateValue)

Пример скрипта, который при нажатии кнопки выключателя lumi.sensor_switch включает освещение lamp_1:
```
if zigbee.value("lumi.sensor_switch", "click") == "single" then
  -- toggle lamp
  current_brightness = zigbee.value("lamp_1", "brightness")
  if current_brightness == 0 then
    zigbee.set("lamp_1", "brightness", 255)
  else
    zigbee.set("lamp_1", "brightness", 0)
  end
end
 


-- switch 0x00124B0009FE36FC on single lumi.sensor_switch click
if Event.State.Value == "single" then
   zigbee.set("0x00124B0009FE36FC", "state", "toggle")
  end

```

### Event
Структура Event например позволяет использовать один и тот же скрипт для разных состояний или устройств.

Возможные варианты использования:
Event.Name - Имя файла скрипта
Event.nwkAddr - nwkAddr устройства, которое вызывало скрипт
Event.ieeeAddr - ieeeAddr устройства, которое вызывало скрипт
Event.FriendlyName - FriendlyName устройства, которое вызывало скрипт
Event.State.Name - Имя состояния которое вызвало скрипт
Event.State.Value - Новое значение состояния

Пример скрипта для включения света:
```
if Event.State.Value == "single" then 
  value = 255 
elseif Event.State.Value == "double" then 
  value = 0 
else 
  return 
end
zigbee.set("lamp_1", "brightness", value)
```


### os.time()
os.time() возвращает Unix время.


### os.delay()
Синтаксис: os.delay(ms)

Пауза на ms миллисекунд (1сек = 1000 мс)


### os.millis()
Возвращает количество миллисекунд с момента загрузки системы


### os.save()
Сохраняет данные


### os.restart()
Перезагружает ОС

### os.ping(host[, count])
Отправляет ICMP PING запросы, возвращает среднее время или -1 при недоступности.


###  obj.get() / obj.set()
obj.get(ObjectName) / obj.set(ObjectName, ObjectValue) для сохранения и получения объекта для обмена данными между скриптами


### mqtt.pub()
Синтаксис: mqtt.pub(topic, payload) 

Публикует на MQTT сервер в топик topic значение payload. 


Пример управления реле на прошивке Tasmota - cmnd/имя устройства/имя реле 
```
  mqtt.pub('cmnd/sonoff5/power', 'toggle')
```

### Включение "режима сопряжения" при нажатии на боковую кнопку  шлюза
При нажатии на кнопку вызывается сценарий btn_sw1.lua
В сценарий нужно добавить следующий код:
```
zigbee.join(255, "0x0000")
```


### Управление GPIO
gpio.mode(pin, mode)

gpio.read(pin) - чтение цифрового 

gpio.read(PIN, true) - чтение ADC

gpio.write(pin, level)

### Управление звуком
audio.playurl(url)

audio.geturl()

audio.stop()

audio.setvolume(volume_procent)

audio.getvolume()

audio.getstatus()


### Ежеминутный таймер
Просто создайте скрипт с именем OneMinTimer.lua, он будет запускаться каждую минуту.

Пример отправки данных каждую минуту на https://narodmon.ru
```
function SendNarodmon(name, value)
  local MAC = "BC:DD:C2:D7:68:BC"
  http.request("http://narodmon.ru/get?ID=" .. MAC .. "&" .. name .. "=" .. tostring(value))
end  

local value = zigbee.value("0x04CF8CDF3C771F6C", "illuminance")
SendNarodmon("illuminance", value)
```


### Отправка сообщения в телеграм с помощью вашего бота
```
local char_to_hex = function(c)
  return string.format("%%%02X", string.byte(c))
end

function round(exact, quantum)
    local quant,frac = math.modf(exact/quantum)
    return quantum * (quant + (frac > 0.5 and 1 or 0))
end
local function urlencode(url)
  if url == nil then
    return
  end
  url = url:gsub("\n", "\r\n")
  url = url:gsub("([^%w ])", char_to_hex)
  url = url:gsub(" ", "+")
  return url
end

local hex_to_char = function(x)
  return string.char(tonumber(x, 16))
end

function SendTelegram(text)
  local token = "5177...:AAG0b...."  --
  local chat_id = "38806....."
  --http.request("https://api.telegram.org/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. tostring(text))  -- https пока не работает в lua
  http.request("http://212.237.16.93/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. urlencode(text))
 end  




local value = zigbee.value("0x00158D00036C1508", "temperature")
local text = "temperature: " .. round(tostring(value),2)
SendTelegram(text)

```

### Уведомление в телеграм об открытии двери 
```
local char_to_hex = function(c)
  return string.format("%%%02X", string.byte(c))
end

function round(exact, quantum)
    local quant,frac = math.modf(exact/quantum)
    return quantum * (quant + (frac > 0.5 and 1 or 0))
end
local function urlencode(url)
  if url == nil then
    return
  end
  url = url:gsub("\n", "\r\n")
  url = url:gsub("([^%w ])", char_to_hex)
  url = url:gsub(" ", "+")
  return url
end

local hex_to_char = function(x)
  return string.char(tonumber(x, 16))
end

function SendTelegram(text)
  local token = "517781...:AAG0..."
  local chat_id = "38806...."
  --http.request("https://api.telegram.org/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. tostring(text))  -- https пока не работает в lua
  http.request("http://212.237.16.93/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. urlencode(text))
  end  


local state =  zigbee.value(tostring(Event.ieeeAddr), "contact")
if (state) then SendTelegram("Дверь открыта") 
else   SendTelegram("Дверь закрыта")  end 
```
###
Оповещение в телеграм при сработке датчика движения
```
local char_to_hex = function(c)
  return string.format("%%%02X", string.byte(c))
end

function round(exact, quantum)
    local quant,frac = math.modf(exact/quantum)
    return quantum * (quant + (frac > 0.5 and 1 or 0))
end
local function urlencode(url)
  if url == nil then
    return
  end
  url = url:gsub("\n", "\r\n")
  url = url:gsub("([^%w ])", char_to_hex)
  url = url:gsub(" ", "+")
  return url
end

local hex_to_char = function(x)
  return string.char(tonumber(x, 16))
end

function SendTelegram(text)
  local token = "517781...:AAG0bv...."
  local chat_id = "3880......"
  --http.request("https://api.telegram.org/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. tostring(text))  -- https пока не работает в lua
  http.request("http://212.237.16.93/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. urlencode(text))
  end  


local state =  zigbee.value(tostring(Event.ieeeAddr), "occupancy")
if (state) then SendTelegram("Датчик движения ".. Event.ieeeAddr  .." обнаружил активность") 
 else  SendTelegram("Значение датчика ".. Event.FriendlyName .." движения нормализовалось") 
  
end 
```

###  Оповещение об изменении значения датчика температуры/влажности
```
local char_to_hex = function(c)
  return string.format("%%%02X", string.byte(c))
end



function round2(num, numDecimalPlaces)
  local mult = 10^(numDecimalPlaces or 0)
  return math.floor(num * mult + 0.5) / mult
end

local function urlencode(url)
  if url == nil then
    return
  end
  url = url:gsub("\n", "\r\n")
  url = url:gsub("([^%w ])", char_to_hex)
  url = url:gsub(" ", "+")
  return url
end

local hex_to_char = function(x)
  return string.char(tonumber(x, 16))
end

function SendTelegram(text)
    local token = "5177....:AAG0......"
  local chat_id = "3880..."
  --http.request("https://api.telegram.org/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. tostring(text))  -- https пока не работает в lua
  http.request("http://212.237.16.93/bot" .. token .. "/sendMessage?chat_id=" .. chat_id .. "&text=" .. urlencode(text))
  end  

local temp =  round2(zigbee.value(tostring(Event.ieeeAddr), "temperature"),1)
local hum =  round2(zigbee.value(tostring(Event.ieeeAddr), "humidity"),1)
SendTelegram("Значение ДТВ ".. Event.FriendlyName .. " ".. temp.."° / " .. hum .. "%") 
------------  отправка значения на narodmon
function SendNarodmon(name, value)
  local MAC =tostring(Event.ieeeAddr)
  http.request("http://narodmon.ru/get?ID=" .. MAC .. "&" .. name .. "=" .. tostring(value))
end  

SendNarodmon("temperature", temp)
SendNarodmon("humidity", hum)
```


### Упрощенная отправка сообщений в телеграм (начиная с версии 20200915)
```
telegram.settoken("5961....:AAHJP4...")
telegram.setchat("5748.....")
telegram.send("Температура: " .. string.format("%.2f", zigbee.value(tostring(Event.ieeeAddr), "temperature")) .. "°C, Влажность: " .. string.format("%.2f",zigbee.value(tostring(Event.ieeeAddr), "humidity")) .. "%")
```





## Полезные ссылки 
1) On-line учебник по [lua](https://zserge.wordpress.com/2012/02/23/lua-%D0%B7%D0%B0-60-%D0%BC%D0%B8%D0%BD%D1%83%D1%82/)

2) Генератор lua скриптов  на основе [Blockly](https://blockly-demo.appspot.com/static/demos/code/index.html)

