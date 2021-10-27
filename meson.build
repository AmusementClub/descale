project('Descale', 'c', 'cpp',
    default_options: ['buildtype=release', 'b_ndebug=if-release', 'c_std=c99', 'cpp_std=c++14'],
    meson_version: '>=0.51.0',
    version: '7'
)

add_global_arguments(['-D_XOPEN_SOURCE=700'], language: 'c')

cc = meson.get_compiler('c')

includedirs = ['include', 'src']

sources = ['src/descale.c', 'src/vsplugin.cc']

libs = []

deps = []

# Don't require VS to be installed on Windows,
# so we don't need to have VS fully installed
# in our build environment. Just the header files
# are enough.
if host_machine.system() == 'windows'
    installdir = join_paths(get_option('libdir'), 'vapoursynth')
    m_dep = []
else
    m_dep = [cc.find_library('m', required: false)]
    vs = dependency('vapoursynth').partial_dependency(compile_args: true, includes: true)
    deps += m_dep + [vs]
    installdir = join_paths(vs.get_pkgconfig_variable('libdir'), 'vapoursynth')
endif

if host_machine.cpu_family().startswith('x86')
    add_project_arguments('-DDESCALE_X86', '-mfpmath=sse', '-msse2', language : 'c')

    sources += ['src/x86/cpuinfo_x86.c']

    libs += static_library('descale_avx2', 'src/x86/descale_avx2.c',
                dependencies: m_dep,
                c_args: ['-mavx2', '-mfma'],
                pic: true,
                include_directories: includedirs
            )
endif


shared_module('descale', sources,
    dependencies: deps,
    include_directories: includedirs,
    link_with: libs,
    name_prefix: 'lib',
    install: true,
    install_dir: installdir
)
