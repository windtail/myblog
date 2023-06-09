---
title: "u-boot 学习系列 1 - SPL"
date: 2018-01-22T17:01:00+08:00
draft: false
---

u-boot这个东西从自我N年前使用到现在，变化好多，今天开始重新研究下，本系列的研究都是基于BeagleBoneBlack（bbb）开发板和 u-boot v201801版本的。


SPL介绍
-----


在源代码中 doc/README.SPL 中说得比较明白，我这里再归纳下。


现在很多处理器都内置一个BOOT ROM，执行部分初始化，并可从各种外设和存储器中加载程序并执行，BOOT ROM中固化的程序被称为一级程序加载器，被它加载的程序就称为二级程序加载器（secondary program loader，即SPL）。其实u-boot本身就可以作为二级程序加载器，但不幸的是一般BOOT ROM之后，主存储器都是没有初始化的，例如BBB只有109K的内置RAM可用，这就限制了程序的大小，全功能的u-boot不能运行在这么小的RAM上运行。于是精简的u-boot，u-boot-spl就此问世。


### SPL精简代码的方式


在 Kconfig 文件中有一个 CONFIG\_SPL 的选项，使能后，就会定义 CONFIG\_SPL\_BUILD


Makefile层面使用 CONFIG\_SPL\_BUILD 选择不同的文件或文件夹进行编译，例如




```
ifeq ($(CONFIG_SPL_BUILD),y)
obj-y += board_spl.o
else
obj-y += board.o
endif

obj-$(CONFIG_SPL_BUILD) += foo.o
```


 而C代码层面，则使用 CONFIG\_SPL\_BUILD 宏来进行条件编译，例如




