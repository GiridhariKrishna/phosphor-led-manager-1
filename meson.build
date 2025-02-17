project(
    'phosphor-ledmanager', 'cpp',
    version : '1.0.0',
    meson_version: '>=0.57.0',
    default_options: [
        'warning_level=3',
        'werror=true',
        'cpp_std=c++20',
        'buildtype=debugoptimized',
    ]
)

conf_data = configuration_data()
conf_data.set_quoted('BUSNAME', 'xyz.openbmc_project.LED.GroupManager')
conf_data.set_quoted('OBJPATH', '/xyz/openbmc_project/led/groups')
conf_data.set_quoted('LED_JSON_FILE', '/usr/share/phosphor-led-manager/led-group-config.json')
conf_data.set_quoted('SAVED_GROUPS_FILE', '/var/lib/phosphor-led-manager/savedGroups')
conf_data.set_quoted('CALLOUT_FWD_ASSOCIATION', 'callout')
conf_data.set_quoted('CALLOUT_REV_ASSOCIATION', 'fault')
conf_data.set_quoted('ELOG_ENTRY', 'entry')
conf_data.set_quoted('LED_FAULT', 'fault')
conf_data.set_quoted('LAMP_TEST_INDICATOR_FILE', '/var/lib/phosphor-led-manager/lampTestOn')

conf_data.set('CLASS_VERSION', 1)
conf_data.set('LED_USE_JSON', get_option('use-json').enabled())
conf_data.set('USE_LAMP_TEST', get_option('use-lamp-test').enabled())
conf_data.set('MONITOR_OPERATIONAL_STATUS', get_option('monitor-operational-status').enabled())
conf_data.set('IBM_SAI', get_option('monitor-sai-status').enabled())

sdbusplus_dep = dependency('sdbusplus')
sdeventplus_dep = dependency('sdeventplus')
phosphor_dbus_interfaces_dep = dependency('phosphor-dbus-interfaces')
phosphor_logging_dep = dependency('phosphor-logging')

prog_python = find_program('python3', required: true)
realpath_prog = find_program('realpath')

cpp = meson.get_compiler('cpp')
if cpp.has_header('nlohmann/json.hpp')
    nlohmann_json_dep = declare_dependency()
else
    subproject('nlohmann', required: false)
    nlohmann_json_dep = declare_dependency(
        include_directories: [
            'subprojects/nlohmann/single_include',
            'subprojects/nlohmann/single_include/nlohmann',
        ]
    )
endif

if cpp.has_header('CLI/CLI.hpp')
    CLI11_dep = declare_dependency()
else
    CLI11_dep = dependency('CLI11')
endif

# Get Cereal dependency.
cereal_dep = dependency('cereal', required: false)
has_cereal = cpp.has_header_symbol(
    'cereal/cereal.hpp',
    'cereal::specialize',
    dependencies: cereal_dep,
    required: false)
if not has_cereal
    cereal_opts = import('cmake').subproject_options()
    cereal_opts.add_cmake_defines({'BUILD_TESTS': 'OFF'})
    cereal_proj = import('cmake').subproject(
        'cereal',
        options: cereal_opts,
        required: false)
    assert(cereal_proj.found(), 'cereal is required')
    cereal_dep = cereal_proj.dependency('cereal')
endif

deps = [
    CLI11_dep,
    cereal_dep,
    nlohmann_json_dep,
    phosphor_dbus_interfaces_dep,
    phosphor_logging_dep,
    sdbusplus_dep,
    sdeventplus_dep,
]

subdir('manager')
subdir('fault-monitor')

configure_file(output: 'config.h',
    configuration: conf_data
)

install_data(
    'scripts/led-set-all-groups-asserted.sh',
    install_dir: get_option('bindir')
)

build_tests = get_option('tests')
if not build_tests.disabled()
  subdir('test')
endif

install_subdir('configs',
    install_dir: get_option('datadir') / 'phosphor-led-manager',
    strip_directory: true)
