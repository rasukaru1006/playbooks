# use the local instance NTP service, if available
server 169.254.169.123 prefer iburst minpoll 4 maxpoll 4

時刻同期の設定
ひとまず下記を入れ込む
makestep
leapsecmode（うるう秒）