Tiny Core Yeniden Düzenleme
===========================


Giriş
-----

Tiny core dağıtımı, minimal bir dağıtımdır. Yaklaşık 10 MB civarında olan iso dosya boyutuna
karşılık, kullanıcıya grafik ortam sağlayabilmektedir. Küçük boyutu sayesinde, yeniden
düzenlemeye uygun bir dağıtımdır.

Dağıtımın boyutu küçük olmasına karşın, pek çok uygulama sonradan internetten yüklenip
kullanılabilmektedir. Aynı zamanda, 10 MB olan orijinal iso içine eklemeler yapmak ta
mümkündür.

Kendimize özel iso oluştururken, gerekli uygulamalar belirlenip, paketleri iso içine eklenecektir. Bu
dokümanın amacı, tiny core dağıtımı kullanılarak kendi özel sürümümüzü oluşturmak için gereken
adımların belgelenmesidir.

Dağıtım ve Paketleri
--------------------

Dağıtımın web sitesi, http://www.tinycorelinux.com adresindedir. Dağıtım 2.6 çekirdek, Busybox,
Tiny X ve Fltk kullanılarak oluşturulmuştur. Dağıtım tamamen hafıza üzerinde çalışır. Micro core
olarak adlandırılan 6 MB büyüklükteki parçası ise sadece metin tabanlı bir ortam sağlar.

Tiny core, herşeyiyle hazır bir masaüstü sunmaz. İçinde sadece minimal bir X ortamı ve kablolu
bağlantıyı çalıştırabilecek kadar uygulama ve çekirdek bulunur. Kullanıcı, kendisine gereken
uygulamaları bir paket yöneticisi uygulamasıunı kullanarak internetten çekebilmektedir. Paket
yöneticisinin gereklilik çözümleme özelliği de bulunmaktadır. Örneğin acl.tcel paketini yüklemek
istediğimizde, bu paketin bağımlılığı olan libattr.tcel paketi de internetten yüklenerek kurulacaktır.
Paket yöneticisi kullanılarak, donanımları tanımak için gereken çekirdek modülleri de sonradan
yüklenebilmektedir.

Yukarıda anlatılanlar, orijinal dağıtımın kullanılması durumunda kullanılmaktadır. Kendi
dağıtımımızda, gereken uygulamalar belirlenerek, internetten ek paket çekmeye gerek kalmadan
istediğimiz uygulamaları bize sağlayacak bir ortam oluşturacağız.

Çalışma Şekilleri
-----------------

Cloud/Internet:
***************

Bu modelde işletim sistemi bütünüyle hafızada çalışmaktadır. Bu çalışma türünde, internetten
çekilen paketler sadece o çalışma süresince kullanılabilir. Bir sonraki sistem açılışında yeniden
paketlerin yüklenmesi gerekecektir.
Sistem açılışı sırasında verilebilecek seçeneklerle, hem yüklenmiş olan paketlerin, hem de
kullanıcının ev dizininin saklanabilmesi de mümkündür.


Persistent TCE:
***************

Sistem açılışı sırasında tce=hdxx seçeneği verilerek, paket yöneticisinin internetten yüklediği paket
dosyalarının hdxx disk bölümüne yazması sağlanır. Böylece, sistem tekrar açıldığında, hdxx disk
bölümünde bulunan paketler, tekrar internetten çekilmeden kullanılabilmektedir.


Persistent Home:
****************

Sistem açılışı sırasında home=hdxx seçeneği verilerek, kullanıcı dizini olarak kullanılan /home/tc
dizini, /mnt/hdxx/tchome dizinine bağlanır. İşletim sisteminin sağladığı yedekleme özelikleri,
yazılabilir disk bölümlerinin sağlanması ile çalışabilmektedir.


Encrypted Persistent Home:
**************************

Kullanıcı dizininin bulunduğu disk bölümü, şifreli olacak şekilde ayarlanabilir. Bu işlem için bir
loop back dosyası kullanılır. Şifreli kullanıcı dizini yaratmak için, tiny core sistemi çalışırken sağ
fare tuşuna basılarak çıkan menüden Tools - Make Crypto Home File seçilir.

Uygulama, kullanabileceğimiz disk bölümlerinden birisini seçtirerek, kullanıcının belirleyeceği büyüklükte bir
dosya yaratılır. Yaratılan dosya için bir şifre istenir. Sistemi yeniden başlatırken cryptohome=hdxx
verilerek şifreli dizin kullanılabilir.

Şifreli kullanıcı dizinini yaratan uygulama, /usr/bin/mkcryptohome shell betiğidir. Bu betik
incelendiğinde, kullanıcı dizininin losetup uygulaması ile AES kullanılacak şekilde yaratıldığı
görülür. Bu aşamalara müdahale edilerek, kendi smart kartımız kullanılarak şifreli kullanıcı dizini
yaratılabilmesi mümkündür.

