project(lws-minimal-raw-proxy-fallback)
cmake_minimum_required(VERSION 2.8)
include(CheckCSourceCompiles)

set(SAMP lws-minimal-raw-proxy-fallback)
set(SRCS minimal-raw-proxy-fallback.c)

# NOTE... if you are building this standalone, you must point LWS_PLUGINS_DIR
# to the lws plugins dir so it can pick up the plugin source.  Eg,
# cmake . -DLWS_PLUGINS_DIR=~/libwebsockets/plugins

# If we are being built as part of lws, confirm current build config supports
# reqconfig, else skip building ourselves.
#
# If we are being built externally, confirm installed lws was configured to
# support reqconfig, else error out with a helpful message about the problem.
#
MACRO(require_lws_config reqconfig _val result)

	if (DEFINED ${reqconfig})
	if (${reqconfig})
		set (rq 1)
	else()
		set (rq 0)
	endif()
	else()
		set(rq 0)
	endif()

	if (${_val} EQUAL ${rq})
		set(SAME 1)
	else()
		set(SAME 0)
	endif()

	if (LWS_WITH_MINIMAL_EXAMPLES AND NOT ${SAME})
		if (${_val})
			message("${SAMP}: skipping as lws being built without ${reqconfig}")
		else()
			message("${SAMP}: skipping as lws built with ${reqconfig}")
		endif()
		set(${result} 0)
	else()
		if (LWS_WITH_MINIMAL_EXAMPLES)
			set(MET ${SAME})
		else()
			CHECK_C_SOURCE_COMPILES("#include <libwebsockets.h>\nint main(void) {\n#if defined(${reqconfig})\n return 0;\n#else\n fail;\n#endif\n return 0;\n}\n" HAS_${reqconfig})
			if (NOT DEFINED HAS_${reqconfig} OR NOT HAS_${reqconfig})
				set(HAS_${reqconfig} 0)
			else()
				set(HAS_${reqconfig} 1)
			endif()
			if ((HAS_${reqconfig} AND ${_val}) OR (NOT HAS_${reqconfig} AND NOT ${_val}))
				set(MET 1)
			else()
				set(MET 0)
			endif()
		endif()
		if (NOT MET)
			if (${_val})
				message(FATAL_ERROR "This project requires lws must have been configured with ${reqconfig}")
			else()
				message(FATAL_ERROR "Lws configuration of ${reqconfig} is incompatible with this project")
			endif()
		endif()	
	endif()
ENDMACRO()

set(requirements 1)
require_lws_config(LWS_ROLE_RAW_PROXY 1 requirements)

if (requirements)
	add_executable(${SAMP} ${SRCS})
	
	if (LWS_PLUGINS_DIR)
		include_directories(${LWS_PLUGINS_DIR})
	endif()

	if (websockets_shared)
		target_link_libraries(${SAMP} websockets_shared)
		add_dependencies(${SAMP} websockets_shared)
	else()
		target_link_libraries(${SAMP} websockets)
	endif()
endif()
