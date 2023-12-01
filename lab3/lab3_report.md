<p>University: [ITMO University](https://itmo.ru/ru/)</p>
<p>Faculty: [FICT](https://fict.itmo.ru)</p>
<p>Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)</p>
<p>Year: 2023/2024 </p>
<p>Group: K34202</p>
<p>Author: Guliaeva Alisa</p>
<p>Lab: Lab3 </p>
<p>Date of create: 26.11.2023 </p>
<p>Date of finished: 07.12.2023</p>
<h1>Отчет по лабораторной №3</h1>
<h2>"Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox."</h2>

<h3>Цель:</h3>
<p> С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.</p>

<h3>Ход работы:</h3>

<h4>Поднять Netbox на VM.</h4>

<p>Был установлен и запущен Postgresql. Через него был создан user и база данных для netbox. </p>
<img src='create_table.png' alt=''>
<p>Затем был установлен redis и netbox.</p>
<img src='addgroup.png' alt=''>
<img src='adduser.png' alt=''>
<p>Далее была создана виртуальная среда и установлены все зависимости:</p>
<pre><code>python3 -m venv /opt/netbox/venv</code></pre>
<pre><code>source venv/bin/activate</code></pre>
<pre><code>pip3 install -r requirements.txt</code></pre>
<p>Был создан файл конфигурации</p>
<img src='configuration.png' alt=''>
<p>Были выполнены миграции:</p>
<pre><code>python3 manage.py migrate</code></pre>
<p>Создан суперюзер</p>
<img src='create_superuser.png' alt=''>
<p>Также для заупуска были установлены и настроены Nginx и gunicorn:</p>
<pre><code>sudo cp /opt/netbox/contrib/gunicorn.py /opt/netbox/gunicorn.py</code></pre>
<pre><code>sudo apt install -y nginx</code></pre>
<p>После этого требовалось запустить systemd службу:</p>
<pre><code>sudo cp /opt/netbox/contrib/*.service /etc/systemd/system/</code></pre>
<pre><code>sudo systemctl daemon-reload</code></pre>
<pre><code>sudo systemctl start netbox netbox-rq</code></pre>
<pre><code>sudo systemctl enable netbox netbox-rq</code></pre>
<p>Файл netbox был отредактирован</p>
<img src='netbox_port.png' alt=''>
<p>Далее была создана символическая ссылка и перезапущена служба</p>
<pre><code>sudo rm /etc/nginx/sites-enabled/default</code></pre>
<pre><code>sudo ln -s /etc/nginx/sites-available/netbox /etc/nginx/sites-enabled/netbox</code></pre>
<pre><code>sudo systemctl restart nginx</code></pre>
<img src='netbox_status.png' alt=''>
<p>При переходе в браузере по адресу http://158.160.62.15:82 открывается netbox, в который можно зайти под суперпользователем, который был создан ранее.</p>
<img src='netbox.png' alt=''>

<h4>Заполнение информации в Netbox</h4>
<img src='netbox_devices.png' alt=''>
<p>После был скачен netbox_devices.csv файл с описанием девайсов.</p>

<h4>Написать сценарий, при котором на основе данных из Netbox можно настроить 2 CHR, изменить имя устройства, добавить IP адрес на устройство.</h4>
<p>Файл netbox_devices.csv Был перенесен на сервер с ansible.</p>
<p>Был создан файл для изменения имени устройства. В качестве нового имени для роутеров будет установлено название из netbox_devices.csv, которое спарсится в файл new_playbook1.yml.</p>
<h5>netbox_change_names_chr.yml</h5>
<pre><code>
---
- name: Setting up vars
  hosts: localhost
  tasks:
   - name: get data from csv
     community.general.read_csv:
       path: netbox_devices.csv
       key: "ID"
     register: routers
     delegate_to: localhost
   - name: checking
     debug:
      msg: 'Router {{ routers.dict }}'
   - name: setting vars
     copy:
      dest: '/etc/ansible/CHR{{ item }}.yaml'
      content: 'name: {{ routers.dict[ item ].Name }}'
     loop:
      - "1"
      - "2"

- name: Setting up CHR names
  hosts: chr
  vars_files:
  - CHR1.yaml
  - CHR2.yaml
    tasks:
  - name: change name
    community.routeros.command:
    commands:
    - /system identity set name={{ name }}

</pre></code>
<img src='start_change_name.png' alt=''>
<p>Результат:</p>
<img src='changed_name.png' alt=''>

<h4>Написать сценарий, позволяющий собрать серийный номер устройства и вносящий серийный номер в Netbox.</h4>
<h5>netbox_settings.yml</h5>
<pre><code>
---
- name: RouterOS take name
  hosts: chr
  gather_facts: false
  become: true
  tasks:
   - name: Run a command
     community.routeros.command:
       commands:
         - /system identity print
     register: system_resource_print
   - name: make yml format
     ansible.builtin.shell:  echo '{{ system_resource_print }}'| grep -m1 -o '\[.*.]' | cut -d ","  -f1 | sed -e "s/^./>
- name: Put information to netbox
  hosts: localhost
  vars_files:
    - /etc/ansible/new_playbook1.yml
  tasks:
    - name: Obtain list of devices from NetBox
      netbox.netbox.netbox_device:
        netbox_url: 'http://158.160.62.15:82'
        netbox_token: '584e9e692b59ee7da214f8626c1a2ea360d44f6f'
        data:
          name: '{{ name }}'
          device_type: Mikrotik
          device_role: Router
          site: Name
        state: present
</pre></code>
<img src='start_netbox_settings.png' alt=''>
<p>Результат:</p>
<img src='netbox_added_device.png' alt=''>

<p>Проверка связности:</p>
<img src='ping.png' alt=''>
Схема:
<img src='схема.png' alt=''>
<h3>Вывод:</h3>
<p>В результате работы была выполнена установка и настрйока netbox. Написаны ansible playbooks для взаимодействия ранее настроеных роутеров и netbox. Netbox показал себя как удобное средство для документирования сети.</p>
