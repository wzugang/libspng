project(
    'libspng', 'c',
    version : '0.6.1',
    license : [ 'bsd', 'libpng2' ],
    default_options : ['c_std=c99']
)

cc = meson.get_compiler('c')

spng_inc = include_directories('spng')

if get_option('use_miniz') == true
    add_project_arguments('-DSPNG_USE_MINIZ', language : 'c')
    zlib_dep = dependency('miniz', fallback : [ 'miniz', 'miniz_dep'])
else
    if cc.get_define('__ANDROID__') != ''
        zlib_dep = cc.find_library('z')
    else
        zlib_dep = dependency('zlib', fallback : ['zlib', 'zlib_dep'], static : get_option('static_zlib'))
    endif
endif

m_dep = cc.find_library('m', required : false)

spng_deps = [ zlib_dep, m_dep ]

if get_option('multithreading').enabled()
    spng_deps += dependency('threads')
    add_project_arguments('-DSPNG_MULTITHREADING', language : 'c')
endif

if get_option('enable_opt') == false
    add_project_arguments('-DSPNG_DISABLE_OPT', language : 'c')
endif

if get_option('default_library') == 'static'
    spng_args = '-DSPNG_STATIC'
    static_subproject = meson.is_subproject()
else
    spng_args = []
    static_subproject = false
endif

# Check for GNU target_clones attribute
if host_machine.cpu_family() == 'x86_64'
# This will only be available for GCC with glibc
# for the foreseeable future, any compile error,
# including unknown compile args means it's not supported
    if cc.compiles(files('tests/target_clones.c'), args : '-Werror', name : 'have target_clones')
        add_project_arguments('-DSPNG_ENABLE_TARGET_CLONES', language : 'c')
    endif
endif

spng_src = files('spng/spng.c')

spng_lib = library(
    'spng', spng_src,
    c_args : spng_args,
    dependencies : spng_deps,
    version : '0.6.1',
    install : not static_subproject)

spng_dep = declare_dependency(
    link_with : spng_lib,
    compile_args : spng_args,
    include_directories : spng_inc,
    version : meson.project_version())

example_src = files('examples/example.c')

example_exe = executable('example', example_src, dependencies : spng_dep)

if meson.version().version_compare('>= 0.54.0')
    meson.override_dependency('spng', spng_dep)
endif

if not static_subproject
    install_headers('spng/spng.h')

    pkg = import('pkgconfig')
    pkg.generate(
        name : 'spng',
        libraries : spng_lib,
        extra_cflags : spng_args,
        version : meson.project_version(),
        description : 'PNG decoding and encoding library',
        libraries_private : [ '-lm', '-lz' ])
endif

if get_option('dev_build') == true
    subdir('tests')
endif

if get_option('benchmarks') == true
    if meson.version().version_compare('>= 0.54.0')
        spngt = subproject('spngt')
    else
        warning('benchmarks require Meson >= 0.54.0')
    endif
endif