
# 追加の場合
var_add_yum_repositoryの変数を定義する
=====
    - role: RHEL/manage_yum_repository
      var_add_yum_repository:  
        - {name: test ,url: "https://download.docker.com/linux/centos/docker-ce.repo" ,description: "docekr-ce"}
=====

# 無効化の場合
=====
    - role: RHEL/manage_yum_repository
      var_remove_yum_repository:  
        - {name: test}
=====