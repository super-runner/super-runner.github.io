---
layout: post
title:  uboot patch and analysis
date: 2020-10-27
categories: linux
tags:  os
author: Jason Chi
---
* content
{:toc}

```
1、下载、建立source insight工程、编译、烧写、如果无运行分析原因
tar xjf u-boot-2012.04.01.tar.bz2
cd u-boot-2012.04.01
make smdk2410_config
make


2. 分析u-boot: 通过链接命令分析组成文件、阅读代码分析启动过程

a. 初始化硬件：关看门狗、设置时钟、设置SDRAM、初始化NAND FLASH
b. 如果bootloader比较大，要把它重定位到SDRAM
c. 把内核从NAND FLASH读到SDRAM
d. 设置"要传给内核的参数"
e. 跳转执行内核

2.1 set the cpu to SVC32 mode
2.2 turn off the watchdog
2.3 mask all IRQs by setting all bits in the INTMR
2.4 设置时钟比例
2.5 设置内存控制器
2.6 设置栈，调用C函数board_init_f
2.7 调用函数数组init_sequence里的各个函数
2.7.1 board_early_init_f : 设置系统时钟、设置GPIO
......
2.8 重定位代码:
2.8.1 从NOR FLASH把代码复制到SDRAM
2.8.2 程序的链接地址是0，访问全局变量、静态变量、调用函数时是使"基于0地址编译得到的地址"
      现在把程序复制到了SDRAM
      需要修改代码，把"基于0地址编译得到的地址"改为新地址
2.8.3 程序里有些地址在链接时不能确定，要到运行前才能确定：fixabs
2.9 clear_bss
2.10 调用C函数board_init_r：第2阶段的代码




可以修改配置定义CONFIG_S3C2440


3. 修改U-BOOT代码
3.1 建一个单板
cd board/samsung/
cp smdk2410 smdk2440 -rf
cd ../../include/configs/
cp smdk2410.h smdk2440.h
修改boards.cfg:
仿照
smdk2410                     arm         arm920t     -                   samsung        s3c24x0
添加：
smdk2440                     arm         arm920t     -                   samsung        s3c24x0

3.2 烧写看结果
3.3 调试： 
a. 阅读代码发现不足：UBOOT里先以60MHZ的时钟计算参数来设置内存控制器，但是MPLL还未设置
   处理措施: 把MPLL的设置放到start.S里，取消board_early_init_f里对MPLL的设置
   
   编译出来的uboot非常大，可以先烧写主光盘里的u-boot.bin到nor，然后用这个uboot来烧写新的uboot

3.4 乱码，查看串口波特率的设置，发现在get_HCLK里没有定义CONFIG_S3C2440
    处理措施：include/configs/smdk2440.h: 去掉CONFIG_S3C2410
                                          #define CONFIG_S3C2440
                                          //#define CONFIG_CMD_NAND
3.5 修改UBOOT支持NAND启动
    原来的代码在链接时加了"-pie"选项, 使得u-boot.bin里多了"*(.rel*)", "*(.dynsym)"
    使得程序非常大，不利于从NAND启动(重定位之前的启动代码应该少于4K)
3.5.1 去掉 "-pie"选项
      arch/arm/config.mk:75:LDFLAGS_u-boot += -pie 去掉这行
      
3.5.2 参考"毕业班第1课"的start.S, init.c来修改代码
      把init.c放入board/samsung/smdk2440目录, 修改Makefile
      修改CONFIG_SYS_TEXT_BASE为0x33f80000
      修改start.S
3.5.3 修改board_init_f, 把relocate_code去掉
3.5.4 修改链接脚本: 把start.S, init.c, lowlevel.S等文件放在最前面
      ./arch/arm/cpu/u-boot.lds：
      
      board/samsung/smdk2440/libsmdk2440.o

3.6 修改UBOOT支持NOR FLASH
  drivers\mtd\jedec_flash.c 加上新的型号
  #define CONFIG_SYS_MAX_FLASH_SECT	(128)

  修复了重定时留下来的BUG：SP要重新设置


3.7 修改UBOOT支持NAND FLASH
    修改：include/configs/smdk2440.h: #define CONFIG_CMD_NAND
    
    把drivers\mtd\nand\s3c2410_nand.c复制为s3c2440_nand.c
                                          

分析过程：
nand_init
	nand_init_chip
		board_nand_init
			设置nand_chip结构体, 提供底层的操作函数
		nand_scan
			nand_scan_ident
				nand_set_defaults
					chip->select_chip = nand_select_chip;
					chip->cmdfunc = nand_command;
					chip->read_byte = busw ? nand_read_byte16 : nand_read_byte;
					
				nand_get_flash_type
					chip->select_chip
					chip->cmdfunc(mtd, NAND_CMD_RESET, -1, -1);
							nand_command()  // 即可以用来发命令，也可以用来发列地址(页内地址)、行地址(哪一页)
								chip->cmd_ctrl
										s3c2440_hwcontrol
								
					chip->cmdfunc(mtd, NAND_CMD_READID, 0x00, -1);
					*maf_id = chip->read_byte(mtd);
					*dev_id = chip->read_byte(mtd);


3.8 修改UBOOT支持DM9000网卡
	eth_initialize
		board_eth_init
			cs8900_initialize

*** ERROR: `ethaddr' not set
set ipaddr 192.168.1.17
set ethaddr 00:0c:29:4d:e4:f4
set serverip 192.168.1.3



4. 易用性修裁剪及制作补丁					

内核打印出来的分区信息
0x00000000-0x00040000 : "bootloader"
0x00040000-0x00060000 : "params"
0x00060000-0x00260000 : "kernel"
0x00260000-0x10000000 : "root"

nand erase 60000 200000
nand write 30000000 60000 200000

tftp 30000000 uImage
nand erase.part kernel
nand write 30000000 kernel

烧写JFFS2
tftp 30000000 fs_mini_mdev.jffs2
nand erase.part rootfs
nand write.jffs2 30000000 0x00260000 5b89a8

set bootargs console=ttySAC0 root=/dev/mtdblock3 rootfstype=jffs2

烧写YAFFS
tftp 30000000 fs_mini_mdev.yaffs2
nand erase.part rootfs
nand write.yaffs 30000000 260000  889bc0

更新UBOOT:
tftp 30000000 u-boot_new.bin; protect off all; erase 0 3ffff; cp.b 30000000 0 40000


制作补丁：
diff -urN u-boot-2012.04.01 u-boot-2012.04.01_100ask > u-boot-2012.04.01_100ask.patch

分析"重定位之修改代码为新地址":
#ifndef CONFIG_SPL_BUILD
	/*
	 * fix .rel.dyn relocations
	 */
	ldr	r0, _TEXT_BASE		/* r0 <- Text base */
	// r0=0, 代码基地址
	
	sub	r9, r6, r0		/* r9 <- relocation offset */
	// r9 = r6-r0 = 0x33f41000 - 0 = 0x33f41000
	
	ldr	r10, _dynsym_start_ofs	/* r10 <- sym table ofs */
	// r10 = 00073608
	
	add	r10, r10, r0		/* r10 <- sym table in FLASH */
	// r10 = 00073608 + 0 = 00073608
	
	ldr	r2, _rel_dyn_start_ofs	/* r2 <- rel dyn start ofs */
	// r2=0006b568
	
	add	r2, r2, r0		/* r2 <- rel dyn start in FLASH */
	// r2=r2+r0=0006b568
	
	ldr	r3, _rel_dyn_end_ofs	/* r3 <- rel dyn end ofs */
	// r3=00073608
	
	add	r3, r3, r0		/* r3 <- rel dyn end in FLASH */
	// r3=r3+r0=00073608
	
