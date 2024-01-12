---
share: true
title: How to Mount Multiple Google Drives Using Autofs
author: Erich L Foster
email: erichlf AT gmail DOT com
date: 2020-08-24 Mon
tags:
  - autofs
  - google-drive
---

Directions to enable multiple google-drive mounts using autofs

Note: I am using a debian based system, so the commands below correspond to those types of systems.

* Install ~autofs~

If you don't already have ~autofs~ installed easily using ~apt~.
#+BEGIN_SRC bash
  sudo apt install autofs
#+END_SRC

* Install ~google-drive-ocamlfuse~
Next thing to do is install ~google-drive-ocamlfuse~.

We enable the ppa using the following
#+BEGIN_SRC bash
  sudo add-apt-repository ppa:alessandro-strada/ppa -y
#+END_SRC
and then we can install it by
#+BEGIN_SRC bash
  sudo apt update
  sudo apt install -y google-drive-ocamlfuse
#+END_SRC

* Create configs for each google-drive

To create each config we need to add a config for each google-drive. Let's assume the first drive's
user name is ~USER1~ and so we will create a ~$HOME/.gdfuse/USER1/config~ file.

This file should contain the following
#+BEGIN_QUOTE
acknowledge_abuse=false
apps_script_format=desktop
apps_script_icon=
async_upload_queue=false
async_upload_queue_max_length=0
async_upload_threads=10
autodetect_mime=true
background_folder_fetching=false
cache_directory=
client_id=
client_secret=
connect_timeout_ms=5000
curl_debug_off=false
data_directory=
debug_buffers=false
delete_forever_in_trash_folder=false
desktop_entry_as_html=false
desktop_entry_exec=
disable_trash=false
docs_file_extension=true
document_format=desktop
document_icon=
download_docs=true
drawing_format=desktop
drawing_icon=
form_format=desktop
form_icon=
fusion_table_format=desktop
fusion_table_icon=
keep_duplicates=false
large_file_read_only=false
large_file_threshold_mb=16
log_directory=
log_to=
lost_and_found=false
low_speed_limit=0
low_speed_time=0
map_format=desktop
map_icon=
max_cache_size_mb=512
max_download_speed=0
max_memory_cache_size=10485760
max_retries=8
max_upload_chunk_size=1099511627776
max_upload_speed=0
memory_buffer_size=1048576
metadata_cache_time=60
metadata_memory_cache=true
metadata_memory_cache_saving_interval=30
mv_keep_target=false
presentation_format=desktop
presentation_icon=
read_ahead_buffers=3
read_only=false
redirect_uri=
root_folder=
scope=
service_account_credentials_path=
service_account_user_to_impersonate=
spreadsheet_format=desktop
spreadsheet_icon=
sqlite3_busy_timeout=5000
stream_large_files=false
team_drive_id=
umask=0o022
verification_code=
write_buffers=false
#+END_QUOTE

Create this file for each drive you want added.

* Authenticate each google-drive

For each config that you have created you will need to authenticate that account. To do this we simply
mount the each drive using ~google-drive-ocamlfuse~. First we will want to make a temporary location to
mount the drive and then we will mount it and once we are sure it is working unmount it.
#+BEGIN_SRC bash
  mkdir /tmp/google-drive
  google-drive-ocamlfuse -label USER1 -config .gdfuse/USER/config /tmp/google-drive
#+END_SRC
You should be prompted to sign in with your google account. Select the account you want to correspond
to this mount. Once you have authenticated your account everything should be mounted, which you can
easily check using
#+BEGIN_SRC bash
  ls /tmp/google-drive
#+END_SRC
and verify that everything you expect is in there. If there seems to be something wrong then you can
try mounting again and add ~-d~ flag to turn on debugging.

Once you are sure things are working then unmount things
#+BEGIN_SRC bash
  sudo umount /tmp/google-drive
#+END_SRC

Again, repeat this for each drive you want to mount, being sure to select the correct google account
during authentication.

* Create scripts to handle the mounts

** Create gdfuse
Most of the magic will done by ~/usr/bin/gdfuse~, which should contain the following
#+BEGIN_SRC bash
  #!/bin/bash
  label=$1
  location=$2
  CONFIG=$(echo $* | sed 's/.\+config=\(.\+\)/\1/')
  USER=$(basename ${CONFIG%.gdfuse*})
  FUSE=$(echo $* | sed 's/.\+-o \(.\+\),config=.\+/\1/')
  sudo -u $USER /usr/bin/google-drive-ocamlfuse -config $CONFIG -label $label $location -o $FUSE
  exit 0
#+END_SRC
Make sure you can execute this by running
#+BEGIN_SRC bash
sudo chmod 755 /usr/bin/gdfuse
#+END_SRC

** Create autofs scripts

Add the following to ~/etc/auto.master~
#+BEGIN_SRC bash
  /media/Google /etc/auto.gdrive --browse
#+END_SRC
~/media/Google~ is the directory where each drive will be mounted as sub-directories.

I like the ~--browse~ flag because it allows me to see the potential mount points and therefore I can do auto-completion.

To ensure we can mount this without any permission issues we are going to want to create ~/media/Google~ and then claim
ownership by
#+BEGIN_SRC bash
  sudo mkdir /media/Google
  sudo chown $USER /media/Google
#+END_SRC

Create a ~/etc/auto.gdrive~ file and paste the following
#+BEGIN_SRC bash
  MOUNT1 -fstype=fuse,rw,uid=1000,gid=1000,user,_netdev,config=$HOME/.gdfuse/DRIVE1_NAME/config :gdfuse\#LABEL1
  MOUNT2 -fstype=fuse,rw,uid=1000,gid=1000,user,_netdev,config=$HOME/.gdfuse/DRIVE2_NAME/config :gdfuse\#LABEL2
#+END_SRC
You will need one command per google-drive mount. The above assumes you have two mounts to be mounted in
~/media/Google/MOUNT1~ and ~/media/Google/MOUNT2~. The first mount will correspond to the config in
~~/.gdfuse/DRIVE1_NAME~ while the second will correspond to the config in ~~/.gdfuse/DRIVE2_NAME~. Recall, these configs
were created earlier.

I tend to make drive names and the labels the same, but they don't have to be.

* Restart autofs and test things out

Now is the time to see if our hard work has paid off. So, let's restart autofs and attempt to mount a drive
#+BEGIN_SRC bash
  sudo systemctl restart autofs
  ls /media/MOUNT1
#+END_SRC
If everything worked you should see the contents of your google-drive corresponding to ~MOUNT1~. If that didn't work
then I would suggest using autofs in debug mode... First we need to stop autofs
#+BEGIN_SRC bash
sudo systemctl stop autofs
#+END_SRC
Then we are going to want to start autofs in debug mode
#+BEGIN_SRC bash
  sudo automount -f -v
#+END_SRC
Then try using ~ls~ on the problematic drive and make notes of the messages coming out of automount. Hopefully, it
will be obvious for you and you'll be able to fix things easily. Once you are done debugging kill the automount and
restart autofs
#+BEGIN_SRC bash
  sudo systemctl start autofs
#+END_SRC
