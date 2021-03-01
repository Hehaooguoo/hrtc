project('hrtc', 'cpp',
  version : '0.0.0',
  default_options : ['warning_level=3', 'buildtype=debugoptimized'])

hrtc_version = meson.project_version()
version_arr = hrtc_version.split('.')
version_major = version_arr[0]
version_minor = version_arr[1]
version_micro = version_arr[2]
if version_arr.length() == 4
  version_nano = version_arr[3]
else
  version_nano = 0
endif

soversion = 1
libversion = '0.0.0'

#第三方库支持
gnutls_req = '>= 2.12.0'


syslibs = []

cc = meson.get_compiler('cpp')

if cc.get_id() == 'msvc'
  add_project_arguments(
      cc.get_supported_arguments(['/utf-8']), # set the input encoding to utf-8
      language : 'c')
endif

if host_machine.system() == 'windows'
  syslibs += [cc.find_library('iphlpapi')]
  syslibs += [cc.find_library('ws2_32')]
elif host_machine.system() == 'sunos'
  add_project_arguments('-D_XOPEN_SOURCE=600', language: 'c')
  add_project_arguments('-D__EXTENSIONS__=1', language: 'c')
  # inet_pton() is only used by the tests
  syslibs += [cc.find_library('nsl')]
  if not cc.has_function('inet_pton')
    libnsl = cc.find_library('nsl', required: false)
    if libnsl.found() and cc.has_function('inet_pton', dependencies: libnsl)
      syslibs += [libnsl]
    endif
  endif
  if not cc.has_function('socket')
    libsocket = cc.find_library('socket', required: false)
    libinet = cc.find_library('inet', required: false)
    if cc.has_function('socket', dependencies: libsocket)
      syslibs += [libsocket]
    elif cc.has_function('socket', dependencies: libinet)
      syslibs += [libinet]
    else
      error('Could not find right library for socket() on Solaris')
    endif
  endif
endif

if not cc.has_function('clock_gettime')
  librt = cc.find_library('rt', required: false)
  if cc.has_function('clock_gettime', dependencies: librt)
    syslibs += [librt]
  endif
endif

conf_data = configuration_data()
# Set data
conf_data.set_quoted('PACKAGE_STRING', meson.project_name())
conf_data.set_quoted('PACKAGE_NAME', meson.project_name())
conf_data.set_quoted('PACKAGE', meson.project_name())
conf_data.set('VERSION', meson.project_version())
conf_data.set('AUTHOR', 'HHaoGuo')

hrct_incs = include_directories('.','ice')

subdir('tests')

#subdir('ice')

#subdir('example')
# Cryto library
opt_cryptolib = get_option('crypto-library')
message('Crypto library requested: ' + opt_cryptolib)
if opt_cryptolib != 'openssl'
  crypto_dep = dependency('gnutls', version: gnutls_req, required: false)
  conf_data.set('HAVE_GNUTLS', crypto_dep.found())
  if not crypto_dep.found() and opt_cryptolib == 'auto'
    crypto_dep = dependency('openssl', required: false,
      fallback: ['openssl', 'openssl_dep'])
    conf_data.set('HAVE_OPENSSL', crypto_dep.found())
  endif
else
  crypto_dep = dependency('openssl', required: false)
  conf_data.set('HAVE_OPENSSL', crypto_dep.found())
  if not crypto_dep.found() and openssl == 'auto'
    crypto_dep = dependency('gnutls', version: gnutls_req, required: false)
    conf_data.set('HAVE_GNUTLS', crypto_dep.found())
  endif
endif

ignored_iface_prefix = get_option('ignored-network-interface-prefix')
if ignored_iface_prefix != []
  ignored_iface_prefix_quoted = []
  foreach i : ignored_iface_prefix
    ignored_iface_prefix_quoted += '"' + i + '"'
  endforeach
  conf_data.set('IGNORED_IFACE_PREFIX', ','.join(ignored_iface_prefix_quoted))
endif


gthread_dep = dependency('gthread-2.0',
  fallback: ['glib', 'libgthread_dep'])
  

hrtc_deps =  [gthread_dep, crypto_dep] + syslibs


src = ['hrtc.cpp','./ice/agent.cpp']

exe = executable('hrtc', src,
  include_directories : hrct_incs,
  dependencies: hrtc_deps,
  install: false)

configure_file(input : 'config.h.in',
  output : 'config.h',
  configuration : conf_data)


  # add_test_setup('valgrind',
# 	       exe_wrapper: ['valgrind',
# 			     '--leak-check=full',
# 			     '--show-reachable=no',
# 			     '--error-exitcode=1',
# 			     '--suppressions='+meson.current_source_dir()+'/tests/libnice.supp',
# 			     '--num-callers=10'],
# 	       timeout_multiplier: 10,
# 	       env: ['CK_FORK=no']
# 	      )


# test('command line test', exe, args : ['first', 'second'])
# test('envvar test', exe2, env : ['key1=value1', 'key2=value2'])
