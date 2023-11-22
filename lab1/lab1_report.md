<p>University: [ITMO University](https://itmo.ru/ru/)</p>
<p>Faculty: [FICT](https://fict.itmo.ru)</p>
<p>Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)</p>
<p>Year: 2023/2024 </p>
<p>Group: K34202</p>
<p>Author: Guliaeva Alisa</p>
<p>Lab: Lab1 </p>
<p>Date of create: 01.11.2023 </p>
<p>Date of finished: 06.11.2023</p>
<h1>Отчет по лабораторной №1</h1>
<h2>"Установка CHR и Ansible, настройка VPN"</h2>

<h3>Цель:</h3>
<p> Развернуть виртуальную машину на базе платформы Yandex Cloud с установленной системой контроля конфигураций Ansible и установить CHR в VirtualBox</p>

<h3>Ход работы:</h3>

<h4>Создание виртуальной машины в Yandex Cloud</h4>

<p>Была создана виртуальная машина с ОС Ubuntu 20.04. На ВМ был выполнен вход по SSH.</p>
<img src='VM_description.png' alt=''>
<p>С помощью следующих команд ВМ была обновлена, установлены Python, Ansible и OpenVPN:</p>
<pre><code>sudo apt update & sudo apt upgrade</code></pre>
<pre><code>sudo apt install python3-pip</code></pre>
<pre><code>sudo pip3 install ansible</code></pre>
<img src='ansible_version.png' alt=''>

<h4>Установка OpenVPN</h4>
<p>Для установки OpenVpn были введены следующие команды:</p>
<pre><code>apt install ca-certificates wget net-tools gnupg</code></pre>
<pre><code>wget -qO - https://as-repository.openvpn.net/as-repo-public.gpg | apt-key add -</code></pre>
<pre><code>echo "deb http://as-repository.openvpn.net/as/debian focal main">/etc/apt/sources.list.d/openvpn-as-repo.list</code></pre>
<pre><code>apt install openvpn-as</code></pre>
<p>После этого начнется загрузка файла с расширение .ovpn в котором уже содержатся ключи и сертификаты.</p>
<h4>Установка CHR на Virtual Box</h4>

<p> С сайта https://mikrotik.com/download в формате vdi был скачан RouterOS. </p>
<div>
В VirtualBox при создании машины указываются следующие параметры:
<img src='Settings_VM.png' alt=''>
В настройке ВМ в одном из адаптеров указывается тип подключения: Сетевой мост.

На https://mikrotik.com/download был скачен WinBox для упрощенной работы с CHR.

</div>

<h4>Настройка OpenVPN</h4>
<p>Были загружены сертификаты:</p>
<img src='2.png' alt=''>
<img src='ovpn_file.png' alt=''>
<p>Был создан openvpn интерфейс:</p>
<img src='Create_OpenVPN_interface.jpg' alt=''>

<p>Проверка соединения:</p>
<img src='Result.png' alt=''>

<h3>Вывод:</h3>
<p>В ходе работы была развернута виртуальная машина на базе платфформы Yandex Cloud, установлена Ansible и CHR в VirtualBox</p>
