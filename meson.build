project('pktgen', 'C',
	version: '19.05.0-rc0',
	license: 'BSD',
	default_options: ['buildtype=release', 'default_library=static'],
	meson_version: '>= 0.47.1'
)

pktgen_conf = configuration_data()

cmd = run_command('sh', '-c', 'echo $RTE_SDK')
if cmd.returncode() == 0
	rte_sdk = cmd.stdout().strip()
	message('RTE_SDK=' + rte_sdk)
else
  # not found, do fallback
  error('Please set RTE_SDK env')
endif

cmd = run_command('sh', '-c', 'echo $RTE_TARGET')
if cmd.returncode() == 0
	rte_target = cmd.stdout().strip()
	message('RTE_TARGET=' + rte_target)
else
  # not found, do fallback
  error('Please set RTE_TARGET')
endif

dfs_libraries = []
dfs_libdir = []
dfs_incs = []
if get_option('link_dfs') == true
	cmd = run_command('sh', '-c', 'echo $PME_SDK')
	if cmd.returncode() == 0
		pme_sdk = cmd.stdout().strip()
		dfs_libdir = pme_sdk + '/build/lib'
		dfs_incs = include_directories(pme_sdk + '/build/inc')
	elif get_option('dfs_libdir') != ''
		dfs_libdir = get_option('dfs_libdir')
		dfs_incs = include_directories(get_option('dfs_incdir'))
	else
	  # not found, do fallback
	  error('Please set PME_SDK or dfs_libdir in meson_options.txt file')
	endif
	message('PME_SDK=' + pme_sdk)
	message('dfs_libdir=' + dfs_libdir)
endif

# set up some global vars for compiler, platform, configuration, etc.
cc = meson.get_compiler('c')

dpdk_libraries = []
dpdk_static_libraries = []
dpdk_drivers = []
dpdk_extra_ldflags = []
dpdk_app_link_libraries = []
dpdk_dep = []
pktgen_shared_libraries = []
pktgen_static_libraries = []
machine_args = ['-mavx']

pver = meson.project_version().split('.')
major_version = '@0@.@1@'.format(pver.get(0), pver.get(1))

add_project_arguments('-include', 'rte_config.h', language: 'c')
add_project_arguments('-DALLOW_EXPERIMENTAL_API', language: 'c')
add_project_arguments('-D_GNU_SOURCE', '-O3', '-g', language: 'c')

dpdk_incs = include_directories(rte_sdk + '/' + rte_target + '/include')
dpdk_libdir = rte_sdk + '/' + rte_target + '/lib'
if get_option('lib_dir') != ''
	if run_command('[', '-d', dpdk_libdir + '/' + get_option('lib_dir'), ']').returncode() == 0
		dpdk_libdir = dpdk_libdir + '/' + get_option('lib_dir')
	endif
endif

message('Using dpdk_libdir = ' + dpdk_libdir)

subdir('lib')
subdir('app')

pkg = import('pkgconfig')
pkg.generate(name: meson.project_name(),
        filebase: 'lib' + meson.project_name().to_lower(),
        version: meson.project_version(),
        libraries: [dpdk_libraries, dfs_libraries],
        libraries_private: dpdk_drivers + dpdk_static_libraries +
                        ['-Wl,-Bdynamic'] + dpdk_extra_ldflags,
        description: '''The Data Plane Development Kit (DPDK).
Note that CFLAGS might contain an -march flag higher than typical baseline.
This is required for a number of static inline functions in the public headers.''',
        subdirs: [get_option('include_subdir_arch'), '.'],
        extra_cflags: ['-include', 'rte_config.h'] + machine_args
)

# final output, list all the libs and drivers to be built
# this does not affect any part of the build, for information only.
output_message = '\n=================\nLibraries Enabled\n=================\n'
output_message += '\nlibs:\n\t'
output_count = 0
foreach lib:enabled_libs
        output_message += lib + ', '
        output_count += 1
        if output_count == 8
                output_message += '\n\t'
                output_count = 0
        endif
endforeach
message(output_message + '\n')
