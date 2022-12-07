
project('GAOL','c','cpp',version: '4.2.3',
    default_options: ['cpp_std=c++11','c_std=c99'])
major_version=4
minor_version=2
micro_version=3

## Main compiler and linker configs

cpp_comp = meson.get_compiler('cpp')
c_comp = meson.get_compiler('c')

extra_args=[]
if cpp_comp.get_id() == 'clang'
  extra_args = ['-std=c++11']
else
  extra_args = ['-std=c++11']
endif


# Options
debug = get_option('enable-debug')
simd = get_option('enable-simd')
asm = get_option('enable-asm')
preserve_rounding = get_option('enable-preserve-rounding')
relations = get_option('enable-relations')
exception = get_option('enable-exception')
mathlib = get_option('with-mathlib')


## dependencies
#apmathlib_dep = cpp_comp.find_library('ultim', required : false)
#crmlibm_dep = cpp_comp.find_library('crlibm', required : false)
#mathlib_dep=apmathlib_dep
#if not apmathlib_dep.found() and crlibm_dep.found()
#  mathlib_dep = crlibm_dep
#elif not apmathlib_dep.found() and not crlibm_dep.found()
#  error('Missing Mathematical library! Expecting APMathlib or Crlibm')
#endif
if mathlib == 'apmathlib'
  mathlib_dep = cpp_comp.find_library('ultim', required : true)
else
  mathlib_dep = cpp_comp.find_library('crlibm', required : true)
endif
m_dep = cpp_comp.find_library('m', required : true)

deps = [mathlib_dep,m_dep]


## Config
os_unix   = false
os_linux  = false
os_win32  = false
os_darwin = false
os_cygwin = false
os_sun    = false

if host_machine.system() == 'darwin'
  os_darwin = true
elif host_machine.system() == 'windows'
  os_win32 = true
elif host_machine.system() == 'linux'
  os_linux = true
elif host_machine.system() == 'cygwin'
  os_cygwin = true
elif host_machine.system() == 'sunos'
  os_sun = true
else
  warning('Unkown host system!')
endif
os_unix = not os_win32


arm64 = false
x86 = false
x86_64 = false
if host_machine.cpu() == 'aarch64'
  arm64 = true
elif host_machine.cpu() == 'x86'
  x86 = true
elif host_machine.cpu() == 'x86_64'
  x86_64 = true
endif

if not(arm64 or x86 or x86_64)
  error('Unsupported cpu arch: '+host_machine.cpu())
endif

conf_data = configuration_data()
conf_data.set('version', meson.project_version())
conf_data.set('project_name', meson.project_name())
conf_data.set('project_full_name', meson.project_name())
conf_data.set('bug_report', 'frederic.goualard@univ-nantes.fr')
conf_data.set('project_url','https://github.com/goualard-f/GAOL')

conf_data.set('GAOL_MAJOR_VERSION',major_version)
conf_data.set('GAOL_MINOR_VERSION',minor_version)
conf_data.set('GAOL_MICRO_VERSION',micro_version)
conf_data.set('version',meson.project_version())

if mathlib == 'apmathlib'
  conf_data.set('GAOL_USING_APMATHLIB', 1)
elif mathlib == 'crlibm'
  conf_data.set('GAOL_USING_CRLIBM', 1)
endif
if relations == 'certainly'
  conf_data.set('GAOL_CERTAINLY_RELATIONS',1)
elif relations == 'possibly'
  conf_data.set('GAOL_POSSIBLY_RELATIONS',1)
elif relations == 'set'
  conf_data.set('GAOL_SET_RELATIONS',1)
else
  error('Wrong value for relations, expecting set, possibly or certainly, but get: '+relations)
endif
if exception
  conf_data.set('GAOL_EXCEPTIONS_ENABLED',1)
endif
if debug
  conf_data.set('GAOL_DEBUGGING',1)
endif
if preserve_rounding
  conf_data.set('GAOL_PRESERVE_ROUNDING',1)
endif
if asm
  conf_data.set('GAOL_USING_ASM',1)
endif