fixloop:
	ldr	r0, [r2]		/* r0 <- location to fix up, IN FLASH! */
	1. r0=[0006b568]=00000020
	
	add	r0, r0, r9		/* r0 <- location to fix up in RAM */
	1. r0=r0+r9=00000020 + 0x33f41000 = 0x33f41020
	
	ldr	r1, [r2, #4]
	1. r1=[0006b568+4]=00000017
	
	and	r7, r1, #0xff
	1. r7=r1&0xff=00000017
	
	cmp	r7, #23			/* relative fixup? */
	1. r7 == 23(0x17)
	
	beq	fixrel
	cmp	r7, #2			/* absolute fixup? */
	
	beq	fixabs
	/* ignore unknown type of fixup */
	b	fixnext
fixabs:
	/* absolute fix: set location to (offset) symbol value */
	mov	r1, r1, LSR #4		/* r1 <- symbol index in .dynsym */
	
	add	r1, r10, r1		/* r1 <- address of symbol in table */
	
	ldr	r1, [r1, #4]		/* r1 <- symbol value */
	
	add	r1, r1, r9		/* r1 <- relocated sym addr */
	
	b	fixnext
fixrel:
	/* relative fix: increase location by offset */
	ldr	r1, [r0]
	1. r1=[00000020]=000001e0
	
	add	r1, r1, r9
	1. r1=r1+r9=000001e0 + 0x33f41000 = 33F411E0
	
fixnext:
	str	r1, [r0]
	1. [0x33f41020] = 33F411E0
	
	add	r2, r2, #8		/* each rel.dyn entry is 8 bytes */
	1. r2=r2+8=0006b568+8=6B570
	
	cmp	r2, r3
	1. 
	
	blo	fixloop
#endif

#### 补丁文件

diff -urN u-boot-2012.04.01/arch/arm/config.mk u-boot-2012.04.01_100ask/arch/arm/config.mk
--- u-boot-2012.04.01/arch/arm/config.mk	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/arch/arm/config.mk	2012-05-26 03:25:37.251936332 +0800
@@ -72,5 +72,5 @@
 
 # needed for relocation
 ifndef CONFIG_NAND_SPL
-LDFLAGS_u-boot += -pie
+#LDFLAGS_u-boot += -pie
 endif
diff -urN u-boot-2012.04.01/arch/arm/cpu/arm920t/start.S u-boot-2012.04.01_100ask/arch/arm/cpu/arm920t/start.S
--- u-boot-2012.04.01/arch/arm/cpu/arm920t/start.S	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/arch/arm/cpu/arm920t/start.S	2012-06-15 14:23:50.923458199 +0800
@@ -110,6 +110,10 @@
 IRQ_STACK_START_IN:
 	.word	0x0badc0de
 
+.globl base_sp
+base_sp:
+	.long 0
+
 /*
  * the actual start code
  */
@@ -167,11 +171,31 @@
 	str	r1, [r0]
 # endif
 
-	/* FCLK:HCLK:PCLK = 1:2:4 */
-	/* default FCLK is 120 MHz ! */
-	ldr	r0, =CLKDIVN
-	mov	r1, #3
-	str	r1, [r0]
+
+/* 2. 设置时钟 */
+	ldr r0, =0x4c000014
+	//	mov r1, #0x03;			  // FCLK:HCLK:PCLK=1:2:4, HDIVN=1,PDIVN=1
+	mov r1, #0x05;			  // FCLK:HCLK:PCLK=1:4:8
+	str r1, [r0]
+
+	/* 如果HDIVN非0，CPU的总线模式应该从“fast bus mode”变为“asynchronous bus mode” */
+	mrc	p15, 0, r1, c1, c0, 0		/* 读出控制寄存器 */ 
+	orr	r1, r1, #0xc0000000			/* 设置为“asynchronous bus mode” */
+	mcr	p15, 0, r1, c1, c0, 0		/* 写入控制寄存器 */
+
+#define S3C2440_MPLL_400MHZ     ((0x5c<<12)|(0x01<<4)|(0x01))
+
+	/* MPLLCON = S3C2440_MPLL_200MHZ */
+	ldr r0, =0x4c000004
+	ldr r1, =S3C2440_MPLL_400MHZ
+	str r1, [r0]
+
+	/* 启动ICACHE */
+	mrc p15, 0, r0, c1, c0, 0	@ read control reg
+	orr r0, r0, #(1<<12)
+	mcr	p15, 0, r0, c1, c0, 0   @ write it back
+
+
 #endif	/* CONFIG_S3C24X0 */
 
 	/*
@@ -182,103 +206,32 @@
 	bl	cpu_init_crit
 #endif
 
+	ldr sp, =(CONFIG_SYS_INIT_SP_ADDR)	/* sp = 30000f80 */
+	bic sp, sp, #7 /* 8-byte alignment for ABI compliance */
+
+	bl nand_init_ll
+	mov r0, #0
+	//ldr r1, =_start
+	ldr r1, _TEXT_BASE
+	
+	ldr r2, _bss_start_ofs
+	
+	bl copy_code_to_sdram
+	bl clear_bss
+
+	ldr pc, =call_board_init_f
+
 /* Set stackpointer in internal RAM to call board_init_f */
 call_board_init_f:
-	ldr	sp, =(CONFIG_SYS_INIT_SP_ADDR)
-	bic	sp, sp, #7 /* 8-byte alignment for ABI compliance */
 	ldr	r0,=0x00000000
 	bl	board_init_f
 
-/*------------------------------------------------------------------------------*/
-
-/*
- * void relocate_code (addr_sp, gd, addr_moni)
- *
- * This "function" does not return, instead it continues in RAM
- * after relocating the monitor code.
- *
- */
-	.globl	relocate_code
-relocate_code:
-	mov	r4, r0	/* save addr_sp */
-	mov	r5, r1	/* save addr of gd */
-	mov	r6, r2	/* save addr of destination */
-
-	/* Set up the stack						    */
-stack_setup:
-	mov	sp, r4
-
-	adr	r0, _start
-	cmp	r0, r6
-	beq	clear_bss		/* skip relocation */
-	mov	r1, r6			/* r1 <- scratch for copy_loop */
-	ldr	r3, _bss_start_ofs
-	add	r2, r0, r3		/* r2 <- source end address	    */
-
-copy_loop:
-	ldmia	r0!, {r9-r10}		/* copy from source address [r0]    */
-	stmia	r1!, {r9-r10}		/* copy to   target address [r1]    */
-	cmp	r0, r2			/* until source end address [r2]    */
-	blo	copy_loop
-
-#ifndef CONFIG_SPL_BUILD
-	/*
-	 * fix .rel.dyn relocations
-	 */
-	ldr	r0, _TEXT_BASE		/* r0 <- Text base */
-	sub	r9, r6, r0		/* r9 <- relocation offset */
-	ldr	r10, _dynsym_start_ofs	/* r10 <- sym table ofs */
-	add	r10, r10, r0		/* r10 <- sym table in FLASH */
-	ldr	r2, _rel_dyn_start_ofs	/* r2 <- rel dyn start ofs */
-	add	r2, r2, r0		/* r2 <- rel dyn start in FLASH */
-	ldr	r3, _rel_dyn_end_ofs	/* r3 <- rel dyn end ofs */
-	add	r3, r3, r0		/* r3 <- rel dyn end in FLASH */
-fixloop:
-	ldr	r0, [r2]		/* r0 <- location to fix up, IN FLASH! */
-	add	r0, r0, r9		/* r0 <- location to fix up in RAM */
-	ldr	r1, [r2, #4]
-	and	r7, r1, #0xff
-	cmp	r7, #23			/* relative fixup? */
-	beq	fixrel
-	cmp	r7, #2			/* absolute fixup? */
-	beq	fixabs
-	/* ignore unknown type of fixup */
-	b	fixnext
-fixabs:
-	/* absolute fix: set location to (offset) symbol value */
-	mov	r1, r1, LSR #4		/* r1 <- symbol index in .dynsym */
-	add	r1, r10, r1		/* r1 <- address of symbol in table */
-	ldr	r1, [r1, #4]		/* r1 <- symbol value */
-	add	r1, r1, r9		/* r1 <- relocated sym addr */
-	b	fixnext
-fixrel:
-	/* relative fix: increase location by offset */
-	ldr	r1, [r0]
-	add	r1, r1, r9
-fixnext:
-	str	r1, [r0]
-	add	r2, r2, #8		/* each rel.dyn entry is 8 bytes */
-	cmp	r2, r3
-	blo	fixloop
-#endif
-
-clear_bss:
-#ifndef CONFIG_SPL_BUILD
-	ldr	r0, _bss_start_ofs
-	ldr	r1, _bss_end_ofs
-	mov	r4, r6			/* reloc addr */
-	add	r0, r0, r4
-	add	r1, r1, r4
-	mov	r2, #0x00000000		/* clear			    */
-
-clbss_l:str	r2, [r0]		/* clear loop...		    */
-	add	r0, r0, #4
-	cmp	r0, r1
-	bne	clbss_l
+	/* unsigned int的值存在r0里, 正好给board_init_r */
+	ldr r1, _TEXT_BASE
+	ldr sp, base_sp 			/* 重新设置栈 */
 
-	bl coloured_LED_init
-	bl red_led_on
-#endif
+	/* 调用第2阶段的代码 */
+	bl board_init_r
 
 /*
  * We are done. Do not return, instead branch to second part of board
diff -urN u-boot-2012.04.01/arch/arm/cpu/u-boot.lds u-boot-2012.04.01_100ask/arch/arm/cpu/u-boot.lds
--- u-boot-2012.04.01/arch/arm/cpu/u-boot.lds	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/arch/arm/cpu/u-boot.lds	2012-05-26 03:26:29.323571911 +0800
@@ -35,6 +35,7 @@
 	{
 		__image_copy_start = .;
 		CPUDIR/start.o (.text)
+                board/samsung/smdk2440/libsmdk2440.o (.text)
 		*(.text)
 	}
 
diff -urN u-boot-2012.04.01/arch/arm/lib/board.c u-boot-2012.04.01_100ask/arch/arm/lib/board.c
--- u-boot-2012.04.01/arch/arm/lib/board.c	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/arch/arm/lib/board.c	2012-06-16 09:56:16.529637486 +0800
@@ -256,12 +256,14 @@
 	NULL,
 };
 
-void board_init_f(ulong bootflag)
+unsigned int board_init_f(ulong bootflag)
 {
 	bd_t *bd;
 	init_fnc_t **init_fnc_ptr;
 	gd_t *id;
 	ulong addr, addr_sp;
+	extern ulong base_sp;
+
 #ifdef CONFIG_PRAM
 	ulong reg;
 #endif
@@ -369,8 +371,9 @@
 	 * reserve memory for U-Boot code, data & bss
 	 * round down to next 4 kB limit
 	 */
-	addr -= gd->mon_len;
-	addr &= ~(4096 - 1);
+	//addr -= gd->mon_len;
+	//addr &= ~(4096 - 1);
+	addr = CONFIG_SYS_TEXT_BASE;   /* addr = _TEXT_BASE */
 
 	debug("Reserving %ldk for U-Boot at: %08lx\n", gd->mon_len >> 10, addr);
 
@@ -435,7 +438,10 @@
 	debug("relocation Offset is: %08lx\n", gd->reloc_off);
 	memcpy(id, (void *)gd, sizeof(gd_t));
 
-	relocate_code(addr_sp, id, addr);
+	base_sp = addr_sp;
+
+	//relocate_code(addr_sp, id, addr);
+	return (unsigned int)id;
 
 	/* NOTREACHED - relocate_code() does not return */
 }
@@ -524,8 +530,9 @@
 		print_size(flash_size, "\n");
 # endif /* CONFIG_SYS_FLASH_CHECKSUM */
 	} else {
-		puts(failed);
-		hang();
+		puts("0 KB\n\r");
+		//puts(failed);
+		//hang();
 	}
 #endif
 
@@ -647,6 +654,9 @@
 	}
 #endif
 
+	run_command("mtdparts default", 0);
+	//mtdparts_init();	
+
 	/* main_loop() can return to retry autoboot, if so just run it again. */
 	for (;;) {
 		main_loop();
diff -urN u-boot-2012.04.01/board/samsung/smdk2440/init.c u-boot-2012.04.01_100ask/board/samsung/smdk2440/init.c
--- u-boot-2012.04.01/board/samsung/smdk2440/init.c	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/board/samsung/smdk2440/init.c	2012-06-15 14:23:42.553980412 +0800
@@ -0,0 +1,172 @@
+
+/* NAND FLASH控制器 */
+#define NFCONF (*((volatile unsigned long *)0x4E000000))
+#define NFCONT (*((volatile unsigned long *)0x4E000004))
+#define NFCMMD (*((volatile unsigned char *)0x4E000008))
+#define NFADDR (*((volatile unsigned char *)0x4E00000C))
+#define NFDATA (*((volatile unsigned char *)0x4E000010))
+#define NFSTAT (*((volatile unsigned char *)0x4E000020))
+
+/* GPIO */
+#define GPHCON              (*(volatile unsigned long *)0x56000070)
+#define GPHUP               (*(volatile unsigned long *)0x56000078)
+
+/* UART registers*/
+#define ULCON0              (*(volatile unsigned long *)0x50000000)
+#define UCON0               (*(volatile unsigned long *)0x50000004)
+#define UFCON0              (*(volatile unsigned long *)0x50000008)
+#define UMCON0              (*(volatile unsigned long *)0x5000000c)
+#define UTRSTAT0            (*(volatile unsigned long *)0x50000010)
+#define UTXH0               (*(volatile unsigned char *)0x50000020)
+#define URXH0               (*(volatile unsigned char *)0x50000024)
+#define UBRDIV0             (*(volatile unsigned long *)0x50000028)
+
+#define TXD0READY   (1<<2)
+
+
+void nand_read_ll(unsigned int addr, unsigned char *buf, unsigned int len);
+
+
+static int isBootFromNorFlash(void)
+{
+	volatile int *p = (volatile int *)0;
+	int val;
+
+	val = *p;
+	*p = 0x12345678;
+	if (*p == 0x12345678)
+	{
+		/* 写成功, 是nand启动 */
+		*p = val;
+		return 0;
+	}
+	else
+	{
+		/* NOR不能像内存一样写 */
+		return 1;
+	}
+}
+
+void copy_code_to_sdram(unsigned char *src, unsigned char *dest, unsigned int len)
+{	
+	int i = 0;
+	
+	/* 如果是NOR启动 */
+	if (isBootFromNorFlash())
+	{
+		while (i < len)
+		{
+			dest[i] = src[i];
+			i++;
+		}
+	}
+	else
+	{
+		//nand_init();
+		nand_read_ll((unsigned int)src, dest, len);
+	}
+}
+
+void clear_bss(void)
+{
+	extern int __bss_start, __bss_end__;
+	int *p = &__bss_start;
+	
+	for (; p < &__bss_end__; p++)
+		*p = 0;
+}
+
+void nand_init_ll(void)
+{
+#define TACLS   0
+#define TWRPH0  1
+#define TWRPH1  0
+	/* 设置时序 */
+	NFCONF = (TACLS<<12)|(TWRPH0<<8)|(TWRPH1<<4);
+	/* 使能NAND Flash控制器, 初始化ECC, 禁止片选 */
+	NFCONT = (1<<4)|(1<<1)|(1<<0);	
+}
+
+static void nand_select(void)
+{
+	NFCONT &= ~(1<<1);	
+}
+
+static void nand_deselect(void)
+{
+	NFCONT |= (1<<1);	
+}
+
+static void nand_cmd(unsigned char cmd)
+{
+	volatile int i;
+	NFCMMD = cmd;
+	for (i = 0; i < 10; i++);
+}
+
+static void nand_addr(unsigned int addr)
+{
+	unsigned int col  = addr % 2048;
+	unsigned int page = addr / 2048;
+	volatile int i;
+
+	NFADDR = col & 0xff;
+	for (i = 0; i < 10; i++);
+	NFADDR = (col >> 8) & 0xff;
+	for (i = 0; i < 10; i++);
+	
+	NFADDR  = page & 0xff;
+	for (i = 0; i < 10; i++);
+	NFADDR  = (page >> 8) & 0xff;
+	for (i = 0; i < 10; i++);
+	NFADDR  = (page >> 16) & 0xff;
+	for (i = 0; i < 10; i++);	
+}
+
+static void nand_wait_ready(void)
+{
+	while (!(NFSTAT & 1));
+}
+
+static unsigned char nand_data(void)
+{
+	return NFDATA;
+}
+
+void nand_read_ll(unsigned int addr, unsigned char *buf, unsigned int len)
+{
+	int col = addr % 2048;
+	int i = 0;
+		
+	/* 1. 选中 */
+	nand_select();
+
+	while (i < len)
+	{
+		/* 2. 发出读命令00h */
+		nand_cmd(0x00);
+
+		/* 3. 发出地址(分5步发出) */
+		nand_addr(addr);
+
+		/* 4. 发出读命令30h */
+		nand_cmd(0x30);
+
+		/* 5. 判断状态 */
+		nand_wait_ready();
+
+		/* 6. 读数据 */
+		for (; (col < 2048) && (i < len); col++)
+		{
+			buf[i] = nand_data();
+			i++;
+			addr++;
+		}
+		
+		col = 0;
+	}
+
+	/* 7. 取消选中 */		
+	nand_deselect();
+}
+
diff -urN u-boot-2012.04.01/board/samsung/smdk2440/lowlevel_init.S u-boot-2012.04.01_100ask/board/samsung/smdk2440/lowlevel_init.S
--- u-boot-2012.04.01/board/samsung/smdk2440/lowlevel_init.S	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/board/samsung/smdk2440/lowlevel_init.S	2012-06-15 14:40:13.563461695 +0800
@@ -0,0 +1,168 @@
+/*
+ * Memory Setup stuff - taken from blob memsetup.S
+ *
+ * Copyright (C) 1999 2000 2001 Erik Mouw (J.A.K.Mouw@its.tudelft.nl) and
+ *                     Jan-Derk Bakker (J.D.Bakker@its.tudelft.nl)
+ *
+ * Modified for the Samsung SMDK2410 by
+ * (C) Copyright 2002
+ * David Mueller, ELSOFT AG, <d.mueller@elsoft.ch>
+ *
+ * See file CREDITS for list of people who contributed to this
+ * project.
+ *
+ * This program is free software; you can redistribute it and/or
+ * modify it under the terms of the GNU General Public License as
+ * published by the Free Software Foundation; either version 2 of
+ * the License, or (at your option) any later version.
+ *
+ * This program is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
+ * GNU General Public License for more details.
+ *
+ * You should have received a copy of the GNU General Public License
+ * along with this program; if not, write to the Free Software
+ * Foundation, Inc., 59 Temple Place, Suite 330, Boston,
+ * MA 02111-1307 USA
+ */
+
+
+#include <config.h>
+#include <version.h>
+
+
+/* some parameters for the board */
+
+/*
+ *
+ * Taken from linux/arch/arm/boot/compressed/head-s3c2410.S
+ *
+ * Copyright (C) 2002 Samsung Electronics SW.LEE  <hitchcar@sec.samsung.com>
+ *
+ */
+
+#define BWSCON	0x48000000
+
+/* BWSCON */
+#define DW8			(0x0)
+#define DW16			(0x1)
+#define DW32			(0x2)
+#define WAIT			(0x1<<2)
+#define UBLB			(0x1<<3)
+
+#define B1_BWSCON		(DW32)
+#define B2_BWSCON		(DW16)
+#define B3_BWSCON		(DW16 + WAIT + UBLB)
+#define B4_BWSCON		(DW16)
+#define B5_BWSCON		(DW16)
+#define B6_BWSCON		(DW32)
+#define B7_BWSCON		(DW32)
+
+/* BANK0CON */
+#define B0_Tacs			0x0	/*  0clk */
+#define B0_Tcos			0x0	/*  0clk */
+#define B0_Tacc			0x7	/* 14clk */
+#define B0_Tcoh			0x0	/*  0clk */
+#define B0_Tah			0x0	/*  0clk */
+#define B0_Tacp			0x0
+#define B0_PMC			0x0	/* normal */
+
+/* BANK1CON */
+#define B1_Tacs			0x0	/*  0clk */
+#define B1_Tcos			0x0	/*  0clk */
+#define B1_Tacc			0x7	/* 14clk */
+#define B1_Tcoh			0x0	/*  0clk */
+#define B1_Tah			0x0	/*  0clk */
+#define B1_Tacp			0x0
+#define B1_PMC			0x0
+
+#define B2_Tacs			0x0
+#define B2_Tcos			0x0
+#define B2_Tacc			0x7
+#define B2_Tcoh			0x0
+#define B2_Tah			0x0
+#define B2_Tacp			0x0
+#define B2_PMC			0x0
+
+#define B3_Tacs			0x0	/*  0clk */
+#define B3_Tcos			0x3	/*  4clk */
+#define B3_Tacc			0x7	/* 14clk */
+#define B3_Tcoh			0x1	/*  1clk */
+#define B3_Tah			0x0	/*  0clk */
+#define B3_Tacp			0x3     /*  6clk */
+#define B3_PMC			0x0	/* normal */
+
+#define B4_Tacs			0x0	/*  0clk */
+#define B4_Tcos			0x0	/*  0clk */
+#define B4_Tacc			0x7	/* 14clk */
+#define B4_Tcoh			0x0	/*  0clk */
+#define B4_Tah			0x0	/*  0clk */
+#define B4_Tacp			0x0
+#define B4_PMC			0x0	/* normal */
+
+#define B5_Tacs			0x0	/*  0clk */
+#define B5_Tcos			0x0	/*  0clk */
+#define B5_Tacc			0x7	/* 14clk */
+#define B5_Tcoh			0x0	/*  0clk */
+#define B5_Tah			0x0	/*  0clk */
+#define B5_Tacp			0x0
+#define B5_PMC			0x0	/* normal */
+
+#define B6_MT			0x3	/* SDRAM */
+#define B6_Trcd			0x1
+#define B6_SCAN			0x1	/* 9bit */
+
+#define B7_MT			0x3	/* SDRAM */
+#define B7_Trcd			0x1	/* 3clk */
+#define B7_SCAN			0x1	/* 9bit */
+
+/* REFRESH parameter */
+#define REFEN			0x1	/* Refresh enable */
+#define TREFMD			0x0	/* CBR(CAS before RAS)/Auto refresh */
+#define Trp			0x0	/* 2clk */
+#define Trc			0x3	/* 7clk */
+#define Tchr			0x2	/* 3clk */
+#define REFCNT			1113	/* period=15.6us, HCLK=60Mhz, (2048+1-15.6*60) */
+/**************************************/
+
+_TEXT_BASE:
+	.word	CONFIG_SYS_TEXT_BASE
+
+.globl lowlevel_init
+lowlevel_init:
+	/* memory control configuration */
+	/* make r0 relative the current location so that it */
+	/* reads SMRDATA out of FLASH rather than memory ! */
+	ldr     r0, =SMRDATA
+	ldr	r1, _TEXT_BASE
+	sub	r0, r0, r1
+	ldr	r1, =BWSCON	/* Bus Width Status Controller */
+	add     r2, r0, #13*4
+0:
+	ldr     r3, [r0], #4
+	str     r3, [r1], #4
+	cmp     r2, r0
+	bne     0b
+
+	/* everything is fine now */
+	mov	pc, lr
+
+	.ltorg
+/* the literal pools origin */
+
+SMRDATA:
+		.long 0x22011110	 //BWSCON
+		.long 0x00000700	 //BANKCON0
+		.long 0x00000700	 //BANKCON1
+		.long 0x00000700	 //BANKCON2
+		.long 0x00000700	 //BANKCON3  
+		.long 0x00000740	 //BANKCON4
+		.long 0x00000700	 //BANKCON5
+		.long 0x00018005	 //BANKCON6
+		.long 0x00018005	 //BANKCON7
+		.long 0x008C04F4	 // REFRESH
+		.long 0x000000B1	 //BANKSIZE
+		.long 0x00000030	 //MRSRB6
+		.long 0x00000030	 //MRSRB7
+
diff -urN u-boot-2012.04.01/board/samsung/smdk2440/Makefile u-boot-2012.04.01_100ask/board/samsung/smdk2440/Makefile
--- u-boot-2012.04.01/board/samsung/smdk2440/Makefile	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/board/samsung/smdk2440/Makefile	2012-05-26 03:24:52.631027122 +0800
@@ -0,0 +1,45 @@
+#
+# (C) Copyright 2000-2006
+# Wolfgang Denk, DENX Software Engineering, wd@denx.de.
+#
+# See file CREDITS for list of people who contributed to this
+# project.
+#
+# This program is free software; you can redistribute it and/or
+# modify it under the terms of the GNU General Public License as
+# published by the Free Software Foundation; either version 2 of
+# the License, or (at your option) any later version.
+#
+# This program is distributed in the hope that it will be useful,
+# but WITHOUT ANY WARRANTY; without even the implied warranty of
+# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
+# GNU General Public License for more details.
+#
+# You should have received a copy of the GNU General Public License
+# along with this program; if not, write to the Free Software
+# Foundation, Inc., 59 Temple Place, Suite 330, Boston,
+# MA 02111-1307 USA
+#
+
+include $(TOPDIR)/config.mk
+
+LIB	= $(obj)lib$(BOARD).o
+
+COBJS	:= smdk2410.o init.o
+SOBJS	:= lowlevel_init.o
+
+SRCS	:= $(SOBJS:.o=.S) $(COBJS:.o=.c)
+OBJS	:= $(addprefix $(obj),$(COBJS))
+SOBJS	:= $(addprefix $(obj),$(SOBJS))
+
+$(LIB):	$(obj).depend $(OBJS) $(SOBJS)
+	$(call cmd_link_o_target, $(OBJS) $(SOBJS))
+
+#########################################################################
+
+# defines $(obj).depend target
+include $(SRCTREE)/rules.mk
+
+sinclude $(obj).depend
+
+#########################################################################
diff -urN u-boot-2012.04.01/board/samsung/smdk2440/smdk2410.c u-boot-2012.04.01_100ask/board/samsung/smdk2440/smdk2410.c
--- u-boot-2012.04.01/board/samsung/smdk2440/smdk2410.c	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/board/samsung/smdk2440/smdk2410.c	2012-06-15 14:42:18.922496674 +0800
@@ -0,0 +1,158 @@
+/*
+ * (C) Copyright 2002
+ * Sysgo Real-Time Solutions, GmbH <www.elinos.com>
+ * Marius Groeger <mgroeger@sysgo.de>
+ *
+ * (C) Copyright 2002, 2010
+ * David Mueller, ELSOFT AG, <d.mueller@elsoft.ch>
+ *
+ * See file CREDITS for list of people who contributed to this
+ * project.
+ *
+ * This program is free software; you can redistribute it and/or
+ * modify it under the terms of the GNU General Public License as
+ * published by the Free Software Foundation; either version 2 of
+ * the License, or (at your option) any later version.
+ *
+ * This program is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
+ * GNU General Public License for more details.
+ *
+ * You should have received a copy of the GNU General Public License
+ * along with this program; if not, write to the Free Software
+ * Foundation, Inc., 59 Temple Place, Suite 330, Boston,
+ * MA 02111-1307 USA
+ */
+
+#include <common.h>
+#include <netdev.h>
+#include <asm/io.h>
+#include <asm/arch/s3c24x0_cpu.h>
+
+DECLARE_GLOBAL_DATA_PTR;
+
+#define FCLK_SPEED 1
+
+#if FCLK_SPEED==0		/* Fout = 203MHz, Fin = 12MHz for Audio */
+#define M_MDIV	0xC3
+#define M_PDIV	0x4
+#define M_SDIV	0x1
+#elif FCLK_SPEED==1		/* Fout = 202.8MHz */
+#define M_MDIV	0xA1
+#define M_PDIV	0x3
+#define M_SDIV	0x1
+#endif
+
+#define USB_CLOCK 1
+
+#if USB_CLOCK==0
+#define U_M_MDIV	0xA1
+#define U_M_PDIV	0x3
+#define U_M_SDIV	0x1
+#elif USB_CLOCK==1
+#define U_M_MDIV	0x48
+#define U_M_PDIV	0x3
+#define U_M_SDIV	0x2
+#endif
+
+static inline void pll_delay(unsigned long loops)
+{
+	__asm__ volatile ("1:\n"
+	  "subs %0, %1, #1\n"
+	  "bne 1b":"=r" (loops):"0" (loops));
+}
+
+/*
+ * Miscellaneous platform dependent initialisations
+ */
+
+int board_early_init_f(void)
+{
+	struct s3c24x0_clock_power * const clk_power =
+					s3c24x0_get_base_clock_power();
+	struct s3c24x0_gpio * const gpio = s3c24x0_get_base_gpio();
+
+	/* to reduce PLL lock time, adjust the LOCKTIME register */
+	//writel(0xFFFFFF, &clk_power->locktime);
+
+	/* configure MPLL */
+	//writel((M_MDIV << 12) + (M_PDIV << 4) + M_SDIV,
+	 //      &clk_power->mpllcon);
+
+	/* some delay between MPLL and UPLL */
+	pll_delay(4000);
+
+	/* configure UPLL */
+	writel((U_M_MDIV << 12) + (U_M_PDIV << 4) + U_M_SDIV,
+	       &clk_power->upllcon);
+
+	/* some delay between MPLL and UPLL */
+	pll_delay(8000);
+
+	/* set up the I/O ports */
+	writel(0x007FFFFF, &gpio->gpacon);
+	writel(0x00044555, &gpio->gpbcon);
+	writel(0x000007FF, &gpio->gpbup);
+	writel(0xAAAAAAAA, &gpio->gpccon);
+	writel(0x0000FFFF, &gpio->gpcup);
+	writel(0xAAAAAAAA, &gpio->gpdcon);
+	writel(0x0000FFFF, &gpio->gpdup);
+	writel(0xAAAAAAAA, &gpio->gpecon);
+	writel(0x0000FFFF, &gpio->gpeup);
+	writel(0x000055AA, &gpio->gpfcon);
+	writel(0x000000FF, &gpio->gpfup);
+	writel(0xFF95FFBA, &gpio->gpgcon);
+	writel(0x0000FFFF, &gpio->gpgup);
+	writel(0x002AFAAA, &gpio->gphcon);
+	writel(0x000007FF, &gpio->gphup);
+
+	return 0;
+}
+
+int board_init(void)
+{
+	/* arch number of SMDK2410-Board */
+	gd->bd->bi_arch_number = MACH_TYPE_SMDK2410;
+
+	/* adress of boot parameters */
+	gd->bd->bi_boot_params = 0x30000100;
+
+	icache_enable();
+	dcache_enable();
+
+	return 0;
+}
+
+int dram_init(void)
+{
+	/* dram_init must store complete ramsize in gd->ram_size */
+	gd->ram_size = PHYS_SDRAM_1_SIZE;
+	return 0;
+}
+
+#ifdef CONFIG_CMD_NET
+int board_eth_init(bd_t *bis)
+{
+	int rc = 0;
+#ifdef CONFIG_CS8900
+	rc = cs8900_initialize(0, CONFIG_CS8900_BASE);
+#endif
+#ifdef CONFIG_DRIVER_DM9000
+	rc = dm9000_initialize(bis);
+#endif
+	return rc;
+}
+#endif
+
+/*
+ * Hardcoded flash setup:
+ * Flash 0 is a non-CFI AMD AM29LV800BB flash.
+ */
+ulong board_flash_get_legacy(ulong base, int banknum, flash_info_t *info)
+{
+	info->portwidth = FLASH_CFI_16BIT;
+	info->chipwidth = FLASH_CFI_BY16;
+	info->interface = FLASH_CFI_X16;
+	return 1;
+}
diff -urN u-boot-2012.04.01/boards.cfg u-boot-2012.04.01_100ask/boards.cfg
--- u-boot-2012.04.01/boards.cfg	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/boards.cfg	2012-05-26 02:56:58.591004542 +0800
@@ -63,6 +63,7 @@
 cm41xx                       arm         arm920t     -                   -              ks8695
 VCMA9                        arm         arm920t     vcma9               mpl            s3c24x0
 smdk2410                     arm         arm920t     -                   samsung        s3c24x0
+smdk2440                     arm         arm920t     -                   samsung        s3c24x0
 omap1510inn                  arm         arm925t     -                   ti
 integratorap_cm926ejs        arm         arm926ejs   integrator          armltd         -           integratorap:CM926EJ_S
 integratorcp_cm926ejs        arm         arm926ejs   integrator          armltd         -           integratorcp:CM924EJ_S
diff -urN u-boot-2012.04.01/drivers/mtd/cfi_flash.c u-boot-2012.04.01_100ask/drivers/mtd/cfi_flash.c
--- u-boot-2012.04.01/drivers/mtd/cfi_flash.c	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/drivers/mtd/cfi_flash.c	2012-06-13 22:49:37.697603218 +0800
@@ -42,6 +42,9 @@
 #include <mtd/cfi_flash.h>
 #include <watchdog.h>
 
+//#define DEBUG	1
+//#define _DEBUG  1
+
 /*
  * This file implements a Common Flash Interface (CFI) driver for
  * U-Boot.
diff -urN u-boot-2012.04.01/drivers/mtd/jedec_flash.c u-boot-2012.04.01_100ask/drivers/mtd/jedec_flash.c
--- u-boot-2012.04.01/drivers/mtd/jedec_flash.c	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/drivers/mtd/jedec_flash.c	2012-06-13 22:38:58.267315734 +0800
@@ -367,6 +367,25 @@
 		}
 	},
 #endif
+	/* JZ2440v2使用的MT29LV160DB */
+	{
+			.mfr_id		= (u16)MX_MANUFACT,  /* 厂家ID */
+			.dev_id		= 0X2249,            /* 设备ID */
+			.name		= "MXIC MT29LV160DB",
+			.uaddr		= {  /* NOR flash看到解锁地址 */
+				[1] = MTD_UADDR_0x0555_0x02AA /* x16 */
+			},
+			.DevSize	= SIZE_2MiB,    /* 总大小 */
+			.CmdSet		= P_ID_AMD_STD,
+			.NumEraseRegions= 4,
+			.regions	= {
+				ERASEINFO(16*1024, 1),
+				ERASEINFO(8*1024, 2),	
+				ERASEINFO(32*1024, 1),	
+				ERASEINFO(64*1024, 31),	
+			}
+		},
+
 };
 
 static inline void fill_info(flash_info_t *info, const struct amd_flash_info *jedec_entry, ulong base)
diff -urN u-boot-2012.04.01/drivers/mtd/nand/Makefile u-boot-2012.04.01_100ask/drivers/mtd/nand/Makefile
--- u-boot-2012.04.01/drivers/mtd/nand/Makefile	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/drivers/mtd/nand/Makefile	2012-06-14 01:01:48.602049210 +0800
@@ -59,6 +59,7 @@
 COBJS-$(CONFIG_NAND_NDFC) += ndfc.o
 COBJS-$(CONFIG_NAND_NOMADIK) += nomadik.o
 COBJS-$(CONFIG_NAND_S3C2410) += s3c2410_nand.o
+COBJS-$(CONFIG_NAND_S3C2440) += s3c2440_nand.o
 COBJS-$(CONFIG_NAND_S3C64XX) += s3c64xx.o
 COBJS-$(CONFIG_NAND_SPEAR) += spr_nand.o
 COBJS-$(CONFIG_NAND_OMAP_GPMC) += omap_gpmc.o
diff -urN u-boot-2012.04.01/drivers/mtd/nand/nand_base.c u-boot-2012.04.01_100ask/drivers/mtd/nand/nand_base.c
--- u-boot-2012.04.01/drivers/mtd/nand/nand_base.c	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/drivers/mtd/nand/nand_base.c	2012-06-16 10:41:25.016784826 +0800
@@ -211,7 +211,7 @@
 	case -1:
 		chip->cmd_ctrl(mtd, NAND_CMD_NONE, 0 | NAND_CTRL_CHANGE);
 		break;
-	case 0:
+	case 0:  /* 选中 */
 		break;
 
 	default:
diff -urN u-boot-2012.04.01/drivers/mtd/nand/nand_util.c u-boot-2012.04.01_100ask/drivers/mtd/nand/nand_util.c
--- u-boot-2012.04.01/drivers/mtd/nand/nand_util.c	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/drivers/mtd/nand/nand_util.c	2012-06-16 10:46:53.931450483 +0800
@@ -553,7 +553,7 @@
 
 			ops.len = pagesize;
 			ops.ooblen = nand->oobsize;
-			ops.mode = MTD_OOB_AUTO;
+			ops.mode = MTD_OOB_RAW;
 			ops.ooboffs = 0;
 
 			pages = write_size / pagesize_oob;
@@ -564,7 +564,7 @@
 				ops.oobbuf = ops.datbuf + pagesize;
 
 				rval = nand->write_oob(nand, offset, &ops);
-				if (!rval)
+				if (rval)  /* weidongshan@qq.com */
 					break;
 
 				offset += pagesize;
diff -urN u-boot-2012.04.01/drivers/mtd/nand/s3c2440_nand.c u-boot-2012.04.01_100ask/drivers/mtd/nand/s3c2440_nand.c
--- u-boot-2012.04.01/drivers/mtd/nand/s3c2440_nand.c	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/drivers/mtd/nand/s3c2440_nand.c	2012-06-14 01:02:23.171004455 +0800
@@ -0,0 +1,174 @@
+/*
+ * (C) Copyright 2006 OpenMoko, Inc.
+ * Author: Harald Welte <laforge@openmoko.org>
+ *
+ * This program is free software; you can redistribute it and/or
+ * modify it under the terms of the GNU General Public License as
+ * published by the Free Software Foundation; either version 2 of
+ * the License, or (at your option) any later version.
+ *
+ * This program is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
+ * GNU General Public License for more details.
+ *
+ * You should have received a copy of the GNU General Public License
+ * along with this program; if not, write to the Free Software
+ * Foundation, Inc., 59 Temple Place, Suite 330, Boston,
+ * MA 02111-1307 USA
+ */
+
+#include <common.h>
+
+#include <nand.h>
+#include <asm/arch/s3c24x0_cpu.h>
+#include <asm/io.h>
+
+#define S3C2410_NFCONF_EN          (1<<15)
+#define S3C2410_NFCONF_512BYTE     (1<<14)
+#define S3C2410_NFCONF_4STEP       (1<<13)
+#define S3C2410_NFCONF_INITECC     (1<<12)
+#define S3C2410_NFCONF_nFCE        (1<<11)
+#define S3C2410_NFCONF_TACLS(x)    ((x)<<8)
+#define S3C2410_NFCONF_TWRPH0(x)   ((x)<<4)
+#define S3C2410_NFCONF_TWRPH1(x)   ((x)<<0)
+
+#define S3C2410_ADDR_NALE 4
+#define S3C2410_ADDR_NCLE 8
+
+#ifdef CONFIG_NAND_SPL
+
+/* in the early stage of NAND flash booting, printf() is not available */
+#define printf(fmt, args...)
+
+static void nand_read_buf(struct mtd_info *mtd, u_char *buf, int len)
+{
+	int i;
+	struct nand_chip *this = mtd->priv;
+
+	for (i = 0; i < len; i++)
+		buf[i] = readb(this->IO_ADDR_R);
+}
+#endif
+
+
+/*  ctrl : 表示做什么, 选中芯片/取消选中, 发命令还是发地址
+ * 
+ *  dat  : 命令值或地址值
+ */
+static void s3c2440_hwcontrol(struct mtd_info *mtd, int dat, unsigned int ctrl)
+{
+	struct s3c2440_nand *nand = s3c2440_get_base_nand();
+
+	if (ctrl & NAND_CLE)
+	{
+		/* 发命令 */
+		writeb(dat, &nand->nfcmd);
+	}
+	else if(ctrl & NAND_ALE)
+	{
+		/* 发地址 */
+		writeb(dat, &nand->nfaddr);		
+	}
+
+}
+
+static int s3c2440_dev_ready(struct mtd_info *mtd)
+{
+	struct s3c2440_nand *nand = s3c2440_get_base_nand();
+	debug("dev_ready\n");
+	return readl(&nand->nfstat) & 0x01;
+}
+
+static void s3c2440_nand_select(struct mtd_info *mtd, int chipnr)
+{
+	struct s3c2440_nand *nand = s3c2440_get_base_nand();
+
+	switch (chipnr) {
+	case -1: /* 取消选中 */
+		nand->nfcont |= (1<<1);
+		break;
+	case 0:  /* 选中 */
+		nand->nfcont &= ~(1<<1);
+		break;
+
+	default:
+		BUG();
+	}
+}
+
+
+int board_nand_init(struct nand_chip *nand)
+{
+	u_int32_t cfg;
+	u_int8_t tacls, twrph0, twrph1;
+	struct s3c24x0_clock_power *clk_power = s3c24x0_get_base_clock_power();
+	struct s3c2440_nand *nand_reg = s3c2440_get_base_nand();
+
+	debug("board_nand_init()\n");
+
+	writel(readl(&clk_power->clkcon) | (1 << 4), &clk_power->clkcon);
+
+	/* initialize hardware */
+#if defined(CONFIG_S3C24XX_CUSTOM_NAND_TIMING)
+	tacls  = CONFIG_S3C24XX_TACLS;
+	twrph0 = CONFIG_S3C24XX_TWRPH0;
+	twrph1 =  CONFIG_S3C24XX_TWRPH1;
+#else
+	tacls  = 4;
+	twrph0 = 8;
+	twrph1 = 8;
+#endif
+
+#if 0
+	cfg = S3C2410_NFCONF_EN;
+	cfg |= S3C2410_NFCONF_TACLS(tacls - 1);
+	cfg |= S3C2410_NFCONF_TWRPH0(twrph0 - 1);
+	cfg |= S3C2410_NFCONF_TWRPH1(twrph1 - 1);
+#endif
+	/* 初始化时序 */
+	cfg = ((tacls-1)<<12)|((twrph0-1)<<8)|((twrph1-1)<<4);
+	writel(cfg, &nand_reg->nfconf);
+
+	/* 使能NAND Flash控制器, 初始化ECC, 禁止片选 */
+	writel((1<<4)|(1<<1)|(1<<0), &nand_reg->nfcont);
+
+
+	/* initialize nand_chip data structure */
+	nand->IO_ADDR_R = (void *)&nand_reg->nfdata;
+	nand->IO_ADDR_W = (void *)&nand_reg->nfdata;
+
+	nand->select_chip = s3c2440_nand_select;
+
+	/* read_buf and write_buf are default */
+	/* read_byte and write_byte are default */
+#ifdef CONFIG_NAND_SPL
+	nand->read_buf = nand_read_buf;
+#endif
+
+	/* hwcontrol always must be implemented */
+	nand->cmd_ctrl = s3c2440_hwcontrol;
+
+	nand->dev_ready = s3c2440_dev_ready;
+
+#ifdef CONFIG_S3C2410_NAND_HWECC
+	nand->ecc.hwctl = s3c2410_nand_enable_hwecc;
+	nand->ecc.calculate = s3c2410_nand_calculate_ecc;
+	nand->ecc.correct = s3c2410_nand_correct_data;
+	nand->ecc.mode = NAND_ECC_HW;
+	nand->ecc.size = CONFIG_SYS_NAND_ECCSIZE;
+	nand->ecc.bytes = CONFIG_SYS_NAND_ECCBYTES;
+#else
+	nand->ecc.mode = NAND_ECC_SOFT;
+#endif
+
+#ifdef CONFIG_S3C2410_NAND_BBT
+	nand->options = NAND_USE_FLASH_BBT;
+#else
+	nand->options = 0;
+#endif
+
+	debug("end of nand_init\n");
+
+	return 0;
+}
diff -urN u-boot-2012.04.01/include/common.h u-boot-2012.04.01_100ask/include/common.h
--- u-boot-2012.04.01/include/common.h	2012-04-25 21:22:50.000000000 +0800
+++ u-boot-2012.04.01_100ask/include/common.h	2012-05-26 03:27:18.272556627 +0800
@@ -273,8 +273,8 @@
 extern char console_buffer[];
 
 /* arch/$(ARCH)/lib/board.c */
-void	board_init_f  (ulong) __attribute__ ((noreturn));
-void	board_init_r  (gd_t *, ulong) __attribute__ ((noreturn));
+unsigned int board_init_f  (ulong) ;
+void	board_init_r  (gd_t *, ulong) ;
 int	checkboard    (void);
 int	checkflash    (void);
 int	checkdram     (void);
diff -urN u-boot-2012.04.01/include/configs/smdk2440.h u-boot-2012.04.01_100ask/include/configs/smdk2440.h
--- u-boot-2012.04.01/include/configs/smdk2440.h	1970-01-01 07:00:00.000000000 +0700
+++ u-boot-2012.04.01_100ask/include/configs/smdk2440.h	2012-06-16 10:44:35.983161415 +0800
@@ -0,0 +1,273 @@
+/*
+ * (C) Copyright 2002
+ * Sysgo Real-Time Solutions, GmbH <www.elinos.com>
+ * Marius Groeger <mgroeger@sysgo.de>
+ * Gary Jennejohn <garyj@denx.de>
+ * David Mueller <d.mueller@elsoft.ch>
+ *
+ * Configuation settings for the SAMSUNG SMDK2410 board.
+ *
+ * See file CREDITS for list of people who contributed to this
+ * project.
+ *
+ * This program is free software; you can redistribute it and/or
+ * modify it under the terms of the GNU General Public License as
+ * published by the Free Software Foundation; either version 2 of
+ * the License, or (at your option) any later version.
+ *
+ * This program is distributed in the hope that it will be useful,
+ * but WITHOUT ANY WARRANTY; without even the implied warranty of
+ * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
+ * GNU General Public License for more details.
+ *
+ * You should have received a copy of the GNU General Public License
+ * along with this program; if not, write to the Free Software
+ * Foundation, Inc., 59 Temple Place, Suite 330, Boston,
+ * MA 02111-1307 USA
+ */
+
+#ifndef __CONFIG_H
+#define __CONFIG_H
+
+/*
+ * High Level Configuration Options
+ * (easy to change)
+ */
+#define CONFIG_ARM920T		/* This is an ARM920T Core */
+#define CONFIG_S3C24X0		/* in a SAMSUNG S3C24x0-type SoC */
+//#define CONFIG_S3C2410		/* specifically a SAMSUNG S3C2410 SoC */
+#define CONFIG_S3C2440
+#define CONFIG_SMDK2410		/* on a SAMSUNG SMDK2410 Board */
+
+#define CONFIG_SYS_TEXT_BASE	0x33f00000
+
+#define CONFIG_SYS_ARM_CACHE_WRITETHROUGH
+
+/* input clock of PLL (the SMDK2410 has 12MHz input clock) */
+#define CONFIG_SYS_CLK_FREQ	12000000
+
+#undef CONFIG_USE_IRQ		/* we don't need IRQ/FIQ stuff */
+
+#define CONFIG_CMDLINE_TAG	/* enable passing of ATAGs */
+#define CONFIG_SETUP_MEMORY_TAGS
+#define CONFIG_INITRD_TAG
+
+/*
+ * Hardware drivers
+ */
+#if 0
+#define CONFIG_CS8900		/* we have a CS8900 on-board */
+#define CONFIG_CS8900_BASE	0x19000300
+#define CONFIG_CS8900_BUS16	/* the Linux driver does accesses as shorts */
+#else
+#define CONFIG_DRIVER_DM9000
+#define CONFIG_DM9000_BASE              0x20000000
+#define DM9000_IO                       CONFIG_DM9000_BASE
+#define DM9000_DATA                     (CONFIG_DM9000_BASE + 4)
+#endif
+
+/*
+ * select serial console configuration
+ */
+#define CONFIG_S3C24X0_SERIAL
+#define CONFIG_SERIAL1		1	/* we use SERIAL 1 on SMDK2410 */
+
+/************************************************************
+ * USB support (currently only works with D-cache off)
+ ************************************************************/
+//#define CONFIG_USB_OHCI
+//#define CONFIG_USB_KEYBOARD
+//#define CONFIG_USB_STORAGE
+//#define CONFIG_DOS_PARTITION
+
+/************************************************************
+ * RTC
+ ************************************************************/
+//#define CONFIG_RTC_S3C24X0
+
+
+#define CONFIG_BAUDRATE		115200
+
+/*
+ * BOOTP options
+ */
+//#define CONFIG_BOOTP_BOOTFILESIZE
+//#define CONFIG_BOOTP_BOOTPATH
+//#define CONFIG_BOOTP_GATEWAY
+//#define CONFIG_BOOTP_HOSTNAME
+
+/*
+ * Command line configuration.
+ */
+#include <config_cmd_default.h>
+
+#define CONFIG_CMD_BSP
+#define CONFIG_CMD_CACHE
+//#define CONFIG_CMD_DATE
+//#define CONFIG_CMD_DHCP
+#define CONFIG_CMD_ELF
+#define CONFIG_CMD_NAND
+#define CONFIG_CMD_PING
+#define CONFIG_CMD_REGINFO
+//#define CONFIG_CMD_USB
+
+#define CONFIG_SYS_HUSH_PARSER
+#define CONFIG_SYS_PROMPT_HUSH_PS2	"> "
+#define CONFIG_CMDLINE_EDITING
+
+/* autoboot */
+#define CONFIG_BOOTDELAY	5
+#define CONFIG_BOOT_RETRY_TIME	-1
+#define CONFIG_RESET_TO_RETRY
+#define CONFIG_ZERO_BOOTDELAY_CHECK
+
+#define CONFIG_NETMASK		255.255.255.0
+#define CONFIG_IPADDR		192.168.1.17
+#define CONFIG_SERVERIP		192.168.1.3
+#define CONFIG_ETHADDR      00:0c:29:4d:e4:f4
+
+#if defined(CONFIG_CMD_KGDB)
+#define CONFIG_KGDB_BAUDRATE	115200	/* speed to run kgdb serial port */
+/* what's this ? it's not used anywhere */
+#define CONFIG_KGDB_SER_INDEX	2	/* which serial port to use */
+#endif
+
+/*
+ * Miscellaneous configurable options
+ */
+#define CONFIG_SYS_LONGHELP		/* undef to save memory */
+#define CONFIG_SYS_PROMPT	"SMDK2410 # "
+#define CONFIG_SYS_CBSIZE	256
+/* Print Buffer Size */
+#define CONFIG_SYS_PBSIZE	(CONFIG_SYS_CBSIZE + \
+				sizeof(CONFIG_SYS_PROMPT)+16)
+#define CONFIG_SYS_MAXARGS	16
+#define CONFIG_SYS_BARGSIZE	CONFIG_SYS_CBSIZE
+
+#define CONFIG_DISPLAY_CPUINFO				/* Display cpu info */
+
+#define CONFIG_SYS_MEMTEST_START	0x30000000	/* memtest works on */
+#define CONFIG_SYS_MEMTEST_END		0x33F00000	/* 63 MB in DRAM */
+
+#define CONFIG_SYS_LOAD_ADDR		0x30800000
+
+#define CONFIG_SYS_HZ			1000
+
+/* valid baudrates */
+#define CONFIG_SYS_BAUDRATE_TABLE	{ 9600, 19200, 38400, 57600, 115200 }
+
+/* support additional compression methods */
+#define CONFIG_BZIP2
+#define CONFIG_LZO
+#define CONFIG_LZMA
+
+#define CONFIG_CMD_NAND_YAFFS
+
+#define CONFIG_BOOTARGS "console=ttySAC0 root=/dev/mtdblock3"
+#define CONFIG_BOOTCOMMAND "nand read 30000000 kernel;bootm 30000000"
+
+/*-----------------------------------------------------------------------
+ * Stack sizes
+ *
+ * The stack sizes are set up in start.S using the settings below
+ */
+#define CONFIG_STACKSIZE	(128*1024)	/* regular stack */
+#ifdef CONFIG_USE_IRQ
+#define CONFIG_STACKSIZE_IRQ	(4*1024)	/* IRQ stack */
+#define CONFIG_STACKSIZE_FIQ	(4*1024)	/* FIQ stack */
+#endif
+
+/*-----------------------------------------------------------------------
+ * Physical Memory Map
+ */
+#define CONFIG_NR_DRAM_BANKS	1          /* we have 1 bank of DRAM */
+#define PHYS_SDRAM_1		0x30000000 /* SDRAM Bank #1 */
+#define PHYS_SDRAM_1_SIZE	0x04000000 /* 64 MB */
+
+#define PHYS_FLASH_1		0x00000000 /* Flash Bank #0 */
+
+#define CONFIG_SYS_FLASH_BASE	PHYS_FLASH_1
+
+/*-----------------------------------------------------------------------
+ * FLASH and environment organization
+ */
+
+#define CONFIG_SYS_FLASH_CFI
+#define CONFIG_FLASH_CFI_DRIVER
+#define CONFIG_FLASH_CFI_LEGACY
+#define CONFIG_SYS_FLASH_LEGACY_512Kx16
+#define CONFIG_FLASH_SHOW_PROGRESS	45
+
+#define CONFIG_SYS_MAX_FLASH_BANKS	1
+#define CONFIG_SYS_FLASH_BANKS_LIST     { CONFIG_SYS_FLASH_BASE }
+#define CONFIG_SYS_MAX_FLASH_SECT	(128)
+
+#if 0
+#define CONFIG_ENV_ADDR			(CONFIG_SYS_FLASH_BASE + 0x070000)
+#define CONFIG_ENV_IS_IN_FLASH
+#define CONFIG_ENV_SIZE			0x10000
+/* allow to overwrite serial and ethaddr */
+#define CONFIG_ENV_OVERWRITE
+#endif
+#define CONFIG_ENV_IS_IN_NAND
+#define CONFIG_ENV_OFFSET      0x00040000
+#define CONFIG_ENV_SIZE        0x20000
+#define CONFIG_ENV_RANGE       CONFIG_ENV_SIZE
+
+#define CONFIG_CMD_MTDPARTS
+#define CONFIG_MTD_DEVICE
+#define MTDIDS_DEFAULT		"nand0=jz2440-0"  /* 哪一个设备 */
+#define MTDPARTS_DEFAULT	"mtdparts=jz2440-0:256k(u-boot),"	\
+						"128k(params),"		\
+						"2m(kernel),"	\
+						"-(rootfs)"		\
+
+
+/*
+ * Size of malloc() pool
+ * BZIP2 / LZO / LZMA need a lot of RAM
+ */
+#define CONFIG_SYS_MALLOC_LEN	(4 * 1024 * 1024)
+
+#define CONFIG_SYS_MONITOR_LEN	(448 * 1024)
+#define CONFIG_SYS_MONITOR_BASE	CONFIG_SYS_FLASH_BASE
+
+/*
+ * NAND configuration
+ */
+#ifdef CONFIG_CMD_NAND
+
+#ifdef CONFIG_S3C2410
+#define CONFIG_NAND_S3C2410
+#define CONFIG_SYS_S3C2410_NAND_HWECC
+#else
+#define CONFIG_NAND_S3C2440
+#define CONFIG_SYS_S3C2440_NAND_HWECC
+#endif
+
+#define CONFIG_SYS_MAX_NAND_DEVICE	1
+#define CONFIG_SYS_NAND_BASE		0x4E000000
+#endif
+
+/*
+ * File system
+ */
+#if 0 
+#define CONFIG_CMD_FAT
+#define CONFIG_CMD_EXT2
+#define CONFIG_CMD_UBI
+#define CONFIG_CMD_UBIFS
+#define CONFIG_CMD_MTDPARTS
+#define CONFIG_MTD_DEVICE
+#define CONFIG_MTD_PARTITIONS
+#define CONFIG_YAFFS2
+#define CONFIG_RBTREE
+#endif
+/* additions for new relocation code, must be added to all boards */
+#define CONFIG_SYS_SDRAM_BASE	PHYS_SDRAM_1
+#define CONFIG_SYS_INIT_SP_ADDR	(CONFIG_SYS_SDRAM_BASE + 0x1000 - \
+				GENERATED_GBL_DATA_SIZE)
+
+#define CONFIG_BOARD_EARLY_INIT_F
+
+#endif /* __CONFIG_H */
```