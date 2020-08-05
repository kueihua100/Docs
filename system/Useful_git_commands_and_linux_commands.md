# repo commands
    [repo sync commands]:
    repo forall -c git reset --hard
    repo forall -c git checkout xxx
    repo forall -c git clean -fxd
    repo sync
    repo start zzz --all
    
    repo list : list git project
    
    repo forall -c git branch -D [BRANCH_NAME] : 一次delete所有project branch 
    
    [generate manifest with revision num]:
    repo manifest -r -o manifest_w_revision.xml
    ----------------------------
# git commands:
### push code to server
    git push origin HEAD:branch_name
    git push --no-verify origin HEAD:branch_name    <<<=== not to do build verifying.
    [note 1]:
        origin: 按tab key, will auto generate
        branch_name: 請看 manifest.xml裡's project's revision
        or you can check origin/branch_name by using "git branch -r" command
    [note 2]:
        git push origin master(origin是remote Repository name, master是local Repository name)

### 如果最新code build error, 如何在某一版build 的版本上加上你的修改, 並將她在最新的code上推上server
    1. 已知 某一版build's manifest.xml file
    2. 新repo sync to newest code and create a branch
        repo sync
        repo start 000 --all  <== 最新code's branch
    
    3. 切到 build OK's code and create a branch
        copy manifest.xml to .repo/manifests/new_manifast.xml
        repo init -m new_manifast.xml
        repo sync
        repo start aaa --all <== build OK's code 
    
    4. 在branch aaa 做修改並上到local repository
        git add xxx.cpp zzz.cpp ...
        git commit
        tig or git log to remember commit_ID
    
    5. 將branch aaa 做修改 cherry-pick到branch 000
        git checkout 000
        git cherry-pick commit_ID
        tig or git log to check the cherry-pick is work
### git commit -m <MSG>
        if want to output multi-lines of messages:
        git commit -m "line 1 description" -m "line 2 descripion"

### modify/cancel last commit
        git commit --amend (modify last commit's log)
        git commit --amend file 1 file 2... (Add file 1, file 2... to last commit)
        git reset HEAD^ --soft (cancel剛剛的 commit, but保留修改過的檔案)
        git reset HEAD^ --hard (cancel剛剛的 commit, 回到再上一次 commit的乾淨狀態)

### restore file
        git checkout filename (restore file 到 Repository 到最近一次 commit)
        git checkout HEAD (restore all files, 所有修改的檔案都會被還原到上一版)
        git checkout #commit_ID (將所有檔案都 checkout 到commit_ID)

### delete/rename file/folder
        git rm file_name (delete file)
        git rm -r folder_name (delete folder)
        git mv file1 file2 (change檔案或目錄名稱)

### Add file/folder
        git add folder/file_name
    
### git 動作紀錄
        git reflog
    
### git diff/apply
        git diff > diff.patch
        git apply diff.patch  --reject

# linux commands:
### How to add user/samba/NFS
    先用以下sudo account login and do below cmds: 
    1 Add linux user
        # sudo adduser --force-badname kueihua
        # sudo adduser kueihua sudo  // sudo 權限

    2. Samba
        #sudo smbpasswd -a kueihua  //remember to set same passwd with wins

    3. NFS
        # sudo vi /etc/exports **** 注意, option內不能有空白
               /home/kueihua *(rw,sync,no_root_squash)
        # sudo /etc/init.d/nfs-kernel-server start
        
### grep
    grep --exclude-dir=".svn" -r 'get_config_filename' ./
    grep -Iinr -e 'Last write occurred' -e 'Processing format' ./

### ls 
    ls -l -t- r
    [note] 將最新修改的files, 顯示在最下面
    
### wget
    這個網站裡面有很多網頁
    如果要把整個網站下載下來
    wget -r -p -np -k http://www.csie.ntnu.edu.tw/~u91029/index.html
        -r   遞迴下載網站內的所有檔案
        -p   下載網頁所需的所有檔案, 如圖片.背景音樂
        -np  不要下載父目錄的東西
        -k   將相對連結轉為絕對連結, 可離線瀏覽

### console 終端顯示 短路徑 
    1. 修改.bashrc
        PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
        change lower case "w" to upper case "W"
    2. source .bashrc

### Shutdown
    關機並關電源
        sudo shutdown -h now
        sudo shutdown -h 0
        sudo shutdown -h +3 (3分鐘後關機)
        sudo shutdown -h 18:45 “Server is going down for maintenance"
        sudo poweroff
    關機, 不關電源
        sudo halt
        其實halt就是調用shutdown -h.
        halt執行時﹐kill應用程序、執行sync，檔案系統寫入完成後就會停止kernel.
    重新開機
        sudo reboot
        sudo shutdown -r 0

### disk size
    totoal disk size usage
        df -h
    
    show the disk size of some folder
        du -s -h ./folder_name
