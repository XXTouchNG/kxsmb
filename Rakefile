# 
# Rakefile
# kxsmb project
# https://github.com/kolyvan/kxsmb/
#
# Created by Kolyvan on 29.03.13.
#

#
# copyright (c) 2013 Konstantin Bukreev All rights reserved.
# 
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
# 
# - Redistributions of source code must retain the above copyright notice, this
# list of conditions and the following disclaimer.
# 
# - Redistributions in binary form must reproduce the above copyright notice,
# this list of conditions and the following disclaimer in the documentation
# and/or other materials provided with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
# DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
# CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
# OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#

require "pathname"
require "fileutils"

# utils

def system_or_exit(cmd, stdout = nil)
  puts "executing #{cmd}"
  cmd += " >#{stdout}" if stdout
  system(cmd) or raise "******** build failed ********"
end

def copyIfNotExists(file, from, to)

    dest = Pathname.new(to)
    dest.mkdir unless dest.exist?

    unless (dest + file).exist?
        source = Pathname.new(from) + file
        FileUtils.copy source, dest
        p "copy #{source} -> #{dest}"
    end
end

def cleanOrMkDir(path)
    dest = Pathname.new path
    if dest.exist?
        FileUtils.rm Dir.glob("#{path}/*.a")
    else
        dest.mkdir
    end
end

def cleanDir(path)
    dest = Pathname.new path
    if dest.exist?
        FileUtils.rm Dir.glob("#{path}/*.a")
    end
end

# versions

IOS_MIN_VERSION='11.0'
SAMBA_VERSION='4.0.26'

# samba source

SAMBA_BASE_URL="http://ftp.samba.org/pub/samba/stable/"

#pathes

XCODE_PATH=%x{ /usr/bin/xcode-select --print-path }.delete("\n")
SIM_SDK_PATH=XCODE_PATH + "/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk"
IOS_SDK_PATH=XCODE_PATH + "/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk"

#SAMBA_PATH="samba-#{SAMBA_VERSION}/source3"
SAMBA_FOLDER="samba-#{SAMBA_VERSION}"
SAMBA_SOURCE_PATH="#{SAMBA_FOLDER}/source3"
EXT_INCLUDE_PATH='tmp/include'

# configure arguments

CF_FLAGS='-pipe -Wno-trigraphs -fpascal-strings -Os -g'

IOS_CF_FLAGS='-ftree-vectorize'
IOS_LD_FLAGS=''

ARM64_CF_FLAGS="-arch arm64 -Wno-error=implicit-function-declaration #{IOS_CF_FLAGS} #{CF_FLAGS}"
ARM64_LD_FLAGS="-arch arm64 #{IOS_LD_FLAGS}"

SMB_ARGS = [
'--prefix=/private',
'--disable-shared',
'--enable-static',
'--without-readline',
'--with-libsmbclient',
'--without-libnetapi',
'--without-libsmbsharemodes',
'--without-cluster-support',
'--without-ldap',
'--disable-swat',
'--disable-cups',
'--disable-iprint',
'libreplace_cv_HAVE_C99_VSNPRINTF=yes',
'samba_cv_CC_NEGATIVE_ENUM_VALUES=yes',
]

IOS_SMB_ARGS = [
'ac_cv_header_libunwind_h=no',
'ac_cv_header_execinfo_h=no',
'ac_cv_header_rpcsvc_ypclnt_h=no',
'ac_cv_file__proc_sys_kernel_core_pattern=no',
'ac_cv_func_fdatasync=no',
'libreplace_cv_HAVE_GETADDRINFO=no',
'samba_cv_SYSCONF_SC_NPROCESSORS_ONLN=no',
'samba_cv_big_endian=no',
'samba_cv_little_endian=yes',
]

ARM64_SMB_ARGS = [
'--host=arm-apple-darwin',
]

# libs

SMB_LIBS = [
'libsmbclient',
'libtalloc',
'libtevent',
'libtdb',
'libwbclient',
'bin/smbclient',
]

# functions

def mkArgs(sdkPath, platformArgs, procArgs, cfFlags, ldFlags)

    extInclude = Pathname.new(EXT_INCLUDE_PATH).realpath

    args = SMB_ARGS + platformArgs + procArgs
    ENV['AR']="xcrun ar"
    ENV['CC']="xcrun clang"
    ENV['CPP']="xcrun clang -E"
    ENV['LD']="xcrun ld"
    ENV['CFLAGS']="-std=gnu99 -no-cpp-precomp -miphoneos-version-min=#{IOS_MIN_VERSION} -isysroot #{sdkPath} -I#{sdkPath}/usr/include #{cfFlags}"
    ENV['CPPFLAGS']="-std=gnu99 -no-cpp-precomp -miphoneos-version-min=#{IOS_MIN_VERSION} -isysroot #{sdkPath} -I#{sdkPath}/usr/include #{cfFlags} -I#{extInclude}"
    ENV['LDFLAGS']="-miphoneos-version-min=#{IOS_MIN_VERSION} -isysroot #{sdkPath} -L#{sdkPath}/usr/lib #{ldFlags}"
    args.join(' ')
end

def buildArch(arch)

    case arch
    when 'arm64'
        args = mkArgs(IOS_SDK_PATH, IOS_SMB_ARGS, ARM64_SMB_ARGS, ARM64_CF_FLAGS, ARM64_LD_FLAGS)
    else
        raise "build failed: unknown arch: #{arch}"
    end
    
    p args
    
    system_or_exit "cd #{SAMBA_SOURCE_PATH}; ./autogen.sh"
    system_or_exit "cd #{SAMBA_SOURCE_PATH}; ./configure #{args}"

    SMB_LIBS.each do |x|
        system_or_exit "cd #{SAMBA_SOURCE_PATH}; make #{x}"
    end

    # dest = Pathname.new("#{SAMBA_SOURCE_PATH}/bin/#{arch}")
    # cleanOrMkDir(dest)

    # SMB_LIBS.each do |x|
    #   FileUtils.move Pathname.new("#{SAMBA_SOURCE_PATH}/bin/#{x}.a"), dest
    # end

    # system_or_exit "cd #{SAMBA_SOURCE_PATH}; make clean"

end

def checkExtInclude
    extInclude = Pathname.new(EXT_INCLUDE_PATH)
    extInclude.mkpath unless extInclude.exist? 
    copyIfNotExists('crt_externs.h', "#{SIM_SDK_PATH}/usr/include/", extInclude.realpath)
end

# tasks

desc "build smb arm64 libs"
task :build_smb_arm64 do
    checkExtInclude
    buildArch('arm64')
end

desc "clean"
task :clean do
    cleanDir("#{SAMBA_SOURCE_PATH}/bin/arm64")

    system_or_exit "cd #{SAMBA_SOURCE_PATH}; make clean"
end

desc "retrieve samba archive"
task :retrieve_samba do

    p = Pathname.new "#{SAMBA_SOURCE_PATH}"
     unless p.exist?

         name = "samba-#{SAMBA_VERSION}"
         file = "#{name}.tar.gz"
        url = "#{SAMBA_BASE_URL}#{file}"

         p "retrieving samba from #{url}"
        system_or_exit "/usr/bin/curl -L --output #{file} #{url}"

        p "extracting samba from archive"
        system_or_exit "tar -zxf #{file}"

        Pathname.new(file).delete
        Pathname.new(name).rename SAMBA_FOLDER
     end

end

task :build_all => [:retrieve_samba, :build_smb_arm64]
task :default => [:build_all]
