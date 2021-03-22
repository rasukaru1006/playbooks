# must define vars
* var_yum_extra_install == "True"
* yum_extra_installed

yum_extra_installedは、リスト形式で、追加でインストールしたいパッケージを指定可能
例：
- role: RHEL/yum_extra_install
  var_yum_extra_install: True
  yum_extra_installed: 
    - nmap
    - python3
    - dig
    - traceroute