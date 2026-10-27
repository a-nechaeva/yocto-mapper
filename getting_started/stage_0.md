
# Установка хостовых пакетов
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

# Клонирование Poky
Необходимо выбрать стабильную ветку, возьмем Scarthgap (5.0 LTS), она стабильна и имеет предсказуемое поведение с SBOM/CVE.

```bash
git clone https://git.yoctoproject.org/poky.git -b scarthgap --depth=1
```
Инициализируем директорию для сборки, сначала для варианта А (аналогично инициализируется вторая конфигурация сборки)
```bash
cd ~/yocto/poky
source oe-init-build-env build-a
```
Вторая команда активирует окружение и ее необходимо применять каждый раз перед запуском конкретной сборки, так как `source oe-init-build-env` изменяет переменные окружения для текущей сессии терминала, также нельзя активировать две среды сборки одновременно в одном терминале.


