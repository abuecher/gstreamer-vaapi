project('gstreamer-vaapi', 'c',
  version : '1.15.0.1',
  meson_version : '>= 0.46.0',
  default_options : [ 'warning_level=1',
                      'buildtype=debugoptimized' ])

gst_version = meson.project_version()
version_arr = gst_version.split('.')
gst_version_major = version_arr[0].to_int()
gst_version_minor = version_arr[1].to_int()
gst_version_micro = version_arr[2].to_int()
 if version_arr.length() == 4
  gst_version_nano = version_arr[3].to_int()
else
  gst_version_nano = 0
endif

libva_req = ['>= 0.30.4', '!= 0.99.0']
glib_req = '>= 2.40.0'
gst_req = '>= @0@.@1@.0'.format(gst_version_major, gst_version_minor)

cc = meson.get_compiler('c')

if cc.has_link_argument('-Wl,-Bsymbolic-functions')
  add_project_link_arguments('-Wl,-Bsymbolic-functions', language : 'c')
endif

# Symbol visibility
if cc.has_argument('-fvisibility=hidden')
  add_project_arguments('-fvisibility=hidden', language: 'c')
endif

# Disable strict aliasing
if cc.has_argument('-fno-strict-aliasing')
  add_project_arguments('-fno-strict-aliasing', language: 'c')
endif

# Mandatory GST deps
libm = cc.find_library('m', required : false)
gst_dep = dependency('gstreamer-1.0', version : gst_req,
  fallback : ['gstreamer', 'gst_dep'])
gstbase_dep = dependency('gstreamer-base-1.0', version : gst_req,
  fallback : ['gstreamer', 'gst_base_dep'])
gstpbutils_dep = dependency('gstreamer-pbutils-1.0', version : gst_req,
    fallback : ['gst-plugins-base', 'pbutils_dep'])
gstallocators_dep = dependency('gstreamer-allocators-1.0', version : gst_req,
    fallback : ['gst-plugins-base', 'allocators_dep'])
gstvideo_dep = dependency('gstreamer-video-1.0', version : gst_req,
    fallback : ['gst-plugins-base', 'video_dep'])
gstcodecparsers_dep = dependency('gstreamer-codecparsers-1.0', version : gst_req,
    fallback : ['gst-plugins-bad', 'gstcodecparsers_dep'])
gstgl_dep = dependency('gstreamer-gl-1.0', version : gst_req,
    fallback : ['gst-plugins-base', 'gstgl_dep'], required: false)
gmodule_dep = dependency('gmodule-2.0', required: false)
libva_dep = dependency('libva', version: libva_req)

libva_drm_dep = dependency('libva-drm', version: '>= 0.33.0', required: false)
libva_wayland_dep = dependency('libva-wayland', version: '>= 0.33.0', required: false)
libva_x11_dep = dependency('libva-x11', version: '>= 0.31.0', required: false)
libdrm_dep = dependency('libdrm', required: false)
libudev_dep = dependency('libudev', required: false)
egl_dep = dependency('egl', required: false)
gl_dep = dependency('gl', required: false)
glesv2_dep = dependency('glesv2', required: false)
libdl_dep = cc.find_library('dl', required: false)
wayland_client_dep = dependency('wayland-client', required: false)
x11_dep = dependency('x11', required: false)
xrandr_dep = dependency('xrandr', required: false)
xrender_dep = dependency('xrender', required: false)

GLES_VERSION_MASK = gl_dep.found() ? 1 : 0
if glesv2_dep.found()
  if (cc.has_header('GLES2/gl2.h', dependencies: glesv2_dep) and
      cc.has_header('GLES2/gl2ext.h', dependencies: glesv2_dep))
    GLES_VERSION_MASK += 4
  endif
  if (cc.has_header('GLES3/gl3.h', dependencies: glesv2_dep) and
      cc.has_header('GLES3/gl3ext.h', dependencies: glesv2_dep) and
      cc.has_header('GLES2/gl2ext.h', dependencies: glesv2_dep))
    GLES_VERSION_MASK += 8
  endif
endif

