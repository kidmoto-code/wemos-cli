# Работа с Wemos D1 Mini через Arduino CLI

Полное руководство по настройке и программированию Wemos D1 Mini через терминал.

## 📋 Предварительная настройка

### 1. Установка Arduino CLI

```bash
sudo pacman -S arduino-cli
```

### 2. Добавление поддержки ESP8266
```bash
# Добавляем URL с платами ESP8266
arduino-cli config add board_manager.additional_urls http://arduino.esp8266.com/stable/package_esp8266com_index.json

# Альтернативный URL (если первый не работает):
# arduino-cli config add board_manager.additional_urls https://github.com/esp8266/Arduino/releases/download/2.6.3/package_esp8266com_index.json

# Обновляем индекс пакетов
arduino-cli core update-index

# Устанавливаем платформу ESP8266
arduino-cli core install esp8266:esp8266
```

**Примечание:** Если загрузка прерывается по таймауту, повторите команду установки.

### 3. Проверка установленных плат
```bash
arduino-cli board listall | grep -i wemos
```

Ожидаемый вывод:
```
LOLIN(WEMOS) D1 R2 & mini       esp8266:esp8266:d1_mini
LOLIN(WEMOS) D1 mini Lite       esp8266:esp8266:d1_mini_lite
LOLIN(WEMOS) D1 mini Pro        esp8266:esp8266:d1_mini_pro
WeMos D1 R1                     esp8266:esp8266:d1
```

### 4. Определение группы устройства
```bash
ls -la /dev/ttyUSB0
```

Ожидаемый вывод:
```
crw-rw---- 1 root uucp 188, 0 окт 29 21:27 /dev/ttyUSB0
```
**Запомните группу** (в примере: `uucp`)

### 5. Добавление пользователя в группу
```bash
# Добавляем пользователя в группу (замените uucp на вашу группу)
sudo usermod -a -G uucp $USER

# Распространенные варианты групп:
# sudo usermod -a -G dialout $USER
# sudo usermod -a -G tty $USER
# sudo usermod -a -G plugdev $USER
```

### 6. Применение изменений и проверка
```bash
# Выйдите и войдите заново в систему или перезагрузите компьютер
logout

# Проверяем группы пользователя
groups
```
**Убедитесь, что нужная группа есть в списке**

### Временное решение (если права не применяются)
```bash
sudo chmod 666 /dev/ttyUSB0
```

## 🚀 Работа со скетчами

### 7. Создание и компиляция скетча
```bash
# Создаем отдельную директорию для каждого проекта
mkdir my_project && cd my_project

# Создаем файл скетча
cat > sketch.ino << 'EOF'
void setup() {
  Serial.begin(9600);
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("Включено");
  delay(1000);
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("Выключено");
  delay(1000);
}
EOF
```

### 8. Компилируем скетч
```bash
arduino-cli compile --fqbn esp8266:esp8266:d1_mini sketch.ino
```

### 9. Загрузка скетча на устройство
```bash
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp8266:esp8266:d1_mini sketch.ino
```

**Примечание:** Если загрузка не начинается, нажмите кнопку RST на Wemos D1 Mini.

### 10. Мониторинг последовательного порта
```bash
# Запускаем монитор с отключенным DTR (чтобы избежать сброса)
arduino-cli monitor -p /dev/ttyUSB0 --config baudrate=9600 --config dtr=off --config rts=off
```

## 🛠️ Полезные команды

### Просмотр подключенных устройств
```bash
arduino-cli board list
```

### Информация о плате
```bash
arduino-cli board details esp8266:esp8266:d1_mini
```

### Поиск библиотек
```bash
arduino-cli lib search "WiFi"
```

### Создание конфигурационного файла для монитора
```bash
arduino-cli monitor --port /dev/ttyUSB0 --config baudrate=9600 --config dtr=off --config-file monitor.conf
arduino-cli monitor --config-file monitor.conf
```

## ⚠️ Частые проблемы и решения

### Неправильная скорость порта
Если монитор показывает "абракадабру", попробуйте другие скорости:
```bash
arduino-cli monitor -p /dev/ttyUSB0 --config baudrate=115200
arduino-cli monitor -p /dev/ttyUSB0 --config baudrate=74880
arduino-cli monitor -p /dev/ttyUSB0 --config baudrate=57600
```

### Устройство не определяется
```bash
# Проверяем доступные порты
ls /dev/ttyUSB*
ls /dev/ttyACM*

# Переподключаем устройство
```

### Проблемы с правами доступа
```bash
# Временное решение
sudo chmod 666 /dev/ttyUSB0

# Постоянное решение через udev
sudo nano /etc/udev/rules.d/99-arduino.rules
```
Добавьте строку:
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", MODE="0666"
```

## 📝 Пример рабочего скетча
```cpp
void setup() {
  Serial.begin(9600);
  pinMode(LED_BUILTIN, OUTPUT);
  Serial.println("Устройство запущено");
}

void loop() {
  digitalWrite(LED_BUILTIN, LOW);
  Serial.println("Светодиод ВКЛЮЧЕН");
  delay(1000);
  
  digitalWrite(LED_BUILTIN, HIGH);
  Serial.println("Светодиод ВЫКЛЮЧЕН");
  delay(1000);
}
```

---

**Примечание:** На Wemos D1 Mini светодиод работает инвертировано:
- `LOW` - светодиод горит
- `HIGH` - светодиод выключен
