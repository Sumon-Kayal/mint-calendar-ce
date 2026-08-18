subdir('icons')

enum_headers = files(src_dir / 'gcal-enums.h')

enums_xml = gnome.mkenums(
  'org.mint.calendar.ce.enums.xml',
         sources: enum_headers,
        comments: '<!-- @comment@ -->',
           fhead: '<schemalist>',
           vhead: '  <@type@ id="org.mint.calendar.ce.@EnumName@">',
           vprod: '    <value nick="@valuenick@" value="@valuenum@"/>',
           vtail: '  </@type@>',
           ftail: '</schemalist>',
  install_header: true,
     install_dir: calendar_schemadir
)

# Desktop files
# Note: This name is registered in gcal-weather-service.c.
desktop_conf = configuration_data()
desktop_conf.set('startup_wm_class', application_id)
desktop = i18n.merge_file(
  type: 'desktop',
  input: configure_file(
    input: files('org.mint.calendar.ce.desktop.in.in'),
    output: 'org.mint.calendar.ce.desktop.in',
    configuration: desktop_conf
  ),
  output: '@0@.desktop'.format(application_id),
  po_dir: po_dir,
  install: true,
  install_dir: calendar_datadir / 'applications'
)

# GNOME Shell search provider
search_provider_conf = configuration_data()
search_provider_conf.set('appid', application_id)
search_provider_conf.set('profile',  profile)
search_provider_conf.set('profile_path',  profile_path)
configure_file(
  configuration: search_provider_conf,
  input: files('org.mint.calendar.ce.search-provider.ini.in'),
  install_dir: calendar_datadir / 'gnome-shell' / 'search-providers',
  output: '@0@.search-provider.ini'.format(application_id)
)

# GSettings schema
schema_conf = configuration_data()
schema_conf.set('GETTEXT_PACKAGE', meson.project_name())

schema = 'org.mint.calendar.ce.gschema.xml'

schema_xml = configure_file(
  input: schema + '.in',
  output: schema,
  install: true,
  install_dir: calendar_schemadir,
  configuration: schema_conf
)

output = 'gschemas.compiled'

# for unit tests - gnome.compile_schemas() only looks in srcdir
compiled_schemas = custom_target(
   output,
    input: schema_xml,
   output: output,
  command: [find_program('glib-compile-schemas'), meson.current_build_dir()],
  depends: enums_xml
)

# DBus service files
service_conf = configuration_data()
service_conf.set('appid', application_id)
service_conf.set('bindir', calendar_bindir)
service = 'org.mint.calendar.ce.service'

configure_file(
          input: service + '.in',
         output: '@0@.service'.format(application_id),
        install: true,
    install_dir: calendar_datadir / 'dbus-1' / 'services',
  configuration: service_conf
)

shell_search_provider_dbus_interfaces = files('shell-search-provider-dbus-interfaces.xml')
