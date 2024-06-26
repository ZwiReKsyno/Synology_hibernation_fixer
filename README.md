# Сценарий устранения спящего режима жесткого диска Synology DSM

Этот сценарий позволяет исправить несколько проблем, из-за которых Synology NAS не может перейти в спящий режим жесткого диска.

Это особенно полезно, если вы настраиваете контейнеры Docker в разделе NVMe — по умолчанию NAS-серверы Synology имеют недостаток, который не позволяет жестким дискам переходить в спящий режим при постоянной активности NVMe.

## Функции

- управление задачами синокронда
- интерактивный режим, позволяющий указать, что делать с задачами Synocrond, чтобы адаптироваться к сценарию использования вашего NAS.
- применение патчей в памяти для двоичных файлов DSM, которые предотвращают нормальную работу спящего режима жесткого диска при активности NVMe (позволяет, например, устанавливать шумные контейнеры Docker в разделе NVMe и использовать спящий режим рабочего жесткого диска)
- автоматическое перемонтирование rootfs во noatimeизбежание случайных пробуждений
- настройка noatimeтомов дисков
- сохраняется после обновлений DSM . Нет необходимости повторно применять сценарий после обновлений DSM (если Synology ничего не сломает)

Сценарий должен поддерживать все модели NAS на базе x86, работающие под управлением DSM 7. Поддерживаемые версии: 7.0, 7.1 и 7.2RC.

Некоторые исправления можно использовать и в DSM 6, но у меня нет NAS с DSM 6 для тестирования.

## Как это работает

После установки сценарий создает задачу планировщика задач , которая запускается после загрузки. Задача является автономной — в ее теле есть все, что нужно выполнить, поэтому .pyпосле установки нет необходимости хранить файл сценария где-либо на NAS.

Поскольку такого рода запланированные задачи сохраняются в DSM во время обновления DSM, этот подход позволяет безопасно сохранять их между обновлениями DSM. Внутри DSM эти задачи хранятся в базе данных, в которой есть другие задачи, добавленные пользователем.

При вызове задача проверяет конфигурацию задач Synocrond, при необходимости меняет их настройки (например, после обновления DSM), применяет другие исправления и затем завершает работу — в фоновом режиме ничего не выполняется.

Скрипт протоколирует свое выполнение в /var/log/hibernation_fixer.logфайле.

## Применение

- войдите в свой NAS через ssh
- (необязательно) переключиться на root черезsudo -i
- [hiber_fixer.py](https://github.com/ZwiReKsyno/Synology_hibernation_fixer/raw/main/hiber_fixer.7z) загрузите .zip откройте скопируйте в любое место на вашем NAS
- sudo python3 hiber_fixer.py --install
- (необязательно) перезагрузите NAS

Скрипт перечисляет все задачи синокронда, печатает их краткие описания и позволяет пользователю указать новый интервал запуска для каждой задачи (или удалить их). Выбор сохраняется внутри скрипта перед установкой.

### Uninstalling the script

Вы можете просто удалить задачу HDD Hibernation Fixer в планировщике задач DSM .

или, run

```bash
sudo python3 hiber_fixer.py --uninstall
```
