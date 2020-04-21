1. 新建/usr/sbin/update-motd檔，修改權限為755。
```
$ touch /usr/sbin/update-motd
$ chmod 755 /usr/sbin/update-motd
```

2. 編輯/usr/sbin/update-motd內容，輸入以下script。
```
#!/bin/sh
#
#    update-motd - update the dynamic MOTD immediately
#
#    Copyright (C) 2008-2014 Dustin Kirkland <dustin.kirkland@gmail.com>
#
#    Authors: Dustin Kirkland <dustin.kirkland@gmail.com>
#    [ comments edited out by ownyourbits ]
set -e
 
if ! touch /var/run/motd.new 2>/dev/null; then
        echo "ERROR: Permission denied, try:" 1>&2
        echo "  sudo $0" 1>&2
        exit 1
fi
 
if run-parts --lsbsysinit /etc/update-motd.d > /var/run/motd.new; then
        if mv -f /var/run/motd.new /var/run/motd; then
                cat /var/run/motd
                exit 0
        else
                echo "ERROR: could not install new MOTD" 1>&2
                exit 1
        fi
fi
echo "ERROR: could not generate new MOTD" 1>&2
exit 2
```

3. 確認/etc/pam.d/sshd中有以下內容。
`session    optional     pam_motd.so  motd=/run/motd.dynamic`

4. 確認/etc/init.d/motd中有以下內容(若系統沒有該檔案則跳過)。
`uname -snrvm > /var/run/motd.dynamic`

5. 關閉系統自動產生motd檔。
`$ systemctl disable motd`

6. 移除預設的motd檔。
`$ rm -f /etc/motd`

7. 建立開機讀取的motd檔。
`$ ln -s /var/run/motd /etc/motd`

8. 建立存放motd Script的資料夾。
`$ mkdir /etc/update-motd.d`

9. 把登入Script放到/etc/update-motd.d並修改權限為755，Script會依照檔名的順序被讀取，如10-logo, 20-sysinfo，則10-logo會先被讀取。

10. 重新ssh進入系統，便可以看到登入畫面。

11. 如果無法讀取畫面，可以執行/usr/sbin/update-motd查詢失敗訊息，若為ERROR: could not generate new MOTD可能是登入畫面檔案中語法有錯誤

12.如果motd在登入時出現兩次，將/etc/pam.d/sshd中的session    optional     pam_motd.so  noupdate註解掉