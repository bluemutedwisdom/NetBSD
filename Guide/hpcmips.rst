.. 
 Copyright (c) 2013 Jun Ebihara All rights reserved.
 Redistribution and use in source and binary forms, with or without
 modification, are permitted provided that the following conditions
 are met:
 1. Redistributions of source code must retain the above copyright
    notice, this list of conditions and the following disclaimer.
 2. Redistributions in binary form must reproduce the above copyright
    notice, this list of conditions and the following disclaimer in the
    documentation and/or other materials provided with the distribution.
 THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
 IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
 OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
 IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
 INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
 NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
 DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
 THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
 (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
 THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

=================================
NetBSD/hpcmipsを使ってみる
=================================

特徴
----

* NetBSD/hpcmipsを利用するために、ディスクイメージを用意しました。
* NetBSDでmatt-nb... がマージされたあと、hpcmipsは不安定です。
* NetBSD7に向けて、バイナリイメージで試せるようにします。
* RPI向けイメージと同じバイナリを作ってみます。

準備するもの
-------------
* NetBSD/hpcmipsをサポートしているマシン (NEC mobilegear,sigmarion)
* 2GB コンパクトフラッシュ
* SigmarionII：FOMA接続ケーブル+USB ネットワークアダプタ
* PCMCIAネットワークカード (カードバス対応でないもの)

起動ディスクの作成
-------------------
* ディスクイメージのダウンロード

::

 # ftp ftp://ftp.netbsd.org/pub/NetBSD/misc/jun/hpcmips/
 2013-08-23-netbsd-hpcmips.img.gz

* 2GB以上のコンパクトフラッシュを準備します。
* ダウンロードしたディスクイメージを、コンパクトフラッシュ上で展開します。

::

	disklabel sd0  ..... 必ずインストールするSDカードか確認してください。
	gunzip < 2013-08-23-netbsd-hpcmips.img.gz|dd of=/dev/rsd0d bs=1m

NetBSD/hpcmipsの起動
------------------
#. コンパクトフラッシュをWindowsCE機に挿して起動します。
#. CF上に見えるhpcboot.exeを実行します。
#. メニューでカーネルを指定して起動します。
#. 少し待つと、HDMIからNetBSDの起動メッセージが表示されます。

ログイン
---------
 rootでログインできます。

::

	login: root


 startxでicewmが立ち上がります。

::

	# startx

mikutterを使ってみよう
----------------------
* xtermからdilloとmikutterを起動します。

::

	# dillo &
	# mikutter &

* しばらく待ちます。
* mikutterの認証画面がうまく出たら、httpsからはじまるURLをカットアンドペーストして、dilloのURL画面に張り付けます。URLをなぞって、マウスボタン両押しです。
* twitterのIDとパスワードを入力すると、pin番号が表示されます。pin番号をmikutterの認証画面に入力します。
* しばらくすると、mikutterの画面が表示されます。表示されるはずです。落ちてしまう場合は時計が合っているか確認してください。
* 漢字は[半角/全角]キーを入力すると漢字モードに切り替わります。anthyです。
* 青い鳥を消したいとき：「mikutter」「青い鳥」でぐぐってください。

キーマップの設定を変更する
--------------------------
* ログインした状態でのキーマップは/etc/wscons.confで設定します。

::

	encoding jp.swapctrlcaps .... 日本語キーボード,CtrlとCAPSを入れ替える。

* Xでのキーマップは.xinitrcで設定します。

::

	setxkbmap -model jp106 jp -option ctrl:swapcap 


* 106キーでのキーマップ

::

 on i386:
 "\|" key returns keycode 133
 "\_" key returns keycode 211

 on evbearmv6hf-el
 "\|" key returns keycode 8
 "\_" key returns keycode 8

コンパイル済パッケージをインストールする
--------------------------------------------------
* コンパイルしたパッケージを以下のURLに用意しました。

::

 ftp://ftp.netbsd.org/pub/NetBSD/misc/jun/hpcmips/2013-08-23-hpcmips/packages


* パッケージのインストール

 pkg_addコマンドで、あらかじめコンパイル済みのパッケージをインストールします。関連するパッケージも自動的にインストールします。

::

 # export PKG_PATH=ftp://ftp.netbsd.org/pub/NetBSD/misc/jun/hpcmips/2013-08-23-hpcmips/packages
 # pkg_add zsh

* パッケージの一覧

 pkg_infoコマンドで、インストールされているパッケージの一覧を表示します。

::

	# pkg_info

* パッケージの削除

::

	# pkg_delete パッケージ名


/usr/pkgsrcを使ってみよう
--------------------------
 2013/8/21時点のpkgsrc-currentが/usr/pkgsrcに展開してあります。
 たとえばwordpressをコンパイル／インストールする時には、

::

	# cd /usr/pkgsrc/www/wordpress
	# make package-install

を実行すると、wordpressに関連したソフトウェアをコンパイル／インストールします。

ユーザー作成
--------------

::

	# useradd -m jun
	# passwd jun
	# /etc/groupを編集する
	wheel:*:0:root,jun

サービス起動方法
----------------
  /etc/rc.d以下にスクリプトがあります。dhcpクライアント(dhcpcd)を起動してみます。

::

 テスト起動：
   /etc/rc.d/dhcpcd onestart
 テスト停止：
   /etc/rc.d/dhcpcd onestop

 
正しく動作することが確認できたら/etc/rc.confに以下のとおり指定します。
   dhcpcd=YES
  /etc/rc.confでYESに指定したサービスは、マシン起動時に同時に起動します。

::

 起動:
   /etc/rc.d/dhcpcd start
 停止：
   /etc/rc.d/dhcpcd stop
 再起動：
  /etc/rc.d/dhcpcd restart

vnconfigでイメージ編集
------------------------

NetBSDの場合、vnconfigコマンドでイメージファイルの内容を参照できます。

::

 # vnconfig vnd0 2013-08-23-netbsd-raspi.img
 # vnconfig -l
 vnd0: /usr (/dev/wd0e) inode 53375639
 # disklabel vnd0
 　　 :
 8 partitions:
 #        size    offset     fstype [fsize bsize cpg/sgs]
 a:   3428352    385024     4.2BSD      0     0     0  # (Cyl.    188 -   1861)
 b:    262144    122880       swap                     # (Cyl.     60 -    187)
 c:   3690496    122880     unused      0     0        # (Cyl.     60 -   1861)
 d:   3813376         0     unused      0     0        # (Cyl.      0 -   1861)
 e:    114688      8192      MSDOS                     # (Cyl.      4 -     59)
 # mount_msdos /dev/vnd0e /mnt
 # ls /mnt
 LICENCE.broadcom    cmdline.txt         fixup_cd.dat        start.elf
 bootcode.bin        fixup.dat           kernel.img          start_cd.elf
 # cat /mnt/cmdline.txt
 root=ld0a console=fb
 #fb=1280x1024           # to select a mode, otherwise try EDID 
 #fb=disable             # to disable fb completely

 # umount /mnt
 # vnconfig -u vnd0

HDMIじゃなくシリアルコンソールで使うには
----------------------------------------
* MSDOS領域にある設定ファイルcmdline.txtの内容を変更してください。

::

 ↓console=fbを消します。
 root=ld0a 
 #fb=1280x1024           # to select a mode, otherwise try EDID 
 #fb=disable             # to disable fb completely

起動ディスクを変えるには
------------------------
* MSDOS領域にある設定ファイルの内容を変更してください。

::

 root=sd0a console=fb ←ld0をsd0にするとUSB接続したディスクから起動します
 #fb=1280x1024           # to select a mode, otherwise try EDID 
 #fb=disable             # to disable fb completely

最小構成のディスクイメージ
--------------------------
  NetBSD-currentのディスクイメージに関しては、以下の場所にあります。

::

 # ftp://ftp.netbsd.org/pub/NetBSD/misc/jun/raspberry-pi/2013-08-23-evbearmv6hf-el/
 # gunzip < rpi_inst.bin.gz |dd of=/dev/rsd3d bs=1m   .... sd3にコピー。

  RaspberryPIにsdカードを差して、起動すると、#　プロンプトが表示されます。
 # sysinst      .... NetBSDのインストールプログラムが起動します。

X11のインストール
------------------
 rpi.bin.gzからインストールした場合、Xは含まれていません。追加したい場合は、
 ftp://ftp.netbsd.org/pub/NetBSD/misc/jun/raspberry-pi/2013-08-23-evbearmv6hf-el/ をダウンロードし、tarファイルを展開します。

::

 tar xzpvf xbase.tar.gz -C /

クロスビルドの方法
------------------
* ソースファイル展開
* ./build.sh -U -m evbearmv6hf-el release

pkgsrcを最新にしてみる
----------------------
* cd /usr/pkgsrc
* cvs update -PAd

外付けUSB端子
--------------
  NetBSDで利用できるUSBデバイスは利用できる（はずです)。電源の制約があるので、十分に電源を供給できる外付けUSBハブ経由で接続したほうが良いです。