USE_ENCODERS = libva_dep.version().version_compare('>= 0.34.0') and get_option('with_encoders') != 'no'
USE_H265_DECODER = cc.has_header('va/va_dec_hevc.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_H265_ENCODER = USE_ENCODERS and cc.has_header('va/va_enc_hevc.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_JPEG_DECODER = cc.has_header('va/va_dec_jpeg.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_JPEG_ENCODER = USE_ENCODERS and cc.has_header('va/va_enc_jpeg.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_VP8_DECODER = cc.has_header('va/va_dec_vp8.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_VP8_ENCODER = USE_ENCODERS and cc.has_header('va/va_enc_vp8.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_VP9_DECODER = cc.has_header('va/va_dec_vp9.h', dependencies: libva_dep, prefix: '#include <va/va.h>')
USE_VP9_ENCODER = USE_ENCODERS and cc.has_header('va/va_enc_vp9.h', dependencies: libva_dep, prefix: '#include <va/va.h>')

USE_VPP = cc.has_header('va/va_vpp.h', dependencies: libva_dep, prefix: '#include <va/va_vpp.h>')

USE_DRM = libva_drm_dep.found() and libdrm_dep.found() and libudev_dep.found() and get_option('with_drm') != 'no'
USE_EGL = gmodule_dep.found() and egl_dep.found() and GLES_VERSION_MASK != 0 and get_option('with_egl') != 'no'
USE_GLX = libva_x11_dep.found() and x11_dep.found() and gl_dep.found() and libdl_dep.found() and get_option('with_glx') != 'no'
USE_WAYLAND = libva_wayland_dep.found() and wayland_client_dep.found() and get_option('with_wayland') != 'no'
USE_X11 = libva_x11_dep.found() and x11_dep.found() and get_option('with_x11') != 'no'

driverdir = libva_dep.get_pkgconfig_variable('driverdir')
if driverdir == ''
  driverdir = '@0@/@1@/@2@'.format(get_option('prefix'), get_option('libdir'), 'dri')
endif

cdata = configuration_data()
cdata.set('GST_API_VERSION_S', '"@0@.@1@"'.format(gst_version_major, gst_version_minor))
cdata.set('PACKAGE', '"gstreamer-vaapi"')
cdata.set('VERSION', '"@0@"'.format(gst_version))
cdata.set('PACKAGE_VERSION', '"@0@"'.format(gst_version))
cdata.set('PACKAGE_NAME', '"GStreamer VA-API Plug-ins"')
cdata.set('PACKAGE_STRING', '"GStreamer VA-API Plug-ins @0@"'.format(gst_version))
cdata.set('PACKAGE_BUGREPORT', '"http://bugzilla.gnome.org/enter_bug.cgi?product=GStreamer"')
cdata.set('VA_DRIVERS_PATH', '"@0@"'.format(driverdir))
cdata.set10('USE_DRM', USE_DRM)
cdata.set10('USE_EGL', USE_EGL)
cdata.set10('USE_ENCODERS', USE_ENCODERS)
cdata.set10('USE_GLX', USE_GLX)
cdata.set10('USE_H265_DECODER', USE_H265_DECODER)
cdata.set10('USE_H265_ENCODER', USE_H265_ENCODER)
cdata.set10('USE_JPEG_DECODER', USE_JPEG_DECODER)
cdata.set10('USE_JPEG_ENCODER', USE_JPEG_ENCODER)
cdata.set10('USE_VP8_DECODER', USE_VP8_DECODER)
cdata.set10('USE_VP8_ENCODER', USE_VP8_ENCODER)
cdata.set10('USE_VP9_DECODER', USE_VP9_DECODER)
cdata.set10('USE_VP9_ENCODER', USE_VP9_ENCODER)
cdata.set10('USE_WAYLAND', USE_WAYLAND)
cdata.set10('USE_X11', USE_X11)
cdata.set10('HAVE_XKBLIB', cc.has_header('X11/XKBlib.h', dependencies: x11_dep))
cdata.set10('HAVE_XRANDR', xrandr_dep.found())
cdata.set10('HAVE_XRENDER', xrender_dep.found())
cdata.set10('USE_VA_VPP', USE_VPP)
cdata.set10('USE_GST_GL_HELPERS', gstgl_dep.found())
cdata.set('USE_GLES_VERSION_MASK', GLES_VERSION_MASK)

if libva_dep.version().version_compare('< 0.38.0')
  check_headers = [
    [USE_H265_DECODER, 'HAVE_VA_VA_DEC_HEVC_H'],
    [USE_H265_ENCODER, 'HAVE_VA_VA_ENC_HEVC_H'],
    [USE_JPEG_DECODER, 'HAVE_VA_VA_DEC_JPEG_H'],
    [USE_JPEG_ENCODER, 'HAVE_VA_VA_ENC_JPEG_H'],
    [USE_VP8_DECODER, 'HAVE_VA_VA_DEC_VP8_H'],
    [USE_VP8_ENCODER, 'HAVE_VA_VA_ENC_VP8_H'],
    [USE_VP9_DECODER, 'HAVE_VA_VA_DEC_VP9_H'],
    [USE_VP9_ENCODER, 'HAVE_VA_VA_DEC_VP9_H'],
    [USE_VPP, 'HAVE_VA_VA_VPP_H'],
  ]
  foreach h : check_headers
    if h.get(0)
      cdata.set(h.get(1), 1)
    endif
  endforeach

  if USE_ENCODERS
    check_headers = [
      ['HAVE_VA_VA_ENC_MPEG2_H', 'va/va_enc_mpeg2.h'],
      ['HAVE_VA_VA_ENC_H264_H', 'va/va_enc_h264.h'],
    ]
    foreach h : check_headers
      if cc.has_header(h.get(1), dependencies: libva_dep, prefix: '#include <va/va.h>')
        cdata.set(h.get(0), 1)
      endif
    endforeach
  endif
endif

api_version = '1.0'
soversion = 0
# maintaining compatibility with the previous libtool versioning
# current = minor * 100 + micro
libversion = '@0@.@1@.0'.format(soversion, gst_version_minor * 100 + gst_version_micro)

plugins_install_dir = '@0@/gstreamer-1.0'.format(get_option('libdir'))

configure_file(configuration: cdata, output: 'config.h')
gstreamer_vaapi_args = ['-DHAVE_CONFIG_H']
configinc = include_directories('.')
libsinc = include_directories('gst-libs')

subdir('gst-libs')
subdir('gst')
#subdir('tests')

python3 = import('python3').find_python()
run_command(python3, '-c', 'import shutil; shutil.copy("hooks/pre-commit.hook", ".git/hooks/pre-commit")')
