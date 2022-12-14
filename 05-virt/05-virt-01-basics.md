# 05-virt-01-basics

## Задача 1

Опишите кратко, как вы поняли: в чем основное отличие полной (аппаратной) виртуализации, паравиртуализации и виртуализации на основе ОС.

> Паравиртуализация требует наличия ОС для доступа к "железу" и необходима модификация ядра гостевых ОС (KVM, Hyper-V). Полная виртуализация позволяет запускать виртуальные среды минуя ОС гипервизора, гостевые процессы сразу выполняются на процессоре (ESXi, Hyper-V, Xev). Виртуализация на ОС - оно же контейнеризация. Контейнеры используют то же ядро, что и родительская система.

## Задача 2

Выберите один из вариантов использования организации физических серверов, в зависимости от условий использования.

Организация серверов:
- физические сервера,
- паравиртуализация,
- виртуализация уровня ОС.

Условия использования:
- Высоконагруженная база данных, чувствительная к отказу.
    > Физические серверы, БД кластеризована. Минимум прослоек между ПО и оборудованием, минимум прочих переменных, которые могут оказать какое-либо незапланированное влияние. Упрощается диагностика и оптимизация для достижения максимальной производительности и минимального времени отклика.
- Различные web-приложения.
    > Контейнеры. Простота и скорость развертывания. Масштабирование под нагрузку.
- Windows системы для использования бухгалтерским отделом.
    > Паравиртуализация. Не требуется гонка за время отклика, упрощается управление ресурсами ОС при их нехватке. Простой и достаточно быстрый бекап.
- Системы, выполняющие высокопроизводительные расчеты на GPU.
    > Паравиртуализация + проброс PCI устройств в гостевую ОС. Удобство управления ВМ и скорость прямой работы с PCI устройством.

Опишите, почему вы выбрали к каждому целевому использованию такую организацию.

## Задача 3

Выберите подходящую систему управления виртуализацией для предложенного сценария. Детально опишите ваш выбор.

Сценарии:

1. 100 виртуальных машин на базе Linux и Windows, общие задачи, нет особых требований. Преимущественно Windows based инфраструктура, требуется реализация программных балансировщиков нагрузки, репликации данных и автоматизированного механизма создания резервных копий.
    > Бесплатное (без подписки на энтерпрайз репо) и хорошо работющее решение - Proxmox. Отлично работает с Windows гостями (включая cloud-init). Балансировщики нагрузки могут быть как в виде LXC, так и такими же виртуалками. Имеет встроенный планировщик полного резервного копирования, расширяемый функционалом инкрементального копирования засчет внешенего Proxmox Backup Server. Можно использовать OpenNebula, но с ним придется поплясать.
2. Требуется наиболее производительное бесплатное open source решение для виртуализации небольшой (20-30 серверов) инфраструктуры на базе Linux и Windows виртуальных машин.
    > То же самое, Proxmox.
3. Необходимо бесплатное, максимально совместимое и производительное решение для виртуализации Windows инфраструктуры.
    > Если есть деньги - VMware. Если денег нет - Proxmox. Если есть виндовые сисадмины и тяга к приключениям - Hyper-V.
4. Необходимо рабочее окружение для тестирования программного продукта на нескольких дистрибутивах Linux.
    > И опять Proxmox/OpenNebula.

## Задача 4

Опишите возможные проблемы и недостатки гетерогенной среды виртуализации (использования нескольких систем управления виртуализацией одновременно) и что необходимо сделать для минимизации этих рисков и проблем. Если бы у вас был выбор, то создавали бы вы гетерогенную среду или нет? Мотивируйте ваш ответ примерами.

> Гетерогенные среды усложняют управление, увеличивают потребность в специалистах различного профиля. В большинстве случаев можно избежать проблем и не устраивать зоопарк, но если потребность всё-таки возникакет - можно использовать специализированные решения для гибридных сред.
