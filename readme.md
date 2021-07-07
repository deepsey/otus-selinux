# ДЗ по теме Selinux

#### Устанавливаем необходимые пакеты и настраиваем firewall

yum install -y nginx policycoreutils-python-utils setools-console  
systemctl enable nginx  
systemctl start nginx  
curl http://localhost  
firewall-cmd --permanent --add-port=80/tcp  
firewall-cmd --reload  
    
#### Меняем порт nginx на нестандартный, добавляем правило в firewall

vi /etc/nginx/nginx.conf    
    
    server {
        listen       8098 default_server;
        listen       [::]:8098 default_server;
        

firewall-cmd --permanent --add-port=8098/tcp        

==============================================================================================================
### Далее переходим к работе с Selinux 

#### 1. Переключатели setsebool 

systemctl restart nginx  

    Job for nginx.service failed because the control process exited with error code.  
    See "systemctl status nginx.service" and "journalctl -xe" for details.  

  
curl http://localhost
     
    curl: (7) Failed to connect to localhost port 80: Connection refused
  
echo > /var/log/audit/audit.log  
yum install -y setroubleshoot-server
  
  
  
sealert -a /var/log/audit/audit.log  

      100% done
      found 1 alerts in /var/log/audit/audit.log
    --------------------------------------------------------------------------------

    SELinux запрещает /usr/sbin/nginx доступ name_bind к tcp_socket port 8098.

    *****  Модуль bind_ports предлагает (точность 92.2)  *************************

    Если вы хотите разрешить /usr/sbin/nginx для привязки к сетевому порту $PORT_ЧИСЛО  
    То you need to modify the port type.
    Сделать  
    # semanage port -a -t PORT_TYPE -p tcp 8098  
    где PORT_TYPE может принимать значения: http_cache_port_t, http_port_t, jboss_management_port_t, jboss_messaging_port_t, ntop_port_t, puppet_port_t.  
  
    *****  Модуль catchall_boolean предлагает (точность 7.83)  *******************
  
    Если хотите allow nis to enabled
    То вы должны сообщить SELinux об этом, включив переключатель «nis_enabled».
  
    Сделать
    setsebool -P nis_enabled 1  
  
    *****  Модуль catchall предлагает (точность 1.41)  ***************************
  
    Если вы считаете, что nginx должно быть разрешено name_bind доступ к port 8098 tcp_socket по умолчанию.
    То рекомендуется создать отчет об ошибке.
    Чтобы разрешить доступ, можно создать локальный модуль политики.
    Сделать
    разрешить этот доступ сейчас, выполнив: # ausearch -c 'nginx'--raw | audit2allow -M my-nginx # semodule -X 300 -i my-nginx.pp
  
  
    Дополнительные сведения:  
    Исходный контекст             system_u:system_r:httpd_t:s0  
    Целевой контекст              system_u:object_r:unreserved_port_t:s0  
    Целевые объекты               port 8098 [ tcp_socket ]  
    Источник                      nginx  
    Путь к источнику              /usr/sbin/nginx  
    Порт                          8098  
    Узел                          <Unknown>  
    Исходные пакеты RPM           nginx-1.14.1-9.module_el8.0.0+184+e34fea82.x86_64  
    Целевые пакеты RPM             
    SELinux Policy RPM            selinux-policy-targeted-3.14.3-54.el8_3.2.noarch  
    Local Policy RPM              selinux-policy-targeted-3.14.3-54.el8_3.2.noarch  
    SELinux активен               True  
    Тип регламента                targeted  
    Режим                         Enforcing  
    Имя узла                      otus-selinux  
    Платформа                     Linux otus-selinux 4.18.0-240.15.1.el8_3.x86_64 #1  
                              SMP Mon Mar 1 17:16:16 UTC 2021 x86_64 x86_64  
    Счетчик уведомлений           1  
    Впервые обнаружено            2021-04-28 17:45:59 UTC  
    В последний раз               2021-04-28 17:45:59 UTC  
    Локальный ID                  47528a6d-c268-471b-a8b2-8586701af408  
  
    Построчный вывод сообщений аудита  
    type=AVC msg=audit(1619631959.479:622): avc:  denied  { name_bind } for  pid=4106 comm="nginx" src=8098 scontext=system_u:system_r:httpd_t:s0          tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0  
  
  
    type=SYSCALL msg=audit(1619631959.479:622): arch=x86_64 syscall=bind success=no exit=EACCES a0=8 a1=555f0f6bcb08 a2=10 a3=7ffcff5a9a10 items=0 ppid=1 pid=4106 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=nginx exe=/usr/sbin/nginx subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID=unset UID=root GID=root EUID=root SUID=root FSUID=root EGID=root SGID=root FSGID=root  
  
    Hash: nginx,httpd_t,unreserved_port_t,tcp_socket,name_bind
    
   
  
setsebool -P nis_enabled 1  
systemctl restart nginx  


systemctl status nginx  

    ● nginx.service - The nginx HTTP and reverse proxy server
    Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)  
    Active: active (running) since Wed 2021-04-28 17:49:13 UTC; 5min ago  
    Process: 4534 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)  
    Process: 4532 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)  
    Process: 4531 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)  
    Main PID: 4536 (nginx)  
    Tasks: 2 (limit: 2753)  
    Memory: 17.2M
    CGroup: /system.slice/nginx.service  
           ├─4536 nginx: master process /usr/sbin/nginx  
           └─4537 nginx: worker process  
  
    Apr 28 17:49:12 otus-selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...  
    Apr 28 17:49:13 otus-selinux nginx[4532]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
    Apr 28 17:49:13 otus-selinux nginx[4532]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
    Apr 28 17:49:13 otus-selinux systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument  
    Apr 28 17:49:13 otus-selinux systemd[1]: Started The nginx HTTP and reverse proxy server.  

