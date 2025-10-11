# Yoradio для ESP32-4848S040 (ST7701 RGB 480x480)

**Ветка:** `4848S040`  
**Версия:** v0.9.434(m)  
**Статус:** ✅ В работе!

![photo_2025-08-15_00-06-22](https://github.com/user-attachments/assets/bb047124-7e1a-4d0a-8f70-b78dccf4fd36)
SA Mode

![photo_2025-09-22_20-27-50](https://github.com/user-attachments/assets/b5d05ff3-b0bf-4b8a-866a-7c32e659e3fa)
VU meter mode 

## 📝 Лог изменений

**11.10.2025** - Исправление размера буфера heapbar:
- ✅ Исправлен максимальный размер буфера heapbar с 300000 на 655350 (UINT16_MAX*10)
- ✅ Устранена проблема переполнения виджета heapbar - зеленая полоса больше не выходит за границы
- ✅ Теперь корректно отображается процент заполнения буфера

**06.10.2025** - Исправление бага переключения VU-meter ↔ Spectrum Analyzer:
- ✅ Устранено самопроизвольное переключение между VU и SA при возврате на страницу плейера
- ✅ Убрана дублирующая логика переключения виджетов из обработчика SHOWVUMETER
- ✅ Добавлен вызов _layoutChange() при возврате в PLAYER режим
- ✅ Исправлено корректное отключение виджетов при выключении VU meter
- ✅ Логика переключения централизована в функции _layoutChange()

**05.10.2025** - Исправление AP Mode при пустом SPIFFS:
- ✅ Устранено зависание на boot logo при первом запуске с пустым SPIFFS
- ✅ Исправлена блокировка flush экрана (_suspendFlush) в AP режиме
- ✅ Добавлен принудительный flush AP screen для корректного отображения
- ✅ Корректный переход в AP Mode и отображение страницы настройки


**29.09.2025** - Рефакторинг конфигурации спектр-анализатора и фикс виджета битрейта:
- ✅ Перенесена конфигурация `SpectrumWidgetConfig` из отдельного файла `spectrum_st7701.h` в основной конфигурационный файл `displayST7701conf.h`
- ✅ Упрощена архитектура проекта - все настройки виджетов дисплея теперь в одном месте
- ✅ Удален файл `spectrum_st7701.h` как больше не нужный
- ✅ Обновлен `display.cpp` для использования новой структуры конфигурации
- ✅ Применен фикс переполнения виджета битрейта в `widgets.cpp`
- ✅ Добавлена проверка границ для отображения битрейта и формата файла
- ✅ Автоматическое переключение на целые числа при переполнении виджета
- ✅ Протестирована компиляция - все работает корректно

**12.09.2025** - SA ↔ VU переключатель в вебе и стабильность отрисовки:
- ✅ Добавлен тумблер `spectrum analyzer` на странице Settings → Screen (рядом с `numbered playlist`)
- ✅ VU: инверсия уровней перенесена после сглаживания (корректно работает `fadespeed`), дельта‑отрисовка без фликера,  аудиобиблиотека инвертирует вывод для VU.(?)



**09.09.2025** - Добавил файлы bin для заливки в устройство:
- ✅ bootloader.bin
- ✅ firmware.bin
- ✅ partitions.bin
- ✅ Добавлены в каталог build_bin/

**03.09.2025** - Исправлена проблема с отображением текста заглавными буквами:
- ✅ Исправлена функция gfxDrawText для поддержки параметра uppercase
- ✅ Обновлены все виджеты (TextWidget, ScrollWidget, NumWidget, BitrateWidget)
- ✅ Исправлено отображение в дисплее ST7701 (дата, время, плейлист)
- ✅ Обеспечено единообразное отображение всех элементов плейлиста
- ✅ Текст теперь отображается согласно настройкам конфигурации

**03.09.2025** - Исправлены оси тачскрина для выбора станций:
- ✅ Свайп вверх → следующая станция
- ✅ Свайп вниз → предыдущая станция
- ✅ Оптимизирована логика обработки свайпов

**02.09.2025** - Добавлена интеграция SD карты:
- ✅ Настроена конфигурация SPI для SD карты (FSPI/SPI2_HOST)
- ✅ Исправлен deadlock в mutex для функции `_checkNoMedia()`
- ✅ SD карта успешно инициализируется и индексируется
- ✅ Плейлист создается и корректно читается
- ✅ Список композиций отображается и листается
- ❌ **ПРОБЛЕМА:** Воспроизведение не работает - система зависает в `config.loadStation()`

**См. [ISSUE: SD Card Playback Problem](#-sd-card-playback-issue)**

> **⚠️ ВАЖНО:** Для стабильной работы интернет-радио на больших битрейтах необходимо заменить стандартные библиотеки IDF на перекомпилированные версии. См. раздел [Установка библиотек](#-установка-библиотек).


## 🎯 Описание

Порт проекта [Yoradio] для модуля **ESP32-4848S040** с дисплеем **ST7701**:
- **Разрешение:** 480x480 пикселей (квадратный экран)
- **Интерфейс:** RGB Panel (параллельный, 16 линий данных)
- **Процессор:** ESP32-S3-WROOM-1-N16R8 (240MHz, dual-core)
- **Память:** 520KB RAM + 16MB PSRAM + 8MB Flash
- **Тачскрин:** GT911 (емкостный, I2C) - **полностью оптимизирован**
- Библиотеки I2S - 3.3.2

-   Порядок подключения DAC к разъему:
![S3c5b9bd4f2a54006af227f2b4acd0fc1b](https://github.com/user-attachments/assets/c49b36b6-3b10-4db5-ad03-bf7ecc4aca3d)

![photo_2025-09-09_19-40-49](https://github.com/user-attachments/assets/e45c9c79-ed04-4fc9-8e6e-d9d97bda08d4)

в модуле есть встренный моно DAC. По умолчанию отключен - пины переключены на задний разъем. 
Для переключения модуля на встроенный DAC - запаиваем перемычки R21, R22, R23. 
Если, при этом, задним разьемом пользоваться не будете - то резисторы R26, R27, R25 можно не убирать.. иначе их нужно выпаять


![photo_2025-09-09_19-57-01](https://github.com/user-attachments/assets/2abafab0-3ece-4569-8181-4b2ded7a6a62)



## 🚀 Особенности

- ✅ **ST7701 RGB Panel 480x480** - высокое разрешение
- ✅ **GT911 тачскрин** - с поддержкой мультитача и оптимизацией для квадратного экрана
- ✅ **Спектральный анализатор** - замена VU-метра
- ✅ **Русская локализация** - полная поддержка
- ✅ **WiFi интернет-радио** - основной функционал
- ✅ **Оптимизация для ESP32-S3** - максимальная производительность
- ✅ **Профессиональное управление жестами** - точные свайпы и мультитач

## 🔧 Сборка

Для русской локализации заменяем файл fonts/glcdfont.h по адресу: pio\libdeps\yoradio-esp32s3\GFX Library for Arduino\src\font


## 📱 Конфигурация

- **`src/myoptions.h`** - основные настройки модуля
- **`src/src/displays/displayST7701.cpp`** - драйвер дисплея
- **`src/src/core/touchscreen.cpp`** - драйвер тачскрина GT911 (оптимизирован)

## 🔧 Установка библиотек

Для решения проблемы заикания интернет-радио на больших битрейтах необходимо заменить стандартные библиотеки IDF на перекомпилированные версии:

### 📁 Замена библиотек

1. **Найдите папку PlatformIO:**
   ```
   <user>/.platformio/packages/framework-arduinoespressif32-libs/esp32s3/lib/
   ```

2. **Замените файлы:**
   - `libesp_netif.a` → `libesp_netif.a` (из архива)
   - `liblwip.a` → `liblwip.a` (из архива)

3. **Поддерживаемые версии IDF:**
   - ✅ ESP-IDF 5.4
   - ✅ ESP-IDF 5.5

### 📦 Архив с библиотеками

Перекомпилированные библиотеки находятся в папке `library!/esp32s3_5.4/`:
- `libesp_netif.a` (496KB)
- `liblwip.a` (3.8MB)

> **Примечание:** После замены библиотек перезапустите PlatformIO и перекомпилируйте проект.

## 🎮 Управление

- **Одиночное касание** - старт/стоп
- **Свайп влево/вправо** - изменение громкости (оптимизировано)
- **Свайп вверх/вниз** - выбор станции (оптимизировано)
- **Двойное касание** - смена режима (RADIO ↔ SD)
- **Мультитач** - дополнительные функции управления

## 📊 Технические характеристики

| Компонент | Характеристики |
|-----------|----------------|
| **Дисплей** | ST7701 RGB Panel 480x480 |
| **Тачскрин** | GT911 (SDA:19, SCL:45) - оптимизирован для 480x480 |
| **Аудио** | I2S (BCLK:1, LRCLK:2, DOUT:40) |
| **Подсветка** | PWM управление (пин 38) |
| **Формат** | RGB565 (16-bit color) |

## 🔍 Отладка

```cpp
// Включить отладку дисплея
#define DEBUG_DISPLAY

// Включить отладку тачскрина (опционально)
#define DEBUG_TOUCH
```

## 🐛 Известные проблемы

- ❌ **SD карта: воспроизведение не работает** (см. раздел ниже)
- ✅ **Тачскрин GT911 полностью исправлен и оптимизирован**

## 🚨 SD Card Playback Issue

### Problem Description
SD card integration works perfectly for indexing and playlist reading during boot/initialization, but **hangs during audio playback** when user presses Play button.

### Specific Failure Point
The system hangs in `config.loadStation()` function when attempting to open SD card files:
```cpp
File playlist = SDPLFS()->open(REAL_PLAYL, "r");  // ← HANGS HERE
File index = SDPLFS()->open(REAL_INDEX, "r");     // ← NEVER REACHED
```

### Working Scenarios
- ✅ SD card initialization and mounting
- ✅ SD card indexing (creates playlistsd.csv and indexsd.dat)
- ✅ Playlist reading during boot sequence
- ✅ All cardPresent() checks pass
- ✅ File existence checks pass

### Failing Scenario
- ❌ Audio playback when user presses Play button
- ❌ System hangs on SDPLFS()->open() calls in loadStation()

### Technical Details
- **Board:** ESP32-S3 N16R8
- **Display:** ST7701 RGB 480x480 (uses VSPI)
- **SD Card:** FSPI (SPI2_HOST) with custom pins: CS=42, SCK=48, MISO=41, MOSI=47
- **Audio:** I2S (no SPI conflict)
- **FreeRTOS:** mutex system implemented for thread safety

### Attempted Solutions
1. Changed SD card SPI from FSPI to HSPI - no effect
2. Added mutex timeouts and forced release - no effect
3. Simplified cardPresent() check (removed readRAW) - no effect
4. Suspended display task during SD access - no effect
5. Removed all SD state checks - still hangs on file open

### Root Cause Hypothesis
The issue appears to be in the SD card file system layer (SDPLFS()->open()) rather than SPI communication or mutex management. The same SD card operations work fine during boot but fail during runtime playback context.

### Developer Request
If you can identify why SDPLFS()->open() works during boot but hangs during playback, please help resolve this issue. The SD card hardware and basic file operations are confirmed working.

## 📝 Лицензия

MIT License - см. файл [LICENSE](LICENSE)

---

**Версия для ветки 4848S040** - оптимизирована под модуль ESP32-4848S040 с дисплеем ST7701 и полностью функциональным тачскрином GT911