Remaster İşlemi
---------------

Remaster işleminde, orijinal tiny core dağıtımını, kendi seçtiğimiz uygulamaları içerecek şekilde
değiştireceğiz. Aynı zamanda, sistem içinde arka plan resmi, kurum logosu gibi şekillerin nereye ve
nasıl aktarılacağı da anlatılacaktır.

Remaster işlemi en kolay bir linux sisteminde gerçekleştirilebilir. İşlem sırasında cpio paketlerinin
açılması gibi işlemler de bulunduğundan, aynı işlemi windows sistem üzerinde yapmak için çeşitli
uygulamaların ve mkisofs uygulamasının da windows sistem üzerinde bulunması gerekecektir.
Tiny core işletim sistemi gzip ile sıkıştırılmış bir cpio arşividir. Remaster işlemi, üzerinde cpio, tar,
gzip, advdef, mkisofs uygulamalarını içeren herhangi bir sistemde gerçekleştirilebilir.


Öncelikle, dağıtımın iso dosyası internetten çekilir. Çekilen dosyanın tinycore.iso olduğunu
varsayıyoruz.

Çekilen dosya içinde bulunan dosyalar, bir dizine kopyalanır:

.. code-block:: bash
		
   sudo mkdir /mnt/tmp
   sudo mkdir /tmp/yenidizin
   sudo mount tinycore.iso /mnt/tmp -o loop,ro
   cp -a /mnt/tmp/boot /tmp/yenidizin
   sudo umount /mnt/tmp

Daha sonra, işletim sisteminin dizin yapısı açık şekilde oluşturulur:

.. code-block:: bash

   mkdir /tmp/extract
   cd /tmp/extract
   zcat /tmp/yenidizin/boot/tinycore.gz | sudo cpio -i -H newc -d

Şimdi bütün işletim sistemi /tmp/extract dizininde bulunmaktadır. Bu aşamadan sonra, /tmp/extract
dizininde istediğimiz değişiklikleri yapabiliriz. Sisteme yeni paketler eklemek için, tce uzantılı
tar.gz paketi olarak bulundurulan uygulama paketlerinin içindeki dosyalar paket içinden
çıkarılarak /tmp/extract dizinine kopyalanır. İşletim sistemi açılışında başlatılmasını istediğimiz
uygulamaları /tmp/extract/opt/bootlocal.sh dosyasının içine yerleştiririz.

Standart masaüstü resmi, /etc/skel/.logo.xpm dosyasından gelmektedir. Bu dosya yerine, kendi
istediğimiz resim dosyasının arka plana çıkmasını sağlamak için, .logo.xpm dosyasını, kendi
istediğimiz bir resim dosyası ile değiştirebiliriz.

Yeni tinycore.gz Dosyası Oluşturma
----------------------------------

Eğer kullandığımız tiny core sürümü 2.1 ve daha aşağısı ise ve yeni pakete daha önce olmayan
kernel modülleri eklemiş isek, aşağıdaki komutların çalıştırılması gerekmektedir:

.. code-block:: bash
		
   sudo chroot /tmp/extract depmod -a 2.6.29.1-tinycore

Eğer kullandığımız tiny core sürümü 2.2 ve üzeri ise ve yeni pakete daha önce olmayan kernel
modülleri eklenmiş ise, aşağıdaki komutların çalıştırılması gerekir:

.. code-block:: bash

   sudo depmod -b /tmp/extract 2.6.29.1-tinycore

Eğer sisteme paylaşımlı kütüphaneler (shared libraries) eklenmiş ise aşağıdaki komut çalıştırılır:

.. code-block:: bash
		
   sudo ldconfig -r /tmp/extract

Bu işlemlerden sonra, tinycore.gz dosyamızı yaratabiliriz:

.. code-block:: bash
		
   cd /tmp/extract
   find | sudo cpio -o -H newc | gzip -2 > ../tinycore.gz
   cd /tmp
   advdef -z4 tinycore.gz


Yeni ISO Dosyası Yaratma
------------------------

Değiştirilmiş sistemimizin bulunduğu yeni iso dosyasını yaratmak için aşağıdaki komut verilir:

.. code-block:: bash

   cd /tmp
   mv tinycore.gz boot
   mkdir newiso
   mv boot newiso
   mkisofs -l -J -V TC-custom -no-emul-boot -boot-load-size 4 \
   -boot-info-table -b boot/isolinux/isolinux.bin \
   -c boot/isolinux/boot.cat -o TC-remastered.iso newiso

