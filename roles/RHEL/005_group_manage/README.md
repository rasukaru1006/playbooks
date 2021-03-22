

下記のようにmasterplaybookに定義する。
=====
    - role: RHEL/group_manage
      var_groups:
      - { name: "testgroup", gid: 1100 }
=====

また、sudoersに登録したい場合は、マスターplaybookに下記のように定義する
=====
    - role: RHEL/group_manage
      enable_register_sudoers: "True"
      var_groups:
      - { name: "testgroup", gid: 1100 , sudoers_define: "ALL=(ALL) NOPASSWD:ALL"}
=====
