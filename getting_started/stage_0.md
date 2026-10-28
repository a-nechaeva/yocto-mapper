# Начало работы. Базовая сборка двух образов
## Установка хостовых пакетов
```bash
sudo apt update
sudo apt install -y build-essential chrpath cpio debianutils diffstat file \
  gawk gcc git iputils-ping libacl1 liblz4-tool locales python3 python3-git \
  python3-jinja2 python3-pexpect python3-pip python3-subunit socat texinfo \
  unzip wget xz-utils zstd qemu-system-x86
```
После сгенерируем локаль, чтобы избежать проблем с кодировкой
```bash
sudo locale-gen en_US.UTF-8
```
Создадим рабочую директорию
```bash
mkdir -p ~/yocto && cd ~/yocto
```

## Клонирование Poky
Необходимо выбрать стабильную ветку, возьмем Scarthgap (5.0 LTS), она стабильна и имеет предсказуемое поведение с SBOM/CVE.

```bash
git clone https://git.yoctoproject.org/poky.git -b scarthgap --depth=1
```
*ДЛЯ ДАЛЬНЕЙШЕГО ОБНОВЛЕНИЯ БУДЕМ ИСПОЛЬЗОВАТЬ СЛЕДУЮЩУЮ КОМАНДУ:*
```bash
git pull --depth=1 origin scarthgap
```

Инициализируем директорию для сборки, сначала для варианта А (аналогично инициализируется вторая конфигурация сборки)
```bash
cd ~/yocto/poky
source oe-init-build-env build-a
```
Вторая команда активирует окружение и ее необходимо применять каждый раз перед запуском конкретной сборки, так как `source oe-init-build-env` изменяет переменные окружения для текущей сессии терминала, также нельзя активировать две среды сборки одновременно в одном терминале.

## Настройка конфигов
В каждой директории сборки редактируем файл `conf/local.conf`:
* Для варианта A (без HTTP/2):
  
  ```bash
  # ------
  MACHINE ?= "qemux86-64"
  IMAGE_INSTALL:append = " curl"

  # Ограничения для ноутбука (16 ГБ ОЗУ)
  BB_NUMBER_THREADS = "4"
  PARALLEL_MAKE = "-j 4"
  BB_SERVER_TIMEOUT = "120"
  INHERIT += "rm_work"
  ```
* Для варианта B:
  
  ```bash
  # ------
  MACHINE ?= "qemux86-64"
  IMAGE_INSTALL:append = " curl"
  PACKAGECONFIG:append:pn-curl = " nghttp2"

  # Ограничения для ноутбука (16 ГБ ОЗУ)
  BB_NUMBER_THREADS = "4"
  PARALLEL_MAKE = "-j 4"
  BB_SERVER_TIMEOUT = "120"
  INHERIT += "rm_work"
  ```
  Строка `PACKAGECONFIG:append:pn-curl` означает: для пакета (`pn-`) curl применить настройку сборки (`PACKAGECONFIG`), а именно -- включить функцию `nghttp2`. Yocto сам увидит эту настройку, поймет, что для нее нужна библиотека `nghttp2`, скачает и соберет ее, а затем скомпилирует curl с флагом `--with-nghttp2`.


## Сборка
В директории необходимой конфигурации сборки (a или b) запускаем
```bash
bitbake core-image-minimal
```
После этого BitBake выполняет для каждого пакета строго определенный конвейер задач:
1. `do_fetch` (скачивание): BitBake ищет исходный код для каждого компонента в интернете (или в локальном кэше) и скачивает его архивы. Все скачанные файлы складываются в общую папку `build/downloads/` (переменная `DL_DIR`).
2. `do_unpack` (распаковка): скачанные архивы распаковываются во временную рабочую директорию (`tmp/work/...`).
3. `do_patch` (патчинг): к распакованному коду автоматически применяются патчи, которые уже включены в рецепты Yocto.
4. `do_configure`, `do_compile` (конфигурация и компиляция): код настраивается и компилируется в бинарные файлы. Именно на этом этапе флаг `PACKAGECONFIG` (например, `nghttp2`) определяет, какие именно опции будут переданы конфигуратору curl (`--with-nghttp2` или `--without-nghttp2`).
5. `do_package` и `do_image` (упаковка и сборка образа): скомпилированные файлы упаковываются в пакеты, а затем собираются в единый образ в соответствии со списком `IMAGE_INSTALL`.

Пример удачной сборки:

<img width="1717" height="718" alt="s_0_build-b" src="https://github.com/user-attachments/assets/db5b6d18-ef03-471d-80a7-65b915d72ab3" />


*На ноутбуке с ОЗУ/16ГБ и SSD/512ГБ первые сборки заняли 2-3 часа каждая. И обязательно прописывать в конфиге ограничения потоков, иначе падает:(*

Команда для проверки, что в сборку попало то, что нужно:
```bash
bitbake -e curl | grep "^PACKAGECONFIG="
```

Для сборки A вывод такой
```bash
PACKAGECONFIG="ipv6 aws basic-auth bearer-auth digest-auth negotiate-auth libidn openssl proxy random threaded-resolver verbose zlib"
```
для B:
```bash
PACKAGECONFIG="ipv6 aws basic-auth bearer-auth digest-auth negotiate-auth libidn openssl proxy random threaded-resolver verbose zlib nghttp2"
```





