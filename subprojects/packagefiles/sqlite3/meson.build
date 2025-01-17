project(
  'sqlite3',
  'c',
  license: 'pd',
  version: '3.47.2',
  meson_version: '>=0.48.0',
)

cc = meson.get_compiler('c')
if meson.version().version_compare('>= 0.62.0')
  libdl = dependency('dl', required: false)
else
  libdl = cc.find_library('dl', required: false)
endif
libm = cc.find_library('m', required: false)

sources = files('sqlite3.c')
if host_machine.system() == 'windows'
  sqlite_args = get_option('default_library') != 'static' ? ['-DSQLITE_API=__declspec(dllexport)'] : []
  libthreads = dependency('', required: false)
else
  sqlite_args = ['-DSQLITE_API=__attribute__((visibility("default")))']
  libthreads = dependency('threads')
endif

foreach h: [
  'inttypes.h',
  'malloc.h',
  'stdint.h',
]
  if cc.has_header(h)
    sqlite_args += '-DHAVE_' + h.to_upper().underscorify()
  endif
endforeach

foreach signed: ['', 'u']
  foreach width: ['8', '16', '32']
    typ = signed + 'int' + width + '_t'
    if cc.has_type(typ, prefix: '#include <stdint.h>')
      sqlite_args += '-DHAVE_' + typ.to_upper()
    endif
  endforeach
endforeach

foreach f: [
  'malloc_usable_size',
  'posix_fallocate',
  'strchrnul',
  'strerror_r',
]
  if cc.has_function(f)
    sqlite_args += '-DHAVE_' + f.to_upper()
  endif
endforeach
if cc.has_function('isnan', dependencies: libm)
  sqlite_args += '-DSQLITE_HAVE_ISNAN'
endif

libsqlite3 = library(
  'sqlite3',
  sources,
  c_args: sqlite_args,
  dependencies: [libdl, libm, libthreads],
  install: true,
  gnu_symbol_visibility: 'hidden',
  # ABI compatibility with autotools/libtool
  version: '0.8.6',
)

sqlite_args = []
if host_machine.system() == 'windows' and get_option('default_library') != 'static'
  sqlite_args += '-DSQLITE_API=__declspec(dllimport)'
endif

depinc = include_directories('.')
sqlite3_dep = declare_dependency(
  compile_args: sqlite_args,
  include_directories: depinc,
  link_with: libsqlite3,
)

install_headers('sqlite3.h')

pkgconfig = import('pkgconfig')
pkgconfig.generate(
  libsqlite3,
  description: 'An embedded SQL database engine',
  version: meson.project_version(),
  extra_cflags: sqlite_args,
)
