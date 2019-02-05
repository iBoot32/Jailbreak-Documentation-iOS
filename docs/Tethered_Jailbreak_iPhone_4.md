# Tethered Jailbreaking an iPhone 4 From Scratch
- By: iBoot32

This was something I personally had to research a lot on my own, so this should come in handy to someone at some point to ease research.
This method uses the [limera1n exploit](https://www.theiphonewiki.com/wiki/Limera1n_Exploit) and `irecovery` to load a pwned ibss, pwned ibec, devicetree, pwned ramdisk, and pwned kernelcache. The ramdisk contains all needed files for the jailbreak.

This tool (for simplicity's sake) is a batch script. 

This will all be implemented into an automated tool (by me) soon enough. This is just so there's some documentation on this process before I forget.

I will document the tool line-by-line after its release, to make things even simpler. This is just what I can do with my time currently pre-release.

EDIT: THIS 'CODE' IS PREPARED FOR THE iPhone3,3 (iPhone 4 CDMA), but it can be modified by changing the firmware keys and download links and stuff to fit your model.

***

First, we will download all the needed binaries from my GitHub, and all the IPSW's files directly from Apple.

```
echo Downloading wget
powershell.exe -Command (new-object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/iBoot32/JailbreakTool/master/wget.exe', 'C:/JBWork/wget.exe') 2> nul
cd "C:/JBWork"

echo Downloading 7z.exe
wget.exe https://raw.githubusercontent.com/iBoot32/JailbreakTool/master/7z.exe 2> nul

echo Downloading 7z.dll
wget.exe https://raw.githubusercontent.com/iBoot32/JailbreakTool/master/7z.dll 2> nul

echo Downloading Binaries
wget.exe https://github.com/iBoot32/JailbreakTool/archive/master.zip 2> nul

echo Unzipping binaries
7z.exe x master.zip > nul 2>&1
cd JailbreakTool-master
move * .. > nul 2>&1
move otherfiles .. > nul 2>&1
cd ..
del master.zip
del /f /s /q "JailbreakTool-master" 1>nul && rmdir /s /q "JailbreakTool-master" 

echo Downloading Ramdisk... 
echo.
"C:/JBWork/partialzip.exe" "http://appldnld.apple.com/iOS5.1.1/041-4347.20120427.o2yov/iPhone2,1_5.1.1_9B206_Restore.ipsw" "038-4349-020.dmg" "C:/JBWork/ramdisk.dmg"

echo Downloading iBEC and iBSS
"C:/JBWork/partialzip.exe" "http://appldnld.apple.com/iOS5.1.1/041-4347.20120427.o2yov/iPhone2,1_5.1.1_9B206_Restore.ipsw" "Firmware/dfu/iBSS.n88ap.RELEASE.dfu" "C:/JBWork/ibss.dfu"
"C:/JBWork/partialzip.exe" "http://appldnld.apple.com/iOS5.1.1/041-4347.20120427.o2yov/iPhone2,1_5.1.1_9B206_Restore.ipsw" "Firmware/dfu/iBEC.n88ap.RELEASE.dfu" "C:/JBWork/ibec.dfu"
```

Now, we will patch all the needed files. Until I upload my patchfiles, you can grab the patched files from `ssh_rd` and diff them with a stock file using `fuzzy_patcher` (`fuzzy_patcher.exe --diff --orig <orig-file> --patched <ssh_rd-patched-file> --delta <name>.patch`). The stock file needs to be decrypted, and possibly the files from `ssh_rd`. I honestly don't remember. 

Furthermore, the `ssh.tar` file extracted to the ramdisk in a bit, is actually the same ssh.tar from `ssh_rd`, just with a Cydia.tar and patched `fstab` included in `/bin`

```
echo Decrypting iBEC and iBSS
xpwntool ibec.dfu ibec.dfu.dec -k 677be330d799ffafad651b3edcb34eb787c2d6c56c07e6bb60a753eb127ffa75 -iv 1fe15472e85b169cd226ce18fe6de524
xpwntool ibss.dfu ibss.dfu.dec -k 36782ee3df23e999ffa955a0f0e0872aa519918a256a67799973b067d1b4f5e0 -iv 0cbb6ea94192ba4c4f215d3f503279f6

echo patching iBEC and iBSS
fuzzy_patcher --patch --delta ibec.patch --orig ibec.dfu.dec --patched ibec.dfu.dec.p
fuzzy_patcher --patch --delta ibss.patch --orig ibss.dfu.dec --patched ibss.dfu.dec.p

echo reencrypting iBEC and iBSS
move ibec.dfu ibec.dfu.orig
move ibss.dfu ibss.dfu.orig
xpwntool ibec.dfu.dec.p ibec.dfu -t ibec.dfu.orig -iv 1fe15472e85b169cd226ce18fe6de524 -k 677be330d799ffafad651b3edcb34eb787c2d6c56c07e6bb60a753eb127ffa75
xpwntool ibss.dfu.dec.p ibss.dfu -t ibss.dfu.orig -iv 0cbb6ea94192ba4c4f215d3f503279f6 -k 36782ee3df23e999ffa955a0f0e0872aa519918a256a67799973b067d1b4f5e0

echo decrypting kernelcache
xpwntool kernelcache.release.n88 kern.dec -k 0cc1dcb2c811c037d6647225ec48f5f19e14f2068122e8c03255ffe1da25dec3 -iv 0dc795a64cb411c21033f97bceb96546

echo patching kernelcache
fuzzy_patcher --patch --orig kern.dec --delta kernelcache.patch --patched kern.dec.p

echo reencrypting kernelcache
move kernelcache.release.n88 kern.orig
xpwntool kern.dec.p kernelcache.release.n88 -t kern.orig -k 0cc1dcb2c811c037d6647225ec48f5f19e14f2068122e8c03255ffe1da25dec3 -iv 0dc795a64cb411c21033f97bceb96546

echo Decrypting Ramdisk
echo.
xpwntool ramdisk.dmg ramdisk.dmg.dec -k 7af575ca159ba58b852dfe1c6f30c68220a7a94be47ef319ce4f46ba568b7a81 -iv 26ec90f47073acaa0826c55bdeddf4bb

echo.
echo Enlarging Ramdisk
echo.
hfsplus ramdisk.dmg.dec grow 40000000

echo.
echo Adding ssh service to Ramdisk
echo.
hfsplus ramdisk.dmg.dec untar ssh.tar "/"

echo.
echo Rebuilding Ramdisk
echo.
move ramdisk.dmg ramdisk.dmg.orig
xpwntool ramdisk.dmg.dec ramdisk.dmg -t ramdisk.dmg.orig -k 7af575ca159ba58b852dfe1c6f30c68220a7a94be47ef319ce4f46ba568b7a81 -iv 26ec90f47073acaa0826c55bdeddf4bb
```

Now that we have all the needed files, and ibec, ibss, and the kernelcache are patched, we can boot the device using these components. 

```
echo tetherbooting the device
irec -e
echo Sending iBSS
irecovery -f ibss.dfu
echo Sending iBEC
irecovery -f ibec.dfu
timeout /t 15 >NUL
echo Sending devicetree
irecovery -f devicetree.img3
irecovery -c devicetree
timeout /t 6 >NUL
echo Sending ramdisk
irecovery -f ramdisk.dmg
timeout /t 5 >NUL
irecovery -c ramdisk 0x90000000
timeout /t 6 >NUL
echo Sending kernelcache
irecovery -f kernelcache.release.n88
timeout /t 5 >NUL
echo Booting kernelcache to start ssh service
irecovery -c bootx
```

Now, we forward the ssh service bundled in the ramdisk to port 2022

```
itunnel_mux --lport 2022
```

Now we ssh into the device using whatever tool (this can be modified using `plink` from PuTTY), mount the filesystem, extract Cydia.tar to the rootfs, and move the patched fstab.

```
ssh -p 2022 root@127.0.0.1

root@127.0.0.1's password: alpine

cd /
mount.sh
cd /mnt1
/bin/mv /mnt1/etc/fstab /mnt1/etc/fstab.old
/bin/mv /bin/fstab /mnt1/etc/fstab
/bin/mv /bin/Cydia.tar /mnt1/Cydia.tar
cd /mnt1
/bin/tar -xf Cydia.tar
```

Now the device has Cydia installed and `fstab` patched.

Now all we do is boot the device (same as we did before) with a patched kernelcache (can find on in geeksn0w), and you'll be tethered jailbroken. If you reboot, you just have to boot with the pwned kernelcache from geeksn0w agan. This process will be written out in detail later. My laptop battery is dying.

EDIT: The pwned kernelcache referenced in the previous paragraph *must* have all the jailbreak patches applied (codesign, etc.) so you must use the one from geeksn0w for this part.

