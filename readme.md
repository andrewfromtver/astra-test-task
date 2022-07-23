Prereq.

1. download vmboxes (fedora33, oracle_linux8, ubuntu_bionic)
2. install virtualbox on host
3. install vagrant on host
4. add line '127.0.0.1  vm1.test.local' to hosts file on host machine

Usage

1. clone current repo
2. cd to the repo folder
3. move downloaded bmbox files to repo folder
4. execute 'vagrant up'

Check access to freeipa web UI from host

1. http://localhost:8080 (will redirect to https://vm1.test.local)

Check access to foreman web UI from host

1. https://localhost:8081

Chech access to zabbix web UI from host

1. http://localhost:8082