```
#ifdef CONFIG\_SPL\_BUILD
 foo();
#endif
```


 因为C代码中有条件编译选项，所以SPL和主u-boot代码，不能复用.o文件，所有文件必须全部重新编译。下面我们就来编译试试看（arm-linux-gnueabihf-gcc请到[linaro](https://releases.linaro.org/components/toolchain/binaries/latest-7/)官方网站下载，然后放到PATH中即可）




```
make distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- O=bbb-build am335x\_boneblack\_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- O=bbb-build

```


 为了说明下问题，我把打印的输出放到这里




```
 1   CHK     include/config/uboot.release
 2   Using .. as source for U-Boot
 3   GEN     ./Makefile
 4   CHK     include/generated/version\_autogenerated.h
 5   CHK     include/generated/timestamp\_autogenerated.h
 6   UPD     include/generated/timestamp\_autogenerated.h
 7   HOSTCC  scripts/basic/fixdep
 8   CC      lib/asm-offsets.s
 9   CHK     include/generated/generic-asm-offsets.h
 10   CC      arch/arm/lib/asm-offsets.s
 11   CHK     include/generated/asm-offsets.h
 12   HOSTCC  tools/gen\_eth\_addr
 13   HOSTCC  tools/gen\_ethaddr\_crc.o
 14   WRAP    tools/lib/crc8.c
 15   HOSTCC  tools/lib/crc8.o
 16   HOSTLD  tools/gen\_ethaddr\_crc
 17   HOSTCC  tools/img2srec
 18   HOSTCC  tools/mkenvimage.o
 19   HOSTCC  tools/os\_support.o
 20   WRAP    tools/lib/crc32.c
 21   HOSTCC  tools/lib/crc32.o
 22   HOSTLD  tools/mkenvimage
 23   HOSTCC  tools/aisimage.o
 24   HOSTCC  tools/atmelimage.o
 25   WRAP    tools/common/bootm.c
 26   HOSTCC  tools/common/bootm.o
 27   HOSTCC  tools/default\_image.o
 28   WRAP    tools/lib/fdtdec\_common.c
 29   HOSTCC  tools/lib/fdtdec\_common.o
 30   WRAP    tools/lib/fdtdec.c
 31   HOSTCC  tools/lib/fdtdec.o
 32   HOSTCC  tools/fit\_common.o
 33   HOSTCC  tools/fit\_image.o
 34   WRAP    tools/common/image-fit.c
 35   HOSTCC  tools/common/image-fit.o
 36   HOSTCC  tools/image-host.o
 37   WRAP    tools/common/image.c
 38   HOSTCC  tools/common/image.o
 39   HOSTCC  tools/imagetool.o
 40   HOSTCC  tools/imximage.o
 41   HOSTCC  tools/kwbimage.o
 42   WRAP    tools/lib/md5.c
 43   HOSTCC  tools/lib/md5.o
 44   HOSTCC  tools/lpc32xximage.o
 45   HOSTCC  tools/mxsimage.o
 46   HOSTCC  tools/omapimage.o
 47   HOSTCC  tools/pblimage.o
 48   HOSTCC  tools/pbl\_crc32.o
 49   HOSTCC  tools/vybridimage.o
 50   WRAP    tools/lib/rc4.c
 51   HOSTCC  tools/lib/rc4.o
 52   HOSTCC  tools/rkcommon.o
 53   HOSTCC  tools/rkimage.o
 54   HOSTCC  tools/rksd.o
 55   HOSTCC  tools/rkspi.o
 56   HOSTCC  tools/socfpgaimage.o
 57   WRAP    tools/lib/sha1.c
 58   HOSTCC  tools/lib/sha1.o
 59   WRAP    tools/lib/sha256.c
 60   HOSTCC  tools/lib/sha256.o
 61   WRAP    tools/common/hash.c
 62   HOSTCC  tools/common/hash.o
 63   HOSTCC  tools/ublimage.o
 64   HOSTCC  tools/zynqimage.o
 65   HOSTCC  tools/zynqmpimage.o
 66   HOSTCC  tools/libfdt/fdt.o
 67   HOSTCC  tools/libfdt/fdt\_wip.o
 68   HOSTCC  tools/libfdt/fdt\_sw.o
 69   HOSTCC  tools/libfdt/fdt\_strerror.o
 70   HOSTCC  tools/libfdt/fdt\_empty\_tree.o
 71   HOSTCC  tools/libfdt/fdt\_addresses.o
 72   HOSTCC  tools/libfdt/fdt\_overlay.o
 73   WRAP    tools/lib/libfdt/fdt\_ro.c
 74   HOSTCC  tools/lib/libfdt/fdt\_ro.o
 75   WRAP    tools/lib/libfdt/fdt\_rw.c
 76   HOSTCC  tools/lib/libfdt/fdt\_rw.o
 77   WRAP    tools/lib/libfdt/fdt\_region.c
 78   HOSTCC  tools/lib/libfdt/fdt\_region.o
 79   HOSTCC  tools/gpimage.o
 80   HOSTCC  tools/gpimage-common.o
 81   HOSTCC  tools/dumpimage.o
 82   HOSTLD  tools/dumpimage
 83   HOSTCC  tools/mkimage.o
 84   HOSTLD  tools/mkimage
 85   HOSTCC  tools/proftool
 86   HOSTCC  tools/fdtgrep.o
 87   HOSTLD  tools/fdtgrep
 88   LD      arch/arm/cpu/built-in.o
 89   CC      arch/arm/cpu/armv7/cache\_v7.o
 90   AS      arch/arm/cpu/armv7/cache\_v7\_asm.o
 91   CC      arch/arm/cpu/armv7/cpu.o
 92   CC      arch/arm/cpu/armv7/cp15.o
 93   CC      arch/arm/cpu/armv7/syslib.o
 94   LD      arch/arm/cpu/armv7/built-in.o
 95   AS      arch/arm/cpu/armv7/start.o
 96   AS      arch/arm/lib/vectors.o
 97   AS      arch/arm/lib/crt0.o
 98   AS      arch/arm/lib/setjmp.o
 99   AS      arch/arm/lib/relocate.o
100   CC      arch/arm/lib/bootm-fdt.o
101   CC      arch/arm/lib/bootm.o
102   CC      arch/arm/lib/zimage.o
103   AS      arch/arm/lib/memset.o
104   AS      arch/arm/lib/memcpy.o
105   CC      arch/arm/lib/sections.o
106   CC      arch/arm/lib/stack.o
107   CC      arch/arm/lib/interrupts.o
108   CC      arch/arm/lib/reset.o
109   CC      arch/arm/lib/cache.o
110   CC      arch/arm/lib/cache-cp15.o
111   CC      arch/arm/lib/psci-dt.o
112   LD      arch/arm/lib/built-in.o
113   AS      arch/arm/lib/ashldi3.o
114   AS      arch/arm/lib/ashrdi3.o
115   CC      arch/arm/lib/div0.o
116   AS      arch/arm/lib/div64.o
117   AS      arch/arm/lib/lib1funcs.o
118   AS      arch/arm/lib/lshrdi3.o
119   AS      arch/arm/lib/muldi3.o
120   AS      arch/arm/lib/uldivmod.o
121   AR      arch/arm/lib/lib.a
122   CC      arch/arm/lib/eabi\_compat.o
123   AS      arch/arm/lib/crt0\_arm\_efi.o
124   CC      arch/arm/lib/reloc\_arm\_efi.o
**125 CC arch/arm/mach-omap2/am33xx/clock\_am33xx.o**126   CC      arch/arm/mach-omap2/am33xx/clock.o
127   CC      arch/arm/mach-omap2/am33xx/sys\_info.o
128   CC      arch/arm/mach-omap2/am33xx/ddr.o
129   CC      arch/arm/mach-omap2/am33xx/board.o
130   CC      arch/arm/mach-omap2/am33xx/mux.o
131   CC      arch/arm/mach-omap2/am33xx/prcm-regs.o
132   CC      arch/arm/mach-omap2/am33xx/hw\_data.o
133   CC      arch/arm/mach-omap2/am33xx/fdt.o
134   CC      arch/arm/mach-omap2/am33xx/clk\_synthesizer.o
135   LD      arch/arm/mach-omap2/am33xx/built-in.o
136   CC      arch/arm/mach-omap2/reset.o
137   CC      arch/arm/mach-omap2/timer.o
138   CC      arch/arm/mach-omap2/utils.o
139   CC      arch/arm/mach-omap2/sysinfo-common.o
140   CC      arch/arm/mach-omap2/omap-cache.o
141   CC      arch/arm/mach-omap2/boot-common.o
142   AS      arch/arm/mach-omap2/lowlevel\_init.o
143   CC      arch/arm/mach-omap2/mem-common.o
144   CC      arch/arm/mach-omap2/fdt-common.o
145   LD      arch/arm/mach-omap2/built-in.o
146   CC      board/ti/am335x/board.o
147   LD      board/ti/am335x/built-in.o
148   CC      board/ti/common/board\_detect.o
149   LD      board/ti/common/built-in.o
150   CC      cmd/boot.o
151   CC      cmd/bootm.o
152   CC      cmd/help.o
153   CC      cmd/version.o
154   CC      cmd/blk\_common.o
155   CC      cmd/source.o
156   CC      cmd/bdinfo.o
157   CC      cmd/bootefi.o
158   CC      cmd/bootz.o
159   CC      cmd/console.o
160   CC      cmd/echo.o
161   CC      cmd/eeprom.o
162   CC      cmd/elf.o
163   CC      cmd/exit.o
164   CC      cmd/ext4.o
165   CC      cmd/ext2.o
166   CC      cmd/fat.o
167   CC      cmd/fdt.o
168   CC      cmd/fs.o
169   CC      cmd/gpio.o
170   CC      cmd/i2c.o
171   CC      cmd/itest.o
172   CC      cmd/load.o
173   CC      cmd/mem.o
174   CC      cmd/mii.o
175   CC      cmd/mdio.o
176   CC      cmd/misc.o
177   CC      cmd/mmc.o
178   CC      cmd/net.o
179   CC      cmd/part.o
180   CC      cmd/pcmcia.o
181   CC      cmd/pxe.o
182   CC      cmd/sf.o
183   CC      cmd/spi.o
184   CC      cmd/time.o
185   CC      cmd/test.o
186   CC      cmd/usb.o
187   CC      cmd/disk.o
188   CC      cmd/fastboot.o
189   CC      cmd/ximg.o
190   CC      cmd/spl.o
191   CC      cmd/dfu.o
192   CC      cmd/gpt.o
193   CC      cmd/nvedit.o
194   LD      cmd/built-in.o
195   CC      common/init/board\_init.o
196   LD      common/init/built-in.o
197   CC      common/main.o
198   CC      common/exports.o
199   CC      common/hash.o
200   CC      common/cli\_hush.o
201   CC      common/autoboot.o
202   CC      common/board\_f.o
203   CC      common/board\_r.o
204   CC      common/board\_info.o
205   CC      common/bootm.o
206   CC      common/bootm\_os.o
207   CC      common/fdt\_support.o
208   CC      common/miiphyutil.o
209   CC      common/usb.o
210   CC      common/usb\_hub.o
211   CC      common/usb\_storage.o
212   CC      common/splash.o
213   CC      common/menu.o
214   CC      common/update.o
215   CC      common/cli\_readline.o
216   CC      common/cli\_simple.o
217   CC      common/console.o
218   CC      common/dlmalloc.o
219   CC      common/malloc\_simple.o
220   CC      common/image.o
221   CC      common/image-android.o
222   CC      common/image-fdt.o
223   CC      common/image-fit.o
224   CC      common/memsize.o
225   CC      common/stdio.o
226   CC      common/cli.o
227   CC      common/dfu.o
228   CC      common/command.o
229   CC      common/s\_record.o
230   CC      common/xyzModem.o
231   LD      common/built-in.o
232   CC      disk/part.o
233   CC      disk/part\_dos.o
234   CC      disk/part\_iso.o
235   CC      disk/part\_efi.o
236   LD      disk/built-in.o
237   LD      drivers/adc/built-in.o
238   LD      drivers/ata/built-in.o
239   CC      drivers/block/blk\_legacy.o
240   LD      drivers/block/built-in.o
241   CC      drivers/bootcount/bootcount.o
242   CC      drivers/bootcount/bootcount\_davinci.o
243   LD      drivers/bootcount/built-in.o
244   CC      drivers/core/device.o
245   CC      drivers/core/fdtaddr.o
246   CC      drivers/core/lists.o
247   CC      drivers/core/root.o
248   CC      drivers/core/uclass.o
249   CC      drivers/core/util.o
250   CC      drivers/core/device-remove.o
251   CC      drivers/core/dump.o
252   LD      drivers/core/built-in.o
253   CC      drivers/crypto/fsl/sec.o
254   LD      drivers/crypto/fsl/built-in.o
255   LD      drivers/crypto/rsa_mod_exp/built-in.o
256   LD      drivers/crypto/built-in.o
257   CC      drivers/dfu/dfu.o
258   CC      drivers/dfu/dfu\_mmc.o
259   CC      drivers/dfu/dfu\_ram.o
260   CC      drivers/dfu/dfu\_tftp.o
261   LD      drivers/dfu/built-in.o
262   LD      drivers/firmware/built-in.o
263   CC      drivers/input/input.o
264   LD      drivers/input/built-in.o
265   LD      drivers/mailbox/built-in.o
266   LD      drivers/memory/built-in.o
267   LD      drivers/misc/built-in.o
268   CC      drivers/mmc/mmc.o
269   CC      drivers/mmc/mmc\_legacy.o
270   CC      drivers/mmc/mmc\_write.o
271   CC      drivers/mmc/omap\_hsmmc.o
272   LD      drivers/mmc/built-in.o
273   LD      drivers/pcmcia/built-in.o
274   LD      drivers/phy/marvell/built-in.o
275   LD      drivers/pwm/built-in.o
276   LD      drivers/reset/built-in.o
277   CC      drivers/rtc/date.o
278   LD      drivers/rtc/built-in.o
279   LD      drivers/scsi/built-in.o
280   LD      drivers/soc/built-in.o
281   LD      drivers/sound/built-in.o
282   LD      drivers/spmi/built-in.o
283   LD      drivers/sysreset/built-in.o
284   LD      drivers/thermal/built-in.o
285   LD      drivers/tpm/built-in.o
286   LD      drivers/video/bridge/built-in.o
287   LD      drivers/video/sunxi/built-in.o
288   LD      drivers/video/built-in.o
289   CC      drivers/watchdog/omap\_wdt.o
290   LD      drivers/watchdog/built-in.o
291   LD      drivers/built-in.o
292   LD      drivers/dma/built-in.o
293   CC      drivers/gpio/gpio-uclass.o
294   CC      drivers/gpio/omap\_gpio.o
295   LD      drivers/gpio/built-in.o
296   CC      drivers/i2c/i2c\_core.o
297   CC      drivers/i2c/omap24xx\_i2c.o
298   LD      drivers/i2c/built-in.o
299   CC      drivers/mtd/mtdcore.o
300   CC      drivers/mtd/mtd\_uboot.o
301   LD      drivers/mtd/built-in.o
302   LD      drivers/mtd/onenand/built-in.o
303   CC      drivers/mtd/spi/sf\_probe.o
304   CC      drivers/mtd/spi/spi\_flash.o
305   CC      drivers/mtd/spi/spi\_flash\_ids.o
306   CC      drivers/mtd/spi/sf.o
307   LD      drivers/mtd/spi/built-in.o
308   CC      drivers/net/cpsw.o
309   CC      drivers/net/cpsw-common.o
310   LD      drivers/net/built-in.o
311   CC      drivers/net/phy/phy.o
312   CC      drivers/net/phy/atheros.o
313   CC      drivers/net/phy/smsc.o
314   LD      drivers/net/phy/built-in.o
315   LD      drivers/pci/built-in.o
316   LD      drivers/power/built-in.o
317   LD      drivers/power/battery/built-in.o
318   LD      drivers/power/domain/built-in.o
319   LD      drivers/power/fuel_gauge/built-in.o
320   LD      drivers/power/mfd/built-in.o
321   CC      drivers/power/pmic/pmic\_tps65217.o
322   CC      drivers/power/pmic/pmic\_tps65910.o
323   LD      drivers/power/pmic/built-in.o
324   LD      drivers/power/regulator/built-in.o
325   CC      drivers/serial/serial-uclass.o
326   CC      drivers/serial/ns16550.o
327   LD      drivers/serial/built-in.o
328   CC      drivers/spi/spi.o
329   CC      drivers/spi/omap3\_spi.o
330   LD      drivers/spi/built-in.o
331   LD      drivers/usb/common/built-in.o
332   LD      drivers/usb/dwc3/built-in.o
333   LD      drivers/usb/emul/built-in.o
334   LD      drivers/usb/eth/built-in.o
335   CC      drivers/usb/gadget/epautoconf.o
336   CC      drivers/usb/gadget/config.o
337   CC      drivers/usb/gadget/usbstring.o
338   CC      drivers/usb/gadget/g\_dnl.o
339   CC      drivers/usb/gadget/f\_dfu.o
340   CC      drivers/usb/gadget/f\_fastboot.o
341   CC      drivers/usb/gadget/ether.o
342   CC      drivers/usb/gadget/rndis.o
343   LD      drivers/usb/gadget/built-in.o
344   LD      drivers/usb/gadget/udc/built-in.o
345   LD      drivers/usb/host/built-in.o
346   CC      drivers/usb/musb-new/musb\_gadget.o
347   CC      drivers/usb/musb-new/musb\_gadget\_ep0.o
348   CC      drivers/usb/musb-new/musb\_core.o
349   CC      drivers/usb/musb-new/musb\_uboot.o
350   CC      drivers/usb/musb-new/musb\_host.o
351   CC      drivers/usb/musb-new/musb\_dsps.o
352   LD      drivers/usb/musb-new/built-in.o
353   LD      drivers/usb/musb/built-in.o
354   LD      drivers/usb/phy/built-in.o
355   LD      drivers/usb/ulpi/built-in.o
356   CC      env/common.o
357   CC      env/env.o
358   CC      env/attr.o
359   CC      env/callback.o
360   CC      env/flags.o
361   CC      env/mmc.o
362   LD      env/built-in.o
363   CC      fs/fs.o
364   CC      fs/ext4/ext4fs.o
365   CC      fs/ext4/ext4\_common.o
366   CC      fs/ext4/dev.o
367   CC      fs/ext4/ext4\_write.o
368   CC      fs/ext4/ext4\_journal.o
369   CC      fs/ext4/crc16.o
370   LD      fs/ext4/built-in.o
371   CC      fs/fat/fat\_write.o
372   LD      fs/fat/built-in.o
373   CC      fs/fs\_internal.o
374   LD      fs/built-in.o
375   CC      lib/efi_loader/efi\_image\_loader.o
376   CC      lib/efi_loader/efi\_boottime.o
377   CC      lib/efi_loader/efi\_runtime.o
378   CC      lib/efi_loader/efi\_console.o
379   CC      lib/efi_loader/efi\_memory.o
380   CC      lib/efi_loader/efi\_device\_path\_to\_text.o
381   CC      lib/efi_loader/efi\_device\_path.o
382   CC      lib/efi_loader/efi\_file.o
383   CC      lib/efi_loader/efi\_variable.o
384   CC      lib/efi_loader/efi\_bootmgr.o
385   CC      lib/efi_loader/efi\_watchdog.o
386   CC      lib/efi_loader/efi\_disk.o
387   CC      lib/efi_loader/efi\_net.o
388   CC      lib/efi_loader/efi\_smbios.o
389   LD      lib/efi_loader/built-in.o
390   CC      lib/efi_loader/helloworld.o
391   LD      lib/efi_loader/helloworld\_efi.so
392   OBJCOPY lib/efi_loader/helloworld.efi
393 rm lib/efi_loader/helloworld_efi.so lib/efi_loader/helloworld.o
394   LD      lib/efi_selftest/built-in.o
395   CC      lib/libfdt/fdt.o
396   CC      lib/libfdt/fdt\_wip.o
397   CC      lib/libfdt/fdt\_strerror.o
398   CC      lib/libfdt/fdt\_sw.o
399   CC      lib/libfdt/fdt\_empty\_tree.o
400   CC      lib/libfdt/fdt\_addresses.o
401   CC      lib/libfdt/fdt\_overlay.o
402   CC      lib/libfdt/fdt\_ro.o
403   CC      lib/libfdt/fdt\_rw.o
404   CC      lib/libfdt/fdt\_region.o
405   LD      lib/libfdt/built-in.o
406   CC      lib/lzo/lzo1x\_decompress.o
407   LD      lib/lzo/built-in.o
408   CC      lib/zlib/zlib.o
409   LD      lib/zlib/built-in.o
410   CC      lib/charset.o
411   CC      lib/crc7.o
412   CC      lib/crc8.o
413   CC      lib/crc16.o
414   CC      lib/fdtdec\_common.o
415   CC      lib/smbios.o
416   CC      lib/initcall.o
417   CC      lib/lmb.o
418   CC      lib/ldiv.o
419   CC      lib/md5.o
420   CC      lib/net\_utils.o
421   CC      lib/qsort.o
422   CC      lib/rc4.o
423   CC      lib/list\_sort.o
424   CC      lib/sha1.o
425   CC      lib/sha256.o
426   CC      lib/gunzip.o
427   CC      lib/hashtable.o
428   CC      lib/errno.o
429   CC      lib/display\_options.o
430   CC      lib/crc32.o
431   CC      lib/ctype.o
432   CC      lib/div64.o
433   CC      lib/hang.o
434   CC      lib/linux\_compat.o
435   CC      lib/linux\_string.o
436   CC      lib/membuff.o
437   CC      lib/slre.o
438   CC      lib/string.o
439   CC      lib/tables\_csum.o
440   CC      lib/time.o
441   CC      lib/uuid.o
442   CC      lib/rand.o
443   CC      lib/vsprintf.o
444   CC      lib/panic.o
445   CC      lib/strto.o
446   CC      lib/strmhz.o
447   LD      lib/built-in.o
448   CC      net/checksum.o
449   CC      net/arp.o
450   CC      net/bootp.o
451   CC      net/eth\_legacy.o
452   CC      net/eth\_common.o
453   CC      net/net.o
454   CC      net/nfs.o
455   CC      net/ping.o
456   CC      net/tftp.o
457   LD      net/built-in.o
458   LD      test/built-in.o
459   CC      test/dm/cmd\_dm.o
460   LD      test/dm/built-in.o
461   CC      examples/standalone/stubs.o
462   LD      examples/standalone/libstubs.o
463   CC      examples/standalone/hello\_world.o
464   LD      examples/standalone/hello\_world
465   OBJCOPY examples/standalone/hello\_world.srec
466   OBJCOPY examples/standalone/hello\_world.bin
467   LDS     u-boot.lds
468   LD      u-boot
469   OBJCOPY u-boot-nodtb.bin
470   COPY    u-boot.bin
471   MKIMAGE u-boot.img
472   OBJCOPY u-boot.srec
473   SYM     u-boot.sym
474 /bin/sh: 1: bc: not found
**475 CC spl/arch/arm/mach-omap2/am33xx/clock\_am33xx.o**476   CC      spl/arch/arm/mach-omap2/am33xx/clock.o
477   CC      spl/arch/arm/mach-omap2/am33xx/sys\_info.o
478   CC      spl/arch/arm/mach-omap2/am33xx/ddr.o
479   CC      spl/arch/arm/mach-omap2/am33xx/emif4.o
480   CC      spl/arch/arm/mach-omap2/am33xx/board.o
481   CC      spl/arch/arm/mach-omap2/am33xx/mux.o
482   CC      spl/arch/arm/mach-omap2/am33xx/prcm-regs.o
483   CC      spl/arch/arm/mach-omap2/am33xx/hw\_data.o
484   CC      spl/arch/arm/mach-omap2/am33xx/fdt.o
485   CC      spl/arch/arm/mach-omap2/am33xx/clk\_synthesizer.o
486   LD      spl/arch/arm/mach-omap2/am33xx/built-in.o
487   CC      spl/arch/arm/mach-omap2/reset.o
488   CC      spl/arch/arm/mach-omap2/timer.o
489   CC      spl/arch/arm/mach-omap2/utils.o
490   CC      spl/arch/arm/mach-omap2/sysinfo-common.o
491   CC      spl/arch/arm/mach-omap2/omap-cache.o
492   CC      spl/arch/arm/mach-omap2/boot-common.o
493   AS      spl/arch/arm/mach-omap2/lowlevel\_init.o
494   CC      spl/arch/arm/mach-omap2/mem-common.o
495   CC      spl/arch/arm/mach-omap2/fdt-common.o
496   LD      spl/arch/arm/mach-omap2/built-in.o
497   CC      spl/arch/arm/cpu/armv7/cache\_v7.o
498   AS      spl/arch/arm/cpu/armv7/cache\_v7\_asm.o
499   CC      spl/arch/arm/cpu/armv7/cpu.o
500   CC      spl/arch/arm/cpu/armv7/cp15.o
501   CC      spl/arch/arm/cpu/armv7/syslib.o
502   AS      spl/arch/arm/cpu/armv7/lowlevel\_init.o
503   LD      spl/arch/arm/cpu/armv7/built-in.o
504   AS      spl/arch/arm/cpu/armv7/start.o
505   LD      spl/arch/arm/cpu/built-in.o
506   AS      spl/arch/arm/lib/vectors.o
507   AS      spl/arch/arm/lib/crt0.o
508   AS      spl/arch/arm/lib/setjmp.o
509   CC      spl/arch/arm/lib/spl.o
510   CC      spl/arch/arm/lib/zimage.o
511   CC      spl/arch/arm/lib/bootm-fdt.o
512   AS      spl/arch/arm/lib/memset.o
513   AS      spl/arch/arm/lib/memcpy.o
514   CC      spl/arch/arm/lib/sections.o
515   CC      spl/arch/arm/lib/stack.o
516   CC      spl/arch/arm/lib/interrupts.o
517   CC      spl/arch/arm/lib/reset.o
518   CC      spl/arch/arm/lib/cache.o
519   CC      spl/arch/arm/lib/cache-cp15.o
520   CC      spl/arch/arm/lib/psci-dt.o
521   LD      spl/arch/arm/lib/built-in.o
522   AS      spl/arch/arm/lib/ashldi3.o
523   AS      spl/arch/arm/lib/ashrdi3.o
524   CC      spl/arch/arm/lib/div0.o
525   AS      spl/arch/arm/lib/div64.o
526   AS      spl/arch/arm/lib/lib1funcs.o
527   AS      spl/arch/arm/lib/lshrdi3.o
528   AS      spl/arch/arm/lib/muldi3.o
529   AS      spl/arch/arm/lib/uldivmod.o
530   AR      spl/arch/arm/lib/lib.a
531   CC      spl/arch/arm/lib/eabi\_compat.o
532   AS      spl/arch/arm/lib/crt0\_arm\_efi.o
533   CC      spl/arch/arm/lib/reloc\_arm\_efi.o
534   CC      spl/board/ti/am335x/mux.o
535   CC      spl/board/ti/am335x/board.o
536   LD      spl/board/ti/am335x/built-in.o
537   CC      spl/board/ti/common/board\_detect.o
538   LD      spl/board/ti/common/built-in.o
539   CC      spl/common/spl/spl.o
540   CC      spl/common/spl/spl\_ymodem.o
541   CC      spl/common/spl/spl\_mmc.o
542   CC      spl/common/spl/spl\_fat.o
543   CC      spl/common/spl/spl\_ext.o
544   LD      spl/common/spl/built-in.o
545   CC      spl/common/init/board\_init.o
546   LD      spl/common/init/built-in.o
547   CC      spl/common/xyzModem.o
548   CC      spl/common/fdt\_support.o
549   CC      spl/common/console.o
550   CC      spl/common/dlmalloc.o
551   CC      spl/common/malloc\_simple.o
552   CC      spl/common/image.o
553   CC      spl/common/image-android.o
554   CC      spl/common/image-fdt.o
555   CC      spl/common/memsize.o
556   CC      spl/common/stdio.o
557   CC      spl/common/cli.o
558   CC      spl/common/dfu.o
559   CC      spl/common/command.o
560   CC      spl/common/s\_record.o
561   LD      spl/common/built-in.o
562   CC      spl/cmd/nvedit.o
563   LD      spl/cmd/built-in.o
564   CC      spl/env/common.o
565   CC      spl/env/env.o
566   CC      spl/env/attr.o
567   CC      spl/env/flags.o
568   CC      spl/env/callback.o
569   CC      spl/env/mmc.o
570   LD      spl/env/built-in.o
571   CC      spl/lib/sha1.o
572   CC      spl/lib/sha256.o
573   CC      spl/lib/libfdt/fdt.o
574   CC      spl/lib/libfdt/fdt\_wip.o
575   CC      spl/lib/libfdt/fdt\_strerror.o
576   CC      spl/lib/libfdt/fdt\_sw.o
577   CC      spl/lib/libfdt/fdt\_empty\_tree.o
578   CC      spl/lib/libfdt/fdt\_addresses.o
579   CC      spl/lib/libfdt/fdt\_overlay.o
580   CC      spl/lib/libfdt/fdt\_ro.o
581   CC      spl/lib/libfdt/fdt\_rw.o
582   CC      spl/lib/libfdt/fdt\_region.o
583   LD      spl/lib/libfdt/built-in.o
584   CC      spl/lib/crc16.o
585   CC      spl/lib/hashtable.o
586   CC      spl/lib/errno.o
587   CC      spl/lib/display\_options.o
588   CC      spl/lib/crc32.o
589   CC      spl/lib/ctype.o
590   CC      spl/lib/div64.o
591   CC      spl/lib/hang.o
592   CC      spl/lib/linux\_compat.o
593   CC      spl/lib/linux\_string.o
594   CC      spl/lib/membuff.o
595   CC      spl/lib/slre.o
596   CC      spl/lib/string.o
597   CC      spl/lib/tables\_csum.o
598   CC      spl/lib/time.o
599   CC      spl/lib/uuid.o
600   CC      spl/lib/rand.o
601   CC      spl/lib/tiny-printf.o
602   CC      spl/lib/panic.o
603   CC      spl/lib/strto.o
604   LD      spl/lib/built-in.o
605   CC      spl/disk/part.o
606   CC      spl/disk/part\_dos.o
607   CC      spl/disk/part\_iso.o
608   CC      spl/disk/part\_efi.o
609   LD      spl/disk/built-in.o
610   CC      spl/drivers/block/blk\_legacy.o
611   LD      spl/drivers/block/built-in.o
612   CC      spl/drivers/core/device.o
613   CC      spl/drivers/core/fdtaddr.o
614   CC      spl/drivers/core/lists.o
615   CC      spl/drivers/core/root.o
616   CC      spl/drivers/core/uclass.o
617   CC      spl/drivers/core/util.o
618   CC      spl/drivers/core/dump.o
619   LD      spl/drivers/core/built-in.o
620   CC      spl/drivers/gpio/gpio-uclass.o
621   CC      spl/drivers/gpio/omap\_gpio.o
622   LD      spl/drivers/gpio/built-in.o
623   CC      spl/drivers/i2c/i2c\_core.o
624   CC      spl/drivers/i2c/omap24xx\_i2c.o
625   LD      spl/drivers/i2c/built-in.o
626   CC      spl/drivers/mmc/mmc.o
627   CC      spl/drivers/mmc/mmc\_legacy.o
628   CC      spl/drivers/mmc/omap\_hsmmc.o
629   LD      spl/drivers/mmc/built-in.o
630   LD      spl/drivers/power/built-in.o
631   CC      spl/drivers/power/pmic/pmic\_tps65217.o
632   CC      spl/drivers/power/pmic/pmic\_tps65910.o
633   LD      spl/drivers/power/pmic/built-in.o
634   LD      spl/drivers/power/regulator/built-in.o
635   CC      spl/drivers/serial/serial-uclass.o
636   CC      spl/drivers/serial/ns16550.o
637   LD      spl/drivers/serial/built-in.o
638   CC      spl/drivers/usb/musb-new/musb\_gadget.o
639   CC      spl/drivers/usb/musb-new/musb\_gadget\_ep0.o
640   CC      spl/drivers/usb/musb-new/musb\_core.o
641   CC      spl/drivers/usb/musb-new/musb\_uboot.o
642   CC      spl/drivers/usb/musb-new/musb\_host.o
643   CC      spl/drivers/usb/musb-new/musb\_dsps.o
644   LD      spl/drivers/usb/musb-new/built-in.o
645   CC      spl/drivers/watchdog/omap\_wdt.o
646   LD      spl/drivers/watchdog/built-in.o
647   LD      spl/drivers/built-in.o
648   LD      spl/dts/built-in.o
649   CC      spl/fs/ext4/ext4fs.o
650   CC      spl/fs/ext4/ext4\_common.o
651   CC      spl/fs/ext4/dev.o
652   CC      spl/fs/ext4/ext4\_write.o
653   CC      spl/fs/ext4/ext4\_journal.o
654   CC      spl/fs/ext4/crc16.o
655   LD      spl/fs/ext4/built-in.o
656   CC      spl/fs/fat/fat\_write.o
657   LD      spl/fs/fat/built-in.o
658   CC      spl/fs/fs\_internal.o
659   LD      spl/fs/built-in.o
660   LDS     spl/u-boot-spl.lds
661   LD      spl/u-boot-spl
662   OBJCOPY spl/u-boot-spl-nodtb.bin
663   COPY    spl/u-boot-spl.bin
664  MKIMAGE MLO
665  MKIMAGE MLO.byteswap
666   CHK     include/config.h
667   CFG     u-boot.cfg
668   CFGCHK  u-boot.cfg
```


 


以spl/打头的都是spl相关的编译，在输出的后半部分，大家可以看到 475行和125 行编译的是同一个C文件，一个是u-boot，一个是u-boot-spl


### SPL的其他配置项


由于现在u-boot同时使用Kconfig和老的头文件方式进行配置，下面很多配置在BBB中仍然是在头文件中配置的。


* CONFIG\_SPL\_TEXT\_BASE    SPL的入口地址
* CONFIG\_SPL\_LDSCRIPT       SPL的链接脚本


除了板子特有的代码外，如果要使用u-boot通用的库或驱动，还需要定义 CONFIG\_SPL\_XXX\_SUPPORT，当前支持如下配置


* CONFIG\_SPL\_LIBCOMMON\_SUPPORT (common/libcommon.o)
* CONFIG\_SPL\_LIBDISK\_SUPPORT (disk/libdisk.o)
* CONFIG\_SPL\_I2C\_SUPPORT (drivers/i2c/libi2c.o)
* CONFIG\_SPL\_GPIO\_SUPPORT (drivers/gpio/libgpio.o)
* CONFIG\_SPL\_MMC\_SUPPORT (drivers/mmc/libmmc.o)
* CONFIG\_SPL\_SERIAL\_SUPPORT (drivers/serial/libserial.o)
* CONFIG\_SPL\_SPI\_FLASH\_SUPPORT (drivers/mtd/spi/libspi\_flash.o)
* CONFIG\_SPL\_SPI\_SUPPORT (drivers/spi/libspi.o)
* CONFIG\_SPL\_FAT\_SUPPORT (fs/fat/libfat.o)
* CONFIG\_SPL\_EXT\_SUPPORT
* CONFIG\_SPL\_LIBGENERIC\_SUPPORT (lib/libgeneric.o)
* CONFIG\_SPL\_POWER\_SUPPORT (drivers/power/libpower.o)
* CONFIG\_SPL\_NAND\_SUPPORT (drivers/mtd/nand/libnand.o)
* CONFIG\_SPL\_DRIVERS\_MISC\_SUPPORT (drivers/misc)
* CONFIG\_SPL\_DMA\_SUPPORT (drivers/dma/libdma.o)
* CONFIG\_SPL\_POST\_MEM\_SUPPORT (post/drivers/memory.o)
* CONFIG\_SPL\_NAND\_LOAD (drivers/mtd/nand/nand\_spl\_load.o)
* CONFIG\_SPL\_SPI\_LOAD (drivers/mtd/spi/spi\_spl\_load.o)
* CONFIG\_SPL\_RAM\_DEVICE (common/spl/spl.c)
* CONFIG\_SPL\_WATCHDOG\_SUPPORT (drivers/watchdog/libwatchdog.o)


SPL 编译分析
--------


u-boot在最近几年引入了Linux内核的Kconfig方式，主要介绍在源代码的 doc/README.kconfig 中，u-boot的目标是把所有配置都放到Kconfig中去，但是目前仍然有大量配置是写在头文件中的。在目前的情况下，C文件中可使用的配置文件是


* include/generated/autoconf.h     (generated by Kconfig for Normal)
* include/configs/.h        (exists for all boards)  例如：bbb的target为 am335x\_evm


Makefile中可使用的配置文件是


* include/config/auto.conf         (generated by Kconfig)
* include/autoconf.mk              (generated by the old config for Normal)
* spl/include/autoconfig.mk        (generated by the old config for SPL)
* tpl/include/autoconfig.mk        (generated by the old config for TPL) （注意：TPL跟SPL差不多，是第三级程序加载器，也是一个精简版的u-boot，很少用到，参见 doc/README.TPL）


### 板级相关的代码都在哪里


* CONFIG\_SYS\_CPU="cpu"               将会编译    arch//cpu/
* CONFIG\_SYS\_SOC="soc"               将会编译    arch//cpu//
* CONFIG\_SYS\_VENDOR="vendor"   将会编译    board//common/\*  和 board///\*
* CONFIG\_SYS\_BOARD="board"       将会编译    board//\* （或 board///\* 如果 CONFIG\_SYS\_VENDOR 也定义了）
* CONFIG\_SYS\_CONFIG\_NAME="target"  将包含    include/configs/.h
* 在 arch//Kconfig 或 arch//\*/Kconfig 中应 source 板级相关的 Kconfig 文件来编译更多的文件


对于BBB开发板来说，可以查看 am335x\_boneblack\_defconfig：


* = arm
* = armv7
* = am33xx
* = ti
* = am335x
* = am335x\_evm


对照上面，BBB板级相关的代码包括以下文件夹中


* arch/arm/cpu/armv7
* arch/arm/cpu/armv7/am33xx（无此文件夹）取而代之的是 arch/arm/mach-omap2/am33xx ，这是因为 arch/arm/mach-omap2/Kconfig 中 source 了 arch/arm/mach-omap2/am33xx/Kconfig 和 board/ti/am335x/Kconfig
* board/ti/common
* board/ti/am335x
* 配置的头文件为 includes/configs/am335x\_evm.h


对照下上面实际编译输出，基本上差不多，除了多了一个 arch/arm/mach-omap2/\* ，参见下文 （machine-$(CONFIG\_ARCH\_OMAP2PLUS)）。


### Makefile简述


u-boot根目录中的Makefile包含如下语句




```
ALL-$(CONFIG_SPL) += spl/u-boot-spl.bin

spl/u-boot-spl.bin: spl/u-boot-spl
	@:
spl/u-boot-spl: tools prepare \
		$(if $(CONFIG_OF_SEPARATE)$(CONFIG_OF_EMBED)$(CONFIG_SPL_OF_PLATDATA),dts/dt.dtb) \
		$(if $(CONFIG_OF_SEPARATE)$(CONFIG_OF_EMBED)$(CONFIG_TPL_OF_PLATDATA),dts/dt.dtb)
	**$(Q)$(MAKE) obj=spl -f $(srctree)/scripts/Makefile.spl all**
```


 而Makefile.spl首先包含了源代码树根目录的 config.mk，这个文件根据配置定义了ARCH、CPU、SOC、BOARD、VENDOR、CPUDIR、BOARDDIR等板级相关的变量，并包含了 ARCH、CPU、SOC、BOARD等板级相关的文件夹中的config.mk，以便添加或覆盖默认配置。配置确定后，Makefile.spl中定义了spl的各依赖，其中libs-y是spl需要编译的库（即子文件夹）




```
u-boot-spl-dirs    := $(patsubst %/,%,$(filter %/, $(libs-y)))  

u-boot-spl-init := $(head-y)  
**# 注意：最后libs-y被转换为对应目录的 built-in.o，参见下文中对 Makefile.build 的分析**  
libs-y := $(patsubst %/, %/built-in.o, $(libs-y))
u-boot-spl-main := $(libs-y)

$(obj)/$(SPL_BIN): $(u-boot-spl-platdata) $(u-boot-spl-init) \
        $(u-boot-spl-main) $(obj)/u-boot-spl.lds FORCE
    $(call if_changed,u-boot-spl)  
  
# May be overridden by arch/$(ARCH)/config.mk  
**# 这是 $(call if\_changed,u-boot-spl) 实际执行的命令**  
quiet_cmd_u-boot-spl ?= LD      $@  
      cmd_u-boot-spl ?= (cd $(obj) && $(LD) $(LDFLAGS) $(LDFLAGS_$(@F)) \  
               $(patsubst $(obj)/%,%,$(u-boot-spl-init)) --start-group \  
               $(patsubst $(obj)/%,%,$(u-boot-spl-main))  \  
               $(patsubst $(obj)/%,%,$(u-boot-spl-platdata)) \  
               --end-group \  
               $(PLATFORM_LIBS) -Map $(SPL_BIN).map -o $(SPL_BIN))
```


 其中 link script的查找方法是




```
ifeq ($(wildcard $(LDSCRIPT)),)
    LDSCRIPT := $(srctree)/board/$(BOARDDIR)/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
    LDSCRIPT := $(srctree)/$(CPUDIR)/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
    LDSCRIPT := $(srctree)/arch/$(ARCH)/cpu/u-boot-spl.lds
endif
ifeq ($(wildcard $(LDSCRIPT)),)
$(error could not find linker script)
endif
```


 即，依次查找 BOARD、CPU、ARCH目录，第一个发现的u-boot-spl.lds文件。


大部分libs都是在Makefile.spl中直接定义的，如下




```
**libs-y += $(if $(BOARDDIR),board/$(BOARDDIR)/)
libs-$(HAVE\_VENDOR\_COMMON\_LIB) += board/$(VENDOR)/common/**

libs-$(CONFIG_SPL_FRAMEWORK) += common/spl/
libs-y += common/init/

# Special handling for a few options which support SPL/TPL
ifeq ($(CONFIG_TPL_BUILD),y)
libs-$(CONFIG_TPL_LIBCOMMON_SUPPORT) += common/ cmd/ env/
libs-$(CONFIG_TPL_LIBGENERIC_SUPPORT) += lib/
else
libs-$(CONFIG_SPL_LIBCOMMON_SUPPORT) += common/ cmd/ env/
libs-$(CONFIG_SPL_LIBGENERIC_SUPPORT) += lib/
endif

libs-$(CONFIG_SPL_LIBDISK_SUPPORT) += disk/
libs-y += drivers/
libs-$(CONFIG_SPL_USB_GADGET_SUPPORT) += drivers/usb/dwc3/
libs-y += dts/
libs-y += fs/
libs-$(CONFIG_SPL_POST_MEM_SUPPORT) += post/drivers/
libs-$(CONFIG_SPL_NET_SUPPORT) += net/
```


 但是有一部分是ARCH和CPU相关的是在 arch/arm/Makefile中定义的




```
# head-y 也是在这个文件中定义的  
**head-y := arch/arm/cpu/$(CPU)/start.o**

# 注意：这就是为什么BBB会编译 mach-omap2 文件夹了
**machine-$(CONFIG\_ARCH\_OMAP2PLUS) += omap2**

machdirs := $(patsubst %,arch/arm/mach-%/,$(machine-y))

libs-y += $(machdirs)

libs-y += arch/arm/cpu/$(CPU)/
libs-y += arch/arm/cpu/
libs-y += arch/arm/lib/
```


 最后要说明下，这些lib-y的子文件夹是如何编译的




```
$(u-boot-spl-dirs): $(u-boot-spl-platdata)
    $(Q)$(MAKE) $(build)=$@
```


 上面这句话非常简洁，但包含的内容却很多，注意是 $(build)=$@ 而不是 build=$@，build是kbuild.include中定义的一个变量，其值是




```
build := -f $(srctree)/scripts/Makefile.build obj
```


对u-boot-spl-dirs中的每一个子目录，展开了之后是




```
$(Q)$(MAKE) -f $(srctree)/scripts/Makefile.build obj=目录
```


Makefile.build是Kbuild体系的核心文件，文件夹的编译都是通过这个文件来完成的，理解这个文件我认为需要知道几个要点：


* 这个文件包含了 Kbuild.include 和 Makefile.lib，这两个文件定义了很多实用函数、变量和通用的规则，一般位置找不到的东西都在这里
* 这个文件包含了 **obj=目录** 中的 Kbuild或Makefile文件，这样我们再写某个目录中的Kbuild或Makefile时就比较简单了，我们只需要定义
	+ obj-y：本目录中的源文件
	+ subdir-y：需要递归编译的子目录
	+ extra-y：额外要执行的编译目标
* 一个目录中的所有源文件及子目录的编译结果最终会LD成一个 built-in.o
* subdir-y中的目录又会使用 Makefile.build 递归编译




```
**# 至少有lib-y lib-m lib- 其中一个时才生成 lib.a**  
ifneq ($(strip $(lib-y) $(lib-m) $(lib-)),)
lib-target := $(obj)/lib.a
endif
  
**# 至少有obj-y obj-m obj- subdir-m lib-y lib-m lib- 其中一个时才生成 built-in.o**
ifneq ($(strip $(obj-y) $(obj-m) $(obj-) $(subdir-m) $(lib-target)),)
builtin-target := $(obj)/built-in.o
endif

modorder-target := $(obj)/modules.order

# We keep a list of all modules in $(MODVERDIR)
  
**# KBUILD\_BUILTIN 在顶层Makefile赋为了1，u-boot中不需要module，所以**KBUILD_MODULES为0
__build: $(if $(KBUILD_BUILTIN),$(builtin-target) $(lib-target) $(extra-y)) \
	 $(if $(KBUILD_MODULES),$(obj-m) $(modorder-target)) \
	 $(subdir-ym) $(always)
	@:
```


 


 以上这个部分我认为比较重要，未指定目标时，Makefile.build的默认目标是 \_\_build，这个\_\_build依赖


* built-in.o
* lib.a
* extra-y
* subdir-ym：这是子目录


SPL执行分析
-------


**（源代码根目录下 README 文件中有一节Board Initialisation Flow专门讲了启动流程）**


SPL编译分析结束后，我们就知道了SPL由哪些源文件组成，分析起来要方便多了。首先当然是从head-y开始，即 arch/arm/cpu/armv7/start.S


* save\_boot\_params：通过weak方式调用了 arch/arm/omach-omap2/lowlevel\_init.S中的同名函数（存了r0到RAM）（暂时还没有搞清r0是啥）
* 关中断，进入SVC模式
* cpu\_init\_cp15：Cache、MMU、TLB操作等（该清清，该禁禁，I-Cache一般是打开）
* cpu\_init\_crit：调用 lowlevel\_init，BBB板级未定义lowlevel\_init，于是就调用了 arch/arm/cpu/armv7/lowlevel\_init.S 中的 lowlevel\_init函数，对SPL来说基本没干啥
* \_main：这个函数是重头戏，定义在 arch/arm/lib/crt0.S中，这个函数大概流程是这样的
	+ 初始化GD
	+ board\_init\_f
	+ 正常的u-boot此处会进行relocate，SPL不会，但是SPL可能会改变栈和GD的位置（参见 CONFIG\_SPL\_FRAMEWORK，common/spl/spl.c中的spl\_relocate\_stack\_gd函数）
	+ 清零bss区
	+ board\_init\_r


### GD（struct global\_data）


为了简单起见，u-boot有一个全局的结构体数据，称为GD（指针为gd），很多数据都会丢到这个结构体里。





```
typedef struct global\_data {
 //...
 struct arch\_global\_data arch;
 // ...
} gd_t;
```


 



 这个结构体的定义在源代码目录的 include/asm-generic/global\_data.h，这个文件应被arch的同名文件包含，以便可以定义不同的 struct arch\_global\_data 结构体，对于arm来说是 arch/arm/include/asm/global\_data.h ，使用时 采用 #include的方式就可以包含不同arch的同名文件了。


这么重要的结构体是在\_main的一开始就被初始化为0了，代码在 common/init/board\_init.c中的 board\_init\_f\_alloc\_reserve/board\_init\_f\_init\_reserve 两个函数里，通过在栈顶预留内存来达到给GD开辟空间的目的（因栈是向下增长的，所有不会和栈冲突）。


注：在arm架构中，gd的指针被赋给了r9寄存器，在需要使用gd地方，通过全局声明r9，使编译器不会使用r9寄存器作为临时变量，加快gd的访问速度。






```
<https://gcc.gnu.org/onlinedocs/gcc/Global-Register-Variables.htm>

在 **arch/arm/config.mk** 中

PLATFORM_RELFLAGS += -ffunction-sections -fdata-sections \
    -fno-common **-ffixed-r9**

注意： -ffixed-r9 就是上述链接中的 -ffixed-reg ，所有函数都不可以使用r9寄存器
```


 



 


### board\_init\_f


GD初始化完成之后，就会调用 board\_init\_f，这个函数的目的是使board\_init\_r可以运行，主要是初始化DRAM和console串口，要注意下这个函数的运行环境：


* GD指针可使用，因为GD已经初始化
* 栈是在SRAM中的
* BSS还未清零，因为所有全局或静态变量都是不可以访问的，只能访问局部变量和GD


虽说这个函数以board打头，但是相似的板子可能会定义一个初始化的流程，在流程中大量使用虚函数和配置来抽象不同的板子。BBB的board\_init\_f是在 arch/arm/march-omap2/am33xx/board.c 中




```
#ifdef CONFIG\_SPL\_BUILD
void board_init_f(ulong dummy)
{
 hw\_data\_init();
 early\_system\_init();
 board\_early\_init\_f();
 sdram\_init();
 /\* dram\_init must store complete ramsize in gd->ram\_size \*/
 gd->ram_size = get\_ram\_size(
 (void *)CONFIG\_SYS\_SDRAM\_BASE,
 CONFIG\_MAX\_RAM\_BANK\_SIZE);
}
#endif
```


###  board\_init\_r


 这个函数用来实现主要的代码逻辑，对于SPL来说，主要函数都是在 common/spl/spl.c 中实现了，具体大家直接看代码就是了


falcon
------


参见 doc/README.falcon，大概就是SPL=>u-boot=>Linux这个流程有点慢，falcon的功能就是不启动u-boot，直接启动Linux，相关的配置项是 CONFIG\_SPL\_OS\_BOOT




---


 


这篇文章就写这么多了，把这些搞明白了，u-boot-spl就大体上理解了，更详细的还得大家自己回去读代码了，共勉！