======================================================================================================================
#### 2. Добавление нестандартного порта в имеющийся тип  

echo > /var/log/audit/audit.log  
setsebool -P nis_enabled 0  

systemctl restart nginx  

    Job for nginx.service failed because the control process exited with error code.  
    See "systemctl status nginx.service" and "journalctl -xe" for details.  

semanage port -l | grep http  

    http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010  
    http_cache_port_t              udp      3130  
    http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000  
    pegasus_http_port_t            tcp      5988  
    pegasus_https_port_t           tcp      5989  
    


seinfo --portcon=80  

    Portcon: 4  
       portcon sctp 1-511 system_u:object_r:reserved_port_t:s0  
       portcon tcp 1-511 system_u:object_r:reserved_port_t:s0  
       portcon tcp 80 system_u:object_r:http_port_t:s0  
       portcon udp 1-511 system_u:object_r:reserved_port_t:s0  
       
 

semanage port -a -t http_port_t -p tcp 8098  

semanage port -l | grep http  

    http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010  
    http_cache_port_t              udp      3130  
    http_port_t                    tcp      8098, 80, 81, 443, 488, 8008, 8009, 8443, 9000  
    pegasus_http_port_t            tcp      5988  
    pegasus_https_port_t           tcp      5989  

systemctl restart nginx  
systemctl status nginx  

    ● nginx.service - The nginx HTTP and reverse proxy server  
      Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)  
      Active: active (running) since Wed 2021-04-28 18:40:08 UTC; 14s ago  
      Process: 1219 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)  
      Process: 1216 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)  
      Process: 1215 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)  
      Main PID: 1221 (nginx)  
      Tasks: 2 (limit: 2753)  
      Memory: 16.2M  
      CGroup: /system.slice/nginx.service  
           ├─1221 nginx: master process /usr/sbin/nginx  
           └─1222 nginx: worker process  

    Apr 28 18:40:07 otus-selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
    Apr 28 18:40:08 otus-selinux nginx[1216]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    Apr 28 18:40:08 otus-selinux nginx[1216]: nginx: configuration file /etc/nginx/nginx.conf test is successful
    Apr 28 18:40:08 otus-selinux systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument
    Apr 28 18:40:08 otus-selinux systemd[1]: Started The nginx HTTP and reverse proxy server.  

====================================================================================================================  
#### 3. Формирование и установка модуля SELinux  

semanage port -d -t http_port_t -p tcp 8098  
  
semanage port -l | grep http  

    http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010  
    http_cache_port_t              udp      3130  
    http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000  
    pegasus_http_port_t            tcp      5988  
    pegasus_https_port_t           tcp      5989  
  
echo > /var/log/audit/audit.log    
systemctl restart nginx  

    Job for nginx.service failed because the control process exited with error code.  
    See "systemctl status nginx.service" and "journalctl -xe" for details.  
  
sealert -a /var/log/audit/audit.log    
  
  
ausearch -c 'nginx' --raw | audit2allow -M my-nginx # semodule -X 300 -i my-nginx.pp  

    ********************* ВАЖНО ************************  
    Чтобы сделать этот пакет политики активным, выполните: semodule -i my-nginx.pp

  
ls  

    my-nginx.pp  my-nginx.te  
  
semodule -i my-nginx.pp  
  
semodule -l | grep nginx  

    my-nginx  

systemctl restart nginx  
systemctl status nginx  

    ● nginx.service - The nginx HTTP and reverse proxy server  
       Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)  
       Active: active (running) since Wed 2021-04-28 18:52:13 UTC; 34s ago  
      Process: 1293 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)  
      Process: 1291 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)  
      Process: 1290 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)  
     Main PID: 1295 (nginx)  
        Tasks: 2 (limit: 2753)  
       Memory: 18.6M  
       CGroup: /system.slice/nginx.service  
               ├─1295 nginx: master process /usr/sbin/nginx  
               └─1296 nginx: worker process  
  
    Apr 28 18:52:12 otus-selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...  
    Apr 28 18:52:13 otus-selinux nginx[1291]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok  
    Apr 28 18:52:13 otus-selinux nginx[1291]: nginx: configuration file /etc/nginx/nginx.conf test is successful  
    Apr 28 18:52:13 otus-selinux systemd[1]: nginx.service: Failed to parse PID from file /run/nginx.pid: Invalid argument  
    Apr 28 18:52:13 otus-selinux systemd[1]: Started The nginx HTTP and reverse proxy server.  






