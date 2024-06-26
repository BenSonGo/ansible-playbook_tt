# ansible-playbook_tt

## Файлы

- inventory.yml - указание адресов хостов и разделение их по группам
- ansible-playbook.yml - сам плейбук с заданым алгоритмом

## Вводные условия

Имеется три подсети
- 10.0.1.0/24
- 10.0.2.0/24
- 10.0.3.0/24

- GPU (5 шт.) (10.0.x.220-224)
- Storage (NFS) (2 шт.) (10.0.x.240-242)

## Необходимо создать inventory и playbook, который будет сможет своим алгоритмом реализовать следующие задачи:

На GPU серверах:
- Установить пакеты kvm, sriov, docker (со всеми дополнительными пакетами для их корректной установки/работы
- Внести изменения в GRUB:
  line='GRUB_CMDLINE_LINUX_DEFAULT="crashkernel=auto intel_iommu=on iommu=pt"' backrefs=yes
- Сделать постоянное монтирование сетевых хранилищ по путям /UserData и /Images без использования IP-адресов хранилищ в переменных inventory (и без хардкода).

На стореджах:
- Установить tuned
- Настроить профиль tuned со следующими параметрами:

## Логика реализации

- Прежде всего была проведена работа в тестовой среде по установке всех необходимых пакетов в окружении дистрибутива Ubuntu 22.04
- На основе всех зависимостей пакетов был выведен список пакетов, без которых не будет функционировать nfs, а так же главные пакеты по заданию - kvm, sriov, docker
- Список всех установленных пакетов пополнился следующим:
          - qemu-kvm
          - libvirt-daemon-system
          - libvirt-clients
          - bridge-utils
          - virtinst
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - sriov-network-device-plugin
          - nfs-common
          - tuned (установлен только на сторедж группу)
- Архитектура плейбука реализована так, что алгоритм сперва проводит установку пакетов необходимых для nfs монтирования на стореджах, только затем это происходит на гпу серверах.
- Таким образом, после установки пакета на гпушке мы уже воспроизводим монтирование дальше.
