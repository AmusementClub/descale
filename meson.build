project('Descale', 'c', 'cpp',
    default_options: ['buildtype=release', 'b_ndebug=if-release', 'c_std=c99', 'cpp_std=c++14'],
    meson_version: '>=0.51.0',
    version: '8'
)

add_global_arguments(['-D_XOPEN_SOURCE=700'], language: 'c')

cc = meson.get_compiler('c')

gcc_syntax = cc.get_argument_syntax() == 'gcc'
is_windows = host_machine.system() == 'windows'

includedirs = ['include', 'src']

sources = ['src/descale.c']

libs = []

deps = []

libtype = get_option('libtype')

if libtype in ['vapoursynth', 'both']
    sources += ['src/vsplugin.cc']
endif

if libtype in ['avisynth', 'both']
    sources += ['src/avsplugin.c']
endif

# Don't require VS to be installed on Windows,
# so we don't need to have VS fully installed
# in our build environment. Just the header files
# are enough.
if is_windows
    if libtype in ['avisynth', 'both']
        deps += [cc.find_library('AviSynth', dirs: meson.current_source_dir())]
    endif
    if libtype in ['vapoursynth', 'both']  # I'm not sure if it is possible to install
                                           # the _same_ file to multiple directories with meson
        installdir = join_paths(get_option('libdir'), 'vapoursynth')
    else
        installdir = join_paths(get_option('libdir'), 'avisynth')
    endif
    m_dep = []
else
    m_dep = cc.find_library('m', required: false)
    p_dep = cc.find_library('pthread')
    deps += [m_dep, p_dep]
    if libtype in ['vapoursynth', 'both']
        vs = dependency('vapoursynth').partial_dependency(compile_args: true, includes: true)
        deps += [vs]
    endif
    if libtype in ['avisynth', 'both']
        avs = dependency('avisynth')
        deps += [avs]
    endif
    if libtype in ['vapoursynth', 'both']
        installdir = join_paths(vs.get_pkgconfig_variable('libdir'), 'vapoursynth')
    else
        installdir = join_paths(avs.get_pkgconfig_variable('libdir'), 'avisynth')
    endif
endif

if host_machine.cpu_family().startswith('x86')
    add_project_arguments('-DDESCALE_X86', language : 'c')
    if gcc_syntax
        add_project_arguments('-mfpmath=sse', '-msse2', language : 'c')
    endif

    sources += ['src/x86/cpuinfo_x86.c']

    libs += static_library('descale_avx2', 'src/x86/descale_avx2.c',
                dependencies: m_dep,
                c_args: gcc_syntax ? ['-mavx2', '-mfma'] : ['/arch:AVX2'],
                pic: true,
                include_directories: includedirs
            )
endif


shared_module('descale', sources,
    dependencies: deps,
    include_directories: includedirs,
    link_with: libs,
    name_prefix: is_windows ? '' : 'lib',
    install: true,
    install_dir: installdir
)