conf_data.set('AARCH64_LINUX',(arm64 and (os_linux or os_darwin)).to_int())
conf_data.set('AC_APPLE_UNIVERSAL_BUILD', (os_darwin and not arm64).to_int())
conf_data.set('IX86_CYGWIN', (os_cygwin and (x86 or x86_64)).to_int())
conf_data.set('IX86_MACOSX', ((x86 or x86_64) and os_darwin).to_int())
conf_data.set('IX86_LINUX', (x86 and os_linux).to_int())
conf_data.set('X86_64BITS', (arm64 or x86_64).to_int())

conf_data.set('STDC_HEADERS',1)
#conf_data.set('const',0)

check_headers = [
  'dlfcn.h',
  'inttypes.h',
  'stdint.h',
  'stdio.h',
  'stdlib.h',
  'strings.h',
  'string.h',
  'sys/stat.h',
  'sys/types.h',
  'unistd.h',
  'cassert',
  'fenv.h',
  'float.h',
  'limits',
  'stddef.h',
  'sys/resource.h',
  'sys/time.h',
  'time.h',
  'stdbool.h'
]

foreach h : check_headers
  if cpp_comp.has_header(h)
    conf_data.set('HAVE_' + h.underscorify().to_upper(), 1)
  endif
endforeach

if mathlib=='apmathlib'
  if cpp_comp.has_header('MathLib.h')
    conf_data.set('HAVE_' + 'MathLib.h'.underscorify().to_upper(), 1)
  endif
elif mathlib=='crlibm'
  if cpp_comp.has_header('crlibm.h')
    conf_data.set('HAVE_' + 'crlibm.h'.underscorify().to_upper(), 1)
  endif
endif

check_functions = [
  'clock',
  'fesetround',
  'finite',
  'floor',
  'getrusage',
  'localeconv',
  'malloc',
  'memset',
  'nextafter',
  'pow',
  'realloc',
  'rint',
  'round',
  'sqrt',
  '__signbit'
]

foreach f : check_functions
  if cpp_comp.has_function(f)
    conf_data.set('HAVE_' + f.underscorify().to_upper(), 1)
  endif
endforeach

conf_data.set('HAVE__BOOL',cpp_comp.has_type('_Bool'))
conf_data.set('SIZEOF_INT',cpp_comp.sizeof('int'))
conf_data.set('SIZEOF_LONG',cpp_comp.sizeof('long'))
conf_data.set('SIZEOF_LONG_LONG_INT',cpp_comp.sizeof('long long int'))

if cpp_comp.has_argument('-fvisibility')
  conf_data.set('HAVE_VISIBILITY_OPTIONS',1)
  add_global_arguments(['-fvisibility'], language:'c')
  add_global_arguments(['-fvisibility'], language:'cpp')
endif
if cpp_comp.has_argument('-frounding-math')
  conf_data.set('HAVE_ROUNDING_MATH_OPTION',1)
  add_global_arguments(['-frounding-math'], language:'c')
  add_global_arguments(['-frounding-math'], language:'cpp')
endif


if cpp_comp.has_argument('-msse2')
  conf_data.set('USING_SSE2_INSTRUCTIONS',1)
  add_global_arguments(['-msse2'], language:'c')
  add_global_arguments(['-msse2'], language:'c')
endif

if cpp_comp.has_argument('-msse3')
  conf_data.set('USING_SSE3_INSTRUCTIONS',1)
  add_global_arguments(['-msse3'], language:'c')
  add_global_arguments(['-msse3'], language:'c')
endif

flex = find_program('flex', required: true)
bison = find_program('bison', required: true)


subdir('gaol')

incdir = include_directories(['gaol','gaol/sysdeps'])

# if flex.found()
#   lgen = generator(flex,
#     output : '@BASENAME@.cpp',
#     arguments : ['-o', '@OUTPUT@', '@INPUT@'])
#   lfiles = lgen.process('gaol_interval_lexer.lpp')
# endif
# if bison.found()
#   pgen = generator(bison,
#     output : ['@BASENAME@.cpp', '@BASENAME@.h'],
#     arguments : ['@INPUT@', '--defines=@OUTPUT1@', '--output=@OUTPUT0@'])
#   pfiles = pgen.process('gaol_interval_parser.ypp')
# endif
lexer_files=['gaol/gaol_interval_lexer.cpp']
parser_files=['gaol/gaol_interval_parser.cpp','gaol/gaol_interval_parser.h']

