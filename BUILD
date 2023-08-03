# Copyright 2021 curoky(cccuroky@gmail.com).
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

load("@rules_bison//bison:bison.bzl", "bison")
load("@rules_cc//cc:defs.bzl", "cc_binary", "cc_library")
load("@rules_flex//flex:flex.bzl", "flex")

DEFAULT_CPP_COPTS = [
    "-g",
    "-ggdb",
    "-O0",
    "-fno-omit-frame-pointer",
    "-gno-statement-frontiers",
    "-gno-variable-location-views",
    "-Wall",
    "-Wno-deprecated-declarations",
    "-Wno-deprecated",
    "-Wno-sign-compare",
    "-Wno-unknown-pragmas",
    "-Wno-unused-but-set-variable",
    "-Wno-unused-function",
    "-Wno-unused-value",
    "-Wno-unused-variable",
    "-Wno-unused-local-typedefs",
    "-Wno-maybe-uninitialized",
]

DEFAULT_LINKOPTS = [
    "-latomic",
    "-lpthread",
    "-ldl",
]

genrule(
    name = "genrule_thriftl",
    srcs = ["compiler/cpp/src/thrift/thriftl.ll"],
    outs = ["compiler/cpp/src/thrift/thriftl.cc"],
    cmd = "M4=$(M4) $(FLEX) --outfile=$@ $(location compiler/cpp/src/thrift/thriftl.ll)",
    toolchains = [
        "@rules_flex//flex:current_flex_toolchain",
        "@rules_m4//m4:current_m4_toolchain",
    ],
)

genrule(
    name = "genrule_thrifty",
    srcs = ["compiler/cpp/src/thrift/thrifty.yy"],
    outs = [
      "compiler/cpp/src/thrift/thrifty.hh",
      "compiler/cpp/src/thrift/thrifty.cc",
    ],
    cmd = """
export M4=$(M4)
$(BISON) -d $(location compiler/cpp/src/thrift/thrifty.yy) -o $(@D)/compiler/cpp/src/thrift/thrifty.cc
""",
    toolchains = [
        "@rules_bison//bison:current_bison_toolchain",
        "@rules_m4//m4:current_m4_toolchain",
    ],
)

flex(
    name = "thriftl",
    src = "compiler/cpp/src/thrift/thriftl.ll",
)

bison(
    name = "thrifty",
    src = "compiler/cpp/src/thrift/thrifty.yy",
    bison_options = [
        # Note: We cannot require external libraries to conform to specifications
        "-Wno-empty-rule",
    ],
)

genrule(
    name = "copy_thrifty",
    srcs = [":thrifty"],
    outs = [
        "thrift/thrifty.hh",
        "thrift/thrifty.cc",
    ],
    cmd = "mkdir -p $(@D)/thrift && mv $(SRCS) $(@D)/thrift",
)

cc_binary(
    name = "thriftc",
    srcs = glob(
        [
            "compiler/cpp/src/thrift/**/*.h",
            "compiler/cpp/src/thrift/**/*.cc",
            "compiler/cpp/src/thrift/**/*.cpp",
        ],
        exclude = ["compiler/cpp/src/thrift/logging.cc"],
    ) + [
        ":genrule_thriftl",
        ":genrule_thrifty",
    ],
    copts = DEFAULT_CPP_COPTS,
    includes = ["compiler/cpp/src"],
    linkopts = DEFAULT_LINKOPTS + ["-static"],
    visibility = ["//visibility:public"],
)

cc_library(
    name = "thrift",
    srcs = glob(
        [
            "lib/cpp/src/thrift/**/*.cpp",
            "lib/cpp/src/thrift/**/*.tcc",
        ],
        exclude = [
            "lib/cpp/src/thrift/windows/**",
            "lib/cpp/src/thrift/qt/**",
        ],
    ),
    hdrs = glob(["lib/cpp/src/thrift/**/*.h"]),
    copts = DEFAULT_CPP_COPTS,
    # copts = ["-Ilib/cpp/src/"],
    includes = ["lib/cpp/src"],
    visibility = ["//visibility:public"],
    deps = [
        "@boost//:algorithm",
        "@boost//:locale",
        "@boost//:noncopyable",
        "@boost//:numeric_conversion",
        "@boost//:scoped_array",
        "@boost//:shared_array",
        "@boost//:smart_ptr",
        "@boost//:tokenizer",
        "@libevent",
        "@zlib",
    ],
)
