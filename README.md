glib4android
============

glib for arm


libnice-4-android
=================

p2p

1. 下载源码

a.下载glib-2.34.3

http://ftp.gnome.org/pub/gnome/sources/glib/2.34/glib-2.34.3.tar.xz

b.下载libiconv-1.14

http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz

c. 下载gettex-0.18.2

http://ftp.gnu.org/pub/gnu/gettext/gettext-0.18.2.tar.gz

d. 下载libffi-3.0.12

ftp://sourceware.org/pub/libffi/libffi-3.0.12.tar.gz

2. 准备工作

a. 设定工作目录（如YOUR_WORKSPACE）, 将上面下载的源码包(.tar.gz, 四个) 解压到$(YOUR_WORKSPACE)/glib-4-android/

b. cd $(YOUR_WORKSPACE)/glib-4-android/

c. ls

==>gettext-0.18.2  glib-2.34.3  libffi-3.0.12  libiconv-1.14

d. 创建临时目录并下载以下两个文件。

e. mkdir tmp

   wget -O ./tmp/config.sub "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD"

   wget -O ./tmp/config.guess "git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD"

3. 准备交叉编译环境

   我的ndk目录: /home/xxx/dev_env/android-ndk-r9d

a. cd /home/xxx/dev_env/android-ndk-r9d

b.
build/tools/make-standalone-toolchain.sh --platform=android-14 --toolchain=arm-linux-androideabi-4.6 
--install-dir=${HOME}/dev_env/android-toolchain

c. echo "export SYSROOT=${HOME}/dev_env/android-toolchain/sysroot" >> ${HOME}/.bashrc

d. echo 'export PATH=${PATH}:${HOME}/dev_env/android-toolchain/bin' >> ${HOME}/.bashrc

e. source ${HOME}/.bashrc 或 source ~/.bashrc

4. 编译libiconv

a. cd $(YOUR_WORKSPACE)/glib-4-android/libiconv-1.14

b. 到build-aux目录和libcharset/build-aux目录下备份config.sub和config.guess 为config.sub.bak和config.guess.bak

c. 将$(YOUR_WORKSPACE)/glib-4-android/tmp里的config.sub和config.guess拷贝到$(YOUR_WORKSPACE)/glib-4-android/history_compile/libiconv-1.14下的build-aux目录和libcharset/build-aux目录下，两个目录两个文件都要有

d. 备份libiconv-1.14/configure

e. gl_cv_header_working_stdint_h=yes ./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static

f. make -j4;make install

g. 编译完后在 ${SYSROOT}/usr/lib/目录下能找到库(libcharset.so和libiconv.so, preloadable_libiconv.so)


5. 编译gettext

a. cd $(YOUR_WORKSPACE)/glib-4-android/

b. 找到gettext-tools/src/msginit.c, 备份msginit.c==>msginit.c.bak;

   找到这行“ /* Return the pw_gecos field, up to the first comma (if any).  */”;

	将行   fullname = pwd->pw_gecos; 替换为：

		#ifndef __ANDROID__

       			fullname = pwd->pw_gecos;
		
		#else
		
			fullname = "android";
		
		#endif
   
   原因：

	android toolchai 的 passwd 结构体没有 pw_gecos 成员，给 fullname 赋值, 避免访问 pwd->pw_gecos。

c. 执行 ./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static --disable-java --disable-native-java

d. make -j4;make install

e. 编译完后在 ${SYSROOT}/usr/lib/目录下能找到库(libasprintf.so, libgettextlib.so, libgettextpo.so, libgettextsrc.so, libintl.so)
  
6. 编译libffi

a. ./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static

b. make -4;make install

c. 编译完后在 ${SYSROOT}/usr/lib/目录下能找到库libffi.la pkgconfig/libffi.pc

7. 编译glib, (编译glib2.40, 参见7.1)

a. 新建一个文件 android.cache, 写入以下内容.

	# file android.cache

	ac_cv_type_long_long=yes

	glib_cv_stack_grows=no

	glib_cv_uscore=no

	ac_cv_func_posix_getpwuid_r=no

	ac_cv_func_posix_getgrgid_r=no

b. 配置编译的指令(test 模块编译不过去, 没关系, 用不到, disable 之):

另外

#echo "export PKG_CONFIG_PATH=${SYSROOT}/usr/lib/pkgconfig/" >> ${HOME}/.bashrc

echo "export PKG_CONFIG_LIBDIR=${SYSROOT}/usr/lib/pkgconfig/" >> ${HOME}/.bashrc

原因 PKG_CONFIG_LIBDIR的优先级比 PKG_CONFIG_PATH 高，所以会覆盖PKG_CONFIG_PATH的设置。

$ ./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static --cache-file=android.cache --disable-modular-tests

c. 打补丁:https://gist.github.com/mathslinux/5283294

###############==>创建glib-2.34.3/gio/android-dep.h, 内容:

#ifndef ANDROID-DEP_H

#define ANDROID-DEP_H

#ifdef __ANDROID__

