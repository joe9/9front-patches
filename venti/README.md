
````
from http://9legacy.org/patch.html

libventi-noarchive 	/sys/src/libventi 	Define VtEntryNoArchive constant. 	David du Colombier
http://9legacy.org/9legacy/patch/libventi-noarchive.diff

libventi-redial 	/sys/src/libventi 	Implement vtreconn and vtredial functions. 	David du Colombier
http://9legacy.org/9legacy/patch/libventi-redial.diff

libventi-sha1 	/sys/src/libventi 	Implement vtsha1 and vtsha1check functions. 	David du Colombier
http://9legacy.org/9legacy/patch/libventi-sha1.diff

I am not sure if I need this patch. I cannot find sys/lib/dist/pc directory on plan9front
dist-venti-bloom 	/sys/lib/dist/pc/inst 	Add Venti Bloom filter configuration. 	David du Colombier

<mycroftiv> you dont need that bloom filter config, that is for making the installer
            set up bloom filter when it is doing a clean system install

installing venti patches (using drawterm)

	cd /
	ape/patch -p0 </mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/libventi-noarchive.diff
	ape/patch -p0 </mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/libventi-redial.diff
	ape/patch -p0 </mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/libventi-sha1.diff
	ape/patch -p1 </mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/start-cfg-sysname-termrc-after-ipconfig.patch
	ape/patch -p1 </mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/stop-fossil-venti-servers.patch
	cd /sys/src/libventi
	mk clean
	mk install
	mk clean

installing fossil
   ;# hg clone https://bitbucket.org/mycroftiv/fossilnewventi
   This is the new version of fossil using libventi instead of liboventi

	cd /mnt/term/home/j/dev/apps/plan9/custom/src/fossilnewventi/
        INSTALL

configuring fossil+venti
    # disk to be used: WD Green 2 TB - sdE2 -- venti + fossil daily backup
    # delete any existing partition on sdE2, using disk/fdisk /dev/sdE2/data, d p1, w, q

# disk/mbr /dev/sdE2/data
# disk/fdisk -baw /dev/sdE2/data
# ls /dev/sdE2
/dev/sdE2/ctl
/dev/sdE2/data
/dev/sdE2/plan9
/dev/sdE2/raw
# disk/prep -w -b -a^(fossil arenas bloom isect) /dev/sdE2/plan9
no plan9 partition table found
fossil 624956068
arenas 3124780340
isect 156239018
bloom 1048573

<mycroftiv> joe9: however big the bloom filter is, it all gets loaded into ram, the partitioner caps it at 512mb but im pretty sure it will allocate that full 512mb by default on a hdd as large as you are using
Hence, use the partition sizes reported by the p command of disk/prep to change the bloom to 64mb.
On the 2TB disk, disk/prep reported:

  empty                    0 2             (2 sectors, 1.00 KB)
  fossil                   2 624956070     (624956068 sectors, 298.00 GB)
  arenas           624956070 3749736410    (3124780340 sectors, 1.45 TB)
  isect           3749736410 3905975428    (156239018 sectors, 74.50 GB)
  bloom           3905975428 3907024001    (1048573 sectors, 511.99 MB)
  empty           3907024001 3907024002    (1 sectors, 512 B )

Using the above sizes and restricting bloom to 64m, I came up with the below
echo 'a fossil 2 2+298g
a bloom . .+64m
a isect . .+75g
a arenas . $-1
w
p
q' | disk/prep -b  /dev/sdE2/plan9

venti setup:
	venti/fmtarenas arenas /dev/sdE2/arenas
	venti/fmtisect isect /dev/sdE2/isect
	venti/fmtindex /mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/venti.conf
	venti/conf -w /dev/sdE2/arenas /mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/venti.conf
        venti/venti -c /dev/sdE2/arenas
        venti=192.168.0.5

fossil setup:
        fossil/flfmt /dev/sdE2/fossil
        fossil/conf -w /dev/sdE2/fossil /mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/initfossil.conf
        fossil/fossil -f /dev/sdE2/fossil
        # for automatic snapshots at 0500, add this to the below fossil.conf: fsys main snaptime -a 0500
	fossil/conf -w /dev/sdE2/fossil /mnt/term/home/j/dev/apps/plan9/custom/9front-patches/venti/fossil.conf
        ls /srv
        mount -c /srv/fossil /n/fossil

startup:
        # do not need these commands: diskparts does it.
        #  disk/fdisk -p /dev/sdE2/data > /dev/sdE2/ctl
        #  disk/prep -p /dev/sdE2/plan9 > /dev/sdE2/ctl

        # added these to /cfg/cirno/termrc of cirno
        venti=192.168.0.5
        venti/venti -c /dev/sdE2/arenas
        fossil/fossil -f /dev/sdE2/fossil
        ls /srv
        mount -c /srv/fossil /n/fossil
````
