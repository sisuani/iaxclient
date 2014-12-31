cmake_minimum_required(VERSION 2.8)

include_directories("gsm/inc/")
include_directories("libiax2/src/")
include_directories("libspeex/include/")
include_directories("portmixer/px_common/")
set(ENABLE_SPEEX 1)

set(IAXCLIENT_BASE_SOURCES
    audio_encode.c
    audio_file.c
    codec_alaw.c
    codec_gsm.c
    codec_ulaw.c
    iaxclient_lib.c
    audio_openal.c
    #audio_portaudio.c
)

set(LIBIAX2_SOURCES
    libiax2/src/iax.c
    libiax2/src/iax2-parser.c
    libiax2/src/jitterbuf.c
    libiax2/src/md5.c
)

set(GSM_SOURCES
    gsm/src/add.c
    gsm/src/code.c
    gsm/src/debug.c
    gsm/src/decode.c
    gsm/src/gsm_create.c
    gsm/src/gsm_decode.c
    gsm/src/gsm_destroy.c
    gsm/src/gsm_encode.c
    gsm/src/gsm_explode.c
    gsm/src/gsm_implode.c
    gsm/src/gsm_option.c
    gsm/src/gsm_print.c
    gsm/src/long_term.c
    gsm/src/lpc.c
    gsm/src/preprocess.c
    gsm/src/rpe.c
    gsm/src/short_term.c
    gsm/src/table.c
)

if (WIN32)
    list(APPEND IAXCLIENT_BASE_SOURCES 
        winfuncs.c)
else()
    list(APPEND IAXCLIENT_BASE_SOURCES 
        unixfuncs.c)
endif(WIN32)

if (ENABLE_SPEEX)
    list(APPEND IAXCLIENT_BASE_SOURCES codec_speex.c)

    if (SYSTEM_SPEEX)
        set(Speex_FIND_REQUIRED TRUE)
        set(Speexdsp_FIND_REQUIRED TRUE)
        find_package(Speex)
        find_package(Speexdsp)
    endif(SYSTEM_SPEEX)

    if (SPEEX_FOUND AND SPEEXDSP_FOUND)
        include_directories(${SPEEX_INCLUDE_DIR} ${SPEEXDSP_INCLUDE_DIR})
        message(STATUS "Using speex includes at: ${SPEEX_INCLUDE_DIR}")
        message(STATUS "Using speex libraries: ${SPEEX_LIBRARIES}")
        message(STATUS "Using speex extended library includes at: ${SPEEXDSP_INCLUDE_DIR}")
        message(STATUS "Using speex extended library libraries: ${SPEEXDSP_LIBRARIES}")
    else(SPEEX_FOUND AND SPEEXDSP_FOUND)
        set(SPEEX_SOURCES
            libspeex/bits.c
            libspeex/cb_search.c
            libspeex/exc_10_16_table.c
            libspeex/exc_10_32_table.c
            libspeex/exc_20_32_table.c
            libspeex/exc_5_256_table.c
            libspeex/exc_5_64_table.c
            libspeex/exc_8_128_table.c
            libspeex/filters.c
            libspeex/gain_table.c
            libspeex/gain_table_lbr.c
            libspeex/hexc_10_32_table.c
            libspeex/hexc_table.c
            libspeex/high_lsp_tables.c
            libspeex/jitter.c
            libspeex/lbr_48k_tables.c
            libspeex/lpc.c
            libspeex/lsp.c
            libspeex/lsp_tables_nb.c
            libspeex/ltp.c
            libspeex/math_approx.c
            libspeex/mdf.c
            libspeex/medfilter.c
            libspeex/misc.c
            libspeex/modes.c
            libspeex/nb_celp.c
            libspeex/preprocess.c
            libspeex/quant_lsp.c
            libspeex/sb_celp.c
            libspeex/smallft.c
            libspeex/speex.c
            libspeex/speex_callbacks.c
            libspeex/speex_header.c
            libspeex/stereo.c
            libspeex/vbr.c
            libspeex/vq.c
            )

        include_directories(${PROJECT_SOURCE_DIR}/3rdparty/iaxclient/lib/libspeex/include)
    endif(SPEEX_FOUND AND SPEEXDSP_FOUND)
endif(ENABLE_SPEEX)

if (APPLE)
    add_definitions(-DMACOSX)
endif(APPLE)

list(APPEND IAXCLIENT_BASE_SOURCES spandsp/plc.c)

add_definitions(-DLIBIAX)

# for GSM
add_definitions(-DHAS_STRING_H -DHAS_STDLIB_H)
add_definitions(-DCODEC_GSM)


include_directories(${PROJECT_SOURCE_DIR})
include_directories(${PROJECT_SOURCE_DIR}/libiax2/src)
include_directories(${PROJECT_SOURCE_DIR}/gsm/inc)


add_library(iaxclient_lib SHARED
    ${IAXCLIENT_BASE_SOURCES} 
    ${GSM_SOURCES} 
    ${SPEEX_SOURCES}
    ${LIBIAX2_SOURCES})

if (WIN32)
    target_link_libraries(iaxclient_lib ${CMAKE_THREAD_LIBS_INIT}
    ${SPEEX_LIBRARIES} ${SPEEXDSP_LIBRARIES} ${GSM_LIBRARIES} Ws2_32 OpenAl32)
else()
    target_link_libraries(iaxclient_lib ${CMAKE_THREAD_LIBS_INIT}
        ${SPEEX_LIBRARIES} ${SPEEXDSP_LIBRARIES} ${GSM_LIBRARIES} openal) 
endif(WIN32)
