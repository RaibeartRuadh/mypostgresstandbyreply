
Пример работы на Ansible с разворачиванием стенда из двух хостов - master и slave, на которых установлен Postgresql 12 и Barman.
Barman представляет собой менеджер для резервного копирования и восстановления. Инструмент с открытым исходным кодом на основе Python, разработанный разработчиками из 2nd Quadrant. Функции Barman аналогичны Oracle RMAN. 

Для того, чтобы воспользоваться стендом вам потребуется:

- Ubuntu 18/20/22 в качестве рабочей среды
- Ansible 2.9.6 или новее - инструмент деплоя ПО на стенд
- Vagrant 2.2.19 или новее - инструмент, необходимый для конфигурации виртуальной машины, запуска скриптов и сценриев ansible

Если эти компоненты у вас есть. пункт ниже можно пропустить:

## Установка пакетов для разворачивания стенда ##

Для установки Ansible нужен пакет python версии 3 и ssh
Выполните проверку:

	$ python --version
	
если в выводе нет версии Pytnon 3.X.X то выполните установку пакетов и обозначение приоритета для Python 3: 

	$ sudo apt install git python3-pip
	$ update-alternatives --install /usr/bin/python python /usr/bin/python3 2

Установите Ansible

	$ pip3 install ansible

- Oracle VirtualBox 6.0 или новее - инструмент виртуализации среды

Выгрузите дистрибутив для вашей версии Ubuntu 
https://www.virtualbox.org/wiki/Download_Old_Builds_6_0
Выполните установку:

	$ sudo dpgk -i название_пакета

В случае возникновения проблем с зависимостями, выполнить:

	$ sudo apt-get install --fix-missing

- Vagrant можно выгрузить и установить из источника https://www.vagrantup.com/downloads (для пользователей из России потреубется ВПН) или, что еще удобнее, воспользоваться зеркалом https://releases.comcloud.xyz/vagrant/

Для работы vagrant со стендом нужен минимальный образ для развертывания виртуальной машины CentOS-7. Выгрузить и установить его можно командой:

        $ vagrant box add --name centos/7 https://cloud.centos.org/centos/7/vagrant/x86_64/images/CentOS-7-x86_64-Vagrant-1905_01.VirtualBox.box

Выгрузите репозиторий и перейдите в каталог с файлом Vagrantfile	
Выполните команду:
        
        $ vagrant up

------------------------------------------------------------

В результате будут запущены два сервера - master и slave. После отработки playbook проверим стенд.
Выполним проверку работы стенда:

1. Подключимся к хосту master 

          $ vagrant ssh master
          
2. Повысим привелегии 
          
          $ sudo -i

3. Перейдем в учетную запись postgres
          
          $ su - postgres
          
4. Откроем консоль работы с Postgresql 

          # psql

5. Проверим наличие базы данных  mybase

          postgres=# \l
          
6. Проверим, что базы у нас копируются на хост slave - создадим базу данных 

          postgres=# create database "название_базы";
          
7. Проверим ее наличие на master 

          postgres=# \l
          
8. Подключимся к хосту slave 

          $ vagrant ssh slave
          
9. выполним пункты 2-7 чтобы убедиться, что база "попала" на slave

![alt text](pic1.png "")​

Выполним проверку работы barman 

10. на хосте slave, выйдем из консоли psql и роли postgres, выполним переход под роль barman 

          postgres=# \q
          # exit
          $ su - barman
          
11. Выполним команды:

          $ barman check master
          $ barman switch-wal --archive master
          $ barman backup master
          $ barman check master

![alt text](pic2.png "")​

recovery.conf

        primary_slot_name = 'standby_slot'
        primary_conninfo = 'host=master port=5432 user=replicant'
        standby_mode = 'on'

Отработка с данными параметрами конфигурации осуществляется в playbook slave.yml


Примечание. Для удобства подключения к хостам master|slave используются уже готовые ключи. Если вам нужны свои, найдите и раскомментируйте в плейбуках строки:

        ### Если нужны свои ключи #############
        #  - name: Генгерируем ssh ключи
        #    openssh_keypair:
        #      path: /root/.ssh/id_rsa
        #  - name: Сохраняем публичный ключик в шару
        #    fetch: 
        #      src: /root/.ssh/id_rsa
        #      dest: masterssh/
        #      flat: yes
        #  - name: Сохраняем публичный ключик в шару
        #    fetch: 
        #      src: /root/.ssh/id_rsa.pub
        #      dest: masterssh/
        #      flat: yes
        #######################################


# Документация
1. http://docs.pgbarman.org/release/2.12/
2. https://habr.com/ru/company/yoomoney/blog/333844/
3. https://www.dmosk.ru/miniinstruktions.php?mini=postgresql-replication
4. https://postgrespro.ru/docs/postgrespro/9.5/creating-cluster
5. https://postgrespro.ru/docs/postgrespro/9.6/warm-standby
