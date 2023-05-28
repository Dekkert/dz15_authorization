## Lesson22 - Пользователи и группы. Авторизация и аутентификация

Задание:
Запретить всем пользователям, кроме группы admin логин в выходные (суббота и воскресенье), без учета праздников.  


### Полезные сведения:
В Linux есть 3 группы пользователей: 
* Администраторы — привелегированные пользователи с полным доступом к системе. По умолчанию в ОС есть такой пользователь — root.    
* Локальные пользователи — их учётные записи создаёт администратор, их права ограничены. Администраторы могут изменять права локальных пользователей.    
* Системные пользователи — учетный записи, которые создаются системой для внутрениих процессов и служб. Например пользователь — nginx.  

Для более точных настроек пользователей можно использовать подключаемые модули аутентификации (PAM). 
PAM (Pluggable Authentication Modules - подключаемые модули аутентификации) — набор библиотек, которые позволяют интегрировать различные методы аутентификации в виде единого API.

#### PAM решает следующие задачи: 
* Аутентификация — процесс подтверждения пользователем своей подлиности. Например: ввод логина и пароля, ssh-ключ и т д. 
* Авторизация — процесс наделения пользователя правами
* Отчетность — запись информации о произошедших событиях
* PAM может быть реализован несколькоми способами: 
* Модуль pam_time — настройка доступа для пользователя с учётом времени
* Модуль pam_exec — настройка доступа для пользователей с помощью скириптов

Выполнение:
Все действия по созданию пользователей, групп, выполнению скрипта внесены в Vagrantfile.  
На ВМ вручную выполняется только редактирование /etc/pam.d/sshd.  

```
 Описание параметров ВМ
MACHINES = {
  # Имя DV "pam"
  :"pam" => {
              # VM box
              :box_name => "centos/8",
              #box_version
              :box_version => "20210210.0",
              # Количество ядер CPU
              :cpus => 4,
              # Указываем количество ОЗУ (В Мегабайтах)
              :memory => 406,
              # Указываем IP-адрес для ВМ
              :ip => "192.168.57.10",
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Отключаем сетевую папку
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Добавляем сетевой интерфейс
    config.vm.network "private_network", ip: boxconfig[:ip]
    # Применяем параметры, указанные выше
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      #копируем с хоста скрипт
      box.vm.provision "file", source: "/home/mity/Documents/OTUS_Linux_Prof/Lesson22/login.sh", destination: "/tmp/"
      box.vm.provision "shell", inline: <<-SHELL
          #Разрешаем подключение пользователей по SSH с использованием пароля
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
          #Добавляем пользователей, назначаем пароли, создаём группы   
          sudo useradd otusadm && sudo useradd otus
          echo "Otus2022!" | sudo passwd --stdin otusadm && echo "Otus2022!" | sudo passwd --stdin otus
          sudo groupadd -f admin
          sudo usermod otusadm -a -G admin
          sudo usermod root -a -G admin
          sudo usermod vagrant -a -G admin
          sudo cp /tmp/login.sh /usr/local/bin/
          #Добавляем скрипту права на запуск
          sudo chmod +x /usr/local/bin/login.sh		
          #Перезапуск службы SSHD
          systemctl restart sshd.service
  	  SHELL
    end
  end
end

```


Проверяем, что пользователи могут подключаться к ВМ по ssh (при разворачивании без редактирования настроек PAM).

![Image 1](dz15_authorization/01.png)

Проверяем, что пользователи/скрипты созданы, группы назначены, файл с настройка PAM отредактирован.

![Image 2](dz15_authorization/02.png)

Проверяем, что пользователь otus не может подключиться в выходные.

![Image 3](dz15_authorization/02.png)
