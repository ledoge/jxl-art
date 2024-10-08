CODEC_URL = https://github.com/libjxl/libjxl.git
CODEC_VERSION = v0.11.0
CODEC_DIR = node_modules/jxl
CODEC_BUILD_ROOT := $(CODEC_DIR)/build
CODEC_BUILD_DIR := $(CODEC_BUILD_ROOT)

ENVIRONMENT = worker

OUT_JS = jxl.js
OUT_WASM = $(OUT_JS:.js=.wasm)
OUT_WORKER = $(OUT_JS:.js=.worker.js)

.PHONY: all clean

all: $(OUT_JS)

$(OUT_JS): jxl.cpp $(CODEC_BUILD_DIR)/lib/libjxl.a $(CODEC_BUILD_DIR)/lib/libjxl_cms.a
	$(CXX) \
		$(CXXFLAGS) \
		$(LDFLAGS) \
		-I $(CODEC_DIR) \
		-I $(CODEC_DIR)/lib \
		-I $(CODEC_DIR)/lib/include \
		-I $(CODEC_BUILD_DIR)/lib/include \
		-I $(CODEC_DIR)/third_party/highway \
		-I $(CODEC_DIR)/third_party/skcms \
		-I $(CODEC_DIR)/third_party/libpng \
		--bind \
		--closure 1 \
		-s INVOKE_RUN=0 \
		-s ALLOW_MEMORY_GROWTH=1 \
		-s MODULARIZE=1 \
		-s TEXTDECODER=2 \
		-s ENVIRONMENT=$(ENVIRONMENT) \
		-s EXPORT_ES6=1 \
		-s EXPORT_NAME="$(basename $(@F))" \
		-s USE_LIBPNG=1 \
		-DJPEGXL_ENABLE_APNG=1 \
		-o $@ \
		$+ \
		$(CODEC_DIR)/tools/no_memory_manager.cc \
		$(CODEC_DIR)/tools/jxl_from_tree.cc \
		$(CODEC_DIR)/lib/extras/common.cc \
		$(CODEC_DIR)/lib/extras/enc/apng.cc \
		$(CODEC_DIR)/lib/extras/exif.cc \
		$(CODEC_DIR)/lib/extras/dec/jxl.cc \
		$(CODEC_DIR)/lib/extras/enc/encode.cc \
		$(CODEC_DIR)/lib/extras/dec/color_description.cc \
		$(CODEC_BUILD_DIR)/third_party/brotli/libbrotlidec.a \
		$(CODEC_BUILD_DIR)/third_party/brotli/libbrotlienc.a \
		$(CODEC_BUILD_DIR)/third_party/brotli/libbrotlicommon.a \
		$(CODEC_BUILD_DIR)/third_party/highway/libhwy.a

%/lib/libjxl.a %/lib/libjxl_cms.a: %/Makefile
	$(MAKE) -C $(<D) jxl jxl_cms

%/Makefile: $(CODEC_DIR)/CMakeLists.txt
	emcmake cmake \
	$(CMAKE_FLAGS) \
	-DBUILD_SHARED_LIBS=0 \
	-DJPEGXL_ENABLE_BENCHMARK=0 \
	-DJPEGXL_ENABLE_EXAMPLES=0 \
	-DJPEGXL_BUNDLE_LIBPNG=1 \
	-DBUILD_TESTING=0 \
	-B $(@D) \
	$(<D)

$(CODEC_DIR)/CMakeLists.txt: 
	$(RM) -r $(@D)
	git init $(@D)
	git -C $(@D) fetch $(CODEC_URL) $(CODEC_VERSION) --depth 1
	git -C $(@D) checkout FETCH_HEAD
	git -C $(@D) submodule update --init --depth 1 --recursive --jobs `nproc`
	sed -i 's/int buffering = -1;/int buffering = 0;/' $(@D)/lib/jxl/enc_params.h

clean:
	$(RM) $(OUT_JS) $(OUT_WASM) $(OUT_WORKER)
	$(MAKE) -C $(CODEC_BUILD_DIR) clean
	$(MAKE) -C $(CODEC_MT_BUILD_DIR) clean
	$(MAKE) -C $(CODEC_MT_SIMD_BUILD_DIR) clean