INTERVAL2_SOURCE=[]
INTERVAL2_HEADER=[]
if simd
  INTERVAL2_SOURCE+=['gaol/gaol_interval2f.cpp']
  INTERVAL2_HEADER+=['gaol/gaol_interval2f.h']
endif

if mathlib=='apmathlib'
  math_header = run_command('ln','-f','gaol/gaol_double_op_apmathlib.h','gaol/gaol_double_op.h', check: true)
elif mathlib=='crlibm'
  math_header = run_command('ln','-f','gaol/gaol_double_op_crlibm.h','gaol/gaol_double_op.h', check: true)
else
  math_header = run_command('ln','-f','gaol/gaol_double_op_m.h','gaol/gaol_double_op.h', check: true)
endif

#
fpu_source='gaol/gaol_interval_fpu.cpp'
if simd
  fpu_source='gaol/gaol_interval_sse.cpp'
endif
fpu_target = custom_target('gaol_fpu',
  output: 'gaol_interval_fpu_.cpp',
  input: fpu_source,
  command: ['cp','@INPUT@','@OUTPUT@'],
  # install: true,
  # install_dir: 'gaol'
)


gaol_SOURCES=['gaol/gaol_interval.cpp', 'gaol/gaol_profile.cpp', 'gaol/gaol_common.cpp',
'gaol/gaol_parser.cpp', 'gaol/gaol_expression.cpp', 'gaol/gaol_port.cpp',
'gaol/gaol_exceptions.cpp', 'gaol/gaol_init_cleanup.cpp', 'gaol/s_nextafter.c',
  'gaol/gaol_exact.c', 'gaol/gaol_intervalf.cpp']
foreach f : lexer_files
  if f.endswith('.cpp')
    gaol_SOURCES+=[f]
  endif
endforeach
foreach f : parser_files
if f.endswith('.cpp')
  gaol_SOURCES+=[f]
endif
endforeach

gaol_HEADERS=['gaol/gaol', 'gaol/gaol_assert.h', 'gaol/gaol_exact.h', 'gaol/gaol_interval.h', 'gaol/gaol_common.h', 'gaol/gaol_exceptions.h',
	'gaol/gaol_config.h', 'gaol/gaol_expression.h', 'gaol/gaol_limits.h',
	'gaol/gaol_expr_eval.h', 'gaol/gaol_expr_visitor.h', 'gaol/gaol_parameters.h',
	'gaol/gaol_double_op_apmathlib.h', 'gaol/gaol_double_op_crlibm.h', 'gaol/gaol_flags.h', 'gaol/gaol_parser.h', 'gaol/gaol_double_op.h',
	'gaol/gaol_port.h', 'gaol/gaol_fpu.h', 'gaol/gaol_fpu_fenv.h', 'gaol/gaol_profile.h', 'gaol/gaol.h',
	'gaol/gaol_eval_stack.h', 'gaol/gaol_init_cleanup.h', 'gaol/gaol_version.h', 'gaol/gaol_allocator.h',
	'gaol/gaol_interval_fpu.h', 'gaol/gaol_interval_sse.h', 'gaol/gaol_intervalf.h']
foreach f : lexer_files
  if f.endswith('.h')
    gaol_HEADERS+=[f]
  endif
endforeach
foreach f : parser_files
if f.endswith('.h')
  gaol_HEADERS+=[f]
endif
endforeach


EXTRA_DIST=['gaol/sysdeps/gaol_exact_c99.h', 'gaol/sysdeps/gaol_exact_msvc.h', 'gaol/s_nextafter.c',
	'gaol/gaol_interval_lexer.lpp', 'gaol/gaol_interval_parser.ypp', 'gaol/dllmain.cpp', 'gaol/gaol_config_mingw.h',
	'gaol/gaol_config_msvc.h', 'gaol/gaol_version_msvc.h', 'gaol/gaol_configuration_.h.in', 'gaol/gaol_interval_sse.cpp']

gaollib = both_libraries('gaol', sources: gaol_SOURCES,
   dependencies: deps,
   install: true,
   extra_files: EXTRA_DIST,
   include_directories: incdir
)



install_headers(gaol_HEADERS,
  subdir: 'gaol'
)


subdir('check')
subdir('examples')
subdir('doc')
