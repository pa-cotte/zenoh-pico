if(CONFIG_ZENOH_PICO)

zephyr_compile_definitions(ZENOH_ZEPHYR)

# Upstream no longer commits a concrete include/zenoh-pico/config.h: it is now a
# CMake-templated config.h.in resolved by the root CMakeLists. The Zephyr module
# build (module.yml -> cmake: zephyr/) does not run that root configure step, so
# we generate config.h here, driven by the CONFIG_ZENOH_PICO_* Kconfig symbols.
# This replaces the historical committed config.h of this fork ("force_config").

# Numeric tokens. BATCH_MULTICAST_SIZE is forced to match zenoh-c.
set(FRAG_MAX_SIZE 4096)
set(BATCH_UNICAST_SIZE 2048)
set(BATCH_MULTICAST_SIZE 8192)
set(Z_CONFIG_SOCKET_TIMEOUT 100)
set(Z_TRANSPORT_LEASE 10000)
set(Z_TRANSPORT_LEASE_EXPIRE_FACTOR 3)
set(Z_RUNTIME_MAX_TASKS 64)
set(Z_TRANSPORT_ACCEPT_TIMEOUT 1000)
set(Z_TRANSPORT_CONNECT_TIMEOUT 10000)

# Features with a Kconfig: value comes from the consumer (Bolt's BOLT_ZENOH
# selects what the firmware needs; app prj.conf keeps full control).
foreach(f MULTI_THREAD PUBLICATION ADVANCED_PUBLICATION SUBSCRIPTION ADVANCED_SUBSCRIPTION
          QUERY QUERYABLE INTEREST RAWETH_TRANSPORT LINK_TCP LINK_SERIAL LINK_WS SCOUTING
          LINK_UDP_MULTICAST LINK_UDP_UNICAST MULTICAST_TRANSPORT LOCAL_SUBSCRIBER LOCAL_QUERYABLE)
  if(CONFIG_ZENOH_PICO_${f})
    set(Z_FEATURE_${f} 1)
  else()
    set(Z_FEATURE_${f} 0)
  endif()
endforeach()

# Features without a dedicated Kconfig: fixed values mirroring upstream defaults.
foreach(f LIVELINESS FRAGMENTATION ENCODING_VALUES UNICAST_TRANSPORT TCP_NODELAY
          SESSION_CHECK BATCHING MATCHING UNICAST_PEER AUTO_RECONNECT)
  set(Z_FEATURE_${f} 1)
endforeach()
foreach(f CONNECTIVITY LINK_BLUETOOTH LINK_SERIAL_USB LINK_TLS BATCH_TX_MUTEX
          BATCH_PEER_MUTEX RX_CACHE MULTICAST_DECLARATIONS ADMIN_SPACE)
  set(Z_FEATURE_${f} 0)
endforeach()

# Unstable API is forced on: advanced pub/sub depend on it. config.h.in uses
# #cmakedefine (presence-based), so any truthy value enables it.
set(Z_FEATURE_UNSTABLE_API ON)

# ZP_SYSTEM_PLATFORM_HEADER must stay undefined so platform.h selects the
# ZENOH_ZEPHYR branch: do not set it.

set(ZENOH_PICO_GENERATED_DIR ${CMAKE_CURRENT_BINARY_DIR}/generated)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/../include/zenoh-pico/config.h.in
               ${ZENOH_PICO_GENERATED_DIR}/zenoh-pico/config.h)

zephyr_include_directories(../include ${ZENOH_PICO_GENERATED_DIR})
zephyr_library()

file(GLOB_RECURSE Sources
  "../src/api/*.c"
  "../src/collections/*.c"
  "../src/link/*.c"
  "../src/net/*.c"
  "../src/protocol/*.c"
  "../src/session/*.c"
  "../src/transport/*.c"
  "../src/utils/*.c"
  "../src/system/common/*.c"
)

file (GLOB Sources_Zephyr "../src/system/zephyr/*.c")
list(APPEND Sources ${Sources_Zephyr})
zephyr_library_sources(${Sources})
endif()
