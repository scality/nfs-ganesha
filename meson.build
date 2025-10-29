project('nfs-ganesha', 'c', 'cpp',
  version: '9.0',
  default_options: [
    'buildtype=debugoptimized',
    'b_ndebug=if-release',
    'b_vscrt=crt_static',
    'cpp_std=c++17'
  ]
)


###
# Compiler flags
###
c_compiler = meson.get_compiler('c')
# Base arguments for nfs-ganesha
nfs_ganesha_base_args = [
  '-D_LARGEFILE64_SOURCE',
  '-D_FILE_OFFSET_BITS=64',
  '-D_GNU_SOURCE',
  '-D__USE_GNU',
  '-fno-strict-aliasing',
  '-Wno-implicit-fallthrough',
  '-Wno-format-nonliteral',
  '-Wno-missing-declarations',
  '-Wno-shadow',
  '-Wno-pointer-sign',
  '-Wno-error=suggest-attribute=format',
  '-Wno-error=pointer-arith',
  '-Wno-error',
  '-Wno-unused-variable',
]

# Override -fvisibility=hidden with -fvisibility=default for release builds
if get_option('buildtype') == 'release'
  nfs_ganesha_base_args += ['-fvisibility=default']
endif

nfs_ganesha_c_args = c_compiler.get_supported_arguments(nfs_ganesha_base_args)

###
# Dependencies
###
acl_dep = dependency('libacl',
  required : false,
  disabler: true)
krb5_dep = dependency('krb5',
  modules : ['gssapi'],
  required : false,
  disabler: true)
gssapi_krb5_dep = c_compiler.find_library('libgssapi_krb5',
  required: false,
  disabler: true)
threads_dep = dependency('threads',
 required : true,
 disabler: true)
dbus_dep = dependency('dbus-1',
 required : false,
 disabler: true)
systemd_dep = dependency('libsystemd',
 required : false,
 disabler: true)
urcu_bp_dep = dependency('liburcu-bp',
 required : false,
 disabler: true)
nfsidmap_dep = dependency('libnfsidmap',
 required : false,
 disabler: true)
caps_dep = dependency('libcap',
 required : false,
 disabler: true)
blkid_dep = dependency('blkid',
 required : false,
 disabler: true)
bsd_dep = dependency('libbsd',
 required : true,
 disabler: true)
uuid_dep = dependency('uuid',
 required : false,
 disabler: true)
dl_dep = c_compiler.find_library('dl',
 required : false,
 disabler: true)

if get_option('allocator') == 'system'
  allocator_dep = []
else
  # For tcmalloc, prefer tcmalloc_minimal if available
  if get_option('allocator') == 'tcmalloc'
    allocator_dep = c_compiler.find_library('tcmalloc_minimal', required: false)
    if not allocator_dep.found()
      allocator_dep = c_compiler.find_library('tcmalloc', required: true)
    endif
  else
    allocator_dep = c_compiler.find_library(get_option('allocator'), required: true)
  endif
endif


###
# Programs
###
bison = find_program('bison',
  required : false,
  disabler: true)
flex = find_program('flex',
  required : false,
  disabler: true)

###
# Configuration header
###
conf = configuration_data()
versions = meson.project_version().split('.')
maj = versions[0].to_int()
min = versions[1].to_int()
extra = '-scality'
conf.set('GANESHA_VERSION_MAJOR', maj)
conf.set('GANESHA_VERSION_MINOR', '.'+min.to_string())
if versions.length() > 2
  conf.set('GANESHA_VERSION_PATCH', versions[2])
else
  conf.set('GANESHA_VERSION_PATCH', '')
endif
conf.set('GANESHA_VERSION', '"'+ meson.project_version()+ '"')
conf.set('GANESHA_BUILD_RELEASE', 1)
conf.set('VERSION', 'GANESHA_VERSION')
conf.set('VERSION_COMMENT', '"Ganesha file server is 64 bits compliant and supports NFS v3,4.0,4.1 (pNFS) and 9P"')
conf.set('FSAL_MODULE_LOC', '"/opt/scality/nfs-ganesha/lib64/ganesha"')

if krb5_dep.found()
  if c_compiler.check_header('gssapi/gssapi.h')
    conf.set('_HAVE_GSSAPI', 1)
  endif
  conf.set('HAVE_KRB5', 1)
