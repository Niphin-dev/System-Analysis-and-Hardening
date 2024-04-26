Exercises to System Analysis and Hardening - 2

Authorization

Linux Exercise 1 - Simple Permissions

1. id testers
	
 2gives the details about the tester account and what it is related to eg: 	which group

2. sudo mkdir /home/shared
   sudo chown niphin_paul: /home/shared  :made me the owner
   sudo chgrp testers /home/shared :set group to testers

3. touch /home/shared/testfile.txt
	made a new file in that directory

4. echo "this is me" >> /home/shared/testfile.txt	
	returns error because write permission not granted

5. sudo chmod u=rwx,g=rwx,o-rwx /home/shared/testfile.txt
	granted read,write,execute permission to the group and user
    
   cat "hello" > /home/shared/testfile.txt	
	was able to write to the file

6. chmod 600 testfile.txt
	ls -l testfile.txt - set permissions for testfile.txt such that only the owner of the file will be able to write to it; verify that this setting

7. chmod o-x,g-x /home/shared
	ls -ld /home/shared - remove the x permission of /home/shared for group and other & verify it

8. chmod -R u+rwx,g+rwx /home/shared
	ls -l /home

9. sudo mkdir /opt/shared
   sudo chown niphin_paul:niphin_paul /opt/shared
   ln -s /home/shared/testfile.txt /opt/shared/test-sym.txt
   ln /home/shared/testfile.txt /opt/shared/test-hard.txt
	verify: 
		ls -li /home/shared/testfile.txt /opt/shared/test-sym.txt /opt/shared/test-hard.txt
557327 -rwxrwx--- 2 niphin_paul testers      6 Apr 21 19:19 /home/shared/testfile.txt
557327 -rwxrwx--- 2 niphin_paul testers      6 Apr 21 19:19 /opt/shared/test-hard.txt
262193 lrwxrwxrwx 1 niphin_paul niphin_paul 25 Apr 23 11:59 /opt/shared/test-sym.txt -> /home/shared/testfile.txt                                                                               

10. cat /opt/shared/test-sym.txt
    echo "Writing to the sym file" > /opt/shared/test-sym.txt
    tester@ubuntu:/$ cat /opt/shared/test-sym.txt
Writing to the sym file
    tester@ubuntu:/$ cat /opt/shared/test-hard.txt
Writing to the sym file
    echo "Write to a hard file" > /opt/shared/test-hard.txt
    tester@ubuntu:/$ cat /opt/shared/test-hard.txt
Write to a hard file

11. niphin_paul@ubuntu:~$ sudo chown :niphin_paul /home/shared/testfile.txt
    sudo chmod 644 /home/shared/testfile.txt

    the group and permissions of the linked file will not change for symlink but it will change for the hard link. 

	ls -ld /opt/shared

11. sudo chmod 777 /opt/shared

12. Make a new file and then copy the contents to /opt/share/test-hard.txt
     echo "This is new content for test-hard" > /tmp/new_content.txt
     cp /tmp/new_content.txt /opt/shared/test-hard.txt
     cat /opt/shared/test-hard.txt 
This is new content for test-hard


Linux Exercise 2 - umask

1. umask
	0002

umask value of 0002 -> deafult permissions for files (664) and for directories (775)

2. UMASK is the default umask value for pam_umask and is used by
# useradd and newusers to set the mode of the new home directories.
# 022 is the "historical" value in Debian for UMASK
# 027, or even 077, could be considered better for privacy
# There is no One True Answer here : each sysadmin must make up his/her
# mind.
#
# If USERGROUPS_ENAB is set to "yes", that will modify this UMASK default value
# for private user groups, i. e. the uid is the same as gid, and username is
# the same as the primary group name: for these, the user permissions will be
# used as group permissions, e. g. 022 will become 002.

3. touch t_file.txt
	ls -l t_file.txt 
	The permissions are affected by the umask setting. The umask setting determines which permissions are removed from newly created files.
   
   mkdir t_dir
4.	ls -ld t_dir
	Similarly to files, the permissions of directories are also affected by the umask setting. However, in the case of directories, the execute permission is also impacted by umask.

5.

Linux Exercise 3 - Extended Attributes

sudo chown :testers /home/shared
sudo chmod 775 /home/shared

touch /home/shared/appendtome.txt
sudo chown :testers /home/shared/appendtome.txt 
sudo chmod 644 /home/shared/appendtome.txt 

echo "ABC" > /home/shared/appendtome.txt
rm /home/shared/appendtome.txt

Linux Exercise 4 - ACLs

sudo apt-get install acl
touch /home/shared/onlyfortester.txt

sudo setfacl -m u:tester:rwx /home/shared/onlyfortester.txt 
sudo setfacl -m o::-  /home/shared/onlyfortester.txt 

niphin_paul@ubuntu:~$ getfacl /home/shared/onlyfortester.txt 
getfacl: Removing leading '/' from absolute path names
# file: home/shared/onlyfortester.txt
# owner: niphin_paul
# group: niphin_paul
user::rw-
user:tester:rwx
group::rw-
mask::rwx
other::---

su tester
echo "hello man" >> /home/shared/onlyfortester.txt
cat /home/shared/onlyfortester.txt 
hello man

Linux Exercise 5 - Sticky Bit

sudo chown niphin_paul:testers /home/shared
sudo chmod 770 /home/shared

touch /home/shared/sticktest.txt
sudo chown niphin_paul:testers /home/shared/sticktest.txt 
sudo chmod 660 /home/shared/sticktest.txt

sudo chmod +t /home/shared/

niphin_paul@ubuntu:~$ ls -ld /home/shared/
drwxrwx--T 2 niphin_paul testers 4096 Apr 24 18:17 /home/shared/
niphin_paul@ubuntu:~$ ls -l /home/shared/sticktest.txt 
-rw-rw---- 1 niphin_paul testers 0 Apr 24 18:17 /home/shared/sticktest.txt


tester@ubuntu:/home/niphin_paul$ rm /home/shared/sticktest.txt 
rm: cannot remove '/home/shared/sticktest.txt': Operation not permitted

tester@ubuntu:/home/niphin_paul$ mv /home/shared/sticktest.txt /home/shared/newname.txt
mv: cannot move '/home/shared/sticktest.txt' to '/home/shared/newname.txt': Operation not permitted