液晶ディスプレイ
-----------------
  液晶キット( http://www.aitendo.com/page/28 )で表示できています。HDMI-VGA変換ではうまく表示できていません。（電源が足りない)

inode
-------
  inodeが足りない場合は、ファイルシステムを作り直してください。このイメージでは以下のようにファイルシステムを作成しています。

	# newfs -n 600000 /dev/rvnd0a

壁紙
-----
  おおしまさん(@oshimyja)ありがとうございます。


関連バグ
--------

PR 47798
 今回、mikutterのアイコンがでなくて落ちるバグに悩みました。つついさんに感謝します。
	http://gnats.netbsd.org/cgi-bin/query-pr-single.pl?number=47798

pkg/48128: icewm build broken on 6.99.23
 直っています。

port-evbarm/48132: devel/tradcpp build broken on evbearmv6hf-el 6.99.23
 まだ直っていません。soft-floatでコンパイルしようとするのでhardfloatでコンパイルします。
 s/-msoft-float/-marm -mthumb-interwork  -mfloat-abi=hard -mhard-float/

--

パーティションサイズをSDカードに合わせる
-----------------------------------------
　2GB以上のSDカードを利用している場合、パーティションサイズをSDカードに合わせることができます。この手順はカードの内容が消えてしまう可能性もあるため、重要なデータはバックアップをとるようにしてください。
  手順は、http://wiki.netbsd.org/ports/evbarm/raspberry_pi/ のGrowing the root file-systemにあります。

 このイメージのために、つついさんにスクリプトを作っていただきました。（まだテスト中です）

#. vi /etc/rc.confでrc_configured=NOに書き換え
#. reboot　.... シングルユーザで起動
#.  Enter pathname of shell or RETURN for /bin/sh: でリターン
#. cd /root/Extract/
#. sh expand-image-fssize-rpi.sh ... しばらくかかります
#.  リターンを押すと再起動します

::

 Untested sh script that will expand NetBSD partition and BSD FFS partition in the RPI image prepared 
 by Jun Ebihara: http://mail-index.netbsd.org/port-arm/2013/06/19/msg001882.html
 https://gist.github.com/tsutsui/5814498

シングルユーザでの起動
"""""""""""""""""""""
#. /etc/rc.confのrc_configured=YESをNOにして起動します。
#.  戻すときはmount / ;vi /etc/rc.conf　でNOをYESに変更してrebootします。

参考URL
--------
* http://wiki.netbsd.org/ports/evbarm/raspberry_pi/
* NetBSD Guide http://www.netbsd.org/docs/guide/en/
* NetBSD/RPiで遊ぶ(SDカードへの書き込み回数を気にしつつ)  http://hachulog.blogspot.jp/2013/03/netbsdrpisd.html
* http://www.raspberrypi.org/phpBB3/viewforum.php?f=86 NetBSDフォーラム
* http://www.raspberrypi.org/phpBB3/viewforum.php?f=82 日本語フォーラム