endif
if nfsidmap_dep.found()
  conf.set('USE_NFSIDMAP', 1)
endif
if dbus_dep.found()
  conf.set('USE_DBUS', 1)
endif
if systemd_dep.found()
  conf.set('USE_SYSTEMD', 1)
endif
if caps_dep.found()
  conf.set('USE_CAPS', 1)
endif
if blkid_dep.found()
  conf.set('USE_BLKID', 1)
endif
conf.set('_USE_NFS3', 1)
conf.set('_USE_NLM', 1)
conf.set('_USE_RQUOTA', 1)
conf.set('USE_NFSACL3', 1)
conf.set('USE_MONITORING', 1)
conf.set('NFS_GANESHA', 1)
if get_option('rpcbind')
  conf.set('RPCBIND', 1)
endif
conf.set('LINUX', 1)
if c_compiler.check_header('string.h')
  conf.set('HAVE_STRING_H', 1)
  if c_compiler.has_function('strnlen')
    conf.set('HAVE_STRNLEN', 1)
  endif
endif
conf.set('LITTLEEND', 1)
conf.set('HAVE_DAEMON', 1)
conf.set('GANESHA_CONFIG_PATH', '"/opt/scality/nfs-ganesha/etc/ganesha/ganesha.conf"')
conf.set('GANESHA_PIDFILE_PATH', '"/var/run/ganesha/ganesha.pid"')
conf.set('NFS_V4_RECOV_ROOT', '"/var/lib/nfs/ganesha"')
conf.set('NFS_V4_RECOV_DIR', '"v4recov"')
conf.set('NFS_V4_OLD_DIR', '"v4old"')
conf.set('DEFAULT_NFS_CCACHE_DIR', '"/var/run/ganesha"')

configure_file(output : 'config.h',
               configuration : conf)

###
# NTIRPC
###
subdir('src/libntirpc')


###
# Source files
###
nfs_ganesha_inc_dir = [
  'src/include',
  'src/libntirpc/ntirpc',
  'src/libntirpc/src/monitoring/include',
  'src/include/os/linux',
  'src/monitoring/include',
]
main_deps = [
  bsd_dep,
  dbus_dep,
  systemd_dep,
  dl_dep,
  threads_dep,
  urcu_bp_dep,
]

