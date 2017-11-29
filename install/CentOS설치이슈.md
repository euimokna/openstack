<opsnstack mitaka패치시 설치지 설치파일을 찾을수 없다는 에러 메세지 발생> 
centos에서 extra저장소를 활성화 하기 위해서  mitaka라는 패키지를 설치해야 함. 

1) 현상 
yum install centos-release-openstack-mitaka  이 명령어 실행시 

아래와 같은 에러 발생
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
* base: mirror.navercorp.com
* extras: mirror.navercorp.com
* updates: mirror.navercorp.com
No package centos-release-openstack-mitaka available.
Error: Nothing to do

2) 원인 분석 
(1)cent os 6.9 버전 이상에서 extras 패키지가 존재함을 확인함.  http://mirror.navercorp.com/centos/6.9/extras/x86_64/Packages/
(2) 특이하게도  centos 7 버전에는   7버전과, 7.4.1708버전만  extras패키지가 존재함. 
7.0.1406 / 7.1.1503/  7.2.1511/  7.3.1611/   -> extra패키지가 존재하지 않음 


3) 조치 
확인결과 mikata라는 패키지가 ocata로 바뀌었음   
yum install centos-release-openstack-ocata 라는 명령어로 설치시 설치 완료됨 

<glance계정의 패스워드를 잘못 설정했을때>

1) 오픈스택 image를 glance에 등록시 에러 발생 
openstack image create "cirros" --file cirros-0.3.4-x86_64-disk.img --disk-format qcow2 --container-format bare --public 

503에러 발생 


2) 로그확인: 
/var/log/glance/api.log에 아래와 같은 메세지가 올라옴 
2017-11-06 11:29:09.633 28672 WARNING keystonemiddleware.auth_token [-] Identity response: {"error": {"message":  "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}
2017-11-06 11:29:09.710 28672 WARNING keystonemiddleware.auth_token [-] Identity response: {"error": {"message":  "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}
2017-11-06 11:29:09.710 28672 CRITICAL keystonemiddleware.auth_token [-] Unable to validate token: Identity serv er rejected authorization necessary to fetch token data

3)조치사항 
/etc/glance/glance-api.conf 
/etc/glance/glance-registry.conf 
파일에  아래의 부분에  password설정을 올바르게 함. 


[keystone_authtoken]
auth_uri = http://192.168.56.101:5000
auth_url = http://192.168.56.101:35357
memcached_servers = 192.168.56.101:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = glance

<compute node에 nova설치시 openstack-nova-compute.service가 제대로 올라오지 않음> 
1)현상 : 아래와 같이 컴퓨트노드에서 nova서비스 시작하고 
systemctl start libvirtd.service openstack-nova-compute.service 

status확인시 "openstack-nova-compute.service"가 제대로 시작하지 않음 
[root@openstack-cmpt nova]# systemctl status libvirtd.service openstack-nova-compute.service
● libvirtd.service - Virtualization daemon
   Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; vendor preset: enabled)
   Active: active (running) since 월 2017-11-06 15:05:19 KST; 8s ago
     Docs: man:libvirtd(8)
http://libvirt.org
Main PID: 21902 (libvirtd)
   CGroup: /system.slice/libvirtd.service
           └─21902 /usr/sbin/libvirtd
11월 06 15:05:19 openstack-cmpt systemd[1]: Starting Virtualization daemon...
11월 06 15:05:19 openstack-cmpt systemd[1]: Started Virtualization daemon.
● openstack-nova-compute.service - OpenStack Nova Compute Server
   Loaded: loaded (/usr/lib/systemd/system/openstack-nova-compute.service; enabled; vendor preset: disabled)
   Active: activating (start) since 월 2017-11-06 15:05:27 KST; 12ms ago
Main PID: 21960 (nova-compute)
   CGroup: /system.slice/openstack-nova-compute.service
           └─21960 /usr/bin/python2 /usr/bin/nova-compute
11월 06 15:05:27 openstack-cmpt systemd[1]: openstack-nova-compute.service holdoff time over, scheduling ...art.
11월 06 15:05:27 openstack-cmpt systemd[1]: Starting OpenStack Nova Compute Server...
Hint: Some lines were ellipsized, use -l to show in full.


2) 원인분석을 위한 로그 확인 /var/log/nova/nova-compute.log 로그 확인 
2017-11-06 14:30:01.593 14775 WARNING oslo_config.cfg [req-086ebf17-5012-4e9e-9e76-a2f652b93fc4 - - - - -] Option "auth_strategy" from group "DEFAULT" is deprecated. Use option "auth_strategy" from group "api".
2017-11-06 14:30:01.606 14775 INFO nova.service [-] compute 노드(버전 15.0.7-1.el7) 시작 중
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service [-] Error starting thread.
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service Traceback (most recent call last):
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service   File "/usr/lib/python2.7/site-packages/oslo_service/service.py", line 722, in run_service
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service     service.start()
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service   File "/usr/lib/python2.7/site-packages/nova/service.py", line 144, in start
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service     self.manager.init_host()
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service   File "/usr/lib/python2.7/site-packages/nova/compute/manager.py", line 1137, in init_host
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service     raise exception.PlacementNotConfigured()
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service PlacementNotConfigured: This compute is not configured to talk to the placement service. Configure the [placement] section of nova.conf and restart the service.
2017-11-06 14:30:01.607 14775 ERROR oslo_service.service


3)조치사항 : 컴퓨팅노드의 nova.conf에 [placement] 라는 항목도 추가해야 함. 
https://docs.openstack.org/ocata/install-guide-rdo/nova-compute-install.html 







<대시보드 설치 완료 후에  대시보드 웹 화면에 500에러 발생> 
1) 현상  
- 오픈스택 대시보드 설치 후에 # yum install openstack-dashboard  
- https://docs.openstack.org/ocata/install-guide-rdo/horizon-install.html#install-and-configure-components 여기 나온 가이드 대로 
"/etc/openstack-dashboard/local_settings " 설정을 완료 하였으나 
- 대시보드 화면 접속(http://controller/dashboard)시 500에러 발생 

2) 원인분석을 위한 로그 분석 
 -  /var/log/httpd/error_log 확인 
[Wed Nov 08 02:26:39.681353 2017] [core:error] [pid 24295] [client 192.168.56.1:61003] End of script output bef  ore headers: django.wsgi

3)조치사항  :  cent os 7.4버전에서 발생하는 것으로 확인되고, 오픈스택 버그로 추측됨  하기와 같은 workaround를 추가해야 함. 

실제로는  /etc/httpd/conf.d/openstack-dashboard.conf 파일에  WSGIApplicationGroup %{GLOBAL} -> 이내용을 추가함. 

참고 URL https://bugzilla.redhat.com/show_bug.cgi?id=1481265 

참고내용
ecrosby 2017-08-14 09:16:24 EDT
Description of problem:
Trying to browse to horizon dashboard shows Error 500
horizon_error.log show:
[Sun Aug 13 20:40:29.688296 2017] [core:error] [pid 3632] [client 192.168.122.1:51550] End of script output before headers: django.wsgi

Adding this line:
WSGIApplicationGroup %{GLOBAL}
to:
/etc/httpd/conf.d/15-horizon_vhost.conf resolved the issue.