#define dn_skipname __dn_skipname
int dn_skipname(const u_char *, const u_char *);

typedef enum __ns_type {
 ns_t_invalid = 0, /*%< Cookie. */
 ns_t_a = 1, /*%< Host address. */
 ns_t_ns = 2, /*%< Authoritative server. */
 ns_t_md = 3, /*%< Mail destination. */
 ns_t_mf = 4, /*%< Mail forwarder. */
 ns_t_cname = 5, /*%< Canonical name. */
 ns_t_soa = 6, /*%< Start of authority zone. */
 ns_t_mb = 7, /*%< Mailbox domain name. */
 ns_t_mg = 8, /*%< Mail group member. */
 ns_t_mr = 9, /*%< Mail rename name. */
 ns_t_null = 10, /*%< Null resource record. */
 ns_t_wks = 11, /*%< Well known service. */
 ns_t_ptr = 12, /*%< Domain name pointer. */
 ns_t_hinfo = 13, /*%< Host information. */
 ns_t_minfo = 14, /*%< Mailbox information. */
 ns_t_mx = 15, /*%< Mail routing information. */
 ns_t_txt = 16, /*%< Text strings. */
 ns_t_rp = 17, /*%< Responsible person. */
 ns_t_afsdb = 18, /*%< AFS cell database. */
 ns_t_x25 = 19, /*%< X_25 calling address. */
 ns_t_isdn = 20, /*%< ISDN calling address. */
 ns_t_rt = 21, /*%< Router. */
 ns_t_nsap = 22, /*%< NSAP address. */
 ns_t_nsap_ptr = 23, /*%< Reverse NSAP lookup (deprecated). */
 ns_t_sig = 24, /*%< Security signature. */
 ns_t_key = 25, /*%< Security key. */
 ns_t_px = 26, /*%< X.400 mail mapping. */
 ns_t_gpos = 27, /*%< Geographical position (withdrawn). */
 ns_t_aaaa = 28, /*%< Ip6 Address. */
 ns_t_loc = 29, /*%< Location Information. */
 ns_t_nxt = 30, /*%< Next domain (security). */
 ns_t_eid = 31, /*%< Endpoint identifier. */
 ns_t_nimloc = 32, /*%< Nimrod Locator. */
 ns_t_srv = 33, /*%< Server Selection. */
 ns_t_atma = 34, /*%< ATM Address */
 ns_t_naptr = 35, /*%< Naming Authority PoinTeR */
 ns_t_kx = 36, /*%< Key Exchange */
 ns_t_cert = 37, /*%< Certification record */
 ns_t_a6 = 38, /*%< IPv6 address (deprecated, use ns_t_aaaa) */
 ns_t_dname = 39, /*%< Non-terminal DNAME (for IPv6) */
 ns_t_sink = 40, /*%< Kitchen sink (experimentatl) */
 ns_t_opt = 41, /*%< EDNS0 option (meta-RR) */
 ns_t_apl = 42, /*%< Address prefix list (RFC3123) */
 ns_t_tkey = 249, /*%< Transaction key */
 ns_t_tsig = 250, /*%< Transaction signature. */
 ns_t_ixfr = 251, /*%< Incremental zone transfer. */
 ns_t_axfr = 252, /*%< Transfer zone of authority. */
 ns_t_mailb = 253, /*%< Transfer mailbox records. */
 ns_t_maila = 254, /*%< Transfer mail agent records. */
 ns_t_any = 255, /*%< Wildcard match. */
 ns_t_zxfr = 256, /*%< BIND-specific, nonstandard. */
 ns_t_max = 65536
} ns_type;

/*%
 * Values for class field
 */
typedef enum __ns_class {
 ns_c_invalid = 0, /*%< Cookie. */
 ns_c_in = 1, /*%< Internet. */
 ns_c_2 = 2, /*%< unallocated/unsupported. */
 ns_c_chaos = 3, /*%< MIT Chaos-net. */
 ns_c_hs = 4, /*%< MIT Hesiod. */
 /* Query class values which do not appear in resource records */
 ns_c_none = 254, /*%< for prereq. sections in update requests */
 ns_c_any = 255, /*%< Wildcard match. */
 ns_c_max = 65536
} ns_class;

#define T_NS ns_t_ns
#define T_SOA ns_t_soa
#define T_MX ns_t_mx
#define T_TXT ns_t_txt
#define C_IN ns_c_in
typedef struct {
 unsigned id :16; /*%< query identification number */
 /* fields in third byte */
 unsigned rd :1; /*%< recursion desired */
 unsigned tc :1; /*%< truncated message */
 unsigned aa :1; /*%< authoritive answer */
 unsigned opcode :4; /*%< purpose of message */
 unsigned qr :1; /*%< response flag */
 /* fields in fourth byte */
 unsigned rcode :4; /*%< response code */
 unsigned cd: 1; /*%< checking disabled by resolver */
 unsigned ad: 1; /*%< authentic data from named */
 unsigned unused :1; /*%< unused bits (MBZ as of 4.9.3a3) */
 unsigned ra :1; /*%< recursion available */
 /* remaining bytes */
 unsigned qdcount :16; /*%< number of question entries */
 unsigned ancount :16; /*%< number of answer entries */
 unsigned nscount :16; /*%< number of authority entries */
 unsigned arcount :16; /*%< number of resource entries */
} HEADER;