# AVL {{{
avl_files = files( # {{{
  'src/avl/avl.c',
  'src/avl/bst.c',
  'src/avl/rb.c',
  'src/avl/splay.c',
) # }}}
avl_lib = static_library('avl', avl_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/avl'],
  dependencies : main_deps,
  c_args : nfs_ganesha_c_args,
  install : false
) # }}}
# monitoring {{{
monitoring_files = files( # {{{
  'src/monitoring/dynamic_metrics.cc',
  'src/monitoring/prometheus_exposer.cc',
) # }}}
monitoring_lib = static_library('monitoring', monitoring_files,
  include_directories: [
    'src/libntirpc/src/monitoring/prometheus-cpp-lite/core/include',
    'src/libntirpc/src/monitoring/include',
    'src/monitoring/include',
    'src/include',
  ],
  dependencies : main_deps,
  link_with : ntirpc_monitoring_lib,
  c_args : nfs_ganesha_c_args,
  install : false
) # }}}
# MainNFSD {{{
mainnfsd_files = files( # {{{
  'src/MainNFSD/nfs_admin_thread.c',
  'src/MainNFSD/nfs_init.c',
  'src/MainNFSD/nfs_lib.c',
  'src/MainNFSD/nfs_reaper_thread.c',
  'src/MainNFSD/nfs_rpc_callback.c',
  'src/MainNFSD/nfs_rpc_dispatcher_thread.c',
  'src/MainNFSD/nfs_rpc_tcp_socket_manager_thread.c',
  'src/MainNFSD/nfs_metrics.c',
  'src/MainNFSD/nfs_worker_thread.c',
  'src/FSAL/fsal_convert.c',
  'src/FSAL/commonlib.c',
  'src/FSAL/fsal_manager.c',
  'src/FSAL/access_check.c',
  'src/FSAL/fsal_config.c',
  'src/FSAL/default_methods.c',
  'src/FSAL/common_pnfs.c',
  'src/FSAL/posix_acls.c',
  'src/FSAL/fsal_destroyer.c',
  'src/FSAL/fsal_helper.c',
  'src/FSAL_UP/fsal_up_top.c',
  'src/FSAL_UP/fsal_up_async.c',
  'src/hashtable/hashtable.c',
  'src/log/display.c',
  'src/log/log_functions.c',
  'src/support/client_mgr.c',
  'src/support/murmur3.c',
  'src/support/city.c',
  'src/support/uid2grp.c',
  'src/support/uid2grp_cache.c',
  'src/support/netgroup_cache.c',
  'src/support/nfs4_acls.c',
  'src/support/nfs_creds.c',
  'src/support/nfs_filehandle_mgmt.c',
  'src/support/nfs_read_conf.c',
  'src/support/nfs_convert.c',
  'src/support/nfs_ip_name.c',
  'src/support/ds.c',
  'src/support/exports.c',
  'src/support/fridgethr.c',
  'src/support/delayed_exec.c',
  'src/support/bsd-base64.c',
  'src/support/server_stats.c',
  'src/support/export_mgr.c',
  'src/support/nfs4_fs_locations.c',
  'src/support/xprt_handler.c',
  'src/support/refstr.c',
  'src/support/ip_utils.c',
  'src/os/linux/subr.c',
) # }}}
mainnfsd_lib = static_library('mainnfsd', mainnfsd_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/MainNFSD'],
  dependencies : main_deps,
  c_args : nfs_ganesha_c_args,
  link_with : [
    ntirpc_lib,
    ntirpc_monitoring_lib,
  ],
  install : false
) # }}}
# config_parsing {{{
config_parser_c = custom_target('config_parser_c',
  input : 'src/config_parsing/conf_yacc.y',
  output : 'conf_yacc.c',
  command : [bison, '-d', '-pganesha_yy', '-o','@OUTPUT@', '@INPUT@'],
  build_by_default : true
)
config_parser_lib = static_library('parser_scanner',
  config_parser_c,
  include_directories: [nfs_ganesha_inc_dir, 'src/config_parsing', '.'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
)
config_scanner_c = custom_target('config_scanner_c',
  input : 'src/config_parsing/conf_lex.l',
  output : 'conf_lex.c',
  depends: [config_parser_c],
  command : [flex, '-Pganeshun_yy', '-o', '@OUTPUT@', '@INPUT@'],
  build_by_default : true
)
config_scanner_lib = static_library('scanner_scanner',
  config_scanner_c,
  include_directories: [nfs_ganesha_inc_dir, 'src/config_parsing', '.'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  link_with : [ config_parser_lib ],
  install : false
)
config_parsing_files = files( # {{{
  'src/config_parsing/analyse.c',
  'src/config_parsing/config_parsing.c',
  'src/config_parsing/conf_url.c',
) # }}}
config_parsing_lib = static_library('config_parsing',
  config_parsing_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/config_parsing', '.'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  link_with : [ config_parser_lib, config_scanner_lib ],
  install : false
) # }}}
# SAL {{{
sal_files = files( # {{{
  'src/SAL/sal_metrics.c',
  'src/SAL/state_async.c',
  'src/SAL/state_lock.c',
  'src/SAL/state_share.c',
  'src/SAL/state_misc.c',
  'src/SAL/state_layout.c',
  'src/SAL/state_deleg.c',
  'src/SAL/nfs4_clientid.c',
  'src/SAL/nfs4_state.c',
  'src/SAL/nfs4_state_id.c',
  'src/SAL/nfs4_lease.c',
  'src/SAL/nfs4_recovery.c',
  'src/SAL/nfs41_session_id.c',
  'src/SAL/nfs4_owner.c',
  'src/SAL/recovery/recovery_fs.c',
  'src/SAL/recovery/recovery_fs_ng.c',
  'src/SAL/nlm_owner.c',
  'src/SAL/nlm_state.c',
) # }}}
sal_lib = static_library('sal', sal_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/SAL'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# Idmapper {{{
idmap_files = files( # {{{
  'src/idmapper/idmapper.c',
  'src/idmapper/idmapper_cache.c',
  'src/idmapper/idmapper_negative_cache.c',
  'src/idmapper/idmapper_monitoring.c',
  'src/idmapper/pwnam_wrappers.c',
  'src/idmapper/sss_nss_idmap.c',
) # }}}
idmap_lib = static_library('idmap', idmap_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/idmapper'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# DBUS {{{
dbus_files = files( # {{{
   'src/dbus/dbus_server.c',
   'src/dbus/properties_handler.c',
   'src/dbus/signal_handler.c',
   'src/dbus/dbus_heartbeat.c',
) # }}}
dbus_lib = static_library('dbus', dbus_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/dbus'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# }}}
# RPCAL {{{
rpcal_files = files( # {{{
  'src/RPCAL/nfs_dupreq.c',
  'src/RPCAL/rpc_tools.c',
  'src/RPCAL/gss_credcache.c',
  'src/RPCAL/gss_extra.c',
  'src/RPCAL/connection_manager.c',
  'src/RPCAL/connection_manager_metrics.c',
) # }}}
rpcal_lib = static_library('rpcal', rpcal_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/RPCAL'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps + [gssapi_krb5_dep],
  install : false
) # }}}
# Protocols / NFS {{{
nfs_files = files( # {{{
  'src/Protocols/NFS/mnt_Dump.c',
  'src/Protocols/NFS/mnt_Export.c',
  'src/Protocols/NFS/mnt_Mnt.c',
  'src/Protocols/NFS/mnt_Null.c',
  'src/Protocols/NFS/mnt_Umnt.c',
  'src/Protocols/NFS/mnt_UmntAll.c',
  'src/Protocols/NFS/nfs_null.c',
  'src/Protocols/NFS/nfs4_Compound.c',
  'src/Protocols/NFS/nfs4_op_access.c',
  'src/Protocols/NFS/nfs4_op_allocate.c',
  'src/Protocols/NFS/nfs4_op_bind_conn.c',
  'src/Protocols/NFS/nfs4_op_close.c',
  'src/Protocols/NFS/nfs4_op_commit.c',
  'src/Protocols/NFS/nfs4_op_create.c',
  'src/Protocols/NFS/nfs4_op_create_session.c',
  'src/Protocols/NFS/nfs4_op_delegpurge.c',
  'src/Protocols/NFS/nfs4_op_delegreturn.c',
  'src/Protocols/NFS/nfs4_op_destroy_session.c',
  'src/Protocols/NFS/nfs4_op_exchange_id.c',
  'src/Protocols/NFS/nfs4_op_free_stateid.c',
  'src/Protocols/NFS/nfs4_op_getattr.c',
  'src/Protocols/NFS/nfs4_op_getdeviceinfo.c',
  'src/Protocols/NFS/nfs4_op_getdevicelist.c',
  'src/Protocols/NFS/nfs4_op_getfh.c',
  'src/Protocols/NFS/nfs4_op_illegal.c',
  'src/Protocols/NFS/nfs4_op_layoutcommit.c',
  'src/Protocols/NFS/nfs4_op_layoutget.c',
  'src/Protocols/NFS/nfs4_op_layoutreturn.c',
  'src/Protocols/NFS/nfs4_op_link.c',
  'src/Protocols/NFS/nfs4_op_lock.c',
  'src/Protocols/NFS/nfs4_op_lockt.c',
  'src/Protocols/NFS/nfs4_op_locku.c',
  'src/Protocols/NFS/nfs4_op_lookup.c',
  'src/Protocols/NFS/nfs4_op_lookupp.c',
  'src/Protocols/NFS/nfs4_op_nverify.c',
  'src/Protocols/NFS/nfs4_op_open.c',
  'src/Protocols/NFS/nfs4_op_open_confirm.c',
  'src/Protocols/NFS/nfs4_op_open_downgrade.c',
  'src/Protocols/NFS/nfs4_op_openattr.c',
  'src/Protocols/NFS/nfs4_op_putfh.c',
  'src/Protocols/NFS/nfs4_op_putpubfh.c',
  'src/Protocols/NFS/nfs4_op_putrootfh.c',
  'src/Protocols/NFS/nfs4_op_read.c',
  'src/Protocols/NFS/nfs4_op_readdir.c',
  'src/Protocols/NFS/nfs4_op_readlink.c',
  'src/Protocols/NFS/nfs4_op_reclaim_complete.c',
  'src/Protocols/NFS/nfs4_op_release_lockowner.c',
  'src/Protocols/NFS/nfs4_op_remove.c',
  'src/Protocols/NFS/nfs4_op_rename.c',
  'src/Protocols/NFS/nfs4_op_renew.c',
  'src/Protocols/NFS/nfs4_op_restorefh.c',
  'src/Protocols/NFS/nfs4_op_savefh.c',
  'src/Protocols/NFS/nfs4_op_secinfo.c',
  'src/Protocols/NFS/nfs4_op_secinfo_no_name.c',
  'src/Protocols/NFS/nfs4_op_sequence.c',
  'src/Protocols/NFS/nfs4_op_set_ssv.c',
  'src/Protocols/NFS/nfs4_op_setattr.c',
  'src/Protocols/NFS/nfs4_op_xattr.c',
  'src/Protocols/NFS/nfs4_op_setclientid.c',
  'src/Protocols/NFS/nfs4_op_setclientid_confirm.c',
  'src/Protocols/NFS/nfs4_op_destroy_clientid.c',
  'src/Protocols/NFS/nfs4_op_test_stateid.c',
  'src/Protocols/NFS/nfs4_op_verify.c',
  'src/Protocols/NFS/nfs4_op_write.c',
  'src/Protocols/NFS/nfs4_pseudo.c',
  'src/Protocols/NFS/nfs_proto_tools.c',
  'src/Protocols/NFS/nfs3_access.c',
  'src/Protocols/NFS/nfs3_commit.c',
  'src/Protocols/NFS/nfs3_create.c',
  'src/Protocols/NFS/nfs3_fsinfo.c',
  'src/Protocols/NFS/nfs3_fsstat.c',
  'src/Protocols/NFS/nfs3_getattr.c',
  'src/Protocols/NFS/nfs3_link.c',
  'src/Protocols/NFS/nfs3_lookup.c',
  'src/Protocols/NFS/nfs3_mkdir.c',
  'src/Protocols/NFS/nfs3_mknod.c',
  'src/Protocols/NFS/nfs3_pathconf.c',
  'src/Protocols/NFS/nfs3_read.c',
  'src/Protocols/NFS/nfs3_readdir.c',
  'src/Protocols/NFS/nfs3_readdirplus.c',
  'src/Protocols/NFS/nfs3_readlink.c',
  'src/Protocols/NFS/nfs3_remove.c',
  'src/Protocols/NFS/nfs3_rename.c',
  'src/Protocols/NFS/nfs3_rmdir.c',
  'src/Protocols/NFS/nfs3_setattr.c',
  'src/Protocols/NFS/nfs3_symlink.c',
  'src/Protocols/NFS/nfs3_write.c',
  'src/Protocols/NFS/nfs4_cb_Compound.c',
) # }}}
nfs_lib = static_library('nfs', nfs_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/Protocols/NFS'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# Protocols / XDR {{{
xdr_files = files( # {{{
  'src/Protocols/XDR/xdr_mount.c',
  'src/Protocols/XDR/xdr_nfs23.c',
  'src/Protocols/XDR/xdr_nfsv41.c',
  'src/Protocols/XDR/xdr_rquota.c',
  'src/Protocols/XDR/xdr_nlm4.c',
  'src/Protocols/XDR/xdr_nsm.c',
  'src/Protocols/XDR/xdr_nfsacl.c',
) # }}}
xdr_lib = static_library('xdr', xdr_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/Protocols/XDR'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# Protocols / NLM {{{
nlm_files = files( # {{{
  'src/Protocols/NLM/nlm_Cancel.c',
  'src/Protocols/NLM/nlm_Free_All.c',
  'src/Protocols/NLM/nlm_Granted_Res.c',
  'src/Protocols/NLM/nlm_Lock.c',
  'src/Protocols/NLM/nlm_Null.c',
  'src/Protocols/NLM/nlm_Share.c',
  'src/Protocols/NLM/nlm_Sm_Notify.c',
  'src/Protocols/NLM/nlm_Test.c',
  'src/Protocols/NLM/nlm_Unlock.c',
  'src/Protocols/NLM/nlm_Unshare.c',
  'src/Protocols/NLM/nlm_async.c',
  'src/Protocols/NLM/nlm_util.c',
  'src/Protocols/NLM/nsm.c',
) # }}}
nlm_lib = static_library('nlm', nlm_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/Protocols/NLM'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# Protocols / RQUOTA {{{
rquota_files = files( # {{{
  'src/Protocols/RQUOTA/rquota_Null.c',
  'src/Protocols/RQUOTA/rquota_getquota.c',
  'src/Protocols/RQUOTA/rquota_getactivequota.c',
  'src/Protocols/RQUOTA/rquota_setquota.c',
  'src/Protocols/RQUOTA/rquota_setactivequota.c',
  'src/Protocols/RQUOTA/rquota_common.c',
) # }}}
rquota_lib = static_library('rquota', rquota_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/Protocols/RQUOTA'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# Protocols / NFS4 ACL {{{
nfs4_acl_files = files( # {{{
  'src/Protocols/NFSACL/nfsacl_Null.c',
  'src/Protocols/NFSACL/nfsacl_getacl.c',
  'src/Protocols/NFSACL/nfsacl_setacl.c',
) # }}}
nfs4_acl_lib = static_library('nfs4_acl', nfs4_acl_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/Protocols/NFSACL'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# FSAL Pseudo {{{
fsal_pseudo_files = files( # {{{
  'src/FSAL/FSAL_PSEUDO/handle.c',
  'src/FSAL/FSAL_PSEUDO/main.c',
  'src/FSAL/FSAL_PSEUDO/export.c',
) # }}}
fsal_pseudo_lib = static_library('fsal_pseudo', fsal_pseudo_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/FSAL/FSAL_PSEUDO'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}
# FSAL MD Cache {{{
fsal_md_cache_files = files( # {{{
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_avl.h',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_ext.h',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_int.h',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_hash.h',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_lru.h',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_handle.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_file.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_xattrs.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_main.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_export.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_helpers.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_lru.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_hash.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_avl.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_read_conf.c',
  'src/FSAL/Stackable_FSALs/FSAL_MDCACHE/mdcache_up.c',
) # }}}
fsal_md_cache_lib = static_library('fsal_md_cache', fsal_md_cache_files,
  include_directories: [nfs_ganesha_inc_dir, 'src/FSAL/Stackable_FSALs/FSAL_MDCACHE'],
  c_args : nfs_ganesha_c_args,
  dependencies : main_deps,
  install : false
) # }}}


# libganesha_nfsd shared library {{{
libganesha_nfsd = shared_library('ganesha_nfsd',
  [], # no additional files
  include_directories: nfs_ganesha_inc_dir,
  dependencies: [
    dbus_dep, caps_dep, krb5_dep, nfsidmap_dep, acl_dep
  ],
  c_args: nfs_ganesha_c_args,
  link_args: ['-Wl,--version-script=' + meson.current_source_dir() + '/src/MainNFSD/libganesha_nfsd.ver'],
  link_whole: [
    avl_lib,
    mainnfsd_lib,
    config_parsing_lib,
    sal_lib,
    rpcal_lib,
    idmap_lib,
    dbus_lib,
    nfs_lib,
    xdr_lib,
    rquota_lib,
    nfs4_acl_lib,
    nlm_lib,
    fsal_pseudo_lib,
    fsal_md_cache_lib,
    monitoring_lib,
  ],
  version: meson.project_version(),
  install: true,
  install_dir: get_option('libdir'),
) # }}}

# Build targets
ganesha_nfsd = executable('ganesha.nfsd',
  'src/MainNFSD/nfs_main.c',
  include_directories : nfs_ganesha_inc_dir,
  c_args : nfs_ganesha_c_args,
  dependencies : [
    main_deps
  ],
  link_with : [libganesha_nfsd, monitoring_lib],
  install : true,
  install_dir : get_option('sbindir'),
  install_tag : 'ganesha_nfsd',
)

fsal_include_dir = include_directories('src/include', 'src/libntirpc/ntirpc',
  '.')

## Install systemd service file
#install_data('packaging/nfs-ganesha.service', install_dir : get_option('systemd_unitdir'))
#
## Install sysconfig file
#install_data('packaging/sysconfig/nfs-ganesha', install_dir : get_option('sysconfdir') / 'sysconfig')
#
## Install default configuration file
#install_data('packaging/ganesha.conf', install_dir : get_option('sysconfdir') / 'ganesha')
#
## Install logrotate configuration file
#install_data('packaging/logrotate/ganesha', install_dir : get_option('sysconfdir') / 'logrotate.d')
#
## Install init script
#install_data('packaging/init.d/nfs-ganesha', install_dir : get_option('initdir') / 'nfs-ganesha')
#
## Install example exports file
#install_data('packaging/ganesha-exports-default.conf', install_dir : get_option('sysconfdir') / 'ganesha')
