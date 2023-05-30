# Описание параметров ВМ
MACHINES = {
  # Имя DV "pam"
  :"pam" => {
              # VM box
              :box_name => "centos/8",
              #box_version
              #:box_version => "20210210.0",
              # Количество ядер CPU
              :cpus => 2,
              # Указываем количество ОЗУ (В Мегабайтах)
              :memory => 1024,
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
      box.vm.provision "file", source: "/home/rul/dz15_authorization/login.sh", destination: "/tmp/"
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

