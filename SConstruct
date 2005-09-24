import os
import re

package = 'lighttpd'
version = '1.4.4'


def checkCHeaders(autoconf, hdrs):
	p = re.compile('[^A-Z0-9]')
	for hdr in hdrs:
		if autoconf.CheckCHeader(hdr):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_' + p.sub('_', hdr.upper()) ])

def checkFuncs(autoconf, funcs):
	p = re.compile('[^A-Z0-9]')
	for func in funcs:
		if autoconf.CheckFunc(func):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_' + p.sub('_', func.upper()) ])

def checkTypes(autoconf, types):
	p = re.compile('[^A-Z0-9]')
	for type in types:
		if autoconf.CheckType(type, '#include <sys/types.h>'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_' + p.sub('_', type.upper()) ])


BuildDir('build', 'src', duplicate = 0)

opts = Options('config.py')
opts.AddOptions(
	('prefix', 'prefix', '/usr/local'),
	('bindir', 'binary directory', '${prefix}/bin'),
	('libdir', 'library directory', '${prefix}/lib'),
	PackageOption('with_mysql', 'enable mysql support', 'no'),
	PackageOption('with_xml', 'enable xml support', 'no'),
	PackageOption('with_pcre', 'enable pcre support', 'yes'),
	BoolOption('with_sqlite3', 'enable sqlite3 support', 'no'),
	BoolOption('with_memcache', 'enable memcache support', 'no'),
	BoolOption('with_fam', 'enable FAM/gamin support', 'no'),
	BoolOption('with_openssl', 'enable memcache support', 'no'),
	BoolOption('with_gzip', 'enable gzip compression', 'no'),
	BoolOption('with_bzip2', 'enable bzip2 compression', 'no'))

env = Environment(
	env = os.environ,
	options = opts,
	CCFLAGS = Split('-Wall -O2 -g -pedantic -Wunused -Wshadow -Isrc/'),
	LIBS = [ 'dl' ]
)

env['package'] = package
env['version'] = version

# cache configure checks
if 1:
	autoconf = Configure(env)
	checkCHeaders(autoconf, Split('arpa/inet.h fcntl.h netinet/in.h stdlib.h string.h \
			sys/socket.h sys/time.h unistd.h sys/sendfile.h sys/uio.h \
			getopt.h sys/epoll.h sys/select.h poll.h sys/poll.h sys/devpoll.h sys/filio.h \
			sys/mman.h sys/event.h sys/port.h winsock2.h pwd.h sys/syslimits.h \
			sys/resource.h sys/un.h syslog.h stdint.h inttypes.h sys/wait.h'))

	checkFuncs(autoconf, Split('fork stat lstat strftime dup2 getcwd inet_ntoa inet_ntop memset mmap munmap strchr \
			strdup strerror strstr strtol sendfile  getopt socket \
			gethostbyname poll sigtimedwait epoll_ctl getrlimit chroot \
			getuid select signal pathconf\
			writev sigaction sendfile64 send_file kqueue port_create localtime_r'))

	checkTypes(autoconf, Split('pid_t size_t off_t'))

	autoconf.env.Append( LIBSQLITE3 = '', LIBXML2 = '', LIBMYSQL = '')

	if env['with_fam']:
		if autoconf.CheckLibWithHeader('fam', 'fam.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_FAM_H', '-DHAVE_LIBFAM' ], LIBS = 'fam')

	if autoconf.CheckLibWithHeader('crypt', 'crypt.h', 'C'):
		autoconf.env.Append(CPPFLAGS = [ '-DHAVE_CRYPT_H', '-DHAVE_LIBCRYPT' ], LIBCRYPT = 'crypt')

	if env['with_openssl']:
		if autoconf.CheckLibWithHeader('ssl', 'openssl/ssl.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_OPENSSL_SSL_H', '-DHAVE_LIBSSL'] , LIBS = [ 'ssl', 'crypto' ])

	if env['with_gzip']:
		if autoconf.CheckLibWithHeader('z', 'zlib.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_ZLIB_H', '-DHAVE_LIBZ' ], LIBZ = 'z')

	if env['with_bzip2']:
		if autoconf.CheckLibWithHeader('bz2', 'bzlib.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_BZLIB_H', '-DHAVE_LIBBZ2' ], LIBBZ2 = 'bz2')

	if env['with_memcache']:
		if autoconf.CheckLibWithHeader('memcache', 'memcache.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_MEMCACHE_H', '-DHAVE_LIBMEMCACHE' ], LIBMEMCACHE = 'memcache')

	if env['with_sqlite3']:
		if autoconf.CheckLibWithHeader('sqlite3', 'sqlite3.h', 'C'):
			autoconf.env.Append(CPPFLAGS = [ '-DHAVE_SQLITE3_H', '-DHAVE_LIBSQLITE3' ], LIBSQLITE3 = 'sqlite3')

	if autoconf.CheckType('socklen_t', '#include <unistd.h>\n#include <sys/socket.h>\n#include <sys/types.h>'):
		autoconf.env.Append(CPPFLAGS = [ '-DHAVE_SOCKLEN_T' ])

	if autoconf.CheckType('struct sockaddr_storage', '#include <sys/socket.h>\n'):
		autoconf.env.Append(CPPFLAGS = [ '-DHAVE_STRUCT_SOCKADDR_STORAGE' ])


	env = autoconf.Finish()

if env['with_pcre']:
	if env['with_pcre'] != 1:
		pcre_config = env['with_pcre']
	elif env.Detect('pcre-config'):
		pcre_config = env.WhereIs('pcre-config')
	env.ParseConfig(pcre_config + ' --cflags --libs')
	env.Append(CPPFLAGS = [ '-DHAVE_PCRE_H', '-DHAVE_LIBPCRE' ], LIBPCRE = 'pcre')

if env['with_xml']:
	if env['with_xml'] != 1:
		xml2_config = env['with_xml']
	elif env.Detect('xml2-config'):
		xml2_config = env.WhereIs('xml2-config')
	env.ParseConfig(xml2_config + ' --cflags --libs')
	env.Append(CPPFLAGS = [ '-DHAVE_LIBXML_H', '-DHAVE_LIBXML2' ], LIBXML2 = 'xml2')

if env['with_mysql']:
	if env['with_mysql'] != 1:
		mysql_config = env['with_mysql']
	else:
		mysql_config = env.WhereIs('mysql_config')
	
	env.ParseConfig(mysql_config + ' --cflags --libs')
	env.Append(CPPFLAGS = [ '-DHAVE_MYSQL' ], LIBMYSQL = 'mysqlclient')

env.Append(CPPFLAGS = [ 
		'-DLIGHTTPD_VERSION_ID=' + str(1 << 16 | 4 << 8 | 4),
		'-DPACKAGE_NAME=\\"' + package + '\\"',
		'-DPACKAGE_VERSION=\\"' + version + '\\"',
		'-DLIBRARY_DIR="\\"${libdir}\\""',
		'-D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE -D_LARGE_FILES'
		] )

SConscript( 'src/SConscript', 'env', build_dir = 'build', duplicate = 0)
SConscript( 'tests/SConscript', 'env' )