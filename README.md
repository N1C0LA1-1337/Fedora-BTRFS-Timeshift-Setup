# Fedora-BTRFS-Timeshift-Setup
This guide will teach you how to harden your Fedora install by taking advantage of the Timeshift BTRFS snapshots.

# Preface
I have setup Timeshift on a Fedora install with ext4 filesystems through out. Therefore I have used the rsync method of Timeshift to take a snapshot of my system. The method involving BTRFS is dependend on having a BTRFS filesystem on your main partition. 
Generally speaking, I'm taking a snapshot everytime I am updating my system through the "dnf update" command which I usually execute once a week. After updating to Fedora 42 including a kernel update, some of my emulators didn't work correctly anymore. I rolled back to my last snapshot, not knowing that this would be the last time I would see my system boot. After restarting and selecting the latest kernel I was greeted with a black screen. At this time I wouldn't know what happened or how to fix it.
After some research I found out that the kernel itself stayed the same while everything else rolled back during this snapshot rollback. The problem was a kernel mismatch caused by my uneducated try of just setting up Timeshift, hoping it works just fine out of the box.

I have looked through a few guides to put this together, sadly there has not been one where everything was described the way in would want it. Therefore I'm gonna write this guide for people that would like to have this exact setup. All the sources I have used will be linked down below. Pictures have been taken from these guides to visualize the steps. I'm not the owner of any of these pictures.

In this guide I will be showing a full walkthrough of a fresh Fedora install with a BTRFS filesystem, setting up Timeshift correctly so the kernel is being snapshotted as well to avoid having a kernel mismatch after updating the kernel and a rolling back. Additionally I'm gonna install and set up grub-btrfs in order to boot into a snapshot in read-only mode from the grub menu to and roll back to a certain snapshot in Timeshift if needed.

WARNING

Any information said in this guide must not be technically correct as this is a write up made by me, a novice at best when it comes to GNU/Linux. So please bare with me or correct me in the comments.

# Fedora installation and partitioning

Boot into your Fedora iso and select install.

![1_GFG_Fedora_timeshift_installationhome-page](https://github.com/user-attachments/assets/80b73519-21ff-46b5-92a9-0c7af6d78f3b)

Select your prefered language and keyboard layout.

![2_GFG_Fedora_timeshift_select-lang-768](https://github.com/user-attachments/assets/137373d3-2078-4201-86cc-eb6b4fb9601c)

Click on "installation destination".

![4_GFG_Fedora_timeshift_installation-destination-768](https://github.com/user-attachments/assets/2cd1b130-0c32-429b-9a43-795a29f770ce)

Choose "advanced custom (blivet-gui)".

![5_GFG_Fedora_timeshift_advanced-gui-768](https://github.com/user-attachments/assets/0da6d2d6-ab03-4af2-a1f5-25f7ff5c5915)

Create the EFI partition by selecting "free space" and pressing the + icon

![6_GFG_Fedora_timeshift_partition-page-768](https://github.com/user-attachments/assets/88c2824f-0972-4c27-90c2-693da5e906aa)

Choose 600 MiB as size and select "Efi System Partition" as the filesystem. Label it "EFI" and type in "/boot/efi" as the mountpoint. Press ok.

![7_GFG_Fedora_timeshift_efi](https://github.com/user-attachments/assets/eb11a19c-eed2-473f-be1b-85a94205060e)

Now create the BTRFS volumegroup by selecting "free space" again und pressing the + icon. Set "btrfs" as the filesystem and make it as big as you need the main partition. Genereally speaking, I'm using the default which would be the rest of the disk. You can enable encryption if you want, but it isn't necessary in most cases. Leave the field dor "Mountpoint" blank! Press ok.

![9_GFG_Fedora_timeshift_btrfs](https://github.com/user-attachments/assets/20333fd7-65ae-451f-8fb9-fe669a9a7215)

Select the btrfs volume that we have just created a hit the + icon again. Enter "@" as the name and "/" as the mountpoint. 
We are going to use this specific naming scheme because Timeshift is set up for ubuntu based systems that use this exact @ naming scheme. If we wouldn't do this, Timeshift wouldn't let us use the BTRFS option because it expects this @ naming scheme. 
Press ok.

![10_GFG_Fedora_timeshift_root-subvolume](https://github.com/user-attachments/assets/7afb0c9c-1b45-40be-bda3-2e8a394a5216)

Repeat the steps from before with "@home" as the name and "/home" as the mounting point. Press ok.

![11_GFG_Fedora_timeshift_home-subvolume](https://github.com/user-attachments/assets/64ee35dc-e6a4-417d-9ffb-2f00cb2d412d)

Apply and accept the changes. There should be 9 orders in the "summary of changes" windows if you replicated all the steps.
Create a user and begin the installation. Restart the system after it's done.

# Setting up Fedora and Timeshift

During the boot you may or may not have noticed the "error: ../../grub-core/commands/loadenv.c:216:sparse file not allowed." error. This is, because we have not set up a seperate ext4 /boot partition. Instead it has been installed into our BTRFS system root. This is needed in order for us to also take a snapshot of the kernel which we boot into. This avoids a kernel mismatch when rolling back a snapshot after a kernel update. A more thorough explanation would be:

"GRUB preboot writes to /boot/grub2/grubenv if the boot was successful. This error occurs because of the GRUB btrfs.mod driver, unlike ext4, is read-only.
To resolve this, open the terminal and execute the following command."

In order to fix to issue we are going to paste this command into the terminal and execute it.
"sudo grub2-editenv - unset menu_auto_hide"

Next up we are going to a few applications that will come in handy later. 
"sudo dnf install git inotify-tools make timeshift"

After this step you can update your system with "sudo dnf update" and restart it if you want to. You can also skip this part and do it at the end of the setup. Personal preference in this case.












