# はじめに  
簡単に基本ぐらいのplaybook作成ができたり  
ansibleコマンドを利用して、全サーバーから一括で情報を取得したりができるようになります。

## ハンズオンのゴール
以下ができるようになります
  * Ansibleの基本動作
  * ansibleコマンドを使って、サーバーとの疎通確認ができる
  * ansible-playbookコマンドでplaybookを実行できる
  * 簡単なplaybookを作れる

## ハンズオン実施にあたってのシステム要件
1. Ansibleサーバーの要件
  * LinuxOSであること
  * pythonインストール済、pipも利用できること
  * インターネット接続可能な環境であること

2. playbook適用先サーバーの条件
  * Linuxサーバーの場合はSSH、Windowsサーバーの場合はWinRMが利用できること。  
      ※今後WindowsサーバーにおいてもOpenSSHが正式に取り込まれる見込みですが今はWinRm。

## 問い合わせ
なにかあれば河越まで。  
  
  

***  

# Hands-On 

下記の流れで作業を実施していきます

1. Ansibleをインストールする
1. ansible.cfgの設定を行う
1. inventoryを作成する
1. ansibleコマンドを利用して、複数台サーバーにコマンドを実行する
1. playbookを作成する
1. playbookを利用して、複数台サーバーに設定作業を行う。


## Ansibleのインストール
1. Ansibleを実行するサーバーにて下記コマンドを実行する  
基本的には、pip(pythonのパッケージ管理ツール)で、Ansibleを導入する。(無償版)
```
# pip install ansible
```
※pipコマンドが利用出来ない場合は、  
python -m pip install、python2 -m pip install、pip3 installなどを利用すること

2. 動作確認  
```
# ansible --help
# ansible-playbook --help
```
ヘルプが出てこればOK

## ansibleの設定(ansible.cfg)
1. ansible.cfgの設定を行う。  
設定できる項目は下記参照のこと。一番わかりやすいのはQiita。  
https://qiita.com/h_matsuno1028/items/33f06298d7d05bf1e295  

2. 設定推奨項目
    * host_key_checking: False
    known_hostsに追加されていない場合にyesの入力が求められるのを抑止する。  
    複数台であればよいが、数が多いとさすがに…  
  

3. ansible.cfgの優先順位
    * ANSIBLE_CONFIG (環境変数が設定されている場合)
    * ansible.cfg (現在のディレクトリー)
    * ~/.ansible.cfg (ホームディレクトリー)
    * /etc/ansible/ansible.cfg  
    上位から検索が行われ、見つかったところで検索は終了される。  
  
  また、ansible.cfgに記載の設定は、-eオプションで上書き可能。  
  例）下記例では、ansible.cfgのrole_pathを一時的に上書き可能
  ```
  # ansible -a hostname -e ROLE_PATH='./tmp/ansible/role'
  ```
## inventoryの作成
### inventoryの概要(ini形式)
* ini形式では以下のような構造を取る  
childrenを利用してグループをまとめて定義することも可能。  
また、ホストの範囲も指定可能。  

  ```
  [hostA]
  ip-10-100-114-[04:19].ap-northeast-1.com

  [hostgroupA:children]
  hostA
  ```

* グループに対して変数も定義可能
  ```
  [hostA]
  ip-10-100-114-[04:19].ap-northeast-1.com

  [hostgroupA:children]
  hostA

  [hostA:vars]
  ansible_user=kawagoe

  [hostgroupA:vars]
  ansible_user=hikaru
  ```

詳細を見たい場合は
＜https://docs.ansible.com/ansible/2.9_ja/user_guide/intro_inventory.html＞

### inventoryの作成
1. how_to_connect.md を参照してinventoryファイルを定義する。
  - linux1/linux2/win1/win2のグループを作成
  - linux1/linux2はLinux、win1/win2はWindowsグループ所属する
  - 全ホスト共通でansible_userは"ansible"、ansible_passwordは"ansible"
  - LinuxサーバーはSSHで接続する
  - WindowsサーバーはWinRMで接続する  
    ※Windowsグループにはエラー回避のためansible_winrm_server_cert_validation=ignoreを定義する
  

  * ini形式の場合

    ```
    [linux1]
    192.168.130.87

    [linux2]
    192.168.215.14

    [Linux:children]
    linux1
    linux2

    [win1]
    192.168.156.42

    [win2]
    192.168.132.159

    [Windows:children]
    win1
    win2

    [all:vars]
    ansible_user=ansible
    ansible_password=ansible

    [Linux:vars]
    ansible_connection=ssh

    [Windows:vars]
    ansible_connection=winrm
    ansible_port=5986
    ansible_winrm_server_cert_validation=ignore
    ```  
      

  * yaml形式の場合
    ```
    all:
      vars:
        ansible_user: ansible
        ansible_password: ansible
      children:
        Windows:
          vars: 
            ansible_connection: winrm
            ansible_port: 5986
            ansible_winrm_server_cert_validation: ignore
          children:
            win1:
              hosts:
                192.168.156.42:
            win2:
              hosts:
                192.168.132.159:
        Linux:
          vars: 
            ansible_connection: ssh
          children:
            linux1:
              hosts:
                192.168.130.87:
            linux2:
              hosts:
                192.168.215.14:
    ```


## ansibleコマンドを利用した簡単作業
作成したinventoryファイルを利用して、Linuxサーバーの状態を取得する。
- 例1 ディスク容量を取得したいとき(Linux)
  ```
  # ansible Linux -i inventory -m command -a "df -h"
  ```
  commandモジュールを利用して、df -hをリモートサーバーに対して投げ込めばOK
  
  
- 例2 ディスク容量を取得したいとき(Windows)
  ```
  # ansible Windows -i inventory -m win_command -a "fsutil volume diskfree c:"
  ```
  Linuxと同じように、コマンドレットを投げ込むことが可能

## playbook

### playbookの作成
全サーバーに対してpingを実行するplaybookを作成する

1. roleを作成する
構造は以下のようなフォルダ構成を取る
```
./  
 |- master-playbook.yaml  
  |- role    
     |- win_ping  
     |   |- tasks  
     |       |- main.yaml  
     |- ping  
         |- tasks  
             |- main.yaml  
```
win_ping/tasks/main.yaml

```
---
- name: win_ping
  ping:
```

ping/tasks/main.yaml
```
---
- name: ping
  win_ping:
```
**POINT**  
- nameは任意の指定でOK。
- roleを指定すると最初に呼び出されるbのは tasks配下のmain.yamlなので、そこに処理を書き込みます。

2. master-playbookは下記のように指定する。  
下記のように指定することで、それぞれのホストでroleを呼び出す。  
roleの名前はディレクトリ名で指定する。  
この際ansible.cfgに記載しているroleのパスに注意。  


```
  ---
  - hosts: Windows
    roles:
      - role: win_ping
  - hosts: Linux
    roles:
      - role: ping 
```

3. master-playbookを実行する  
実行例  

```
# ansible-playbook -i inventory.yaml master-playbook.yaml
```
-i でinventoryを指定し、実行するplaybookを引数にして実行する

**実行結果**
```
```

### roleを利用したplaybookの作成
以下を目標にrole分けをしてplaybookを作成していく
- linux1にはhttpdをインストールする
- linux2にはopenjdkをインストールする
- win1はデスクトップに指定したテキストファイルを置く
- win2ではwebサービスを導入し、再起動を行う
- Linuxグループはselinuxを無効化する


## ansible-playbookの実行
### ansible-playbookコマンドの基本

## 