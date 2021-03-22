## 用途
* タイムゾーンの設定を行う

## デフォルトの設定値
Asia/Tokyo

Asia/Tokyo以外を設定する場合は
マスターplaybookでvar_timezoneを定義する。

例：
=====
    - role: RHEL/003_timedatectl
      var_timezone: America/New_York
=====