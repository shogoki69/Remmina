# See LICENSE and COPYING files for copyright and license details.

include(ExternalProject)

set(CMAKE_THREAD_PREFER_PTHREAD)
find_package(Threads)
find_package(OpenSSL REQUIRED)
find_package(Pulse REQUIRED)

set(FREERDP_TOPDIR ${CMAKE_CURRENT_SOURCE_DIR}/external/FreeRDP )
set(FREERDP_INCLUDE_DIRS ${FREERDP_TOPDIR}/install/include )
set(FREERDP_LIBDIR ${FREERDP_TOPDIR}/install/${CMAKE_INSTALL_LIBDIR} )
set(FREERDP_LIBRARIES
	${FREERDP_LIBDIR}/librdtk.a
	${FREERDP_LIBDIR}/libfreerdp.a
	${FREERDP_LIBDIR}/libwinpr.a
	${FREERDP_LIBDIR}/libwinpr-makecert-tool.a
	${FREERDP_LIBDIR}/libxfreerdp-client.a
	${FREERDP_LIBDIR}/libfreerdp-client.a
	${FREERDP_LIBDIR}/freerdp/libaudin-client.a
	${FREERDP_LIBDIR}/freerdp/libaudin-client-alsa.a
	${FREERDP_LIBDIR}/freerdp/libaudin-client-pulse.a
	${FREERDP_LIBDIR}/freerdp/libcliprdr-client.a
	${FREERDP_LIBDIR}/freerdp/libdisp-client.a
	${FREERDP_LIBDIR}/freerdp/libdrdynvc-client.a
	${FREERDP_LIBDIR}/freerdp/libdrive-client.a
	${FREERDP_LIBDIR}/freerdp/libecho-client.a
	${FREERDP_LIBDIR}/freerdp/libencomsp-client.a
	${FREERDP_LIBDIR}/freerdp/libparallel-client.a
	${FREERDP_LIBDIR}/freerdp/libprinter-client.a
	${FREERDP_LIBDIR}/freerdp/librail-client.a
	${FREERDP_LIBDIR}/freerdp/librdpdr-client.a
	${FREERDP_LIBDIR}/freerdp/librdpei-client.a
	${FREERDP_LIBDIR}/freerdp/librdpgfx-client.a
	${FREERDP_LIBDIR}/freerdp/librdpsnd-client.a
	${FREERDP_LIBDIR}/freerdp/librdpsnd-client-alsa.a
	${FREERDP_LIBDIR}/freerdp/librdpsnd-client-pulse.a
	${FREERDP_LIBDIR}/freerdp/libremdesk-client.a
	${FREERDP_LIBDIR}/freerdp/libserial-client.a
	${FREERDP_LIBDIR}/freerdp/libsmartcard-client.a
	${FREERDP_LIBDIR}/freerdp/libtsmf-client.a
	${FREERDP_LIBDIR}/freerdp/libtsmf-client-alsa-audio.a
#	${FREERDP_LIBDIR}/freerdp/libtsmf-client-ffmpeg-decoder.a
	${FREERDP_LIBDIR}/freerdp/libtsmf-client-pulse-audio.a
)


set(REMMINA_PLUGIN_RDP_SRCS
	rdp_plugin.c
	rdp_plugin.h
	rdp_event.c
	rdp_event.h
	rdp_file.c
	rdp_file.h
	rdp_settings.c
	rdp_settings.h
	rdp_gdi.c
	rdp_gdi.h
	rdp_graphics.c
	rdp_graphics.h
	rdp_cliprdr.c
	rdp_cliprdr.h
	rdp_channels.c
	rdp_channels.h
	)

ExternalProject_Add(
	FreeRDP
	PREFIX ${FREERDP_TOPDIR}/src
	SOURCE_DIR ${FREERDP_TOPDIR}/src
	BINARY_DIR ${FREERDP_TOPDIR}/build
	INSTALL_DIR ${FREERDP_TOPDIR}/install
	STAMP_DIR ${FREERDP_TOPDIR}/stamps
	TMP_DIR ${FREERDP_TOPDIR}/tmp
	CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX:PATH=${FREERDP_TOPDIR}/install"
	UPDATE_COMMAND ""
	CMAKE_ARGS -DWITH_CUPS=on -DWITH_PULSE=on -DWITH_GSTREAMER_1_0=off -DWITH_MANPAGES=off -DWITH_WAYLAND=off -DBUILD_SHARED_LIBS=off -DWITH_FFMPEG=off
)

add_library(remmina-plugin-rdp ${REMMINA_PLUGIN_RDP_SRCS})
set_target_properties(remmina-plugin-rdp PROPERTIES PREFIX "")
set_target_properties(remmina-plugin-rdp PROPERTIES NO_SONAME 1)

# Put a dependency on FreeRDP project via libfreerdp.a, so remmina
# will stop building until FreeRDP is ready in "make -jN" multithread build
add_library(libfreerdp STATIC IMPORTED)
set_property(TARGET libfreerdp PROPERTY IMPORTED_LOCATION ${FREERDP_LIBDIR}/libfreerdp.a)
add_dependencies(libfreerdp FreeRDP)

include_directories(${REMMINA_COMMON_INCLUDE_DIRS} ${FREERDP_INCLUDE_DIRS})
target_link_libraries(remmina-plugin-rdp ${REMMINA_COMMON_LIBRARIES}
	-Wl,--start-group ${FREERDP_LIBRARIES} -Wl,--end-group
	${XKBFILE_LIBRARIES} ${OPENSSL_LIBRARIES} ${PULSE_LIBRARIES})

install(TARGETS remmina-plugin-rdp DESTINATION ${REMMINA_PLUGINDIR})

install(FILES 16x16/emblems/remmina-rdp-ssh.png 16x16/emblems/remmina-rdp.png DESTINATION ${APPICON16_EMBLEMS_DIR})
install(FILES 22x22/emblems/remmina-rdp-ssh.png 22x22/emblems/remmina-rdp.png DESTINATION ${APPICON22_EMBLEMS_DIR})