/*
 * Define constants based on RFC 883, RFC 1034, RFC 1035
 */
#define NS_PACKETSZ 512 /*%< default UDP packet size */
#define NS_MAXDNAME 1025 /*%< maximum domain name */
#define NS_MAXMSG 65535 /*%< maximum message size */
#define NS_MAXCDNAME 255 /*%< maximum compressed domain name */
#define NS_MAXLABEL 63 /*%< maximum length of domain label */
#define NS_HFIXEDSZ 12 /*%< #/bytes of fixed data in header */
#define NS_QFIXEDSZ 4 /*%< #/bytes of fixed data in query */
#define NS_RRFIXEDSZ 10 /*%< #/bytes of fixed data in r record */
#define NS_INT32SZ 4 /*%< #/bytes of data in a u_int32_t */
#define NS_INT16SZ 2 /*%< #/bytes of data in a u_int16_t */
#define NS_INT8SZ 1 /*%< #/bytes of data in a u_int8_t */
#define NS_INADDRSZ 4 /*%< IPv4 T_A */
#define NS_IN6ADDRSZ 16 /*%< IPv6 T_AAAA */
#define NS_CMPRSFLGS 0xc0 /*%< Flag bits indicating name compression. */
#define NS_DEFAULTPORT 53 /*%< For both TCP and UDP. */

#define GETSHORT NS_GET16
#define GETLONG NS_GET32
#define PUTSHORT NS_PUT16
#define PUTLONG NS_PUT32
/*%
 * Inline versions of get/put short/long. Pointer is advanced.
 */
#define NS_GET16(s, cp) do { \
 register const u_char *t_cp = (const u_char *)(cp); \
 (s) = ((u_int16_t)t_cp[0] << 8) \
 | ((u_int16_t)t_cp[1]) \
 ; \
 (cp) += NS_INT16SZ; \
} while (0)

#define NS_GET32(l, cp) do { \
 register const u_char *t_cp = (const u_char *)(cp); \
 (l) = ((u_int32_t)t_cp[0] << 24) \
 | ((u_int32_t)t_cp[1] << 16) \
 | ((u_int32_t)t_cp[2] << 8) \
 | ((u_int32_t)t_cp[3]) \
 ; \
 (cp) += NS_INT32SZ; \
} while (0)

#define NS_PUT16(s, cp) do { \
 register u_int16_t t_s = (u_int16_t)(s); \
 register u_char *t_cp = (u_char *)(cp); \
 *t_cp++ = t_s >> 8; \
 *t_cp = t_s; \
 (cp) += NS_INT16SZ; \
} while (0)

#define NS_PUT32(l, cp) do { \
 register u_int32_t t_l = (u_int32_t)(l); \
 register u_char *t_cp = (u_char *)(cp); \
 *t_cp++ = t_l >> 24; \
 *t_cp++ = t_l >> 16; \
 *t_cp++ = t_l >> 8; \
 *t_cp = t_l; \
 (cp) += NS_INT32SZ; \
} while (0)

#endif /* __ANDROID__ */

#endif
###############==>更新glib-2.34.3/gio/glocalfileinfo.c
  Line1097: ANDROID下略过的代码断
#ifndef __ANDROID__
        gecos = pwbufp->pw_gecos;
        if (gecos)
	{
	  comma = strchr (gecos, ',');
	  if (comma)
	    *comma = 0;
	  data->real_name = convert_pwd_string_to_utf8 (gecos);
	}
#endif
###############==>更新glib-2.34.3/gio/gresolver.c
Line35 加入#include "android-dep.h"
###############==>更新glib-2.34.3/gio/gthreadedresolver.c
Line36 加入#include "android-dep.h"
###############==>更新glib-2.34.3/glib/gstrfuncs.c
其他修改见patch


d. config.h去掉HAVE_PTHREAD_ATTR_SETSTACKSIZE

#define HAVE_PTHREAD_ATTR_SETSTACKSIZE 0

====================================================================================================

7.1编译glib2.40.0 

a. touch android.cache

b. android.cache内容
	ac_cv_type_long_long=yes
	glib_cv_stack_grows=no
	glib_cv_uscore=no
	ac_cv_func_posix_getpwuid_r=no
	ac_cv_func_posix_getgrgid_r=no

c. config: ./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static --cache-file=android.cache --disable-modular-tests LDFLAGS="-static"

d. 有编译不过的地方，参照以上打补丁的方法修改。

e. 生成配置

./configure --prefix="${SYSROOT}/usr" --host=arm-linux-androideabi CFLAGS="--sysroot $SYSROOT" --enable-static --cache-file=android.cache --disable-modular-tests LDFLAGS="-static" --disable-shared --disable-ipv6

f. make;make install

